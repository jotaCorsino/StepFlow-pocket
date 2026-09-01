# Bloco 12 — Estrutura oficial + plano da Fase 2

**Status:** EM ANÁLISE — ANÁLISES 1–2 APROVADAS  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-09-01

## Objetivo

Fechar o último gate da Fase 1 e transformar os contratos já aprovados em uma fundação executável planejada, sem iniciar implementação funcional neste PR.

O Bloco 12 deve decidir:

1. árvore oficial do repositório e fronteiras entre componentes;
2. estratégia de workspace/build/configuração de desenvolvimento;
3. localização e disciplina de migrations, scripts e testes iniciais;
4. parâmetros finais ainda pendentes que não podem ficar à escolha do executor;
5. sincronização segura do checkout local antes do primeiro trabalho executável;
6. plano da Fase 2 e critério explícito que autoriza o primeiro scaffold/runtime oficial.

## Restrições herdadas

Continuam obrigatórias:

- Client Tauri 2 + HTML/CSS/JavaScript modular;
- Launcher Rust x64 pequeno/self-contained;
- Controller + Host Rust; Host com Tokio/Axum + `rusqlite` bundled;
- SQLite acessado somente pelo Host;
- contrato Pocket sem instalador/manualidade/admin/Internet/toolchain em produção;
- sem Windows Service, Task Scheduler, watchdog, tray ou daemon como baseline;
- nenhuma decisão de UX/produto já aprovada é reaberta sem bloqueador concreto;
- nenhuma migration oficial, scaffold executável ou código de negócio é criado antes do gate final deste bloco.

---

# Análise 1 — árvore fonte e fronteiras de responsabilidade

**Status:** APROVADA PELO PO EM 2026-09-01 — D12.1–D12.18.

## Princípio

A árvore de **código-fonte** não deve imitar a pasta publicada em produção. A publicação Pocket é artefato gerado; o repositório deve privilegiar manutenção, testes e ownership claro.

Direção aprovada:

```text
StepFlow/
├── Cargo.toml
├── apps/
│   ├── client/
│   │   ├── web/
│   │   │   ├── index.html
│   │   │   ├── css/
│   │   │   └── js/
│   │   │       ├── app/
│   │   │       ├── components/
│   │   │       ├── features/
│   │   │       ├── services/
│   │   │       └── shared/
│   │   └── src-tauri/
│   ├── launcher/
│   ├── controller/
│   └── host/
├── crates/
│   ├── protocol/
│   ├── domain/
│   ├── documents/
│   └── platform-windows/
├── scripts/
├── tests/
│   └── e2e/
├── docs/
├── .gitignore
└── README.md
```

A árvore é uma baseline de ownership, não autorização para criar todos os diretórios imediatamente. Diretório/crate só entra no scaffold quando houver responsabilidade concreta na etapa correspondente da Fase 2.

## Fronteiras aprovadas

- `apps/client`: Tauri local, frontend HTML/CSS/JavaScript em ES modules, `src-tauri` fino e sem regra de negócio/autorização duplicada;
- `apps/launcher`: manifest/deployment, preparação local, validação/ativação do Client e encerramento;
- `apps/controller`: lifecycle central, exclusividade, readiness/shutdown, relaunch bounded e Recovery local;
- `apps/host`: autoridade funcional, HTTP/JSON/WebSocket, auth, SQLite/migrations, domínio, documentos, Backup/Restore e auditoria;
- `crates/protocol`: contratos/tipos realmente compartilhados;
- `crates/domain`: invariantes puras com uso concreto;
- `crates/documents`: pipeline documental Host-side isolável;
- `crates/platform-windows`: adapters Win32 reutilizáveis/testáveis quando houver necessidade real.

## Publicação Pocket aprovada

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\
    │   ├── manifest.json
    │   ├── deployment.json
    │   └── releases\<versao>\...
    └── server\
        ├── app\
        │   ├── StepFlowController.exe
        │   └── StepFlowHost.exe
        ├── config\
        ├── data\
        ├── logs\
        └── backups\
