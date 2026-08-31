# Documentação do StepFlow Pocket

Esta pasta contém a documentação ativa do projeto. O `README.md` da raiz é apenas o painel executivo.

## Precedência

Em caso de divergência:

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `01-produto/visao-geral.md`;
5. tarefa dentro das decisões vigentes;
6. histórico Git.

Proposta não aprovada não pode ser implementada como decisão.

## Ownership documental

Cada camada possui uma função principal:

- **Governança** — regras de trabalho, ambientes e execução assistida;
- **Produto** — o que o StepFlow é, domínio, requisitos e limites;
- **Telas** — comportamento e UX das superfícies;
- **Arquitetura** — mecanismos, componentes, persistência, segurança e restrições técnicas;
- **Planejamento** — fases, blocos, dependências e gates ainda ativos;
- **Progresso** — decisões vigentes e marcos históricos.

Regra de higiene: documento técnico estável não anuncia “próximo bloco”. Estado corrente pertence ao `README.md`, ao plano da fase e ao roadmap.

## Leitura eficiente

Sempre:

1. `AGENTS.md`;
2. enunciado da tarefa;
3. este índice;
4. documento específico indicado.

Conforme impacto, consultar também registro de decisões, plano da fase, arquitetura vigente e contexto de ambientes.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — ambientes e limites de validação;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de capacidade.

### Produto — `01-produto`

- `visao-geral.md` — propósito, requisitos e limites;
- `categorizacao-atendimentos-equipamentos.md` — domínio `Procedimento × Atendimento × Equipamento`.

### Telas — `02-telas`

- `README.md` — índice das superfícies;
- `01-login.md` a `15-estados-transversais.md` — contratos UX consolidados;
- `13-backup-restauracao.md` — UX de Backup/Restore;
- `14-exportacao-impressao-ficha.md` — UX documental e Ficha compacta.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — mapa arquitetural;
- `modelo-dados-schema-fase-1.md` — modelo conceitual e histórico;
- `concorrencia-fila-conflitos-eventos.md` — concorrência e eventos;
- `implantacao-pocket.md` — implantação e ciclo central;
- `launcher-distribuicao-client.md` — distribuição Pocket do Client;
- `compatibilidade-windows-client.md` — Windows/WebView2;
- `host-pocket.md` — Controller/Host;
- `comunicacao-client-host.md` — HTTP/JSON/WebSocket;
- `autenticacao-sessao-autorizacao.md` — autenticação/autorização.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases;
- `plano-oficial-fase-1.md` — estado, gates e pendências da Fase 1;
- `bloco-9-atendimentos-execucao-checklist.md` — contrato operacional consolidado;
- `bloco-10-exportacao-impressao-ficha.md` — mapa técnico consolidado do Bloco 10;
- `bloco-10-etapa-11-validacao-tecnica-final.md` — matriz técnica final do Bloco 10;
- `bloco-11-backup-restauracao.md` — mapa principal do Bloco 11 em análise;
- `bloco-11-analise-3-catalogo-retencao-coordenacao.md` — Análise 3 aprovada: catálogo, retenção e coordenação;
- `bloco-11-analise-4-restore-safety-compatibilidade.md` — Análise 4 aprovada: Restore, safety backup e compatibilidade;
- `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md` — proposta da Análise 5: restart, sessões, reconexão e falhas;
- `tarefas-codex/README.md` — regra para tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes, pendências e gates;
- `changelog-projeto.md` — marcos cronológicos relevantes;
- `diario-de-progresso.md` — histórico inicial congelado;
- `revisao-cruzada-fase-0.md` — evidência histórica da Fase 0.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

A **Fase 1 permanece em andamento**. Blocos 0–10 estão documentalmente encerrados; no Bloco 11, Análises 1–4 estão aprovadas e a Análise 5 está em revisão. O Bloco 12 fecha estrutura oficial e plano da Fase 2.

Nenhuma implementação funcional oficial foi iniciada.
