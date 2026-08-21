# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para Codex e outros agentes que atuem neste repositório.

## Fonte de verdade e fase atual

- GitHub é a fonte principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Desenvolvimento atual: computador pessoal fora da LAN corporativa.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–7 da Fase 1 estão fechados; Bloco 8 (UI/UX) é o próximo.

## Precedência e autoridade da tarefa

O enunciado da tarefa define **o trabalho autorizado**, mas não revoga silenciosamente decisões vigentes.

Em caso de conflito, aplicar esta ordem:

1. `AGENTS.md`;
2. decisão consolidada mais recente em `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente da funcionalidade/arquitetura/tela/fase;
4. `docs/01-produto/visao-geral.md`;
5. enunciado da tarefa, apenas dentro das decisões vigentes;
6. material histórico.

Se o enunciado exigir contrariar uma decisão consolidada, só prosseguir quando ele declarar explicitamente que existe **nova decisão aprovada pelo PO** e incluir a atualização dos documentos vigentes afetados. Caso contrário, parar e reportar o conflito.

Ambiguidade nunca autoriza escolher a alternativa mais conveniente ao código.

## Leitura do Codex por camadas

### Sempre ler antes de alterar qualquer arquivo

1. `AGENTS.md`;
2. o enunciado da tarefa;
3. `docs/README.md`;
4. os documentos específicos indicados pela tarefa.

### Ler quando houver impacto correspondente

- `docs/05-progresso/registro-de-decisoes.md` — produto/arquitetura/regra já consolidada;
- `docs/04-planejamento/plano-oficial-fase-1.md` — autorização/gate da fase atual;
- `docs/03-arquitetura/arquitetura-vigente.md` — impacto arquitetural ou integração entre componentes;
- `docs/00-governanca/contexto-ambientes.md` — ambiente, ferramentas, rede, SMB, instalação ou validação externa;
- demais documentos técnicos específicos do assunto.

`docs/00-governanca/metodo-padrao-trabalho-assistido.md` e `docs/00-governanca/politica-capacidade-codex.md` são governança do fluxo PO/Assistente e **não precisam ser relidos pelo Codex em toda tarefa**, salvo se a própria tarefa tratar dessas políticas.

## Papéis

- **PO:** define produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** analisa, arquiteta, documenta e transforma decisões aprovadas em tarefas.
- **Codex:** executa tecnicamente o escopo recebido, sem inventar produto ou ampliar tarefa.

## Pré-flight de capacidade

A seleção de modelo/raciocínio é responsabilidade do Assistente + PO antes do envio da tarefa ao Codex. O Codex não deve alterar escopo ou comportamento com base nessa seleção.

## Base Git obrigatória da tarefa

Toda tarefa que permita alteração do repositório deve informar:

- branch/base esperada;
- commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` não corresponder ao SHA esperado, **não** fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo automaticamente. Parar e reportar o estado encontrado.

## Proteção absoluta do working tree

Qualquer alteração preexistente ao início da tarefa deve ser tratada como trabalho do PO ou de outro fluxo.

É proibido, salvo autorização explícita e específica do PO para aquela ação:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alterações locais;
- sobrescrever arquivo modificado preexistente;
- trocar branch de modo que descarte trabalho;
- incluir alteração preexistente no commit da tarefa.

Se um arquivo necessário à tarefa já estiver modificado antes do início, parar e reportar o conflito em vez de tentar “limpar” o checkout.

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
- testes dependentes da LAN corporativa feitos fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- protótipos descartáveis não podem ser promovidos silenciosamente a produção.

## Ambiente Codex versus sessão normal do PO

Restrições específicas do sandbox/execução do Codex não viram requisito do produto.

O Codex não deve tentar reparar o próprio ambiente por meio de:

- alteração de ACL/permissões globais;
- mudanças de Schannel/políticas de segurança;
- alterações de registro/PATH global;
- reinstalação de ferramentas já válidas no ambiente normal;
- sequência aberta de microdiagnósticos sem nova evidência.

Quando uma operação realmente exigir credenciais, Internet confiável, elevação ou instalação/configuração global, reportar a necessidade para execução controlada na sessão Windows normal do PO. Não contornar limitações do sandbox enfraquecendo o sistema.

## Regras Pocket obrigatórias

- implantação central baseada em copiar/publicar pasta pronta;
- nenhuma toolchain de desenvolvimento exigida no servidor de produção;
- não usar Windows Service persistente, serviço auto-start, Task Scheduler, watchdog, tray agent ou daemon residente como padrão;
- Host e Controller iniciam sob demanda;
- o Controller aberto representa o ciclo central ativo; quando esse ciclo for encerrado, nenhum processo StepFlow deve permanecer ativo no servidor;
- fechar um Client individual não encerra o Host central;
- não inventar auto-shutdown por ausência de Clients ou timeout sem decisão explícita;
- Client operacional roda localmente na estação, preparado por launcher transitório;
- launcher também encerra após iniciar o Client;
- dados/configuração/logs permanecem separados dos binários substituíveis.

Qualquer exceção futura a essas regras exige mudança explícita do requisito pelo PO.

## Regras técnicas já consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON para API e WebSocket para eventos;
- SQLite somente pelo Host local aos dados;
- writer coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessões opacas e autorização sempre no Host;
- Argon2id para senha, com parâmetros operacionais finais ainda sujeitos à decisão documentada;
- processos documentais usam revisões imutáveis;
- PDF, DOCX e impressão são requisitos do produto;
- estado das marcações do checklist durante execução ainda é pendência.

## Tarefa Codex

Toda tarefa deve declarar objetivo, base Git esperada, fonte de verdade, escopo incluído, fora do escopo, critérios de aceite, validações e documentação impactada.

O relatório final deve informar: objetivo executado, base/estado inicial observado, arquivos alterados, decisões técnicas, validações/resultados, riscos/pendências, documentação atualizada e próximos passos sugeridos.

## Gate de implementação da Fase 1

Durante a Fase 1, trabalho “estrutural” significa apenas documentação, organização documental ou **PoC explicitamente descartável** autorizada pelo plano/tarefa.

Não criar scaffold oficial, módulos runtime definitivos, árvore final de Client/Host/Launcher, código de negócio ou implementação de produção antes do Bloco 12/Fase 2 autorizar explicitamente.

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental ou investigativo explicitamente solicitado.