```

Experiência:

```text
abrir pasta StepFlow
→ duplo clique em StepFlow.exe
→ Launcher prepara/valida Client local
→ Client abre
→ Launcher encerra
```

Regras:

- `StepFlow.exe` é o Launcher com nome/ícone amigáveis e único ponto de entrada normal;
- usuário comum não precisa navegar na árvore técnica;
- `.lnk` não é requisito baseline;
- Hidden/System em `_internal/` é somente acabamento opcional;
- source tree e publicação Pocket são estruturas diferentes.

## Decisões D12.1–D12.18

- **D12.1:** workspace Rust na raiz, `apps/` para binários e `crates/` para bibliotecas reutilizáveis;
- **D12.2:** binários oficiais: `client`, `launcher`, `controller`, `host`;
- **D12.3:** frontend em `apps/client/web/` com ES modules por feature/componente/serviço/shared, sem monólito;
- **D12.4:** `src-tauri/` permanece shell fino;
- **D12.5:** Launcher transitório, Controller lifecycle central, Host autoridade funcional/SQLite;
- **D12.6:** `crates/protocol` somente para contratos/tipos compartilhados;
- **D12.7:** `crates/domain` somente para invariantes puras com uso concreto;
- **D12.8:** geração documental pode ficar em `crates/documents`, mantendo propriedade Host-side;
- **D12.9:** adapters Win32 reutilizáveis podem ficar em `crates/platform-windows`;
- **D12.10:** migrations pertencem ao Host;
- **D12.11:** scripts root são tooling de dev/build/test/package, nunca runtime de produção;
- **D12.12:** unit/integration junto ao owner; `tests/e2e/` para fluxos entre componentes;
- **D12.13:** source tree e publicação são distintas; `dist/`, `target/`, dados reais e pacotes gerados não são versionados;
- **D12.14:** estrutura aprovada não autoriza scaffold antes do gate final + sincronização segura do checkout;
- **D12.15:** raiz publicada possui único ponto de entrada normal `StepFlow.exe`;
- **D12.16:** artefatos técnicos ficam encapsulados sob `_internal/`;
- **D12.17:** `.lnk` não é requisito baseline;
- **D12.18:** Hidden/System em `_internal/` é acabamento opcional, nunca requisito funcional.

---

# Análise 2 — workspace, build, dependências e configuração

**Status:** APROVADA PELO PO EM 2026-09-01 — D12.19–D12.34.

Fonte detalhada: `bloco-12-analise-2-workspace-build-dependencias.md`.

Contrato aprovado:

- Rust `1.98.0`, target `x86_64-pc-windows-msvc`, rustup `minimal`, `rustfmt` e `clippy`;
- Edition 2024 + Cargo resolver 3;
- virtual workspace na raiz;
- `rust-toolchain.toml` e `Cargo.lock` versionados;
- builds/test/package lockfile-aware e sem `cargo update` incidental;
- dependências só entram quando houver uso real;
- baseline inicial: Tauri 2.11.x, tauri-build 2.6.x, Tauri CLI 2.11.x, Tokio ~1.51, Axum 0.8.9, rusqlite 0.40.2 bundled, Serde 1.0.229, serde_json 1.0.151, tracing 0.1.44 e tower-http 0.6.8, somente onde aplicável;
- frontend vanilla modular sem Node/npm/pnpm/yarn/Vite/bundler/framework no baseline;
- Tauri CLI apenas em desenvolvimento;
- configuração separada entre build/dev, deployment e runtime central;
- `target/`/`dist/` descartáveis e produção montada por packaging;
- LTO/strip/panic/codegen tuning somente por benchmark;
- scripts PowerShell finos, sem requisito para o usuário final.

Decisões vigentes: **D12.19–D12.34** conforme fonte detalhada e registro de decisões.

---

## Estado das análises

1. Análise 1 — árvore fonte + publicação Pocket: **APROVADA** — D12.1–D12.18;
2. Análise 2 — workspace/build/dependências/configuração: **APROVADA** — D12.19–D12.34;
3. Análise 3 — migrations/scripts/testes/fixtures: **EM REVISÃO** — P12.35–P12.55 em `bloco-12-analise-3-migrations-testes-fixtures.md`;
4. parâmetros finais de autenticação + Backup/Restore + configuração da empresa/categoria arquivada: pendente;
5. plano detalhado da Fase 2 e sequência de tarefas Codex: pendente;
6. gate de encerramento da Fase 1 e autorização explícita do primeiro scaffold: pendente.

## Fora do escopo deste checkpoint

- criar `Cargo.toml`, lockfile ou toolchain file oficiais;
- criar app/crate vazio;
- criar migration SQL oficial;
- criar scripts/fixtures executáveis;
- implementar UI, Host, Launcher ou Controller;
- sincronizar/alterar o checkout local do PO;
- criar task Codex executável antes do gate correspondente.
