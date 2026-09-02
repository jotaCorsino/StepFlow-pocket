# Modelo de dados local

## Entidades

### Company
Identidade visual e dados de contato.

### ResponsiblePerson
Rótulo local para campos de Responsável. Não é usuário nem identidade autenticada.

### Category
Classificação de Procedimentos com estado ativo/arquivado.

### Procedure
Identidade lógica do Procedimento.

### ProcedureRevision
Snapshot editorial imutável de uma revisão.

### ProcedureStage
Etapa/página do manual, com ordem, passos, observações e blocos de instrução.

### Attendance
Registro de trabalho real com lifecycle local.

### AttendanceProcedure
Vínculo entre Atendimento e revisão exata de Procedimento utilizada.

### AttendanceStageState
Checklist e observação operacional por Etapa dentro do Atendimento.

### Equipment
Ativo opcional/reutilizável.

## Identificadores

IDs técnicos devem ser estáveis e locais, preferencialmente UUID/crypto-random quando útil. Códigos humanos `AT-000001`, `EQP-000001` e códigos de Procedimento permanecem separados de IDs técnicos.

## Histórico

Revisões e vínculos históricos devem permitir reproduzir o conteúdo usado no Atendimento sem depender do estado atual dos cadastros.
