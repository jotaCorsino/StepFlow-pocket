# Modelo de Dados, Schema, Migrations e Histórico — StepFlow

**Status:** NÚCLEO CONSOLIDADO / EXTENSÃO DE CATEGORIAS, EQUIPAMENTOS E ATENDIMENTOS INCORPORADA  
**Atualização:** 2026-08-21

## Princípios

- SQLite é acessado somente pelo Host;
- foreign keys habilitadas;
- transações em alterações compostas;
- timestamps persistidos em UTC;
- identificadores estáveis, preferencialmente UUID textual para entidades de negócio/histórico;
- códigos legíveis de operação são separados da identidade interna;
- arquivos como logo/avatar ficam no filesystem administrado pelo Host; o banco guarda referência/metadados;
- exclusão normal prefere arquivamento/desativação quando há histórico;
- detalhes finais do lifecycle de atendimento/checklist são fechados no Bloco 9, sem impedir a definição conceitual das entidades agora necessárias.

## Usuários e sessões

`users` contém identidade estável, login único normalizado, `password_hash`, nome, cargo, avatar, preset, estado e timestamps.

`sessions` contém `session_id`, `token_hash`, `user_id`, criação/atividade/expiração e revogação. Token reutilizável não é persistido em texto puro.

## Empresa

`company_settings` contém identidade institucional e referência controlada ao logo. Bind/porta/paths operacionais permanecem em configuração da implantação, não como dado de negócio.

## Categorias de procedimentos

Categorias são dados configuráveis, não enumeração hardcoded.

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

Um procedimento pode pertencer a uma ou mais categorias por relação muitos-para-muitos.

```text
process_category_assignments
- process_id
- category_id
```

Regras:

- nome normalizado deve evitar duplicação acidental equivalente;
- arquivar categoria preserva histórico/relações existentes;
- não criar hierarquia complexa no schema inicial sem requisito adicional;
- categorias devem ser indexáveis para filtro/listagem.

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

O conteúdo mutável fica em revisões imutáveis.

A categorização operacional atual pode ser mantida em relação estável ao processo; quando necessário para fidelidade histórica/exportação, o snapshot de revisão deve preservar os nomes/IDs de categoria exibidos naquele momento ou referência equivalente que não reescreva o passado silenciosamente.

## Revisão técnica x versão exibida

- `revision_no`: inteiro monotônico por processo, usado para concorrência e histórico técnico;
- `display_version`: versão editorial apresentada ao usuário (`1.0`, `1.1`, etc.).

O Host nunca usa `display_version` para controle de concorrência.

## Revisões imutáveis

`process_revisions` registra snapshot de metadados/conteúdo do processo, autor, datas, `revision_no`, `base_revision_no`, `display_version` e resumo de alteração.

Regra:

- revisão confirmada nunca é editada in-place;
- salvar alteração cria nova revisão + filhos em uma única transação;
- `processes.current_revision_id` só muda junto do commit da nova revisão.

## Etapas e blocos

Cada revisão possui `process_stages` ordenadas. Cada etapa possui `stage_blocks` ordenados com `block_type` e `payload_json` validado pelo Host.

Tipos iniciais:

```text
paragraph
numbered_steps
checklist
note
warning
command
code
```

Não armazenar HTML arbitrário como fonte principal. O checklist aqui representa a definição documental; o estado operacional das marcações pertence ao Bloco 9.

## Equipamentos

Equipamento é entidade opcional e reutilizável entre atendimentos.

Identidade conceitual:

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

`equipment_id` é a identidade canônica. `equipment_code` é o identificador legível operacional; formato final será decidido antes da implementação.

MAC não é chave canônica. Como um equipamento pode possuir mais de uma interface, armazenar identificadores de rede em estrutura própria ou lista normalizada equivalente:

```text
equipment_network_identifiers
- network_identifier_id
- equipment_id
- kind
- normalized_value
- label NULL
```

Inicialmente `kind` pode representar MAC; a modelagem não deve impedir múltiplos valores por equipamento.

Campos físicos que não se aplicam permanecem nulos; não criar obrigatoriedade artificial para procedimentos sem computador.

