# Bloco 12 — Análise 3 — Migrations, scripts, testes e fixtures

**Status:** APROVADA PELO PO — D12.35–D12.55  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Contrato consolidado

Migrations pertencem ao Host, são parte do produto e precisam ser reproduzíveis, auditáveis e recuperáveis.

```text
source migration versionada
→ embutida no StepFlowHost.exe
→ Host valida histórico/checksums antes de readiness
→ banco existente com pendências recebe pre_migration backup
→ lote pendente aplica de forma atômica
→ quick_check + foreign_key_check
→ readiness
```

### Localização e runner

```text
apps/host/
├── migrations/
│   ├── 000001_<slug>.sql
│   └── ...
└── src/persistence/migrations.rs
```

- IDs decimais de seis dígitos, crescentes, nunca reutilizados;
- slug `snake_case` descritivo;
- migration publicada é imutável; correção recebe novo ID;
- não criar migration vazia para reservar número;
- Host usa runner próprio pequeno, sem framework externo no baseline;
- SQL oficial é embutido no executável; produção não depende de `.sql` solto.

Tracking mínimo:

```text
schema_migrations
- migration_id INTEGER PRIMARY KEY
- name TEXT NOT NULL
- checksum_sha256 TEXT NOT NULL
- applied_at_utc TEXT NOT NULL
```

Checksum divergente, migration aplicada desconhecida ou cadeia inválida bloqueiam startup/readiness.

### Banco novo e banco existente

Banco novo é criado pelo Host, recebe pragmas, `schema_migrations` e migrations reais existentes. Não há seed genérico de produção nem ADM/senha padrão.

Banco existente com pelo menos uma migration pendente exige um único backup `origin=system`, motivo `pre_migration`, confirmado antes do lote. Falha desse backup impede migration.

### Atomicidade

Baseline:

```text
BEGIN IMMEDIATE
→ migration N + registro N
→ migration N+1 + registro N+1
→ ...
→ quick_check
→ foreign_key_check
→ COMMIT
```

Falha causa rollback integral e bloqueia readiness. `PRAGMA writable_schema=ON` não é atalho permitido. Mudança de schema complexa segue operações/procedimentos seguros do SQLite. Exceção futura ao modelo transacional exige nova análise explícita.

Não existem down migrations automáticas. Correções são forward migrations; rollback de binário respeita compatibilidade e Restore.

### Testes e fixtures

- unitários validam IDs, ordem, unicidade, checksum e ausência de migration vazia;
- integração usa arquivo SQLite temporário real;
- cobrir banco vazio, cada prefixo suportado, no-op atual, checksum divergente, schema mais novo, rollback sintético e integridade/FKs;
- falhas são injetadas por migrations sintéticas do harness, não por arquivos oficiais quebrados;
- snapshots `.sqlite` históricos versionados não são baseline: prefixes das migrations geram os estados antigos;
- fixtures/builders são sintéticos, mínimos, sem dados reais/segredos e nunca seed de produção;
- testes dividem-se em unitários, integração do owner/Host e `tests/e2e/` entre binários;
- baseline sem Node não é quebrado apenas para obter test runner frontend.

Scripts iniciais autorizáveis na Fase 2:

```text
scripts/
├── check.ps1
├── test.ps1
├── build.ps1
└── package.ps1
```

São wrappers finos, lockfile-aware, sem instalação silenciosa de toolchain e sem configuração corporativa embutida.

## Decisões D12.35–D12.55

- **D12.35:** migrations oficiais em `apps/host/migrations/NNNNNN_<slug>.sql`, IDs crescentes de seis dígitos, imutáveis e nunca reutilizados;
- **D12.36:** não criar migration vazia para reservar número;
- **D12.37:** runner próprio pequeno do Host, registry compilado e SQL embutido no `StepFlowHost.exe`;
- **D12.38:** `schema_migrations` guarda ID, nome, SHA-256 e timestamp UTC;
- **D12.39:** checksum divergente, migration desconhecida ou cadeia inválida bloqueiam readiness;
- **D12.40:** banco novo nasce sem seed genérico de produção ou credencial padrão;
- **D12.41:** bootstrap do primeiro ADM permanece fluxo local/controlado fora de fixtures/migrations;
- **D12.42:** banco existente com pendências exige um único `pre_migration` backup confirmado antes do lote;
- **D12.43:** lote pendente é transacional no baseline; falha causa rollback integral;
- **D12.44:** `quick_check` válido + `foreign_key_check` vazio são exigidos antes de readiness;
- **D12.45:** `writable_schema` não é atalho permitido; exceção futura exige revisão explícita;
- **D12.46:** não existem down migrations automáticas;
- **D12.47:** testes do registry validam IDs, ordem, unicidade, checksum e ausência de migration vazia;
- **D12.48:** integração usa SQLite temporário real e cobre prefixes, no-op, mismatch, schema mais novo e integridade/FKs;
- **D12.49:** rollback é provado com migrations sintéticas do harness;
- **D12.50:** não versionar `.sqlite` histórico como baseline; prefixes imutáveis geram estados antigos;
- **D12.51:** fixtures/builders são sintéticos, mínimos e sem dados reais/segredos;
- **D12.52:** fixture nunca é seed/migration de produção nem cria ADM padrão reutilizável;
- **D12.53:** testes = unitários junto ao owner + integração package/Host + E2E entre binários;
- **D12.54:** Node não entra apenas para test runner frontend;
- **D12.55:** scripts iniciais podem ser `check.ps1`, `test.ps1`, `build.ps1`, `package.ps1`, wrappers finos e lockfile-aware.

## Fora do escopo

Nenhuma migration SQL, script, fixture, banco ou scaffold é criado neste PR. Implementação continua bloqueada até o gate final do Bloco 12.
