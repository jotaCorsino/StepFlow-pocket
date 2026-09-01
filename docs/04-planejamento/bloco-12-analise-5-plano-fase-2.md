# Bloco 12 — Análise 5 — Plano detalhado da Fase 2 e sequência de tarefas Codex

**Status:** PROPOSTA PARA REVISÃO DO PO  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Transformar D12.1–D12.79 em uma sequência pequena, verificável e executável para a **Fase 2 — Fundação técnica executável**, sem antecipar funcionalidades das Fases 3+.

Esta análise não autoriza implementação enquanto o Bloco 12/Fase 1 não passar pelo gate final, Git remoto não estiver limpo e o checkout local não tiver sido sincronizado de forma segura.

## Princípio de execução

A Fase 2 não será um “scaffold de tudo”. Cada tarefa cria somente o componente/estrutura que já possui responsabilidade concreta naquele passo.

Fluxo por tarefa:

```text
main limpo
→ pré-flight do PO separado do prompt Codex
→ branch própria
→ Codex confirma HEAD + working tree
→ implementação pequena
→ validações mecânicas
→ draft PR
→ revisão/aprovação
→ squash merge
→ delete branch
→ verificar remoto limpo
→ próxima tarefa
```

Nenhuma tarefa posterior está automaticamente autorizada pela conclusão da anterior.

---

# Gate de transição Fase 1 → Fase 2

Antes da primeira tarefa executável:

1. PR do Bloco 12 aprovado e squash-merged em `main`;
2. branch remota do Bloco 12 removida;
3. zero PRs abertos e remoto limpo;
4. `main` remoto identificado por SHA exato;
5. checkout local `C:\dev\StepFlow` inspecionado com:
   - `git rev-parse HEAD`;
   - `git status --short --branch`;
6. qualquer alteração local preexistente do PO é preservada e tratada antes de sincronização;
7. não usar `reset --hard`, `clean`, `stash` ou descarte para “alinhar” o checkout sem autorização explícita;
8. somente depois da sincronização segura o primeiro branch de implementação pode nascer.

A sincronização local é um **gate operacional**, não uma tarefa Codex que possa apagar trabalho local.

---

# Sequência proposta da Fase 2

## F2-T01 — Workspace oficial + tooling mínimo + Host mínimo compilável

**Objetivo:** criar a primeira árvore executável real e um único package concreto (`apps/host`) suficiente para provar toolchain/workspace/build, sem domínio de negócio.

Inclui:

- `rust-toolchain.toml` conforme D12.19–D12.21;
- `Cargo.toml` virtual workspace Edition 2024/resolver 3;
- `Cargo.lock` versionado;
- `.gitignore` para `target/`, `dist/`, dados/artefatos locais;
- `apps/host` como primeiro package real;
- `scripts/check.ps1`, `test.ps1`, `build.ps1` como wrappers finos;
- logging mínimo de processo somente se necessário para o binário iniciar de forma diagnosticável.

Não inclui:

- Controller, Client, Launcher;
- SQLite/migrations;
- HTTP/API;
- autenticação;
- domínio;
- packaging Pocket.

Aceite mínimo:

- `cargo fmt --check`;
- `cargo check --workspace --locked`;
- `cargo clippy --workspace --all-targets --locked -- -D warnings`;
- `cargo test --workspace --locked`;
- build x64 MSVC no ambiente aplicável;
- nenhum Node/package manager introduzido.

## F2-T02 — Host runtime: configuração, logging e health/readiness

**Objetivo:** transformar o Host mínimo em processo central observável, ainda sem banco/domínio.

Inclui:

- configuração runtime central com defaults seguros somente para desenvolvimento/teste;
- resolução explícita da raiz de deployment/teste;
- logging estruturado e rotação técnica conforme D12.74;
- Axum/Tokio;
- endpoints mínimos não autenticados de infraestrutura, versionados, para:
  - health de processo;
  - readiness;
  - metadata/compatibilidade de contrato necessária ao bootstrap futuro;
- shutdown gracioso interno/testável;
- portas efêmeras em teste;
- erros sem path/stack/segredo na resposta externa.

Readiness neste passo representa somente a fundação já disponível; passos posteriores acrescentam SQLite/recovery como pré-condições antes de uso real.

## F2-T03 — SQLite + runner de migrations determinístico

**Objetivo:** implementar a infraestrutura D12.35–D12.55 sem criar schema de negócio artificial.

Inclui:

- `rusqlite` bundled;
- criação controlada de `data/stepflow.sqlite` em raiz temporária/configurada;
- pragmas obrigatórios e foreign keys;
- `schema_migrations` como metadata técnica do runner;
- registry compilado/embedding;
- validação de IDs/checksum/cadeia;
- runner transacional;
- `quick_check` + `foreign_key_check`;
- testes com SQLite temporário real e migrations sintéticas do harness;
- readiness passa a depender de persistência/migrations válidas.

Não criar `000001` vazia. A primeira migration oficial numerada só nasce quando houver mudança real de schema persistente em fase posterior.

