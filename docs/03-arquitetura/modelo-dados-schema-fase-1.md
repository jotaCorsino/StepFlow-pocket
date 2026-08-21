# Modelo de Dados, Schema, Migrations e Histórico — StepFlow

**Status:** NÚCLEO CONSOLIDADO / EXTENSÃO OPERACIONAL CONSOLIDADA CONCEITUALMENTE  
**Atualização:** 2026-08-21

## Princípios

- SQLite acessado somente pelo Host;
- foreign keys habilitadas;
- transações em alterações compostas;
- timestamps em UTC;
- identificadores internos estáveis para entidades de negócio/histórico;
- códigos legíveis de operação separados da identidade interna;
- arquivos administrados ficam no filesystem do Host e o banco guarda referências/metadados;
- exclusão normal prefere arquivamento/desativação quando há histórico;
- migrations versionadas e imutáveis após publicação;
- revisões de procedimento imutáveis;
- lifecycle/checklist operacional permanece para o Bloco 9.

## Usuários, sessões e empresa

`users`, `sessions` e `company_settings` permanecem conforme os documentos vigentes de autenticação/empresa.

## Processos / procedimentos

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

`process_revisions` preserva snapshots imutáveis. `revision_no` técnico permanece separado de `display_version` editorial.

Cada revisão possui `process_stages` e `stage_blocks` ordenados. Tipos iniciais:

```text
paragraph
numbered_steps
checklist
note
warning
command
code
```

O checklist aqui representa a definição documental; estado operacional pertence ao Bloco 9.

## Categorias de procedimentos

Categorias são configuráveis e não hardcoded.

Conceitualmente:

```text
process_categories
- category_id
- name
- normalized_name
- description NULL
- is_archived
- created_at / created_by_user_id
- updated_at / updated_by_user_id
```

Como um procedimento pode possuir múltiplas categorias:

```text
process_category_assignments
- process_id
- category_id
```

Regras:

- evitar duplicação acidental de nomes equivalentes;
- arquivar categoria preserva histórico/relações existentes;
- não criar hierarquia complexa na primeira versão;
- categorias devem ser indexáveis para filtro/listagem.

## Equipamentos

Equipamento é entidade opcional e reutilizável entre atendimentos.

Modelo conceitual:

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
- created_at / created_by_user_id
- updated_at / updated_by_user_id
```

`equipment_id` é a identidade canônica. `equipment_code` é o identificador legível operacional; formato final será decidido no Bloco 9/implementação.

MAC não é chave canônica. Múltiplos identificadores de rede podem ser armazenados em estrutura própria:

```text
equipment_network_identifiers
- network_identifier_id
- equipment_id
- kind
- normalized_value
- label NULL
```

Campos não aplicáveis permanecem nulos.

## Atendimento / execução

O atendimento representa uma ocorrência real de trabalho.

Modelo conceitual:

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
- created_at / created_by_user_id
- updated_at / updated_by_user_id
```

Regras consolidadas:

- equipamento é opcional;
- OS/referência externa é opcional e não substitui identidade interna;
- mesmo equipamento pode participar de atendimentos diferentes;
- um atendimento pode usar múltiplos procedimentos;
- concorrência relevante usa `row_revision`/controle otimista equivalente;
- lifecycle/status exatos permanecem pendentes do Bloco 9.

## Procedimentos utilizados em atendimento

O vínculo preserva a revisão realmente utilizada:

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
- added_at
- added_by_user_id
```

O snapshot textual evita que relatório histórico mude apenas porque título/código do procedimento foi alterado depois.

Progresso por etapa, checklist marcado e regras de reabertura pertencem ao Bloco 9.

## Busca

Consultas devem ser atendidas inicialmente por campos normalizados/índices adequados, sem motor externo.

Procedimentos:

- código;
- título/termos compatíveis;
- área/departamento;
- categoria.

Equipamentos/atendimentos:

- `equipment_code`;
- `service_code`;
- OS/referência externa;
- nome do equipamento;
- cliente/solicitante/responsável;
- serial;
- patrimônio;
- MAC normalizado.

FTS5 só entra se a necessidade real de busca justificar.

## Ficha compacta imprimível

A ficha de atendimento/equipamento deve ser derivada do estado confirmado de:

- empresa;
- atendimento;
- equipamento associado;
- procedimentos/revisões utilizados;
- usuário responsável;
- observações/resumo.

Não criar tabela exclusiva apenas para apresentação. Se o Bloco 10 exigir snapshot adicional para reprodução histórica exata, documentar antes da migration correspondente.

## Histórico e auditoria

Histórico oficial combina:

1. snapshots em `process_revisions`;
2. referências/snapshots operacionais de atendimento;
3. eventos append-only em `audit_events`.

Ações relevantes de categoria/equipamento/atendimento devem ser auditáveis de forma proporcional. Nunca guardar segredos em auditoria.

## Arquivos persistentes

```text
data\
├── stepflow.sqlite
├── company\
└── avatars\
```

Backup inclui banco e arquivos administrados.

## Arquivamento/desativação

- processos: arquivar;
- categorias: arquivar;
- equipamentos: arquivar quando houver histórico;
- usuários: desativar;
- revisões/auditoria/atendimentos históricos: não excluir por operação normal sem regra explícita.

## Migrations

Usar migrations numeradas/versionadas e `schema_migrations` com identificador, nome, data e checksum.

- migration publicada é imutável;
- Host aplica pendências na inicialização;
- usar transação quando SQLite permitir;
- falha bloqueia readiness;
- correção exige nova migration.

## Rollback

Antes de migration que possa impedir rollback simples, criar backup consistente.

- binário anterior só retorna sem restore se suportar schema atual;
- caso contrário, restaurar backup correspondente;
- não usar down migration destrutiva automática por conveniência.

## Fora do schema inicial

- CRM completo;
- faturamento/financeiro;
- estoque de peças;
- RMM/inventário automatizado;
- descoberta automática de hardware;
- chat social;
- workflow complexo;
- fila distribuída;
- banco externo.

## Relação com concorrência

Processos continuam usando `base_revision`/revisão otimista. Categorias, equipamentos e atendimentos usam mecanismo equivalente quando houver risco de perda concorrente.

Detalhes gerais permanecem em `concorrencia-fila-conflitos-eventos.md`; regras específicas do atendimento serão fechadas no Bloco 9.