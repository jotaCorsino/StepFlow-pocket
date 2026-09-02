# PROMPT / ENUNCIADO PARA O CODEX

## Objetivo

<resultado técnico pequeno e verificável>

## Base Git obrigatória

- branch base:
- SHA esperado:
- branch da tarefa:

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se houver divergência ou alteração local preexistente, parar e reportar.

## Escopo permitido

- arquivos/componentes:
- comportamento:

## Fora do escopo

- não ampliar produto;
- não introduzir dependência sem autorização;
- não alterar contratos documentados fora da tarefa.

## Validação

- lint/check/test/build aplicáveis;
- tamanho do bundle quando houver dependência/frontend;
- `git diff --check`;
- resumo objetivo das mudanças.
