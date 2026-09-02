# Método padrão de trabalho assistido — StepFlow

## Fluxo

```text
necessidade
→ análise do Assistente
→ decisão do PO
→ branch
→ draft PR
→ implementação/documentação
→ validação
→ aprovação
→ squash merge
→ apagar branch
→ verificar remoto limpo
```

Uma tarefa lógica por branch/PR.

## Papéis

- **PO:** produto, prioridade e aprovação.
- **Assistente:** arquitetura, coerência, pesquisa, gates e preparação de tarefas.
- **Codex:** execução técnica do escopo aprovado.

## Working tree

Alterações locais preexistentes pertencem ao PO. Sem autorização explícita é proibido usar `reset --hard`, `clean`, `stash`, descartar/restaurar alterações ou sobrescrever trabalho local.

## Tarefas Codex

Antes de cada tarefa técnica o Assistente entrega separadamente:

1. `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. `PROMPT / ENUNCIADO PARA O CODEX`.

O prompt informa base/branch/SHA e critérios de aceite.

## Dependências

Biblioteca externa entra somente quando reduzir complexidade ou risco de forma concreta. Preferir APIs nativas da Web; medir impacto no bundle e registrar licença/versão.
