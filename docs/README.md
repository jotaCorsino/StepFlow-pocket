# Documentação do StepFlow Pocket

Esta pasta contém apenas a documentação ativa necessária para entender e evoluir o projeto. Troubleshooting concluído, tarefas antigas e provas descartáveis devem ser consultados pelo histórico Git quando necessário, não mantidos como documentos vigentes.

## Precedência

Em caso de divergência durante uma tarefa:

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md` — decisão consolidada mais recente;
3. documento específico vigente de produto/arquitetura/tela/fase;
4. `01-produto/visao-geral.md`;
5. enunciado da tarefa, dentro das decisões vigentes;
6. material histórico no Git.

O enunciado autoriza o escopo, mas não revoga silenciosamente decisão consolidada. Para alterar uma decisão vigente, a tarefa deve declarar explicitamente a nova decisão aprovada pelo PO e atualizar os documentos afetados.

Ambiguidade relevante volta ao PO/Assistente; não é autorização para o executor escolher sozinho.

## Leitura eficiente para Codex

O Codex não precisa reler toda a governança a cada tarefa.

Sempre:

- `AGENTS.md`;
- enunciado da tarefa;
- este índice;
- documentos específicos indicados.

Conforme impacto:

- registro de decisões;
- plano da fase;
- arquitetura vigente;
- contexto de ambientes;
- documento técnico relacionado.

`metodo-padrao-trabalho-assistido.md` e `politica-capacidade-codex.md` orientam principalmente PO/Assistente e só precisam ser lidos pelo Codex quando a tarefa tratar dessas políticas.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — sessão normal do PO, sandbox Codex e ambiente corporativo;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de modelo/raciocínio antes de tarefas Codex.

Regras específicas e obrigatórias de execução ficam em `../AGENTS.md`, evitando duplicação.

### Produto — `01-produto`

- `visao-geral.md` — propósito, usuários, requisitos e limites do StepFlow.

### Telas — `02-telas`

- `README.md` — mapa das telas, limites e ordem de especificação do Bloco 8.

As especificações individuais serão criadas apenas conforme forem analisadas/aprovadas.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — visão consolidada Client/Launcher/Host/Data;
- `implantacao-pocket.md` — requisitos inegociáveis de implantação e ciclo de vida;
- `compatibilidade-windows-client.md` — Tauri/Windows/WebView2;
- `host-pocket.md` — tecnologia, Controller, Host, ciclo de vida, paths, shutdown e atualização;
- `launcher-distribuicao-client.md` — cópia local/versionamento do Client;
- `comunicacao-client-host.md` — HTTP/JSON, WebSocket e compatibilidade;
- `autenticacao-sessao-autorizacao.md` — decisões consolidadas e parâmetros ainda pendentes de autenticação/permissões;
- `modelo-dados-schema-fase-1.md` — schema conceitual, revisões e migrations;
- `concorrencia-fila-conflitos-eventos.md` — writer, fila, conflitos e eventos.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases do projeto;
- `plano-oficial-fase-1.md` — estado dos blocos, gates e limites da fase atual;
- `tarefas-codex/README.md` — lista somente de tarefas Codex ativas. Tarefas concluídas são removidas da árvore ativa e permanecem no histórico Git.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes e pendências;
- `changelog-projeto.md` — marcos relevantes;
- `diario-de-progresso.md` — registro cronológico histórico; não é fonte superior de decisão;
- `revisao-cruzada-fase-0.md` — evidência histórica do gate da Fase 0, mantida por referência do diário.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md` — inclui base Git, proteção do working tree e regras de parada.

## Estado atual

**Fase 1 em andamento. Blocos 0–7 encerrados no núcleo arquitetural; parâmetros operacionais explicitamente pendentes continuam bloqueados para implementação até decisão. Próximo: Bloco 8 — UI/UX.**

Não há código funcional oficial ainda.
