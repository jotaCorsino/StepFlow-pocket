# Tela 03 — Leitor de Procedimento

Experiência visual de livro/manual.

## Estrutura

```text
Visão geral
→ Etapa 1
→ Etapa 2
→ ...
```

Uma Etapa é uma página lógica. O stepper horizontal representa navegação, não conclusão operacional.

Cada página pode conter:

- título e objetivo da Etapa;
- passos numerados;
- observações;
- checklist de referência;
- blocos de comando/instrução;
- botão icon-only para copiar conteúdo de comando;
- navegação anterior/próxima.

Comandos preservam whitespace. O botão de copiar possui nome acessível/tooltip sem adicionar rótulo textual permanente desnecessário.

No Reader standalone, checklist não altera o Procedimento. Dentro de um Atendimento, o mesmo conteúdo pode receber estado operacional persistente pertencente ao Atendimento.