A pipeline `pre_migration` depende da implementação de Backup, que pertence à Fase 8. Até lá, qualquer mudança real de schema em banco existente que exija essa proteção não pode ser promovida a produção sem o mecanismo correspondente; durante a fundação, os testes usam bancos temporários controlados.

## F2-T04 — Controller: ciclo central, exclusividade e shutdown

**Objetivo:** implementar o lifecycle mínimo Controller → Host → readiness → shutdown, sem serviço residente.

Inclui:

- `apps/controller`;
- resolução da raiz de implantação;
- instância única por deployment/data root;
- start do Host como processo-filho;
- espera bounded de readiness conforme D12.71;
- diagnóstico de falha de startup;
- shutdown gracioso;
- confirmar saída do processo-filho;
- smoke test de segunda instância recusada quando tecnicamente aplicável.

Não inclui ainda Restore relaunch/recovery completo; só a fundação necessária para evoluir depois sem mudar ownership.

## F2-T05 — Client Tauri mínimo + compatibilidade Client↔Host

**Objetivo:** criar o primeiro Client local real, com frontend vanilla modular e integração mínima com o Host, sem UI de produto.

Inclui:

- `apps/client/web/` com HTML/CSS/JS ES modules;
- `apps/client/src-tauri/` fino;
- Tauri 2 conforme D12.26–D12.30;
- sem Node/npm/Vite/bundler/framework;
- leitura de `deployment.json` local não sensível;
- consulta à metadata/compatibilidade do Host;
- estados mínimos de bootstrap: carregando, Host indisponível, incompatível, conexão válida;
- timeout de conexão/request conforme D12.72;
- nenhuma credencial/login ainda;
- nenhum design final das telas de produto além de uma superfície técnica mínima coerente.

WebSocket autenticado permanece para a fase que implementar sessão/autorização; não criar canal operacional anônimo para antecipá-lo.

## F2-T06 — Launcher e preparação local do Client

**Objetivo:** provar o núcleo do contrato Pocket: executar um único Launcher pelo pacote publicado e iniciar o Client a partir de `%LOCALAPPDATA%`.

Inclui:

- `apps/launcher` Rust x64 pequeno/transitório;
- leitura/validação de `manifest.json` + `deployment.json`;
- release Client versionada no pacote de teste;
- hash/verificação dos artefatos;
- staging local e ativação determinística em `%LOCALAPPDATA%` ou root temporário equivalente nos testes;
- execução do Client local;
- encerramento do Launcher;
- sem admin, instalador, PATH global, Internet ou updater residente.

O nome técnico do binário no source pode continuar `launcher`; `StepFlow.exe` é nome de packaging.

## F2-T07 — Packaging Pocket reproduzível

**Objetivo:** montar automaticamente a árvore publicada aprovada, sem usar `target/release` como procedimento de produção.

Saída mínima:

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\
    │   ├── manifest.json
    │   ├── deployment.json.example/test fixture conforme contexto
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

Inclui:

- `scripts/package.ps1`;
- `dist/` descartável;
- cópia somente de artefatos validados;
- `StepFlow.exe` como Launcher na raiz;
- `_internal/` técnico;
- manifests/hashes necessários ao Launcher;
- nenhuma credencial, endpoint corporativo real ou banco real no pacote versionado.

## F2-T08 — Smoke integrado e gates Pocket/Windows da fundação

**Objetivo:** provar a fundação completa da Fase 2 antes de iniciar autenticação/Fase 3.

Automatizável em ambiente de desenvolvimento:

1. build limpo `--locked`;
2. package limpo;
3. Controller inicia Host e obtém readiness;
4. Host cria/reabre SQLite determinístico;
5. runner de migrations é idempotente no schema técnico atual;
6. Client consulta compatibilidade do Host;
7. Launcher parte da pasta publicada, prepara Client local e o inicia;
8. fechar Client não encerra Host;
9. shutdown central não deixa processo StepFlow iniciado pelo teste;
10. pacote não contém Node/runtime dev/toolchain/configuração corporativa/segredo;
11. `git diff --check` e estado Git limpo após testes gerados/temporários.

Validações que dependem de ambiente corporativo real são executadas pelo PO/sessão Windows apropriada e registradas sem falsificação:

- execução do Launcher a partir do share SMB real;
- WebView2 Evergreen nas estações-alvo;
- PoC de Fixed Version local em `%LOCALAPPDATA%` somente se o fallback for realmente necessário;
- Windows 10/11 alvo, ACL/EDR/antivírus pertinentes;
- ausência de elevação/manualidade;
- nenhum runtime/toolchain exigido na estação/servidor de produção além dos componentes Windows previstos.

Falha em requisito Pocket obrigatório é blocker; não é autorização para instalar dependência manualmente na estação.

---

# Ordem e dependências

```text
Gate Fase 1 + sync local seguro
        ↓
F2-T01 workspace + Host mínimo
        ↓
F2-T02 Host runtime/readiness
        ↓
F2-T03 SQLite + migrations runner
        ↓
F2-T04 Controller lifecycle
        ↓
F2-T05 Client Tauri + compatibilidade
        ↓
F2-T06 Launcher/localização Client
        ↓
F2-T07 packaging Pocket
        ↓
F2-T08 smoke integrado + gates Windows/Pocket
        ↓
Gate Fase 2
        ↓
Fase 3 — autenticação, usuários e shell
```

