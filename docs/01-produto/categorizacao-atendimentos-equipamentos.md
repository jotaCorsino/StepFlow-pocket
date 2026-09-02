# Categorias, Atendimentos e Equipamentos

## Categorias

Procedimentos podem possuir múltiplas categorias configuráveis. O baseline não exige árvore/hierarquia.

Categoria pode ser arquivada. Uma categoria arquivada já herdada de uma revisão anterior pode permanecer com aviso ou ser removida; não pode ser adicionada como nova associação enquanto arquivada.

## Revisões de Procedimento

- salvamento é explícito;
- revisão publicada/histórica não é reescrita silenciosamente;
- `revision_no` técnico pode ser separado da versão editorial exibida;
- histórico registra alterações relevantes do documento;
- nova revisão parte do conteúdo anterior, preservando vínculos históricos.

## Atendimento

Estados:

```text
Em andamento
├─→ Concluído
└─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

O primeiro save gera um código local sequencial, por exemplo `AT-000001`.

Conclusão exige Responsável e Resumo do trabalho. Checklist incompleto avisa, mas não conclui/cancela automaticamente.

## Checklist operacional

Checklist do Reader standalone é apenas referência. Quando o Procedimento é usado dentro de um Atendimento, o estado do checklist pertence ao Atendimento.

O progresso do Atendimento deriva do checklist. 100% de checklist não conclui automaticamente o Atendimento.

## Observação do serviço

Cada Etapa usada no Atendimento pode receber uma observação operacional opcional, separada do texto oficial do Procedimento.

## Equipamento

Equipamento é opcional e reutilizável. Código local sugerido: `EQP-000001`.

Serial, MAC e patrimônio podem ajudar na busca, mas não são identificador técnico canônico.

## Histórico

Atendimentos concluídos devem continuar reproduzíveis com a revisão exata do Procedimento utilizada, mesmo após novas revisões do documento ou alterações posteriores no cadastro do Equipamento.
