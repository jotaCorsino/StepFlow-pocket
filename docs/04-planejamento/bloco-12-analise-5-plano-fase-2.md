# Bloco 12 — Análise 5 — Plano detalhado da Fase 2 e sequência de tarefas Codex

**Status:** CONSOLIDADO / APROVADO PELO PO — D12.80–D12.98  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Transformar D12.1–D12.79 em uma sequência pequena, verificável e executável para a **Fase 2 — Fundação técnica executável**, sem antecipar funcionalidades das Fases 3+.

A aprovação deste plano não inicia implementação. A Fase 2 só pode começar após o fechamento Git do Bloco 12 e a sincronização segura do checkout local.

## Gate Fase 1 → Fase 2

Antes da primeira tarefa executável:

1. PR do Bloco 12 aprovado e squash-merged em `main`;
2. branch remota removida;
3. zero PRs abertos e remoto limpo;
4. `main` remoto identificado por SHA exato;
5. checkout local `C:\dev\StepFlow` inspecionado com `git rev-parse HEAD` e `git status --short --branch`;
6. alterações locais preexistentes do PO preservadas;
7. nenhum `reset --hard`, `clean`, `stash` ou descarte para forçar alinhamento;
8. primeiro branch de implementação somente depois da sincronização segura.

## Sequência da Fase 2

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
→ Fase 3
```

### F2-T01 — Workspace + tooling + Host mínimo

Cria `rust-toolchain.toml`, workspace virtual, lockfile, `.gitignore`, scripts finos e somente `apps/host` como primeiro package concreto. Não cria Controller, Client, Launcher, SQLite, HTTP, domínio ou crates vazios.

Aceite mínimo: fmt/check/clippy/test `--locked`, build x64 MSVC no ambiente aplicável e nenhum Node/package manager.

### F2-T02 — Host runtime

Configuração runtime, logging, Axum/Tokio, health/readiness/metadata de compatibilidade e shutdown gracioso. Ainda sem banco/domínio.

### F2-T03 — SQLite + migrations runner

`rusqlite` bundled, `data/stepflow.sqlite`, `schema_migrations`, registry/checksums, runner transacional, `quick_check` + `foreign_key_check` e testes em SQLite temporário real.

Não criar `000001` vazia. Até Backup `pre_migration` existir, mudança real de schema em banco persistente existente não pode ser promovida a produção sem a proteção correspondente.

### F2-T04 — Controller lifecycle

Controller inicia Host, garante exclusividade, espera readiness bounded e coordena shutdown gracioso, sem serviço/watchdog.

### F2-T05 — Client Tauri mínimo

HTML/CSS/JavaScript em ES modules + `src-tauri` fino, sem Node/bundler. Lê `deployment.json`, consulta compatibilidade do Host e mostra estados técnicos mínimos. Login/UI de produto ficam fora.

WebSocket operacional autenticado não é antecipado por canal anônimo.

### F2-T06 — Launcher Pocket

Launcher Rust transitório valida manifest/deployment/release, prepara Client em `%LOCALAPPDATA%`, ativa versão local, inicia o Client e encerra. Sem admin, instalador, Internet ou updater residente.

### F2-T07 — Packaging Pocket

`scripts/package.ps1` monta `dist/` reproduzível:

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\
    └── server\
```

`StepFlow.exe` é o Launcher empacotado. Produção não é executada diretamente de `target/release` e o pacote não contém segredo, endpoint corporativo versionado ou banco real.

### F2-T08 — Smoke integrado e gates Pocket/Windows

Prova build/package limpos, Controller→Host→readiness, SQLite/migrations, Client↔Host, Launcher→Client local, shutdown sem processo residual e ausência de dependência dev no pacote.

SMB, WebView2, Windows alvo, ACL/EDR/antivírus e ausência real de elevação/manualidade são validados no ambiente corporativo apropriado. Fora dele, registrar `NÃO APLICÁVEL NESTE AMBIENTE`, nunca PASS presumido.

## Branches previstas

```text
feat/f2-01-workspace-host
feat/f2-02-host-runtime
feat/f2-03-sqlite-migrations
feat/f2-04-controller-lifecycle
feat/f2-05-client-foundation
feat/f2-06-launcher-pocket
feat/f2-07-packaging-pocket
feat/f2-08-foundation-validation
```

Cada tarefa nasce de `main` consolidada após o gate da anterior e usa pré-flight PO separado do prompt Codex.

## Critério de saída da Fase 2

- workspace/toolchain/build reproduzíveis e lockfile-aware;
- Host/readiness/config/logging funcionando;
- SQLite e migrations determinísticos;
- Controller lifecycle bounded;
- Client Tauri vanilla integrado ao Host;
- Launcher preparando Client local automaticamente;
- packaging `StepFlow.exe + _internal/` reproduzível;
- smoke integrado aprovado no ambiente aplicável;
- gates Pocket corporativos registrados como PASS ou blocker real;
- nenhuma antecipação de autenticação, domínio, documentos ou Backup/Restore funcional.

## Decisões D12.80–D12.98

- **D12.80:** Fase 2 só começa após fechamento Git do Bloco 12 e sincronização local segura;
- **D12.81:** cada tarefa da Fase 2 usa branch/PR próprios, nasce de `main` consolidada e recebe pré-flight separado;
- **D12.82:** F2-T01 cria workspace/tooling/lockfile e somente `apps/host`, sem scaffold vazio dos demais componentes;
- **D12.83:** F2-T02 implementa configuração, logging, Axum/Tokio, health/readiness/compatibilidade mínima e shutdown;
- **D12.84:** F2-T03 implementa SQLite bundled + `schema_migrations` + runner/checksums/testes sem migration vazia ou schema artificial;
- **D12.85:** até `pre_migration` estar operacional, schema de banco persistente existente não é promovido sem proteção D11/D12;
- **D12.86:** F2-T04 implementa Controller lifecycle/exclusividade/readiness/shutdown bounded sem serviço/watchdog;
- **D12.87:** F2-T05 cria Client Tauri vanilla modular e compatibilidade HTTP sem login/UI de produto;
- **D12.88:** WebSocket operacional autenticado não é antecipado por canal anônimo;
- **D12.89:** F2-T06 implementa Launcher transitório e preparação/ativação local do Client sem admin/instalador/Internet;
- **D12.90:** F2-T07 implementa packaging reproduzível `StepFlow.exe + _internal/` em `dist/`;
- **D12.91:** F2-T08 executa smoke integrado e separa gates dependentes do ambiente corporativo real;
- **D12.92:** teste fora do ambiente corporativo não declara PASS para SMB/EDR/WebView2 real;
- **D12.93:** falha Pocket em estação suportada é blocker, não autorização para manualidade/admin;
- **D12.94:** Fase 2 não antecipa autenticação/usuários, Procedimentos, Atendimentos, documentos ou Backup/Restore funcional;
- **D12.95:** crates compartilhados só nascem quando houver reutilização real concreta;
- **D12.96:** cada tarefa termina com validação mecânica proporcional, diff/status Git e documentação viva quando afetada;
- **D12.97:** saída da Fase 2 exige integração reproduzível de Host/SQLite/Controller/Client/Launcher/packaging sem processo residual ou dependência dev em produção;
- **D12.98:** concluir Fase 2 apenas autoriza o gate para Fase 3; não executa automaticamente a próxima fase.

## Fora do escopo

- criar branches/tasks agora;
- criar prompt Codex executável antes do gate final da Fase 1;
- criar scaffold/código;
- reabrir contratos funcionais das Fases 3–9;
- declarar validação corporativa sem executá-la no ambiente real.
