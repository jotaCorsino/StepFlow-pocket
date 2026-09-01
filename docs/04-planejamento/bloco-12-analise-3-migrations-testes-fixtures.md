# Bloco 12 — Análise 3 — Migrations, scripts, testes e fixtures

**Status:** PROPOSTA PARA REVISÃO DO PO  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Fechar a disciplina executável de evolução do SQLite e a base de testes da Fase 2 sem criar ainda migration SQL oficial, banco, fixture, script ou scaffold neste PR.

Esta análise parte de D12.1–D12.34 e dos contratos já consolidados de SQLite Host-only, foreign keys, WAL, Backup/Restore e migrations imutáveis.

## Princípio

Migrations são parte do produto e precisam ser reproduzíveis, auditáveis e recuperáveis. Testes e fixtures devem provar o mecanismo sem transformar dados sintéticos em estado de produção.

Direção:

```text
source migration versionada
→ embutida no StepFlowHost.exe
→ Host abre banco antes de readiness
→ valida histórico/checksums
→ se houver pendências em banco existente, confirma pre_migration backup
→ aplica lote pendente de forma atômica
→ valida integridade/FKs
→ só então libera readiness
```

## Localização e naming

Migrations oficiais pertencem ao Host:

```text
apps/host/
├── migrations/
│   ├── 000001_<slug>.sql
│   ├── 000002_<slug>.sql
│   └── ...
└── src/
    └── persistence/
        └── migrations.rs
```

Regras:

- identificador decimal de seis dígitos, crescente e nunca reutilizado;
- slug em `snake_case`, descritivo e sem data/hora como identidade;
- migration publicada é imutável;
- correção posterior recebe novo identificador;
- não renumerar para “organizar” histórico;
- não criar migration vazia apenas para reservar número.

A primeira migration numerada só deve existir quando houver uma mudança real de schema persistente. O mecanismo de tracking pode existir antes disso como infraestrutura interna do runner.

## Runner e embedding

O Host possui runner próprio e pequeno; não adicionar framework externo de migrations sem necessidade concreta.

As migrations aprovadas ficam versionadas como SQL no source e são **embutidas no Host em build time**. Produção não depende da presença de arquivos `.sql` soltos ao lado do executável.

O registry compilado associa, no mínimo:

```text
migration_id
name
sql bytes
checksum SHA-256
```

O Host cria/garante sua tabela técnica de tracking antes de processar a cadeia:

```text
schema_migrations
- migration_id INTEGER PRIMARY KEY
- name TEXT NOT NULL
- checksum_sha256 TEXT NOT NULL
- applied_at_utc TEXT NOT NULL
```

`schema_migrations` é metadata do próprio runner; migrations numeradas representam mudanças reais do schema de aplicação.

## Validação do histórico

Antes de executar qualquer pendência, o Host compara banco × registry embutido.

Bloqueios obrigatórios:

- migration aplicada com checksum diferente do source atual → `MIGRATION_CHECKSUM_MISMATCH`;
- identificador aplicado inexistente no binário atual → schema/binário incompatível;
- cadeia aplicada com gap/impossibilidade de reconstrução → histórico inválido;
- duas migrations embutidas com mesmo ID/nome incompatível → erro de build/test, nunca escolha em runtime.

Migration já aplicada nunca é executada novamente por alteração silenciosa do arquivo.

## Banco novo

Banco novo começa sem dados de negócio.

Fluxo:

```text
criar arquivo SQLite controlado pelo Host
→ configurar pragmas obrigatórios
→ garantir schema_migrations
→ aplicar migrations reais existentes em ordem
→ quick_check
→ foreign_key_check
→ readiness
```

Não existe “seed de produção” genérico. Bootstrap do primeiro ADM continua sendo fluxo local/controlado próprio e não uma fixture ou migration com credencial padrão.

## Banco existente e pre_migration backup

Se o banco já contém schema de aplicação e existe ao menos uma migration pendente:

1. Host permanece sem readiness normal;
2. obtém coordenação administrativa de `MIGRATION`;
3. cria **um único** backup `origin=system`, motivo `pre_migration`, usando a pipeline D11;
4. somente após backup confirmado inicia o lote de migrations;
5. falha do backup bloqueia a migration e preserva o banco anterior.

Não criar um backup separado para cada migration do mesmo lote.

Banco realmente novo/vazio não exige `pre_migration` porque não há estado anterior de aplicação a recuperar.

## Atomicidade do lote

Baseline: todas as migrations pendentes de um startup devem ser compatíveis com uma única transação explícita de escrita.

