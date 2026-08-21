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

O enunciado autoriza escopo, mas não revoga decisão consolidada. Proposta marcada como proposta não pode ser implementada como decisão sem aprovação do PO.

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
- `categorizacao-atendimentos-equipamentos.md` — **requisitos novos confirmados** de categorização/registro de serviço/equipamento/ficha compacta + **modelagem recomendada ainda em proposta**.

### Telas — `02-telas`

- `README.md` — mapa/limites do Bloco 8;
- `01-login.md` — Login consolidado;
- `02-shell-sidebar.md` — núcleo consolidado, extensão operacional em aprovação;
- `03-dashboard.md` — Dashboard em análise/proposta.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — arquitetura consolidada + extensão operacional explicitamente proposta;
- `implantacao-pocket.md` — implantação/ciclo de vida;
- `compatibilidade-windows-client.md` — Tauri/Windows/WebView2;
- `host-pocket.md` — Controller/Host;
- `launcher-distribuicao-client.md` — distribuição local do Client;
- `comunicacao-client-host.md` — HTTP/JSON/WebSocket;
- `autenticacao-sessao-autorizacao.md` — autenticação e parâmetros pendentes;
- `modelo-dados-schema-fase-1.md` — schema consolidado original + extensão operacional proposta;
- `concorrencia-fila-conflitos-eventos.md` — writer/fila/conflitos/eventos.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases;
- `plano-oficial-fase-1.md` — estado/gates/pendências;
- `tarefas-codex/README.md` — somente tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes, requisitos novos confirmados e propostas claramente separadas;
- `changelog-projeto.md` — marcos relevantes;
- `diario-de-progresso.md` — histórico; não é fonte superior de decisão;
- `revisao-cruzada-fase-0.md` — evidência histórica da Fase 0.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

**Fase 1 em andamento; Bloco 8 em execução. Requisito novo de categorização + ficha/registro de serviço/equipamento confirmado. Modelagem específica ainda aguarda aprovação do PO.**

Não há código funcional oficial.
