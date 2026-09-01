# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para agentes que atuem neste repositório. Este arquivo governa **como trabalhar**; detalhes de produto e arquitetura pertencem às fontes específicas.

## Fonte de verdade e estado

- GitHub é a fonte operacional principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–11 estão encerrados.
- Bloco 12 está em validação final; D12.1–D12.98 estão aprovadas e P12.99–P12.108 permanecem em revisão.
- Nenhuma implementação funcional oficial foi iniciada.

Estado executivo corrente pertence ao `README.md` e ao plano da fase, não a documentos técnicos estáveis.

## Precedência

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. tarefa, dentro das decisões vigentes;
6. histórico Git.

O enunciado não revoga silenciosamente decisão consolidada. Se houver conflito ou ambiguidade material, parar e retornar ao PO/Assistente.

## Leitura por camadas

### Sempre

1. `AGENTS.md`;
2. enunciado da tarefa;
3. `docs/README.md`;
4. documento específico indicado.

### Conforme impacto

- `docs/05-progresso/registro-de-decisoes.md`;
- plano/roadmap vigentes;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- modelo de dados;
- contexto de ambientes;
- documento de Produto/Tela/Arquitetura afetado.

## Papéis

- **PO:** autoridade de produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** análise, arquitetura, coerência documental, gates e preparação de tarefas.
- **Codex:** execução técnica do escopo aprovado, sem inventar produto ou ampliar tarefa.

## Pré-flight para Codex

Antes de **cada nova tarefa Codex**, inclusive PoC:

1. entregar `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. entregar `PROMPT / ENUNCIADO PARA O CODEX` separado;
3. usar o menor perfil de capacidade suficiente conforme a política Codex.

## Base Git obrigatória

Toda tarefa que permita alteração informa branch/base e commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` divergir, não fazer pull/merge/rebase/reset/checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Alteração preexistente pertence ao PO/outro fluxo. Sem autorização explícita, é proibido:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alteração local;
- sobrescrever arquivo modificado preexistente;
- trocar branch descartando trabalho;
- incluir alteração preexistente no commit da tarefa.

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

Branch mergeada não está encerrada enquanto permanecer no remoto. Cada tarefa da Fase 2 seguirá branch/PR próprios conforme D12.81.

## Regras operacionais

- uma tarefa lógica por vez;
- não declarar parcial como concluído;
- não criar funcionalidade/dependência/estrutura fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta/exemplo/valor provisório em decisão;
- manter documentação e implementação sincronizadas;
- atualizar o painel quando houver mudança real de fase/bloco;
- preservar modularidade e evitar monólito HTML/JS;
- não versionar credenciais, segredos, banco real ou dados pessoais;
- exemplos de IP/hostname/share/path nunca viram configuração oficial;
- teste corporativo impossível fora do ambiente correspondente = `NÃO APLICÁVEL NESTE AMBIENTE`, nunca PASS presumido;
- PoC descartável não vira produção silenciosamente;
- documento técnico estável não carrega próximo bloco nem gate Git consumido.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não altera ACL, Schannel, registro, PATH global, políticas de segurança ou reinstala ferramenta válida para “consertar” o sandbox.

Operações que exijam credenciais, Internet confiável, elevação ou configuração global devem ser encaminhadas à sessão Windows normal do PO.

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

Obrigatório no uso normal:

- zero instalador tradicional por estação;
- zero configuração manual de dependência;
- zero privilégio administrativo;
- zero toolchain em produção;
- nenhuma Internet obrigatória;
- Client operacional local, não permanente em SMB;
- atualização central + versões locais validadas;
- Host/Controller sob demanda;
- fechar Client individual não encerra Host;
- nenhum processo residual após encerramento central;
- sem Service/auto-start/Task Scheduler/watchdog/tray/daemon baseline.

Se dependência exigir instalação/elevação/preparação manual por computador, a solução não atende ao Pocket e deve ser redesenhada ou tratada como blocker.

### WebView2

Evergreen compatível já presente é preferível. Fixed Version não pode executar de UNC/SMB; fallback autocontido deve ser local e só entra após PoC sem instalação, elevação ou ação manual. Falha em estação suportada é blocker, não autorização para enfraquecer Pocket.

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
- D12.1–D12.18: estrutura/publicação `StepFlow.exe + _internal/`;
- D12.19–D12.34: toolchain/workspace/build/dependências;
- D12.35–D12.55: migrations/testes/fixtures;
- D12.56–D12.79: parâmetros iniciais;
- D12.80–D12.98: sequência F2-T01…F2-T08 e gates da Fase 2.

Aprovação D12 não autoriza scaffold antes do gate final da Fase 1.

## Pendências ainda abertas

- P12.99–P12.108 — validação final da Fase 1;
- gate Git do Bloco 12;
- sincronização segura do checkout local;
- autorização explícita de F2-T01;
- gates corporativos de Windows/WebView2/Launcher/SMB/Word/impressoras/filesystem/ACL/EDR/adapters.

## Gate de implementação

Não criar scaffold oficial, módulos runtime definitivos, migration oficial ou código de negócio antes do Bloco 12/Fase 1 ser encerrado.

Antes do primeiro trabalho de implementação, o checkout local deve ser sincronizado explicitamente sem apagar, sobrescrever, descartar ou incorporar indevidamente alterações preexistentes do PO. A semântica positiva de sincronização permanece em revisão em P12.103–P12.104.

## Regra final

Executar somente o escopo autorizado, preservar o consolidado e preferir referência à fonte específica em vez de duplicar contratos em governança.