## Atendimento / execução

O novo requisito de produto torna necessária uma entidade formal de ocorrência operacional.

Conceitualmente:

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
- ordem de serviço/referência externa é opcional e não substitui identidade interna;
- um atendimento pode usar múltiplos procedimentos;
- lifecycle/status exatos são pendência do Bloco 9;
- concorrência relevante usa `row_revision`/controle otimista equivalente;
- atendimento concluído deve manter histórico consistente.

## Procedimentos utilizados em atendimento

O vínculo deve preservar qual revisão foi efetivamente usada:

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

O snapshot textual evita que um relatório histórico mude apenas porque o título/código exibido do procedimento foi alterado no futuro.

A decisão sobre progresso por etapa, checklist marcado, timestamps por ação e reabertura pertence ao Bloco 9.

## Busca operacional

Índices/normalização devem suportar consultas reais sem motor externo.

Procedimentos:

- código;
- título/campos normalizados;
- categoria.

Equipamentos/atendimentos:

- `equipment_code`;
- `service_code`;
- `external_reference`/ordem de serviço;
- nome do equipamento;
- cliente/solicitante/responsável;
- serial;
- patrimônio;
- MAC normalizado.

FTS5 continua opcional somente se necessidade real justificar; não introduzir serviço externo de busca.

## Ficha compacta imprimível

A ficha/etiqueta de atendimento não exige uma tabela exclusiva apenas para apresentação. Ela deve ser derivada do estado confirmado de:

- empresa;
- atendimento;
- equipamento associado;
- procedimentos/revisões utilizados;
- usuário responsável;
- observações/resumo.

Se o Bloco 10 exigir snapshot adicional para garantir reprodução histórica exata, essa necessidade deverá ser documentada antes da migration correspondente.

## Histórico e auditoria

Histórico oficial combina:

1. snapshots em `process_revisions`;
2. referências/snapshots operacionais dos atendimentos;
3. eventos append-only em `audit_events`.

Auditoria registra ator, ação, entidade, data, `request_id` e contexto sanitizado quando necessário; não duplica snapshots completos nem guarda segredos.

Ações relevantes de equipamento/atendimento também devem ser auditáveis conforme a política final.

## Arquivos persistentes

```text
data\
├── stepflow.sqlite
├── company\
└── avatars\
```

Nomes físicos são gerenciados pelo Host; upload é validado por tipo/tamanho; backup inclui banco e arquivos administrados.

## Arquivamento/desativação

- processos: arquivar por padrão;
- categorias: arquivar por padrão;
- equipamentos: arquivar por padrão quando houver histórico;
- usuários: desativar por padrão;
- revisões/auditoria/atendimentos históricos: não excluir por operação normal sem regra explícita.

## Migrations

Usar migrations numeradas/versionadas e tabela `schema_migrations` com identificador, nome, data e checksum.

- migration publicada é imutável;
- Host verifica/aplica pendências na inicialização;
- usar transação quando SQLite permitir;
- falha bloqueia readiness e gera diagnóstico;
- correção exige nova migration, não edição silenciosa da antiga.

## Rollback e schema

Antes de migration que possa impedir rollback simples, criar backup consistente.

- se binário anterior suporta schema atual, rollback pode trocar apenas binário;
- se não suporta, restaurar backup correspondente;
- não usar down migration destrutiva automática por conveniência.

## Fora do schema inicial

Continuam fora por ausência de requisito:

- CRM completo de clientes;
- faturamento/financeiro;
- estoque de peças;
- RMM/inventário automatizado;
- descoberta automática de hardware pela rede;
- chat/comentários sociais;
- workflow complexo;
- notificações persistentes genéricas;
- fila distribuída;
- banco externo.

## Relação com concorrência

Mutações de processo recebem `base_revision`; o writer do Host compara com a revisão atual antes de criar a próxima revisão.

Equipamentos, categorias e atendimentos devem adotar revisão otimista equivalente quando houver risco de perda concorrente. Detalhes gerais permanecem em `concorrencia-fila-conflitos-eventos.md` e regras específicas de execução serão fechadas no Bloco 9.
