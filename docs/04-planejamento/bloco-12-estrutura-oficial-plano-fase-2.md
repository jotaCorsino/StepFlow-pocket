# Bloco 12 — Estrutura oficial + plano da Fase 2

**Status:** EM ANÁLISE — ANÁLISE 1 APROVADA  
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

**Status:** APROVADA PELO PO EM 2026-09-01.

## Princípio

A árvore de **código-fonte** não deve imitar a pasta publicada em produção. A publicação Pocket é artefato gerado; o repositório deve privilegiar manutenção, testes e ownership claro.

Direção aprovada:

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

## Fronteiras aprovadas

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

Na publicação Pocket, esse binário é o **único ponto de entrada destinado ao usuário comum** e pode ser exposto na raiz com o nome amigável `StepFlow.exe`, independentemente do nome técnico do crate/binário no source tree.

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

Direção aprovada como ownership; detalhes permanecem para análises seguintes do Bloco 12:

- migrations pertencem ao Host e devem ficar próximas da persistência que as executa;
- scripts do root servem build/test/package/dev e **não** viram dependência operacional de produção;
- unit tests ficam próximos ao módulo/crate;
- integration tests ficam no componente proprietário;
- `tests/e2e/` fica reservado a smoke/end-to-end entre binários quando a Fase 2 tiver fundação suficiente;
- artefatos `target/`, `dist/`, pacote Pocket e dados reais não são versionados.

## Código-fonte × publicação Pocket

A saída de build/package deve separar claramente **superfície do usuário** e **estrutura interna**.

Direção aprovada para a pasta publicada:

```text
StepFlow\
├── StepFlow.exe                 # único ponto de entrada normal; Launcher com ícone do produto
└── _internal\
    ├── client\
    │   ├── manifest.json
    │   ├── deployment.json
    │   └── releases\
    │       └── <versao>\
    │           └── artefatos do Client
    └── server\
        ├── app\
        │   ├── StepFlowController.exe
        │   └── StepFlowHost.exe
        ├── config\
        ├── data\
        ├── logs\
        └── backups\
```

Experiência aprovada para o usuário:

```text
abrir a pasta StepFlow no compartilhamento
→ ver StepFlow.exe como entrada evidente
→ duplo clique
→ Launcher prepara/valida o Client local
→ Client abre
→ Launcher encerra
```

Regras dessa superfície:

- não exigir que o usuário navegue por `client/`, `releases/`, `app/` ou outras pastas técnicas para iniciar o produto;
- executáveis internos de Controller/Host não ficam apresentados como alternativas de início para o usuário comum;
- a subpasta `_internal/` é encapsulamento organizacional; o funcionamento não depende de atributo Hidden/System do Windows;
- packaging pode marcar `_internal/` como oculto apenas como acabamento visual, desde que isso não seja requisito funcional e não complique update/cópia/ACL;
- não usar `.lnk` como baseline obrigatório: shortcut do Windows pode depender de target/path e ficar frágil quando a pasta/share for movido;
- o próprio Launcher exposto como `StepFlow.exe` cumpre a função de “atalho” portátil e verificável;
- o nome técnico do crate/binário de desenvolvimento pode continuar distinto (`launcher`/`stepflow-launcher`); o nome amigável é decisão de packaging;
- source tree e publicação continuam estruturas diferentes.

Essa pasta é **produto de packaging/deploy**, não a organização do source tree.

## Decisões D12.1–D12.18

- **D12.1:** repositório executável usa workspace Rust na raiz, com `apps/` para binários e `crates/` para bibliotecas reutilizáveis;
- **D12.2:** binários oficiais são `client`, `launcher`, `controller` e `host`, preservando as responsabilidades já consolidadas;
- **D12.3:** frontend do Client fica em `apps/client/web/` com ES modules organizados por feature/componente/serviço/shared, sem monólito HTML/JS;
- **D12.4:** `apps/client/src-tauri/` permanece shell fino e não vira segunda camada de negócio/autorização;
- **D12.5:** `launcher` permanece Rust self-contained e transitório; `controller` permanece lifecycle central; `host` permanece autoridade funcional/SQLite;
- **D12.6:** `crates/protocol` só carrega contratos/tipos realmente compartilhados e nunca acesso ao banco;
- **D12.7:** `crates/domain` contém somente invariantes puras com uso concreto; abstração antecipada e `utils` genérico não são baseline;
- **D12.8:** geração documental pode ser isolada em `crates/documents`, mantendo execução/propriedade Host-side;
- **D12.9:** integrações Win32 reutilizáveis/testáveis podem ser isoladas em `crates/platform-windows`;
- **D12.10:** migrations pertencem ao Host e ficam próximas à persistência executora; naming/conteúdo inicial serão fechados em análise específica antes de criação;
- **D12.11:** scripts do root são apenas tooling de desenvolvimento/build/test/package e não criam requisito de runtime em produção;
- **D12.12:** unit/integration tests ficam próximos ao owner; `tests/e2e/` é reservado a testes entre componentes;
- **D12.13:** source tree e pasta Pocket publicada são estruturas distintas; `dist/`, `target/`, dados reais e pacotes gerados não são versionados;
- **D12.14:** aprovação desta árvore não autoriza scaffold imediato; implementação só começa após o gate final do Bloco 12 + sincronização segura do checkout local;
- **D12.15:** a raiz da pasta Pocket publicada possui **um único ponto de entrada normal**, `StepFlow.exe`, que é o Launcher empacotado com nome/ícone amigáveis;
- **D12.16:** artefatos técnicos da publicação ficam encapsulados sob `_internal/`, separando `client/` e `server/`; usuário comum não precisa navegar nessa árvore para iniciar o StepFlow;
- **D12.17:** `.lnk` não é requisito baseline; o executável Launcher na raiz fornece a experiência de atalho portátil sem depender de target absoluto/UNC estável;
- **D12.18:** atributo Hidden/System para `_internal/` pode existir apenas como acabamento opcional de packaging; integridade, atualização e execução nunca dependem dele.

## Estado das análises

1. Análise 1 — árvore fonte + publicação Pocket: **APROVADA** — D12.1–D12.18;
2. Análise 2 — workspace/build/dependências/configuração: **EM REVISÃO** — P12.19–P12.34 em `bloco-12-analise-2-workspace-build-dependencias.md`;
3. migrations/scripts/testes iniciais e fixtures: pendente;
4. parâmetros finais de autenticação + Backup/Restore + configuração da empresa/categoria arquivada: pendente;
5. plano detalhado da Fase 2 e sequência de tarefas Codex: pendente;
6. gate de encerramento da Fase 1 e autorização explícita do primeiro scaffold: pendente.

## Fora do escopo deste checkpoint

- criar `Cargo.toml` oficial;
- criar app/crate vazio;
- criar migration SQL oficial;
- implementar UI, Host, Launcher ou Controller;
- sincronizar/alterar o checkout local do PO;
- criar task Codex executável antes do gate correspondente.
