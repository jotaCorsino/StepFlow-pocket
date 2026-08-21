# Tarefas Codex — StepFlow Pocket

Esta pasta deve conter somente tarefas Codex **ativas ou imediatamente preparadas**.

Tarefas concluídas, troubleshooting e provas descartáveis antigas são removidos da árvore documental ativa e permanecem recuperáveis pelo histórico Git.

## Estado atual

Não há tarefa Codex aberta neste momento.

A Fase 1 está no **Bloco 8 — UI/UX**, que deve primeiro produzir/aprovar especificações de tela antes de abrir implementação.

## Regra para nova tarefa

Antes de qualquer prompt ao Codex:

1. confirmar que o bloco/gate correspondente autoriza a tarefa;
2. apresentar ao PO o pré-flight de capacidade separado do enunciado;
3. usar `docs/templates/template-tarefa-codex.md`;
4. registrar branch/base e commit SHA esperado;
5. indicar somente documentos vigentes necessários;
6. declarar escopo e fora do escopo;
7. identificar alterações locais conhecidas que devem permanecer intocadas;
8. definir critérios de aceite e validações;
9. confirmar que nenhum parâmetro pendente está sendo tratado como decisão definitiva.

O Codex deve ler `AGENTS.md` antes de executar e inspecionar `HEAD`/working tree antes de alterar arquivos.

Se a base divergir, houver arquivo necessário já modificado ou o prompt conflitar com decisão vigente, a tarefa deve parar em vez de tentar corrigir o checkout ou reinterpretar o produto.
