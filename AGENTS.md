# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para agentes que atuem neste repositório. Este arquivo governa **como trabalhar**; detalhes de produto e arquitetura pertencem às fontes específicas.

## Fonte de verdade e estado

- GitHub é a fonte operacional principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- **Fase 1 — Fechamento arquitetural e especificação: documental e tecnicamente concluída.**
- Blocos 0–12 estão encerrados em seus contratos documentais/técnicos.
- Decisões finais da Fase 1: D12.1–D12.108.
- Nenhuma implementação funcional oficial foi iniciada.
- A Fase 2 só começa após gate Git limpo, sincronização local segura e autorização explícita do PO para F2-T01.

Estado executivo corrente pertence ao `README.md` e ao plano/roadmap, não a documentos técnicos estáveis.

## Precedência

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. tarefa, dentro das decisões vigentes;
6. histórico Git.

O enunciado não revoga silenciosamente decisão consolidada. Conflito ou ambiguidade material volta ao PO/Assistente.

## Leitura por camadas

Sempre: `AGENTS.md` → enunciado → `docs/README.md` → documento específico. Conforme impacto, consultar registro de decisões, plano/roadmap, arquitetura, modelo de dados, contexto de ambientes e documento de Produto/Tela afetado.

## Papéis

- **PO:** autoridade de produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** análise, arquitetura, coerência documental, gates e preparação de tarefas.
- **Codex:** execução técnica do escopo aprovado, sem inventar produto ou ampliar tarefa.

## Pré-flight para Codex

Antes de cada nova tarefa Codex, inclusive PoC:

1. entregar `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. entregar `PROMPT / ENUNCIADO PARA O CODEX` separado;
3. usar o menor perfil de capacidade suficiente conforme a política Codex.

## Base Git obrigatória

Toda tarefa que altere arquivos informa branch/base e commit SHA esperado. Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` divergir, não fazer pull/merge/rebase/reset/checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Alteração preexistente pertence ao PO/outro fluxo. Sem autorização explícita, é proibido usar `git reset --hard`, `git clean`, `git stash`, descartar/restaurar alteração local, sobrescrever arquivo modificado, trocar branch descartando trabalho ou incluir alteração preexistente no commit da tarefa.

Se arquivo necessário já estiver modificado, parar e reportar.

## Disciplina Git

```text
1 trabalho lógico
→ 1 branch ativa
→ 1 PR
→ revisão/aprovação
→ squash merge em main
→ apagar branch encerrada
→ verificar remoto limpo
→ próximo trabalho
```

Branch mergeada não está encerrada enquanto permanecer no remoto. Cada tarefa da Fase 2 usa branch/PR próprios conforme D12.81.

## Regras operacionais

- uma tarefa lógica por vez;
- não declarar parcial como concluído;
- não criar funcionalidade/dependência/estrutura fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta/exemplo/valor provisório em decisão;
- manter documentação e implementação sincronizadas;
- preservar modularidade e evitar monólito HTML/JS;
- não versionar credenciais, segredos, banco real ou dados pessoais;
- exemplos de IP/hostname/share/path nunca viram configuração oficial;
- teste corporativo impossível fora do ambiente correspondente = `NÃO APLICÁVEL NESTE AMBIENTE`, nunca PASS presumido;
- PoC descartável não vira produção silenciosamente;
- documento técnico estável não carrega próximo bloco nem gate Git consumido.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não altera ACL, Schannel, registro, PATH global, políticas de segurança ou reinstala ferramenta válida para “consertar” o sandbox. Operações que exijam credenciais, Internet confiável, elevação ou configuração global vão para a sessão Windows normal do PO.

## Contrato Pocket obrigatório

```text
pasta pronta publicada no servidor Windows
→ estação acessa compartilhamento
→ usuário executa StepFlow.exe na raiz
→ Launcher prepara/valida Client local automaticamente
→ Client abre de %LOCALAPPDATA%
→ Launcher encerra
```

`StepFlow.exe` é o Launcher amigável e único ponto de entrada normal. Artefatos publicados ficam sob `_internal/`; `.lnk` não é requisito baseline.

Obrigatório no uso normal: zero instalador tradicional por estação; zero configuração manual de dependência; zero privilégio administrativo; zero toolchain em produção; nenhuma Internet obrigatória; Client operacional local; Controller/Host sob demanda; fechar Client individual não encerra Host; nenhum processo residual após encerramento central; sem Service/auto-start/Task Scheduler/watchdog/tray/daemon baseline.

Se dependência exigir instalação/elevação/preparação manual por computador, a solução não atende ao Pocket e deve ser redesenhada ou tratada como blocker.

### WebView2

Evergreen compatível já presente é preferível. Fixed Version não pode executar de UNC/SMB; fallback autocontido deve ser local e só entra após PoC sem instalação, elevação ou ação manual. Falha em estação suportada é blocker.

## Invariantes técnicas consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular, sem Node/bundler baseline;
- Host: Rust + Tokio/Axum + `rusqlite` bundled;
- HTTP/JSON + WebSocket autenticado quando sessão existir;
- SQLite somente pelo Host;
- WAL + writer lógico + fila bounded + revisão otimista;
- Procedimentos usam revisões imutáveis;
- Atendimento preserva revisão usada; checklist/observação operacional persistem somente em Atendimento;
- geração documental Host-side por snapshot + `DocumentModel`;
- PDF Typst, DOCX OOXML Rust, impressão pelo mesmo PDF via WebView2;
- Ficha válida = exatamente uma A4; `2+` = `SHEET_OVERFLOW`;
- Backup/Restore segue D11.1–D11.116;
- Backup = ADM/Gerência; Restore = ADM-only;
- disaster recovery local/transitório;
- Restore destrutivo invalida sessões antigas;
- `uncertain` bloqueia readiness/mutações/cleanup;
- safety backup mantém barrier até primeiro rename;
- paths Backup/Restore usam semântica Windows estrita e provenance;
- D12.1–D12.108 definem estrutura, toolchain/build, migrations/testes, parâmetros, plano da Fase 2 e gates finais.

## Gate de transição para implementação

O primeiro scaffold não nasce automaticamente após o merge documental.

```text
fechar PR/branch do Bloco 12
→ verificar main + zero PRs
→ inspecionar C:\dev\StepFlow
→ se main estiver limpa: git fetch --prune origin + git merge --ff-only origin/main
→ confirmar HEAD/status
→ PO autoriza F2-T01
→ pré-flight F2-T01
→ prompt Codex F2-T01
→ branch feat/f2-01-workspace-host
```

Se houver alteração local, branch inesperada ou divergência deliberada, parar e reportar. Não usar reset/clean/stash/rebase para forçar alinhamento.

## Pendências ainda abertas

- gate Git do Bloco 12 e remoto limpo;
- sincronização segura do checkout local;
- autorização explícita da F2-T01;
- gates corporativos de Windows/WebView2/Launcher/SMB/Word/impressoras/filesystem/ACL/EDR/adapters nas fases aplicáveis.

## Regra final

Executar somente o escopo autorizado, preservar o consolidado e preferir referência à fonte específica em vez de duplicar contratos em governança.