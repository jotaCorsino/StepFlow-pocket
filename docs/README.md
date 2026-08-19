# Documentação do StepFlow Pocket

## Objetivo

Esta pasta concentra a documentação viva e operacional do StepFlow Pocket. Ela deve permitir que PO, Assistente e Codex entendam o estado vigente do produto sem depender da memória de conversas anteriores.

## Princípios

- GitHub é a fonte principal de verdade do projeto.
- Toda evolução relevante deve deixar rastreabilidade documental proporcional ao impacto.
- Decisões consolidadas devem ser separadas de hipóteses, propostas e pendências.
- Implementação e documentação devem permanecer sincronizadas.
- Documentos históricos não devem ser silenciosamente reescritos para parecer que uma decisão sempre existiu.
- Quando uma decisão for substituída, registrar a nova decisão e indicar a anterior como superada quando necessário.

## Estrutura

### `00-governanca`

Regras de trabalho, papéis, precedência documental, guia mestre e método reutilizável de desenvolvimento assistido.

### `01-produto`

Visão do produto, atores, objetivos, requisitos funcionais e não funcionais e limites de escopo.

### `02-telas`

Uma especificação por tela ou superfície relevante. Deve cobrir layout, componentes, interações, estados, dados, permissões, erros, regras e critérios de aceite.

### `03-arquitetura`

Arquitetura do sistema, estrutura do repositório, decisões técnicas, compatibilidade Windows, modelo de dados, contratos, segurança, concorrência, persistência e ADRs.

Documentos já relevantes:

- `arquitetura-inicial.md` — separação lógica Client/Host/Data;
- `compatibilidade-windows-client.md` — investigação da Fase 1 sobre Tauri, WebView2, Windows e distribuição do Client.

### `04-planejamento`

Roadmap, fases, dependências, gates de entrada/saída, planos oficiais e tarefas fechadas para o Codex.

Documentos vigentes:

- `roadmap.md`;
- `plano-oficial-fase-0.md` — concluído;
- `plano-oficial-fase-1.md` — vigente;
- `tarefas-codex/` — tarefas investigativas/implementáveis em escopo fechado.

### `05-progresso`

Registro de decisões, diário de progresso, changelog e evidências de revisões/gates.

### `templates`

Modelos reutilizáveis para análises e tarefas.

## Precedência documental

Em caso de divergência, usar esta ordem prática até que uma decisão específica determine algo diferente:

1. decisão consolidada mais recente em `05-progresso/registro-de-decisoes.md`;
2. documento específico e vigente da tela, fluxo, arquitetura ou fase;
3. `00-governanca/guia-mestre-desenvolvimento.md`;
4. visão geral de produto;
5. material histórico ou conversas antigas.

Se ainda houver ambiguidade relevante, ela deve ser tratada como pendência, não como licença para o executor escolher sozinho.

## Estado atual

A **Fase 0 — Fundação documental e governança** foi concluída em 2026-08-19.

A fase vigente é **Fase 1 — Fechamento arquitetural e especificação**.

O StepFlow ainda não possui implementação funcional. A Fase 1 permite investigação técnica, documentação e provas explicitamente descartáveis, mas continua bloqueando a antecipação de funcionalidades de negócio.

O primeiro bloco vigente é **Plataforma Windows, Client e distribuição**. A primeira tarefa Codex preparada é:

`docs/04-planejamento/tarefas-codex/f1-b1-t01-inventario-ambiente-windows.md`
