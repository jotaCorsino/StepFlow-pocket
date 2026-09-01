# Documentação do StepFlow Pocket

Esta pasta contém a documentação ativa do projeto. O `README.md` da raiz é apenas o painel executivo.

## Precedência

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `01-produto/visao-geral.md`;
5. tarefa dentro das decisões vigentes;
6. histórico Git.

Proposta não aprovada não pode ser implementada como decisão.

## Ownership documental

- **Governança** — regras de trabalho, ambientes e execução assistida;
- **Produto** — propósito, domínio, requisitos e limites;
- **Telas** — comportamento e UX;
- **Arquitetura** — mecanismos, componentes, persistência, segurança e restrições;
- **Planejamento** — fases, blocos, dependências e gates ativos;
- **Progresso** — decisões vigentes e marcos históricos.

Documento técnico estável não anuncia próximo bloco. Estado corrente pertence ao README raiz, plano da fase e roadmap.

## Leitura eficiente

Sempre: `AGENTS.md` → enunciado → este índice → documento específico. Conforme impacto, consultar registro de decisões, plano, arquitetura e contexto de ambientes.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — ambientes/gates reais;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de capacidade.

### Produto — `01-produto`

- `visao-geral.md`;
- `categorizacao-atendimentos-equipamentos.md`.

### Telas — `02-telas`

- `README.md` — índice das superfícies;
- `01-login.md` a `15-estados-transversais.md` — contratos UX;
- `13-backup-restauracao.md` — UX de Backup/Restore;
- `14-exportacao-impressao-ficha.md` — UX documental/Ficha.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — mapa arquitetural;
- `modelo-dados-schema-fase-1.md` — modelo conceitual/histórico;
- `concorrencia-fila-conflitos-eventos.md`;
- `implantacao-pocket.md`;
- `launcher-distribuicao-client.md`;
- `compatibilidade-windows-client.md`;
- `host-pocket.md`;
- `comunicacao-client-host.md`;
- `autenticacao-sessao-autorizacao.md`.

### Planejamento — `04-planejamento`

- `roadmap.md`;
- `plano-oficial-fase-1.md`;
- `bloco-9-atendimentos-execucao-checklist.md`;
- `bloco-10-exportacao-impressao-ficha.md`;
- `bloco-10-etapa-11-validacao-tecnica-final.md`;
- `bloco-11-backup-restauracao.md` — D11.1–D11.116 + parâmetros D12 aplicáveis;
- análises 3–7 do Bloco 11 — detalhes técnicos consolidados;
- `bloco-12-estrutura-oficial-plano-fase-2.md` — mapa principal;
- `bloco-12-analise-2-workspace-build-dependencias.md` — D12.19–D12.34;
- `bloco-12-analise-3-migrations-testes-fixtures.md` — D12.35–D12.55;
- `bloco-12-analise-4-parametros-finais.md` — D12.56–D12.79;
- `bloco-12-analise-5-plano-fase-2.md` — D12.80–D12.98;
- `bloco-12-analise-6-validacao-final-fase-1.md` — proposta P12.99–P12.108;
- `tarefas-codex/README.md` — regra para tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes/gates;
- `changelog-projeto.md` — marcos;
- `diario-de-progresso.md` — histórico inicial congelado;
- `revisao-cruzada-fase-0.md` — evidência histórica.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

A **Fase 1 permanece em andamento**. Blocos 0–11 estão concluídos; Bloco 12 está na validação final. D12.1–D12.98 estão aprovadas e P12.99–P12.108 permanecem em revisão.

Nenhuma implementação funcional oficial foi iniciada.
