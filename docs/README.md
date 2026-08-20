# Documentação do StepFlow Pocket

Esta pasta contém apenas a documentação ativa necessária para entender e evoluir o projeto. Troubleshooting concluído, tarefas antigas e provas descartáveis devem ser consultados pelo histórico Git quando necessário, não mantidos como documentos vigentes.

## Precedência

Em caso de divergência:

1. `05-progresso/registro-de-decisoes.md`;
2. documento específico vigente de produto/arquitetura/tela/fase;
3. `01-produto/visao-geral.md`;
4. material histórico no Git.

Ambiguidade relevante volta ao PO; não é autorização para o executor escolher sozinho.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — desenvolvimento local versus ambiente corporativo;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de modelo/raciocínio antes de tarefas Codex.

Regras específicas de execução ficam em `../AGENTS.md`, evitando duplicação.

### Produto — `01-produto`

- `visao-geral.md` — propósito, usuários, requisitos e limites do StepFlow.

### Telas — `02-telas`

- `README.md` — mapa das telas e ordem de especificação do Bloco 8.

As especificações individuais serão criadas apenas conforme forem analisadas/aprovadas.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — visão consolidada Client/Launcher/Host/Data;
- `implantacao-pocket.md` — requisitos inegociáveis de implantação e ciclo de vida;
- `compatibilidade-windows-client.md` — Tauri/Windows/WebView2;
- `host-pocket.md` — tecnologia, Controller, Host, paths, shutdown e atualização;
- `launcher-distribuicao-client.md` — cópia local/versionamento do Client;
- `comunicacao-client-host.md` — HTTP/JSON, WebSocket e compatibilidade;
- `autenticacao-sessao-autorizacao.md` — usuários, sessão e permissões;
- `modelo-dados-schema-fase-1.md` — schema conceitual, revisões e migrations;
- `concorrencia-fila-conflitos-eventos.md` — writer, fila, conflitos e eventos.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases do projeto;
- `plano-oficial-fase-1.md` — estado dos blocos e gates da fase atual;
- `tarefas-codex/README.md` — lista somente de tarefas Codex ativas. Tarefas concluídas são removidas da árvore ativa e permanecem no histórico Git.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes e pendências;
- `changelog-projeto.md` — marcos relevantes;
- `diario-de-progresso.md` — registro cronológico histórico; não é fonte superior de decisão;
- `revisao-cruzada-fase-0.md` — evidência histórica do gate da Fase 0, mantida por referência do diário.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

**Fase 1 em andamento. Blocos 0–7 encerrados em nível arquitetural. Próximo: Bloco 8 — UI/UX.**

Não há código funcional oficial ainda.
