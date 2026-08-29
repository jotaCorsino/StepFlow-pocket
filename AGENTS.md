# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para agentes que atuem neste repositório. Este arquivo governa **como trabalhar**; detalhes de produto e arquitetura pertencem às fontes específicas.

## Fonte de verdade e estado

- GitHub é a fonte operacional principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–10 estão encerrados em seus escopos documentais.
- Bloco 11 fecha Backup/Restore técnico.
- Bloco 12 fecha estrutura oficial, parâmetros finais e plano da Fase 2.
- Nenhuma implementação funcional oficial foi iniciada.

O estado executivo corrente deve ser lido no `README.md` e no plano da fase, não replicado em documentos técnicos estáveis.

## Precedência

Em caso de conflito:

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. tarefa, dentro das decisões vigentes;
6. histórico Git.

O enunciado autoriza trabalho, mas não revoga silenciosamente decisão consolidada. Se a tarefa contrariar uma decisão vigente, parar e retornar ao PO/Assistente até existir nova decisão explícita e sincronização documental. Ambiguidade nunca autoriza escolher a alternativa mais conveniente.

## Leitura por camadas

### Sempre

1. `AGENTS.md`;
2. enunciado da tarefa;
3. `docs/README.md`;
4. documento específico indicado.

### Conforme impacto

- `docs/05-progresso/registro-de-decisoes.md`;
- `docs/04-planejamento/plano-oficial-fase-1.md`;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- `docs/03-arquitetura/modelo-dados-schema-fase-1.md`;
- `docs/00-governanca/contexto-ambientes.md`;
- documento de Produto/Tela/Arquitetura diretamente afetado.

## Papéis

- **PO:** autoridade de produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** análise, arquitetura, coerência documental, gates e preparação de tarefas.
- **Codex:** execução técnica do escopo aprovado, sem inventar produto ou ampliar tarefa.

## Pré-flight para Codex

Antes de **cada nova tarefa Codex**, inclusive PoC ou trabalho técnico preparatório, o Assistente entrega separadamente:

1. `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. `PROMPT / ENUNCIADO PARA O CODEX`.

Usar o menor perfil de capacidade suficiente com margem de segurança, conforme `docs/00-governanca/politica-capacidade-codex.md`.

## Base Git obrigatória

Toda tarefa que permita alteração informa branch/base e commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` divergir, não fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Alteração preexistente pertence ao PO/outro fluxo.

Sem autorização explícita e específica do PO, é proibido:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alterações locais;
- sobrescrever arquivo modificado preexistente;
- trocar branch descartando trabalho;
- incluir alteração preexistente no commit da tarefa.

Se arquivo necessário já estiver modificado, parar e reportar.

## Disciplina Git do fechamento da Fase 1

```text
1 trabalho lógico
→ 1 branch ativa
→ 1 PR
→ revisão/aprovação
→ squash merge em main
→ apagar branch encerrada
→ verificar remoto somente com main e zero PRs abertos
→ próximo trabalho
```

- branch mergeada não está encerrada enquanto permanecer no remoto;
- remoto é a fonte operacional;
- sincronização do checkout local permanece adiada até antes do primeiro trabalho de implementação, preservando mudanças do PO.

## Regras operacionais

- uma tarefa lógica por vez;
- não declarar parcial como concluído;
- não criar funcionalidade, dependência ou estrutura relevante fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta, exemplo ou valor provisório em decisão;
- manter documentação e implementação sincronizadas;
- atualizar o painel do `README.md` quando houver mudança real de fase/bloco;
- preservar modularidade, responsabilidades claras e baixo acoplamento;
- evitar monólito HTML/JS;
- não versionar credenciais, segredos, banco real ou dados pessoais da empresa;
- exemplos de IP/hostname/share/path nunca viram configuração oficial;
- testes de LAN corporativa fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- PoC descartável não vira produção silenciosamente;
- documento técnico estável não deve carregar “próximo bloco” nem gate Git histórico consumido.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não repara o próprio ambiente alterando ACL, Schannel, registro, PATH global, políticas de segurança ou reinstalando ferramentas válidas.

