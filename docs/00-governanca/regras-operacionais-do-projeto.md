# Regras Operacionais do Projeto — StepFlow Pocket

## Objetivo

Definir as regras mínimas que governam análise, planejamento, implementação, revisão e documentação do StepFlow Pocket.

## Regras gerais

- GitHub é a fonte principal de verdade.
- O projeto é conduzido por fases com gates explícitos.
- A execução técnica ocorre em tarefas pequenas, fechadas e revisáveis.
- Antes de cada nova tarefa destinada ao Codex, o Assistente deve realizar um pré-flight de capacidade e recomendar ao PO o modelo e nível de raciocínio adequados.
- O pré-flight de capacidade é uma instrução ao PO e deve aparecer separado do prompt técnico; não deve ser enviado ao Codex como parte do enunciado.
- A escolha de capacidade deve seguir `docs/00-governanca/politica-capacidade-codex.md`, buscando a menor capacidade que mantenha margem adequada de segurança.
- Implementação parcial não deve ser apresentada como entrega concluída.
- Decisões consolidadas devem ser separadas de propostas e pendências.
- Toda mudança estrutural relevante deve ser registrada.
- Toda evolução funcional relevante deve atualizar a documentação correspondente.
- Não criar funcionalidades fora do escopo documentado.
- Não adicionar campos, fluxos ou burocracia sem valor de produto aprovado.
- Não alterar direção visual ou UX aprovada por conveniência técnica.
- Não esconder problemas de concorrência, segurança ou persistência em nome de simplificação.
- Evitar monólitos e também evitar abstrações excessivas sem necessidade.

## Regra de execução atual

A fase vigente é **Fase 1 — Fechamento arquitetural e especificação**.

A Fase 1 autoriza investigação, documentação, decisões técnicas e provas descartáveis explicitamente aprovadas. Não autoriza antecipar funcionalidades de negócio, criar produção definitiva a partir de protótipos ou ignorar os gates definidos em `docs/04-planejamento/plano-oficial-fase-1.md`.

## Pré-flight obrigatório de capacidade do Codex

Antes de apresentar qualquer novo prompt de tarefa, o Assistente deve avaliar:

- complexidade;
- risco;
- escopo;
- quantidade de domínios/arquivos envolvidos;
- ambiguidade;
- impacto arquitetural;
- concorrência;
- segurança;
- dados/migrations;
- dificuldade de debugging;
- reversibilidade.

O Assistente deve então informar ao PO, em bloco separado do enunciado do Codex:

- modelo recomendado;
- nível de raciocínio recomendado;
- justificativa curta;
- condição de escalonamento, quando pertinente.

A capacidade deve ser escolhida por tarefa e não herdada automaticamente da tarefa anterior.

## Rastreabilidade mínima

Toda tarefa técnica relevante deve produzir evidência proporcional ao impacto e registrar:

- objetivo;
- arquivos alterados;
- validações;
- decisões técnicas relevantes;
- pendências/riscos;
- documentação impactada.

## Conflitos

Se uma demanda nova conflitar com decisão vigente:

1. registrar o conflito;
2. avaliar impacto;
3. consolidar a nova decisão com o PO;
4. atualizar documentos afetados;
5. somente então implementar a mudança.
