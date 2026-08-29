# Bloco 11 — Backup / Restauração técnico

**Status:** PROPOSTA / EM ANÁLISE — NÃO CONSOLIDADO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29

## Objetivo

Fechar o contrato técnico de Backup/Restore antes da estrutura oficial e da Fase 2, sem criar implementação funcional nesta etapa.

## Contratos já consolidados

A UX permanece definida em `../02-telas/13-backup-restauracao.md`.

Este bloco não reabre, sem bloqueador técnico concreto:

- Backup/Restore coordenado pelo Host;
- Client sem acesso direto ao SQLite;
- autorização Host-side;
- Restore normal somente com backup elegível e confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva do Restore normal;
- falha do safety backup bloqueia o Restore normal;
- recuperação sem Host funcional pertence a disaster recovery local/controlado;
- sucesso somente após confirmação do Host;
- Backup separado de exportação documental;
- ausência de scheduler periódico por inferência;
- Restore de Gerência não autorizado; Gerência × Backup permanece pendente;
- contrato Pocket como gate superior.

## Tópicos que o Bloco 11 deve fechar

1. conjunto exato de dados e arquivos que formam o estado recuperável;
2. snapshot consistente de SQLite + arquivos administrados;
3. formato e identidade do backup;
4. manifesto, verificação e compatibilidade entre versões/schema;
5. escrita completa, promoção e tratamento de backup parcial;
6. catálogo e retenção inicial;
7. coordenação com mutações e operações administrativas;
8. Restore normal e safety backup;
9. restart, reconexão e sessões após Restore;
10. falhas parciais e resultado incerto;
11. disaster recovery local quando o Host não inicia;
12. capacidades e auditoria;
13. validação técnica final.

## Ordem de análise proposta

1. estado recuperável + envelope do backup;
2. consistência + escrita/promoção/verificação;
3. catálogo + retenção + coordenação;
4. Restore + safety backup + compatibilidade;
5. restart/sessões/reconexão + falhas;
6. disaster recovery + capacidades/auditoria;
7. validação técnica final.

A ordem organiza o trabalho; não aprova antecipadamente nenhuma alternativa técnica.

## Análise 1 — estado recuperável + envelope do backup

**Status da análise:** PROPOSTA PARA REVISÃO DO PO.

### 1.1 Fronteira do estado recuperável

A arquitetura vigente separa:

```text
StepFlow\
├── app\
├── config\
├── data\
│   ├── stepflow.sqlite
│   ├── company\
│   └── avatars\
├── logs\
└── backups\
```

A proposta é tratar o Backup normal do StepFlow como **backup de estado da aplicação**, não como imagem completa da implantação/servidor.

Entram no estado recuperável inicial:

```text
data\stepflow.sqlite
data\company\**
data\avatars\**
```

Regras propostas:

- `stepflow.sqlite` contém o estado relacional oficial;
- `company/` contém arquivos administrados da identidade da empresa que não estejam no SQLite;
- `avatars/` contém arquivos administrados de usuários;
- novo tipo de arquivo persistente só entra no backup após inclusão explícita no contrato/allowlist do Host;
- não copiar recursivamente qualquer conteúdo apenas por estar abaixo da pasta StepFlow.

Ficam fora do Backup normal inicial:

- `app/` — binários substituíveis;
- `config/` — configuração operacional/de implantação;
- `logs/` — diagnóstico, não estado funcional a restaurar;
- `backups/` — nunca incluir backups dentro de backup;
- exportações PDF/DOCX/Ficha salvas pelo usuário;
- temporários do Host/Client;
- cópias locais do Client em `%LOCALAPPDATA%`.

Motivo para `config/` ficar fora: um Restore de dados não deve reverter silenciosamente endereço, porta, paths ou outras escolhas específicas da implantação e tornar o Host inacessível. Configuração funcional que precise acompanhar o estado restaurado deve residir no banco/arquivos administrados ou ser adicionada explicitamente ao contrato futuro.

Consequência: este Backup permite reconstruir o **estado StepFlow** dentro de uma implantação compatível; não pretende ser backup bare-metal do Windows nem da pasta de deployment inteira.

### 1.2 Envelope físico proposto

Proposta inicial: **um único arquivo de backup imutável por snapshot**, administrado pelo Host em `backups/`.

Formato lógico:

```text
backup-<utc>-<backup_id>.stepflow-backup

manifest.json
payload/
├── stepflow.sqlite
├── company/
└── avatars/
```

Direção proposta para o container:

- container ZIP padrão sob extensão própria `.stepflow-backup`;
- baseline sem compressão (`Stored`) para reduzir CPU, dependências e variabilidade;
- compressão futura, se realmente necessária, exige nova versão/capacidade do formato;
- pacote é lido/escrito pelo StepFlow; nenhuma ferramenta externa é requisito operacional;
- a extensão própria identifica que o arquivo possui semântica StepFlow e não é um ZIP genérico para edição manual.

Motivos para arquivo único em vez de diretório de backup:

