# Regras Operacionais do Projeto — StepFlow Pocket

## Objetivo

Definir as regras mínimas que governam análise, planejamento, implementação, revisão e documentação do StepFlow Pocket.

## Regras gerais

- GitHub é a fonte principal de verdade.
- O projeto é conduzido por fases com gates explícitos.
- A execução técnica ocorre em tarefas pequenas, fechadas e revisáveis.
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

Enquanto a Fase 0 estiver em andamento, o trabalho é documental e arquitetural. Não há autorização para iniciar lógica de negócio, instalar a stack definitiva ou criar um app funcional parcial sem tarefa explícita que altere o gate.

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