```text
BEGIN IMMEDIATE
→ migration N
→ registrar N em schema_migrations
→ migration N+1
→ registrar N+1
→ ...
→ PRAGMA quick_check
→ PRAGMA foreign_key_check
→ COMMIT
```

Se qualquer etapa falhar, o Host executa rollback explícito e não declara readiness.

Regras:

- `foreign_keys` fica habilitado conforme baseline do banco;
- migration não usa `PRAGMA writable_schema=ON` como atalho;
- preferir operações `ALTER TABLE` suportadas e, quando necessário, o procedimento seguro de reconstrução de tabela documentado pelo SQLite;
- se uma migration futura realmente não puder obedecer ao modelo transacional/foreign-key vigente, isso exige análise técnica explícita antes da implementação; o executor não relaxa o contrato por conta própria.

O SQLite documenta transações atômicas e `BEGIN IMMEDIATE` como início explícito de transação de escrita; também documenta cautelas específicas para alterações de schema e reconstrução de tabelas.

## Rollback de versão

Permanece vigente:

- não há down migration automática;
- binário anterior só pode abrir schema que declare suportar;
- se schema atual for incompatível com o binário anterior, rollback operacional exige Restore de backup correspondente;
- correção de migration publicada é sempre nova migration forward.

## Testes de migrations

### Registry/unit

Testes rápidos validam:

- IDs estritamente crescentes e únicos;
- nomes/IDs coerentes;
- checksum determinístico;
- nenhuma entrada duplicada;
- nenhuma migration oficial vazia.

### Integração em SQLite real temporário

Testes de persistência/migrations usam **arquivo SQLite temporário real**, não apenas `:memory:`, para exercitar comportamento de filesystem, journaling e reabertura.

Cobertura mínima:

1. banco vazio → schema atual;
2. cada prefixo suportado da cadeia → schema atual;
3. reaplicar runner em schema atual → no-op;
4. checksum alterado → bloqueio;
5. migration desconhecida/mais nova → bloqueio;
6. falha sintética no meio do lote → rollback integral e `schema_migrations` sem avanço parcial;
7. `quick_check` e `foreign_key_check` válidos após migração.

Os testes de falha usam migrations sintéticas do harness, não arquivos oficiais deliberadamente inválidos.

## Fixtures

Fixtures existem somente para desenvolvimento/teste e devem ser sintéticas.

Direção:

```text
tests/support/
├── fixtures/
└── builders/
```

ou suporte equivalente junto ao package proprietário quando isso reduzir acoplamento.

Regras:

- fixture não é migration;
- fixture não é seed de produção;
- não conter dados reais da empresa/pessoas;
- não conter senha/token/segredo reutilizável;
- builders/factories criam somente o mínimo necessário para o cenário;
- IDs/horários podem ser determinísticos quando isso facilitar asserções;
- snapshot/binário `.sqlite` versionado não é baseline para representar schemas antigos: os próprios prefixes imutáveis de migrations devem gerar esses estados em teste;
- fixture incompatível com schema corrente falha no teste; não é automaticamente “migrada” escondendo problema.

## Testes por camada

Baseline aprovado na Análise 1 é detalhado assim:

- **unitários:** próximos ao módulo/crate, sem I/O quando a unidade não precisa de I/O;
- **integração de package:** APIs internas do owner com diretórios/bancos temporários reais quando persistência/filesystem importam;
- **integração Host:** inicia componentes necessários em porta efêmera e storage temporário, sem IP/hostname corporativo;
- **E2E/smoke:** `tests/e2e/` executa binários reais e raiz de implantação temporária quando a fundação estiver madura o suficiente;
- **ambiente corporativo:** testes que exigem SMB/EDR/WebView2/Word/impressora real continuam gates separados e não recebem resultado inventado fora desse ambiente.

O baseline sem Node não será quebrado apenas para adicionar um test runner JavaScript. Testes frontend que futuramente exijam tooling adicional precisam justificar explicitamente a ferramenta; o smoke inicial do Client pode validar abertura/assets/Tauri sem introduzir Node por conveniência.

## Scripts iniciais

Quando o scaffold for autorizado, a superfície inicial de scripts pode ser:

```text
scripts/
├── check.ps1
├── test.ps1
├── build.ps1
└── package.ps1
```

Responsabilidades:

- `check.ps1` → formatação/check/clippy conforme estágio;
- `test.ps1` → suíte local aplicável;
- `build.ps1` → build lockfile-aware;
- `package.ps1` → monta `dist/` conforme layout Pocket aprovado.

Regras:

- wrappers finos; lógica real permanece nos tools/manifests/código proprietário;
- `Set-StrictMode`/tratamento equivalente e falha propagada;
- sem instalação silenciosa de toolchain;
- sem credencial/configuração corporativa embutida;
- CI futuro deve chamar os mesmos comandos ou equivalentes, não criar uma segunda lógica de build paralela.

## Propostas P12.35–P12.55

- **P12.35:** migrations oficiais ficam em `apps/host/migrations/` com nome `NNNNNN_<slug>.sql`, IDs crescentes de seis dígitos, imutáveis e nunca reutilizados;
- **P12.36:** não criar migration vazia para reservar número; a primeira migration numerada exige mudança real de schema persistente;
- **P12.37:** Host usa runner próprio pequeno e registry compilado; SQL oficial fica no source e é embutido no `StepFlowHost.exe`, sem arquivo `.sql` externo necessário em produção;
- **P12.38:** `schema_migrations` pertence à infraestrutura do runner e guarda `migration_id`, `name`, `checksum_sha256` e `applied_at_utc`;
- **P12.39:** checksum divergente, migration aplicada desconhecida ou cadeia inválida bloqueiam startup/readiness; migration aplicada nunca é alterada/reexecutada silenciosamente;
- **P12.40:** banco novo é criado pelo Host, recebe tracking e cadeia real existente, sem seed genérico de produção ou credencial padrão;
- **P12.41:** bootstrap do primeiro ADM continua fora de fixtures/migrations e segue fluxo local/controlado próprio;
- **P12.42:** banco existente com qualquer migration pendente exige um backup `pre_migration` confirmado antes do lote; banco realmente novo/vazio não exige esse backup;
- **P12.43:** lote pendente usa uma transação explícita de escrita no baseline, registrando migrations na mesma transação; falha causa rollback integral e bloqueia readiness;
- **P12.44:** após aplicar o lote e antes de aceitar readiness, a validação exige `quick_check` válido e `foreign_key_check` vazio;
- **P12.45:** `PRAGMA writable_schema=ON` não é atalho permitido no baseline; mudanças complexas seguem operações suportadas/procedimento seguro SQLite, e exceção futura exige revisão explícita;
- **P12.46:** não existem down migrations automáticas; correção é nova migration forward e rollback de binário respeita compatibilidade/Restore;
- **P12.47:** testes do registry validam IDs, ordem, unicidade, checksum e ausência de migration oficial vazia;
- **P12.48:** integração de migration usa arquivo SQLite temporário real e cobre banco vazio, cada prefixo suportado, no-op atual, checksum divergente, schema mais novo e integridade/FKs;
- **P12.49:** harness de teste injeta migrations sintéticas para provar rollback do lote sem contaminar a cadeia oficial com arquivos inválidos;
- **P12.50:** snapshots binários `.sqlite` versionados não são baseline para schemas históricos; prefixes das migrations imutáveis geram estados antigos em teste;
- **P12.51:** fixtures/builders são sintéticos, mínimos, sem dados reais/segredos e exclusivos de desenvolvimento/teste;
- **P12.52:** fixture nunca é seed/migration de produção e não cria ADM padrão reutilizável;
- **P12.53:** testes ficam divididos em unitários próximos ao owner, integração de package/Host com recursos temporários e `tests/e2e/` somente para fluxos entre binários;
- **P12.54:** o baseline sem Node não é quebrado apenas para adicionar test runner frontend; tooling JavaScript adicional exige necessidade concreta e decisão explícita;
- **P12.55:** scripts iniciais podem ser `check.ps1`, `test.ps1`, `build.ps1` e `package.ps1`, wrappers finos/lockfile-aware, sem instalar dependências ou carregar configuração corporativa silenciosamente.

## Evidência técnica

SQLite documenta que transações explícitas agrupam alterações e que `BEGIN IMMEDIATE` inicia a transação de escrita de forma explícita. Também documenta que alterações de schema mais complexas exigem procedimento cuidadoso e transacional, com verificação de foreign keys quando aplicável.

O contrato do StepFlow continua mais restritivo que a capacidade bruta do SQLite: se uma migration futura exigir exceção ao modelo acima, essa exceção precisa ser decidida antes de entrar na cadeia oficial.

## Fora do escopo desta análise

- criar migration SQL oficial;
- decidir nomes físicos finais de todas as tabelas de negócio;
- criar `schema_migrations` no repositório neste PR;
- criar scripts/fixtures executáveis;
- introduzir framework externo de migration/test;
- definir parâmetros finais de autenticação/Backup;
- sincronizar checkout local;
- autorizar scaffold.