- unidade operacional mais simples para listar, mover e validar;
- evita backup parcial espalhado por vários arquivos finais;
- facilita safety backup e disaster recovery;
- permite promoção final do pacote como uma única unidade de filesystem;
- mantém o conteúdo lógico explícito por manifesto.

### 1.3 Manifesto mínimo proposto

`manifest.json` deve conter pelo menos:

- `format_version`;
- `backup_id` opaco;
- `created_at` em UTC;
- `origin` (`manual` ou `system`);
- ator responsável quando aplicável;
- motivo técnico quando `origin = system`, quando aplicável;
- versão StepFlow de origem;
- versão/schema ou migration de origem necessária à compatibilidade;
- lista ordenada de entradas do payload;
- path lógico normalizado de cada entrada;
- tamanho em bytes;
- SHA-256 de cada entrada.

SHA-256 aqui é verificação de integridade/corrupção, não assinatura/autenticidade do pacote.

O manifesto não deve carregar senha, token reutilizável ou conteúdo de negócio desnecessário.

### 1.4 Paths e segurança do envelope

- somente paths relativos e normalizados;
- rejeitar entrada absoluta ou que escape do namespace lógico;
- não seguir reparse points/symlinks durante coleta ou restauração;
- Restore mapeia nomes lógicos conhecidos para destinos controlados pelo Host; nunca confia em path arbitrário vindo do pacote;
- entrada desconhecida em versão de formato não suportada torna o pacote incompatível, não é copiada por conveniência.

### 1.5 Lifecycle físico proposto

Um backup ainda em construção não aparece como backup válido.

```text
staging interno
→ materializar snapshot
→ construir pacote
→ finalizar escrita
→ verificar manifesto + payload
→ promover para filename final
→ somente então listar como confirmado
```

- staging não usa filename final;
- falha antes da promoção deixa somente resíduo não confirmado para cleanup conservador;
- colisão nunca sobrescreve backup confirmado existente;
- promoção final deve ocorrer no mesmo filesystem sempre que possível;
- a etapa seguinte do Bloco 11 fechará flush/durabilidade/crash safety em detalhe.

### 1.6 SQLite — direção técnica para a próxima análise

Para o banco, a candidata preferencial é a **SQLite Online Backup API**, exposta pelo `rusqlite` através da feature `backup`.

Razões:

- produz cópia consistente de banco ativo por API oficial;
- evita copiar `stepflow.sqlite` cru enquanto transações/WAL estão ativos;
- integra-se ao Host Rust já escolhido;
- pode copiar toda a base em uma operação ou trabalhar incrementalmente;
- usa menos CPU que `VACUUM INTO` segundo a documentação SQLite.

`VACUUM INTO` permanece alternativa tecnicamente válida, mas não é a preferência inicial porque o objetivo do backup não exige compactar/reorganizar a base a cada snapshot.

A decisão formal sobre janela de mutações, coordenação com WAL e arquivos externos pertence à **Análise 2 — consistência + escrita/promoção/verificação**.

### 1.7 Propostas resultantes da Análise 1

Para aprovação futura, sem consolidar ainda:

- **P11.1:** Backup normal protege estado da aplicação, não a implantação inteira;
- **P11.2:** payload inicial = `stepflow.sqlite` + `company/**` + `avatars/**`;
- **P11.3:** `app/`, `config/`, `logs/`, `backups/`, exportações, temporários e Client local ficam fora;
- **P11.4:** inclusão futura de novo arquivo persistente exige allowlist/contrato explícito;
- **P11.5:** um backup confirmado é um único pacote imutável `.stepflow-backup`;
- **P11.6:** container proposto = ZIP padrão, método `Stored` no baseline;
- **P11.7:** pacote contém `manifest.json` + `payload/` em paths lógicos controlados;
- **P11.8:** manifesto versionado registra origem, compatibilidade, tamanho e SHA-256 por entrada;
- **P11.9:** pacote parcial nunca é listado como válido; staging precede promoção;
- **P11.10:** Online Backup API é candidata preferencial para o snapshot SQLite na análise seguinte.

## Critérios de fechamento

O bloco só pode ser considerado concluído quando as decisões acima permitirem implementação futura sem escolhas críticas deixadas ao executor e quando:

- a UX da Tela 13 continuar coerente;
- o modelo de dados/migrations souber quais impactos precisará absorver na fase executável;
- o contrato Pocket permanecer intacto;
- nenhum backup parcial puder ser tratado como válido;
- Restore tiver estados de falha e recuperação definidos;
- disaster recovery possuir fronteira clara em relação ao Restore normal;
- decisões aprovadas forem sincronizadas nas fontes específicas.

## Fora do escopo desta abertura

- implementação funcional;
- migrations oficiais;
- scheduler periódico;
- serviço persistente de backup;
- backup em nuvem;
- integração com destino externo específico;
- nova UX;
- números finais de performance sem evidência.

## Próxima análise

Após revisão das propostas P11.1–P11.10, avançar para **consistência + escrita/promoção/verificação**, incluindo a janela de mutações necessária para capturar SQLite e arquivos administrados como um único snapshot lógico.
