# Bloco 12 — Estrutura oficial + plano da Fase 2

**Status:** EM ANÁLISE — ANÁLISES 1–3 APROVADAS  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-09-01

## Objetivo

Fechar o último gate da Fase 1 e transformar os contratos já aprovados em uma fundação executável planejada, sem iniciar implementação funcional neste PR.

O Bloco 12 fecha:

1. árvore oficial e fronteiras entre componentes;
2. workspace/build/dependências/configuração;
3. migrations, scripts, testes e fixtures;
4. parâmetros finais que não podem ficar à escolha do executor;
5. plano detalhado da Fase 2;
6. sincronização segura do checkout local;
7. gate explícito que autoriza o primeiro scaffold/runtime oficial.

Restrições herdadas permanecem: Pocket sem instalador/manualidade/admin/Internet/toolchain em produção; Client Tauri 2 + HTML/CSS/JS modular; Host/Controller/Launcher Rust; SQLite Host-only; sem serviço/watchdog/daemon baseline; nenhuma implementação antes do gate final.

---

## Análise 1 — estrutura e publicação Pocket

**APROVADA — D12.1–D12.18.**

Source tree baseline:

```text
StepFlow/
├── Cargo.toml
├── apps/
│   ├── client/
│   ├── launcher/
│   ├── controller/
│   └── host/
├── crates/
│   ├── protocol/
│   ├── domain/
│   ├── documents/
│   └── platform-windows/
├── scripts/
├── tests/e2e/
└── docs/
```

Publicação Pocket:

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\...
    └── server\...
```

- `StepFlow.exe` é o Launcher com nome/ícone amigáveis e único ponto de entrada normal;
- `_internal/` encapsula artefatos técnicos;
- `.lnk` não é requisito;
- frontend Client usa ES modules e `src-tauri` fino;
- `apps/`/`crates/` representam ownership, não obrigação de criar abstrações vazias;
- aprovação estrutural não autoriza scaffold antes do gate final.

---

## Análise 2 — workspace/build/dependências

**APROVADA — D12.19–D12.34.**

Fonte: `bloco-12-analise-2-workspace-build-dependencias.md`.

Contrato:

- Rust 1.98.0, Edition 2024, resolver 3, target Windows x64 MSVC;
- `rust-toolchain.toml` + `Cargo.lock` versionados;
- dependências lockfile-aware e adotadas somente com uso real;
- baseline inicial Tauri/Tokio/Axum/rusqlite/Serde/tracing conforme D12.26;
- Client vanilla sem Node/npm/Vite/bundler/framework no baseline;
- configuração build/dev × deployment × runtime central;
- `target/` e `dist/` descartáveis; produção montada por packaging;
- scripts PowerShell finos;
- tuning e dependências futuras entram apenas com necessidade/benchmark explícitos.

---

## Análise 3 — migrations/scripts/testes/fixtures

**APROVADA — D12.35–D12.55.**

Fonte: `bloco-12-analise-3-migrations-testes-fixtures.md`.

Contrato:

- migrations em `apps/host/migrations/NNNNNN_<slug>.sql`, imutáveis;
- runner pequeno do Host + registry compilado + SQL embutido no executável;
- `schema_migrations` com ID/nome/checksum/timestamp;
- banco existente com pendências exige um `pre_migration` backup confirmado;
- lote pendente transacional no baseline;
- `quick_check` + `foreign_key_check` antes de readiness;
- sem down migration automática ou `writable_schema` como atalho;
- testes de migration usam SQLite temporário real e prefixes da cadeia;
- fixtures são sintéticas e nunca seed de produção;
- scripts iniciais autorizáveis: `check.ps1`, `test.ps1`, `build.ps1`, `package.ps1`;
- nenhuma migration/script/fixture foi criada neste PR.

---

## Análise 4 — parâmetros finais

**EM REVISÃO — P12.56–P12.79.**

Fonte: `bloco-12-analise-4-parametros-finais.md`.

A proposta fecha:

- Argon2id, senha, throttling, token e sessão;
- Gerência × configuração da empresa;
- limites de campos/logo;
- regra de categoria arquivada em nova revisão;
- retenção/limites/espaço/timeouts de Backup/Restore;
- readiness/relaunch/reconexão;
- rotação inicial de logs/admin audit.

Nada de P12.56–P12.79 é contrato até aprovação do PO.

---

## Estado das análises

1. Estrutura + publicação Pocket — **APROVADA** — D12.1–D12.18;
2. Workspace/build/dependências — **APROVADA** — D12.19–D12.34;
3. Migrations/scripts/testes/fixtures — **APROVADA** — D12.35–D12.55;
4. Parâmetros finais — **EM REVISÃO** — P12.56–P12.79;
5. Plano detalhado da Fase 2 + sequência de tarefas Codex — pendente;
6. Validação final da Fase 1 + sincronização segura do checkout + autorização do primeiro scaffold — pendente.

## Gate vigente

Até o encerramento do Bloco 12 permanece proibido:

- criar `Cargo.toml`/toolchain/lockfile oficiais;
- criar apps/crates/scaffold runtime;
- criar migration SQL oficial;
- criar scripts/fixtures executáveis;
- implementar UI/Host/Launcher/Controller;
- sincronizar ou alterar automaticamente o checkout local do PO;
- iniciar tarefa Codex de implementação.
