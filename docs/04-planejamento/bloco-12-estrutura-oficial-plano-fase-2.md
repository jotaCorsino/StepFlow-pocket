# Bloco 12 — Estrutura oficial + plano da Fase 2

**Status:** EM ANÁLISE — PROPOSTA NÃO CONSOLIDADA  
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

**Status:** PROPOSTA PARA REVISÃO DO PO.

## Princípio

A árvore de **código-fonte** não deve imitar a pasta publicada em produção. A publicação Pocket é artefato gerado; o repositório deve privilegiar manutenção, testes e ownership claro.

Direção proposta:

```text
StepFlow/
├── Cargo.toml                  # workspace Rust
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

A árvore é uma **baseline de ownership**, não autorização para criar todos os diretórios imediatamente. Diretório/crate só entra no scaffold quando houver responsabilidade concreta na etapa correspondente da Fase 2.

## Fronteiras propostas

### `apps/client`

Aplicativo Tauri executado localmente na estação.

- `web/` usa HTML/CSS/JavaScript com **ES modules**;
- organização por features/componentes/serviços, evitando `index.html` ou `app.js` monolítico;
- classes podem ser usadas quando houver estado/lifecycle claro, mas não são exigidas artificialmente onde funções/módulos simples forem melhores;
- `src-tauri/` fica fino: bootstrap Tauri, comandos/adapters realmente locais e integração Windows necessária ao Client;
- regra de negócio/autorização não migra para o shell Tauri.

### `apps/launcher`

Binário Rust responsável apenas por manifest/deployment, preparação local, validação, ativação de versão Client e encerramento.

Não inicia Host remoto, não acessa SQLite e não vira updater residente.

### `apps/controller`

Binário Rust da máquina central responsável pelo ciclo de vida do Host, exclusividade, readiness, shutdown, relaunch bounded de Restore e modo Recovery local/transitório.

Não absorve regras de domínio que pertencem ao Host.

### `apps/host`

Servidor central e autoridade funcional.

- HTTP/JSON + WebSocket;
- autenticação/autorização;
- SQLite/migrations;
- writer/fila/conflitos/eventos;
- domínio de Procedimentos/Atendimentos/Equipamentos;
- documentos, Backup/Restore e auditoria conforme contratos vigentes.

### `crates/protocol`

Tipos/DTOs/envelopes/códigos compartilhados entre componentes Rust quando necessário. Não contém acesso ao banco nem regra de negócio que dependa do Host.

### `crates/domain`

Invariantes puras e tipos de domínio que realmente tenham reutilização/testabilidade independente de transporte/persistência.

Evitar transformar `domain` em depósito genérico ou antecipar abstrações sem uso concreto.

### `crates/documents`

Pipeline documental compartilhável dentro do Host: `DocumentModel`, Typst/PDF, OOXML/DOCX e contratos de rendering. A existência do crate não muda a propriedade Host-side da geração.

### `crates/platform-windows`

Adapters Windows que mereçam isolamento/testes próprios: impressão, operações de filesystem/rename quando aplicável, integração WebView2/Win32 estritamente necessária.

Não vira camada genérica para qualquer código específico do Windows sem necessidade real.

## Migrations, scripts e testes

Direção para análise posterior do próprio Bloco 12:

- migrations pertencem ao Host e devem ficar próximas da persistência que as executa;
- scripts do root servem build/test/package/dev e **não** viram dependência operacional de produção;
- unit tests ficam próximos ao módulo/crate;
- integration tests ficam no componente proprietário;
- `tests/e2e/` fica reservado a smoke/end-to-end entre binários quando a Fase 2 tiver fundação suficiente;
- artefatos `target/`, `dist/`, pacote Pocket e dados reais não são versionados.

## Código-fonte × publicação Pocket

A saída de build/package futura deve produzir algo conceitualmente compatível com:

```text
publicação/
├── client/
│   ├── StepFlowLauncher.exe
│   ├── manifest.json
│   ├── deployment.json
│   └── releases/<versao>/...
└── server/
    ├── app/
    │   ├── StepFlowController.exe
    │   └── StepFlowHost.exe
    ├── config/
    ├── data/
    ├── logs/
    └── backups/
```

Essa pasta é **produto de packaging/deploy**, não a organização do source tree.

## Propostas P12.1–P12.14

- **P12.1:** repositório executável usa workspace Rust na raiz, com `apps/` para binários e `crates/` para bibliotecas reutilizáveis;
- **P12.2:** binários oficiais são `client`, `launcher`, `controller` e `host`, preservando as responsabilidades já consolidadas;
- **P12.3:** frontend do Client fica em `apps/client/web/` com ES modules organizados por feature/componente/serviço/shared, sem monólito HTML/JS;
- **P12.4:** `apps/client/src-tauri/` permanece shell fino e não vira segunda camada de negócio/autorização;
- **P12.5:** `launcher` permanece Rust self-contained e transitório; `controller` permanece lifecycle central; `host` permanece autoridade funcional/SQLite;
- **P12.6:** `crates/protocol` só carrega contratos/tipos realmente compartilhados e nunca acesso ao banco;
- **P12.7:** `crates/domain` contém somente invariantes puras com uso concreto; abstração antecipada e `utils` genérico não são baseline;
- **P12.8:** geração documental pode ser isolada em `crates/documents`, mantendo execução/propriedade Host-side;
- **P12.9:** integrações Win32 reutilizáveis/testáveis podem ser isoladas em `crates/platform-windows`;
- **P12.10:** migrations pertencem ao Host e ficam próximas à persistência executora; naming/conteúdo inicial serão fechados em análise específica antes de criação;
- **P12.11:** scripts do root são apenas tooling de desenvolvimento/build/test/package e não criam requisito de runtime em produção;
- **P12.12:** unit/integration tests ficam próximos ao owner; `tests/e2e/` é reservado a testes entre componentes;
- **P12.13:** source tree e pasta Pocket publicada são estruturas distintas; `dist/`, `target/`, dados reais e pacotes gerados não são versionados;
- **P12.14:** aprovação desta árvore não autoriza scaffold imediato; implementação só começa após o gate final do Bloco 12 + sincronização segura do checkout local.

## Próximas análises do Bloco 12

Após decisão sobre P12.1–P12.14:

1. workspace/build/dependências/configuração e versões pinadas;
2. migrations/scripts/testes iniciais e fixtures;
3. parâmetros finais de autenticação + Backup/Restore + configuração da empresa/categoria arquivada;
4. plano detalhado da Fase 2 e sequência de tarefas Codex;
5. gate de encerramento da Fase 1 e autorização explícita do primeiro scaffold.

## Fora do escopo desta abertura

- criar `Cargo.toml` oficial;
- criar app/crate vazio;
- criar migration SQL oficial;
- escolher valores numéricos ainda não analisados;
- implementar UI, Host, Launcher ou Controller;
- sincronizar/alterar o checkout local do PO;
- criar task Codex executável antes do gate correspondente.
