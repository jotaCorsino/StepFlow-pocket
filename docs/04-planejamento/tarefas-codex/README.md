# Tarefas Codex — StepFlow Pocket

Esta pasta deve conter somente tarefas Codex **ativas ou imediatamente preparadas**.

Tarefas concluídas, troubleshooting e provas descartáveis antigas permanecem recuperáveis pelo histórico Git e não devem acumular na árvore documental ativa.

## Estado

Não há tarefa Codex aberta neste momento.

O estado corrente do projeto deve ser consultado em:

- `../../../README.md`;
- `../plano-oficial-fase-1.md`;
- `../roadmap.md`.

Este arquivo não replica número de bloco/fase corrente para evitar envelhecimento documental.

## Regra para nova tarefa

Antes de qualquer prompt ao Codex:

1. confirmar que o bloco/gate correspondente autoriza implementação;
2. apresentar ao PO o pré-flight de capacidade separado do enunciado;
3. usar `../../templates/template-tarefa-codex.md` quando aplicável;
4. registrar branch/base e commit SHA esperado;
5. indicar somente documentos vigentes necessários;
6. declarar escopo e fora do escopo;
7. identificar alterações locais conhecidas que devem permanecer intocadas;
8. definir critérios de aceite e validações;
9. confirmar que nenhum parâmetro pendente está sendo tratado como decisão definitiva.

O Codex deve ler `AGENTS.md`, inspecionar `HEAD`/working tree antes de alterar arquivos e interromper a execução quando a base divergir ou quando houver conflito com alterações preexistentes do PO.