A ordem pode receber microajuste por bloqueador técnico comprovado, mas uma tarefa não pode puxar implementação funcional de fase futura para “facilitar” a fundação.

---

# Branches e PRs da Fase 2

Convenção proposta, ajustável apenas no sufixo descritivo:

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

Cada branch nasce de `main` após o gate da anterior. Não acumular duas tarefas Codex independentes no mesmo PR.

---

# Critério de saída da Fase 2

A Fase 2 pode ser considerada concluída somente quando:

- workspace/toolchain/build são reproduzíveis e `--locked`;
- Host inicia com configuração/logging/readiness e storage temporário/controlado;
- SQLite abre/cria deterministicamente e runner de migrations é testado;
- Controller possui lifecycle bounded sem serviço/watchdog;
- Client Tauri vanilla abre sem Node/bundler e valida compatibilidade do Host;
- Launcher prepara e executa Client local automaticamente;
- packaging gera `StepFlow.exe + _internal/` sem segredo/dado real;
- smoke integrado passa no ambiente aplicável;
- validações Pocket corporativas exigidas para a fundação estão registradas como PASS ou blocker real, nunca presumidas;
- documentação viva reflete o executado;
- não houve antecipação de autenticação, Procedimentos, Atendimentos, documentos ou Backup/Restore funcional das fases posteriores.

---

# Propostas P12.80–P12.98

- **P12.80:** Fase 2 só começa após fechamento Git do Bloco 12 e sincronização local segura, preservando alterações preexistentes do PO;
- **P12.81:** cada tarefa técnica da Fase 2 usa branch/PR próprios, nasce de `main` já consolidada e passa por pré-flight Codex separado;
- **P12.82:** F2-T01 cria workspace/toolchain/lockfile/tooling e somente o primeiro package concreto `apps/host`, sem scaffold vazio dos demais componentes;
- **P12.83:** F2-T02 implementa configuração, logging, Axum/Tokio, health/readiness/compatibilidade mínima e shutdown do Host, ainda sem domínio;
- **P12.84:** F2-T03 implementa SQLite bundled + `schema_migrations` + runner/checksums/testes sem criar migration numerada vazia ou schema de negócio artificial;
- **P12.85:** até Backup ser implementado na Fase 8, mudanças reais de schema em banco existente que dependam de `pre_migration` não podem ser promovidas a produção sem a proteção D11/D12 correspondente;
- **P12.86:** F2-T04 implementa Controller lifecycle, exclusividade, start/readiness/shutdown bounded sem serviço/watchdog;
- **P12.87:** F2-T05 cria Client Tauri vanilla modular, lê deployment e valida compatibilidade HTTP do Host sem implementar login/UI de produto;
- **P12.88:** WebSocket operacional autenticado não é antecipado na Fase 2 por canal anônimo; entra junto da fundação de sessão/autorização apropriada;
- **P12.89:** F2-T06 implementa Launcher transitório, staging/ativação Client local, verificação de artefatos e execução local sem admin/instalador/Internet;
- **P12.90:** F2-T07 implementa packaging reproduzível em `dist/` com `StepFlow.exe` na raiz e `_internal/client|server`, sem executar produção diretamente de `target/release`;
- **P12.91:** F2-T08 executa smoke integrado automatizável e separa explicitamente os gates que dependem do ambiente corporativo real;
- **P12.92:** teste fora da LAN/Windows corporativo não pode declarar PASS para SMB/EDR/WebView2 real; deve registrar `NÃO APLICÁVEL NESTE AMBIENTE` e encaminhar validação controlada;
- **P12.93:** falha Pocket real em estação suportada é blocker e não autoriza instalação/manualidade/admin para contornar o requisito;
- **P12.94:** nenhuma tarefa da Fase 2 implementa autenticação/usuários, Procedimentos, Atendimentos, geração documental ou Backup/Restore funcional antes das fases proprietárias;
- **P12.95:** crates compartilhados só são criados quando duas responsabilidades reais justificarem reutilização; não criar `protocol/domain/documents/platform-windows` vazios na fundação;
- **P12.96:** cada tarefa deve terminar com validação mecânica proporcional, diff/status Git conferidos e documentação viva atualizada quando afetada;
- **P12.97:** critério de saída da Fase 2 exige Host/SQLite/Controller/Client/Launcher/packaging integrados e reproduzíveis, sem processo residual ou dependência dev em produção;
- **P12.98:** concluir a Fase 2 apenas autoriza entrada na Fase 3; não autoriza executar automaticamente a próxima tarefa/fase sem novo gate.

## Fora do escopo desta análise

- criar os branches/tasks agora;
- criar prompt Codex executável antes do gate final da Fase 1;
- criar scaffold/código;
- definir detalhes internos de cada módulo além do necessário para sequenciamento;
- reabrir contratos funcionais das Fases 3–9;
- declarar validação corporativa sem executá-la no ambiente real.
