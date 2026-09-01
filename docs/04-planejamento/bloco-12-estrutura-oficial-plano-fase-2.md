# Bloco 12 — Estrutura oficial + plano da Fase 2

**Status:** EM ANÁLISE — ANÁLISES 1–5 APROVADAS / VALIDAÇÃO FINAL EM REVISÃO  
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

## Análises aprovadas

### Análise 1 — estrutura e publicação — D12.1–D12.18

- workspace por `apps/`/`crates/`;
- Client modular em ES modules e `src-tauri` fino;
- source tree distinto da publicação;
- `StepFlow.exe` como único ponto de entrada normal;
- `_internal/` encapsula a árvore técnica;
- não criar abstrações/crates vazios por antecipação.

### Análise 2 — workspace/build/dependências — D12.19–D12.34

Fonte: `bloco-12-analise-2-workspace-build-dependencias.md`.

- Rust 1.98.0, Edition 2024, resolver 3, Windows x64 MSVC;
- toolchain + `Cargo.lock` versionados;
- dependências lockfile-aware e somente com uso real;
- Client vanilla sem Node/npm/Vite/bundler/framework;
- configuração build/dev × deployment × runtime;
- produção montada por packaging.

### Análise 3 — migrations/scripts/testes/fixtures — D12.35–D12.55

Fonte: `bloco-12-analise-3-migrations-testes-fixtures.md`.

- migrations Host-side imutáveis e embutidas;
- registry + `schema_migrations` com checksum;
- lote transacional e validação de integridade/FKs;
- sem down migration automática;
- testes em SQLite temporário real;
- fixtures sintéticas;
- scripts finos de check/test/build/package.

### Análise 4 — parâmetros finais — D12.56–D12.79

Fonte: `bloco-12-analise-4-parametros-finais.md`.

- Argon2id/senha/blocklist/throttling/token/sessão;
- Gerência pode alterar configuração da empresa;
- limites de identidade/logo;
- categoria arquivada herdada preservada com aviso, mas não adicionável enquanto arquivada;
- retenção/limites/espaço/timeouts de Backup/Restore;
- readiness/relaunch/reconexão;
- rotação de logs/admin audit;
- parâmetros centralizados, sem defaults inseguros silenciosos.

### Análise 5 — plano detalhado da Fase 2 — D12.80–D12.98

Fonte: `bloco-12-analise-5-plano-fase-2.md`.

```text
Gate Fase 1 + sync local seguro
→ F2-T01 workspace/tooling + Host mínimo
→ F2-T02 Host runtime/readiness
→ F2-T03 SQLite + migrations runner
→ F2-T04 Controller lifecycle
→ F2-T05 Client Tauri + compatibilidade
→ F2-T06 Launcher Pocket
→ F2-T07 packaging Pocket
→ F2-T08 smoke integrado + gates Windows/Pocket
→ Gate Fase 2
```

Cada tarefa usa branch/PR próprios, nasce de `main` consolidada, recebe pré-flight separado e não antecipa funcionalidades das fases posteriores.

## Análise 6 — validação final da Fase 1

**EM REVISÃO — P12.99–P12.108.**

Fonte: `bloco-12-analise-6-validacao-final-fase-1.md`.

A validação encontrou apenas refinamentos operacionais/documentais:

- configuração inválida de retenção deve falhar explicitamente;
- cada parâmetro tem owner único e só vira knob quando documentado como configurável;
- `deployment.json` real de produção precisa de input explícito, sem placeholder silencioso;
- sync local limpo usa `fetch --prune` + `merge --ff-only` e para diante de qualquer alteração/divergência inesperada;
- gates corporativos não bloqueiam o encerramento documental da Fase 1, mas podem bloquear a saída da Fase 2/produção;
- merge do Bloco 12 não cria autorização automática de scaffold.

## Estado das análises

1. Estrutura + publicação — **APROVADA** — D12.1–D12.18;
2. Workspace/build/dependências — **APROVADA** — D12.19–D12.34;
3. Migrations/scripts/testes/fixtures — **APROVADA** — D12.35–D12.55;
4. Parâmetros finais — **APROVADA** — D12.56–D12.79;
5. Plano da Fase 2 — **APROVADA** — D12.80–D12.98;
6. Validação final da Fase 1 — **EM REVISÃO** — P12.99–P12.108.

## Gate vigente

Até a aprovação da validação final e o fechamento Git do Bloco 12 permanece proibido:

- criar workspace/toolchain/lockfile oficiais;
- criar apps/crates/scaffold runtime;
- criar migration SQL oficial;
- criar scripts/fixtures executáveis;
- implementar UI/Host/Launcher/Controller;
- sincronizar ou alterar automaticamente o checkout local do PO;
- iniciar tarefa Codex de implementação.
