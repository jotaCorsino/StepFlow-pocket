# Modelo de Dados, Schema, Migrations e Histórico — StepFlow

**Status:** NÚCLEO CONSOLIDADO / EXTENSÃO OPERACIONAL PROPOSTA PARA APROVAÇÃO  
**Atualização:** 2026-08-21

## Princípios consolidados

- SQLite acessado somente pelo Host;
- foreign keys habilitadas;
- transações em alterações compostas;
- timestamps em UTC;
- identificadores estáveis para entidades de negócio/histórico;
- arquivos administrados ficam no filesystem do Host e o banco guarda referências/metadados;
- exclusão normal prefere arquivamento/desativação quando há histórico;
- migrations são versionadas/imutáveis após publicação;
- revisões de procedimento são imutáveis.

## Usuários, sessões e empresa

`users`, `sessions` e `company_settings` permanecem conforme o núcleo já consolidado de autenticação/empresa.

## Processos

`processes` mantém identidade estável e ponteiros para revisões:

```text
process_id
code UNIQUE
current_revision_id
published_revision_id NULL
is_archived
created_at / created_by_user_id
archived_at / archived_by_user_id
```

`process_revisions` preserva snapshots imutáveis. `revision_no` técnico continua separado de `display_version` editorial.

Cada revisão possui `process_stages` e `stage_blocks` ordenados. Tipos iniciais permanecem:

```text
paragraph
numbered_steps
checklist
note
warning
command
code
```

O estado operacional do checklist continua pendente do Bloco 9.

## Extensão nova — requisito confirmado, schema proposto

O PO confirmou necessidade de:

- categorização de procedimentos;
- registro de informações específicas de computador/equipamento em serviços aplicáveis;
- busca por identificadores/referências úteis;
- resumo dos procedimentos realizados;
- ficha compacta imprimível.

As estruturas abaixo são **PROPOSTAS DE MODELAGEM**, não contrato de implementação até aprovação da direção `Procedimento × Atendimento × Equipamento`.

## Categorias — proposta

Direção recomendada:

```text
process_categories
- category_id
- name
- normalized_name
- description NULL
- is_archived
- timestamps / usuário responsável
```

Se múltiplas categorias forem aprovadas:

```text
process_category_assignments
- process_id
- category_id
```

A alternativa categoria única/hierárquica ainda pode ser escolhida pelo PO antes da implementação. Não criar taxonomia complexa sem requisito.

## Equipamentos — proposta

Se a entidade reutilizável de equipamento for aprovada:

```text
equipment
- equipment_id
- equipment_code UNIQUE
- name
- equipment_type NULL
- client_or_owner_name NULL
- processor NULL
- ram_summary NULL
- storage_summary NULL
- os_name NULL
- os_version NULL
- serial_number NULL
- asset_tag NULL
- battery_health_percent NULL
- observations NULL
- is_archived
- row_revision
- timestamps / usuários responsáveis
```

Recomendação:

- `equipment_id` como identidade interna canônica;
- código legível operacional separado;
- serial/patrimônio/MAC como atributos de busca;
- campos não aplicáveis permanecem nulos.

Para múltiplos identificadores de rede, a proposta é estrutura equivalente a:

```text
equipment_network_identifiers
- network_identifier_id
- equipment_id
- kind
- normalized_value
- label NULL
```

MAC não deve ser a única chave de identidade.

## Atendimento / registro de serviço — proposta

Se a separação operacional for aprovada:

```text
service_records
- service_record_id
- service_code UNIQUE
- external_reference NULL
- client_or_requester_name NULL
- equipment_id NULL
- responsible_user_id
- status
- started_at NULL
- completed_at NULL
- summary NULL
- observations NULL
- row_revision
- timestamps / usuários responsáveis
```

Recomendações:

- equipamento opcional;
- OS/referência externa opcional;
- controle otimista por `row_revision` ou equivalente;
- lifecycle/status exatos somente após Bloco 9.

## Procedimentos utilizados no serviço — proposta

Para preservar histórico, recomenda-se vincular o registro de serviço à revisão exata do procedimento:

```text
service_record_processes
- service_record_process_id
- service_record_id
- process_id
- process_revision_id
- position NULL
- code_snapshot
- title_snapshot
- display_version_snapshot NULL
- added_at / added_by_user_id
```

O snapshot textual evita que relatório histórico mude apenas porque o procedimento oficial foi renomeado depois.

Permitir um ou múltiplos procedimentos por atendimento ainda aguarda aprovação explícita do PO.

## Busca operacional

Requisito confirmado: localizar registros pelas informações úteis do trabalho/equipamento.

Índices/normalização propostos devem suportar, quando os campos existirem:

- código do procedimento;
- título;
- categoria;
- código do equipamento/atendimento;
- OS/referência externa;
- nome do equipamento;
- cliente/solicitante;
- serial;
- patrimônio;
- MAC normalizado.

FTS5 permanece opcional somente se necessidade real justificar; não introduzir motor externo por antecipação.

## Ficha compacta imprimível

Requisito confirmado: gerar uma ficha/relatório compacto do serviço/equipamento.

A direção recomendada é derivar a ficha do estado confirmado de empresa + registro de serviço + equipamento + procedimentos utilizados + responsável, evitando tabela exclusiva apenas para apresentação.

Se o Bloco 10 provar necessidade de snapshot adicional para reprodução histórica, documentar antes da migration correspondente.

## Histórico e auditoria

Núcleo consolidado:

- snapshots em `process_revisions`;
- `audit_events` append-only;
- sem segredos em auditoria.

Se equipamentos/atendimentos forem aprovados como entidades, suas mudanças relevantes também deverão participar de histórico/auditoria proporcional.

## Arquivamento

Consolidado para entidades existentes: processos/usuários preferem arquivar/desativar.

Proposto para novas entidades:

- categorias: arquivar;
- equipamentos: arquivar quando houver histórico;
- registros de serviço históricos: não excluir destrutivamente por operação normal sem regra explícita.

## Migrations e rollback

Mantêm-se as regras consolidadas:

- `schema_migrations` numerada/versionada com checksum;
- migration publicada é imutável;
- Host aplica/verifica no startup;
- falha bloqueia readiness;
- correção exige nova migration;
- backup consistente antes de mudança incompatível;
- rollback de binário somente com schema compatível, caso contrário restaurar backup correspondente.

## Fora do schema inicial sem novo requisito

- CRM completo;
- faturamento/financeiro;
- estoque de peças;
- RMM/inventário automatizado;
- descoberta automática de hardware;
- chat/social;
- workflow complexo;
- notificações persistentes genéricas;
- fila distribuída;
- banco externo.

## Relação com concorrência

Processos mantêm `base_revision` + writer coordenado conforme arquitetura vigente.

Se categorias/equipamentos/atendimentos forem aprovados, seguirão o mesmo princípio de fila coordenada e revisão otimista quando houver risco de perda concorrente. Regras específicas de execução serão fechadas no Bloco 9.
