# Documentação do StepFlow Pocket

Esta pasta contém a documentação ativa necessária para entender e evoluir o projeto. O `README.md` da raiz funciona como **painel resumido de acompanhamento**, mas não substitui as fontes de decisão abaixo.

## Precedência

Em caso de divergência durante uma tarefa:

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md` — decisão consolidada mais recente;
3. documento específico vigente de produto/arquitetura/tela/fase;
4. `01-produto/visao-geral.md`;
5. enunciado da tarefa, dentro das decisões vigentes;
6. material histórico no Git.

O enunciado autoriza escopo, mas não revoga decisão consolidada. Proposta não pode ser implementada como decisão sem aprovação do PO.

## Leitura eficiente para Codex

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

`metodo-padrao-trabalho-assistido.md` e `politica-capacidade-codex.md` orientam principalmente PO/Assistente.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — sessão normal do PO, sandbox Codex e ambiente corporativo;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de modelo/raciocínio.

Regras obrigatórias de execução ficam em `../AGENTS.md`.

### Produto — `01-produto`

- `visao-geral.md` — propósito, usuários, requisitos e limites;
- `categorizacao-atendimentos-equipamentos.md` — categorização e domínio `Procedimento × Atendimento × Equipamento`, já aprovados conceitualmente, além das pendências dos Blocos 9/10.

### Telas — `02-telas`

- `README.md` — mapa/limites do Bloco 8;
- `01-login.md` — Login consolidado;
- `02-shell-sidebar.md` — Shell consolidado;
- `03-dashboard.md` — Dashboard consolidado;
- `04-lista-pesquisa-processos.md` — Lista/Pesquisa de Processos consolidada;
- `05-leitor-processo.md` — Leitor em formato livro consolidado;
- `06-editor-processo.md` — Editor de Processo + categorias consolidado;
- `07-historico-revisoes.md` — Histórico/Revisões consolidado;
- `08-lista-pesquisa-atendimentos.md` — Lista/Pesquisa de Atendimentos consolidada;
- `09-atendimento-execucao-equipamento.md` — Atendimento/Execução + Equipamento consolidado;
- `10-usuarios-permissoes.md` — Usuários/Permissões consolidado;
- `11-meu-perfil.md` — Meu perfil consolidado;
- `12-configuracoes-categorias.md` — Configurações + Categorias consolidado;
- próxima especificação: Tela 13 — Backup/Restauração — UX.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — arquitetura consolidada;
- `implantacao-pocket.md` — implantação/ciclo de vida;
- `compatibilidade-windows-client.md` — Tauri/Windows/WebView2;
- `host-pocket.md` — Controller/Host;
- `launcher-distribuicao-client.md` — distribuição local do Client;
- `comunicacao-client-host.md` — HTTP/JSON/WebSocket;
- `autenticacao-sessao-autorizacao.md` — autenticação e parâmetros pendentes;
- `modelo-dados-schema-fase-1.md` — schema conceitual consolidado, incluindo extensão operacional;
- `concorrencia-fila-conflitos-eventos.md` — writer/fila/conflitos/eventos.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases;
- `plano-oficial-fase-1.md` — estado/gates/pendências;
- `tarefas-codex/README.md` — somente tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes e pendências;
- `changelog-projeto.md` — marcos relevantes;
- `diario-de-progresso.md` — histórico; não é fonte superior de decisão;
- `revisao-cruzada-fase-0.md` — evidência histórica da Fase 0.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

**Fase 1 em andamento; Bloco 8 em execução. Telas 01–12 consolidadas. Próxima: Tela 13 — Backup/Restauração — UX.**

A modelagem `Procedimento × Atendimento/Execução × Equipamento`, categorias múltiplas, identidade interna de equipamento e a UX de identidade da empresa/gestão de categorias estão aprovadas. Lifecycle/permissões/checklist permanecem para o Bloco 9; formato técnico da ficha compacta, para o Bloco 10; Backup/Restore técnico, para o Bloco 11.

Não há código funcional oficial.