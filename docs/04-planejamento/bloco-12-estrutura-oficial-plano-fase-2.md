# Bloco 12 — Estrutura oficial + plano da Fase 2

**Status:** EM ANÁLISE — ANÁLISES 1–4 APROVADAS  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-09-01

## Objetivo

Fechar o último gate da Fase 1 e transformar os contratos aprovados em uma fundação executável planejada, sem iniciar implementação funcional neste PR.

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

- workspace por `apps/`/`crates/`;
- Client modular em ES modules e `src-tauri` fino;
- Launcher/Controller/Host com ownership separado;
- source tree distinto da publicação;
- publicação com `StepFlow.exe` como único ponto de entrada normal e `_internal/` técnico;
- `.lnk` não é requisito;
- não criar abstrações/crates vazios por antecipação.

---

## Análise 2 — workspace/build/dependências

**APROVADA — D12.19–D12.34.**

Fonte: `bloco-12-analise-2-workspace-build-dependencias.md`.

- Rust 1.98.0, Edition 2024, resolver 3, Windows x64 MSVC;
- `rust-toolchain.toml` + `Cargo.lock` versionados;
- dependências lockfile-aware e somente com uso real;
- Client vanilla sem Node/npm/Vite/bundler/framework;
- configuração build/dev × deployment × runtime;
- produção montada por packaging;
- scripts PowerShell finos e sem tuning prematuro.

---

## Análise 3 — migrations/scripts/testes/fixtures

**APROVADA — D12.35–D12.55.**

Fonte: `bloco-12-analise-3-migrations-testes-fixtures.md`.

- migrations Host-side `NNNNNN_<slug>.sql`, imutáveis;
- registry compilado + SQL embutido;
- `schema_migrations` com ID/nome/checksum/timestamp;
- lote transacional, validação de integridade/FKs e sem down migration automática;
- banco existente com pendências exige `pre_migration` backup quando esse mecanismo estiver operacional;
- testes em SQLite temporário real;
- fixtures sintéticas, sem seed de produção;
- scripts `check/test/build/package` como wrappers finos.

---

## Análise 4 — parâmetros finais

**APROVADA — D12.56–D12.79.**

Fonte: `bloco-12-analise-4-parametros-finais.md`.

Contrato aprovado:

- Argon2id 64 MiB / 3 passes / 4 lanes; salt 16 bytes; output 32 bytes;
- senha 15–128 caracteres Unicode após NFKC, sem composition rule/rotação periódica;
- blocklist offline ≥10.000 e throttling progressivo;
- token opaco 256 bits, Client-memory-only e digest server-side;
- sessão 30 min idle / 8 h absoluta;
- GERENCIA pode alterar configuração da empresa; Funcionário não;
- identidade 120/160/200/254 e logo PNG/JPEG até 2 MiB / 2048×2048;
- categoria arquivada herdada é preservada com aviso e removível, mas não adicionável enquanto arquivada;
- retenção default 20, faixa 5–100;
- Backup/Restore bounded conforme D12.67–D12.73;
- logs técnicos e admin audit rotacionam conforme D12.74–D12.75;
- parâmetros ficam centralizados e não podem cair silenciosamente em defaults inseguros.

---

## Análise 5 — plano detalhado da Fase 2

**EM REVISÃO — P12.80–P12.98.**

Fonte: `bloco-12-analise-5-plano-fase-2.md`.

Sequência proposta:

```text
Gate Fase 1 + sync local seguro
→ F2-T01 workspace/tooling + Host mínimo
→ F2-T02 Host runtime/readiness
→ F2-T03 SQLite + migrations runner
→ F2-T04 Controller lifecycle
→ F2-T05 Client Tauri + compatibilidade
→ F2-T06 Launcher Pocket/local Client
→ F2-T07 packaging Pocket
→ F2-T08 smoke integrado + gates Windows/Pocket
→ Gate Fase 2
```

Princípios:

- uma tarefa lógica por branch/PR;
- cada tarefa nasce de `main` consolidada;
- pré-flight PO e prompt Codex permanecem separados;
- não criar crates vazios;
- não antecipar autenticação, domínio, documentos ou Backup/Restore funcional;
- validações corporativas não são declaradas PASS fora do ambiente real.

Nada de P12.80–P12.98 é contrato até aprovação do PO.

---

## Estado das análises

1. Estrutura + publicação Pocket — **APROVADA** — D12.1–D12.18;
2. Workspace/build/dependências — **APROVADA** — D12.19–D12.34;
3. Migrations/scripts/testes/fixtures — **APROVADA** — D12.35–D12.55;
4. Parâmetros finais — **APROVADA** — D12.56–D12.79;
5. Plano detalhado da Fase 2 + sequência Codex — **EM REVISÃO** — P12.80–P12.98;
6. Validação final da Fase 1 + gate Git + sincronização segura do checkout + autorização do primeiro scaffold — pendente.

## Gate vigente

Até o encerramento do Bloco 12 permanece proibido:

- criar workspace/toolchain/lockfile oficiais;
- criar apps/crates/scaffold runtime;
- criar migration SQL oficial;
- criar scripts/fixtures executáveis;
- implementar UI/Host/Launcher/Controller;
- sincronizar ou alterar automaticamente o checkout local do PO;
- iniciar tarefa Codex de implementação.
