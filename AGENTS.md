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

## Leitura obrigatória antes de implementar

1. `README.md`;
2. `docs/README.md`;
3. `docs/00-governanca/contexto-ambientes.md`;
4. `docs/00-governanca/metodo-padrao-trabalho-assistido.md`;
5. `docs/00-governanca/politica-capacidade-codex.md`;
6. `docs/01-produto/visao-geral.md`;
7. `docs/03-arquitetura/arquitetura-vigente.md`;
8. `docs/03-arquitetura/implantacao-pocket.md`;
9. `docs/05-progresso/registro-de-decisoes.md`;
10. `docs/04-planejamento/plano-oficial-fase-1.md` enquanto a Fase 1 estiver vigente;
11. documentos específicos da tarefa.

## Papéis

- **PO:** define produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** analisa, arquiteta, documenta e transforma decisões aprovadas em tarefas.
- **Codex:** executa tecnicamente o escopo recebido, sem inventar produto ou ampliar tarefa.

## Pré-flight obrigatório

Antes de cada nova tarefa destinada ao Codex, o Assistente deve apresentar ao PO, separadamente do prompt técnico:

- modelo recomendado;
- nível de raciocínio;
- motivo;
- condição de escalonamento, quando pertinente.

Seguir `docs/00-governanca/politica-capacidade-codex.md` e buscar a menor capacidade que mantenha margem adequada de segurança.

## Regras operacionais

- uma tarefa por vez;
- não declarar trabalho parcial como concluído;
- não criar funcionalidade, dependência ou estrutura relevante fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta em decisão;
- manter documentação e implementação sincronizadas;
- preservar modularidade e baixo acoplamento;
- não versionar credenciais, senhas, tokens, banco real ou dados pessoais da empresa;
- exemplos de IP/hostname/share/path nunca viram configuração oficial;
- testes dependentes da LAN corporativa feitos fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- protótipos descartáveis não podem ser promovidos silenciosamente a produção.

## Regras Pocket obrigatórias

- implantação central baseada em copiar/publicar pasta pronta;
- nenhuma toolchain de desenvolvimento exigida no servidor de produção;
- não usar Windows Service persistente, serviço auto-start, Task Scheduler, watchdog, tray agent ou daemon residente como padrão;
- Host e Controller iniciam sob demanda;
- quando o StepFlow está fechado/sem uso, nenhum processo StepFlow deve permanecer ativo no servidor;
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
- Argon2id para senha;
- processos documentais usam revisões imutáveis;
- PDF, DOCX e impressão são requisitos do produto;
- estado das marcações do checklist durante execução ainda é pendência.

## Tarefa Codex

Toda tarefa deve declarar objetivo, fonte de verdade, escopo incluído, fora do escopo, critérios de aceite, validações e documentação impactada.

O relatório final deve informar: objetivo executado, arquivos alterados, decisões técnicas, validações/resultados, riscos/pendências, documentação atualizada e próximos passos sugeridos.

## Gate de implementação

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental, investigativo ou estrutural explicitamente solicitado.
