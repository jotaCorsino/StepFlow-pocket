# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para Codex e outros agentes que atuem neste repositório.

## Fonte de verdade e fase atual

- GitHub é a fonte principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Desenvolvimento atual: computador pessoal fora da LAN corporativa.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–7 estão fechados no núcleo arquitetural; Bloco 8 (UI/UX) está em andamento.
- Novo requisito de 2026-08-21 incorporou categorização, atendimentos/execuções formais, equipamentos opcionais e ficha compacta imprimível.

## Precedência e autoridade da tarefa

O enunciado da tarefa define **o trabalho autorizado**, mas não revoga silenciosamente decisões vigentes.

Em caso de conflito, aplicar esta ordem:

1. `AGENTS.md`;
2. decisão consolidada mais recente em `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente da funcionalidade/arquitetura/tela/fase;
4. `docs/01-produto/visao-geral.md`;
5. enunciado da tarefa, apenas dentro das decisões vigentes;
6. material histórico.

Se o enunciado exigir contrariar uma decisão consolidada, só prosseguir quando ele declarar explicitamente nova decisão aprovada pelo PO e incluir a atualização dos documentos vigentes afetados. Caso contrário, parar e reportar.

Ambiguidade nunca autoriza escolher a alternativa mais conveniente ao código.

## Leitura do Codex por camadas

### Sempre ler antes de alterar qualquer arquivo

1. `AGENTS.md`;
2. o enunciado da tarefa;
3. `docs/README.md`;
4. os documentos específicos indicados pela tarefa.

### Ler quando houver impacto correspondente

- `docs/05-progresso/registro-de-decisoes.md` — produto/arquitetura/regra consolidada;
- `docs/04-planejamento/plano-oficial-fase-1.md` — autorização/gate da fase;
- `docs/03-arquitetura/arquitetura-vigente.md` — impacto arquitetural;
- `docs/00-governanca/contexto-ambientes.md` — ambiente/rede/instalação/toolchain;
- `docs/01-produto/categorizacao-atendimentos-equipamentos.md` — categorias, atendimentos, equipamentos e ficha compacta;
- demais documentos técnicos específicos.

`metodo-padrao-trabalho-assistido.md` e `politica-capacidade-codex.md` orientam principalmente PO/Assistente e não precisam ser relidos pelo Codex em toda tarefa, salvo quando a tarefa tratar dessas políticas.

## Papéis

- **PO:** define produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** analisa, arquiteta, documenta e transforma decisões aprovadas em tarefas.
- **Codex:** executa tecnicamente o escopo recebido, sem inventar produto ou ampliar tarefa.

## Pré-flight de capacidade

A seleção de modelo/raciocínio é responsabilidade do Assistente + PO antes do envio da tarefa ao Codex. O Codex não altera escopo/comportamento com base nessa seleção.

## Base Git obrigatória da tarefa

Toda tarefa que permita alteração do repositório deve informar branch/base esperada e commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` não corresponder ao SHA esperado, não fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Qualquer alteração preexistente pertence ao PO ou outro fluxo.

É proibido, salvo autorização explícita e específica do PO:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alterações locais;
- sobrescrever arquivo modificado preexistente;
- trocar branch descartando trabalho;
- incluir alteração preexistente no commit da tarefa.

Se arquivo necessário já estiver modificado, parar e reportar conflito.

## Regras operacionais

- uma tarefa por vez;
- não declarar trabalho parcial como concluído;
- não criar funcionalidade, dependência ou estrutura relevante fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta ou parâmetro provisório em decisão;
- manter documentação e implementação sincronizadas;
- preservar modularidade e baixo acoplamento;
- não versionar credenciais, senhas, tokens, banco real ou dados pessoais da empresa;
- exemplos de IP/hostname/share/path nunca viram configuração oficial;
- testes dependentes da LAN corporativa fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- protótipos descartáveis não podem ser promovidos silenciosamente a produção.

## Ambiente Codex versus sessão normal do PO

Restrições do sandbox Codex não viram requisito do produto.

Codex não deve tentar reparar o próprio ambiente por alteração de ACL/permissões globais, Schannel/políticas de segurança, registro/PATH global, reinstalação de ferramentas válidas ou sequência aberta de microdiagnósticos.

Quando a operação exigir credenciais, Internet confiável, elevação ou instalação/configuração global, reportar para execução controlada na sessão Windows normal do PO.

## Regras Pocket obrigatórias

- implantação central baseada em copiar/publicar pasta pronta;
- nenhuma toolchain de desenvolvimento exigida no servidor de produção;
- não usar Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon residente como padrão;
- Host e Controller iniciam sob demanda;
- Controller aberto representa ciclo central ativo; encerrado o ciclo, nenhum processo StepFlow permanece ativo;
- fechar Client individual não encerra Host;
- não inventar auto-shutdown por ausência de Clients/timeout;
- Client roda localmente na estação, preparado por launcher transitório;
- launcher encerra após iniciar Client;
- dados/config/logs permanecem separados dos binários substituíveis.

Qualquer exceção exige mudança explícita do requisito pelo PO.

## Regras técnicas já consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON para API e WebSocket para eventos;
- SQLite somente pelo Host local aos dados;
- writer coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessões opacas e autorização sempre no Host;
- Argon2id para senha, com parâmetros finais ainda sujeitos à decisão documentada;
- procedimentos documentais usam revisões imutáveis;
- categorias de procedimentos são configuráveis e podem ser múltiplas;
- atendimento/execução formal é requisito vigente quando houver rastreabilidade operacional;
- equipamento é opcional e separado do procedimento;
- atendimento preserva a revisão de procedimento efetivamente utilizada;
- MAC/serial/patrimônio podem ser usados para busca, mas não substituem identidade interna estável do equipamento;
- PDF, DOCX e impressão são requisitos da documentação;
- ficha compacta imprimível de atendimento/equipamento é requisito, com formato técnico pendente do Bloco 10;
- lifecycle do atendimento, checklist/progresso e matriz operacional de permissões permanecem pendentes do Bloco 9.

## Tarefa Codex

Toda tarefa deve declarar objetivo, base Git esperada, fonte de verdade, escopo incluído, fora do escopo, critérios de aceite, validações e documentação impactada.

O relatório final informa objetivo, base/estado inicial, arquivos alterados, decisões técnicas, validações/resultados, riscos/pendências, documentação e próximos passos sugeridos.

## Gate de implementação da Fase 1

Durante a Fase 1, trabalho “estrutural” significa apenas documentação, organização documental ou PoC explicitamente descartável autorizada.

Não criar scaffold oficial, módulos runtime definitivos, árvore final de Client/Host/Launcher, código de negócio ou implementação de produção antes do Bloco 12/Fase 2 autorizar explicitamente.

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental ou investigativo explicitamente solicitado.