Operações que realmente exijam credenciais, Internet confiável, elevação ou configuração global devem ser reportadas para execução apropriada na sessão Windows normal do PO.

## Contrato Pocket obrigatório

A experiência suportada é:

```text
pasta pronta publicada no servidor Windows
→ estação acessa o compartilhamento
→ usuário executa StepFlowLauncher.exe
→ Launcher prepara/valida o Client local automaticamente
→ Client abre de %LOCALAPPDATA%
→ Launcher encerra
```

É obrigatório no uso normal:

- zero instalador tradicional por estação;
- zero configuração manual de dependência;
- zero privilégio administrativo;
- zero toolchain de desenvolvimento em produção;
- nenhuma Internet obrigatória;
- Client operacional local, não executado permanentemente do SMB;
- atualização central pela pasta publicada + versões locais validadas;
- Host/Controller sob demanda na máquina central;
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo;
- sem Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon como baseline.

Se dependência exigir instalação, elevação ou preparação manual por computador, a solução não atende ao Pocket e deve ser redesenhada ou tratada como bloqueador.

### WebView2

- Tauri usa WebView2 no Windows;
- Evergreen compatível já presente é preferível;
- disponibilidade real deve ser detectada;
- não baixar/instalar runtime silenciosamente pela Internet em produção;
- Fixed Version não pode ser executado de UNC/SMB;
- fallback autocontido deve ser local e só entra após PoC provar `%LOCALAPPDATA%` sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo;
- falha dessa PoC em estação que deva ser suportada é bloqueador do fallback, não autorização para enfraquecer o contrato Pocket.

Fontes: `docs/03-arquitetura/implantacao-pocket.md`, `launcher-distribuicao-client.md`, `compatibilidade-windows-client.md`.

## Invariantes técnicas consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- WAL + writer lógico coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessão opaca e autorização Host-side;
- Procedimentos usam revisões imutáveis;
- `Procedimento`, `Atendimento/Execução` e `Equipamento` são domínios distintos;
- Atendimento preserva revisão efetivamente usada;
- checklist persiste somente em Atendimento;
- `Observação do serviço` persiste opcionalmente por Etapa somente em Atendimento;
- estado histórico relevante precisa ser reproduzível após conclusão/reabertura;
- Ficha compacta é prestação de contas resumida ao cliente;
- Reader usa experiência livro/manual e stepper de navegação, nunca como conclusão operacional;
- geração documental pertence ao Host e usa snapshot consistente + `DocumentModel`;
- PDF de Procedimentos usa Typst embutido; DOCX usa OOXML direto em Rust;
- impressão Windows usa o mesmo PDF oficial via WebView2;
- Ficha válida possui exatamente uma A4; `2+` páginas geram `SHEET_OVERFLOW`;
- artefatos persistentes e temporários têm lifecycles separados.

Detalhes ficam nas fontes específicas; não ampliar estas invariantes por inferência.

## Pendências ainda abertas

- parâmetros finais de Argon2id/senha/sessão/token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- mecanismo técnico do Bloco 11;
- estrutura oficial e plano da Fase 2 no Bloco 12;
- gates corporativos de Windows/WebView2/Launcher/SMB/Word/impressoras/EDR.

## Gate de implementação

Na Fase 1, trabalho estrutural significa documentação ou PoC explicitamente descartável autorizada. Não criar scaffold oficial, módulos runtime definitivos, árvore final ou código de negócio antes do Bloco 12/Fase 2 autorizar.

Antes do primeiro trabalho de implementação, sincronizar explicitamente o checkout local com o remoto **sem apagar, sobrescrever, descartar ou incorporar indevidamente** alterações preexistentes do PO.

## Regra final

Executar somente o escopo autorizado, preservar o que já está consolidado e preferir referência à fonte específica em vez de duplicar contratos em documentos de governança.
