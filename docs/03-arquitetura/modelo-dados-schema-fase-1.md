# Modelo de Dados, Schema, Migrations e Histórico — StepFlow Pocket

**Status:** NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS CONCEITUALMENTE PARA A FASE 1  
**Atualização:** 2026-08-29

Este documento define **semântica e invariantes conceituais**. Nomes físicos finais de tabelas/colunas e migrations oficiais serão fechados no gate de estrutura/Fase 2.

## Princípios

- SQLite acessado somente pelo Host;
- foreign keys habilitadas;
- transações em alterações compostas;
- timestamps em UTC;
- identificadores internos estáveis;
- códigos legíveis separados da identidade interna;
- arquivos administrados ficam no filesystem do Host e o banco guarda referências/metadados;
- exclusão normal prefere arquivamento/desativação quando existe histórico;
- migrations versionadas e imutáveis após publicação;
- revisões de Procedimento são imutáveis;
- concorrência relevante usa revisão otimista/controle equivalente;
- nenhuma migration oficial é criada durante esta fase documental.

## Usuários, sessões e empresa

`users`, `sessions` e `company_settings` seguem os contratos vigentes de autenticação, Telas 10–12 e configuração da empresa.

Parâmetros finais de segurança permanecem pendentes e não devem ser inferidos pelo schema.

## Procedimentos e revisões

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

Checklist aqui é definição documental. Estado de execução e observações feitas durante o serviço ficam separados no domínio do Atendimento.

## Categorias

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

Relação múltipla conceitual:

```text
process_category_assignments
- process_id
- category_id
```

Regras:

- evitar nomes normalizados equivalentes;
- arquivamento preserva histórico/relações;
- categoria arquivada não é opção normal para nova associação;
- sem hierarquia complexa inicialmente;
- índices adequados para filtro/listagem;
- gestão por ADM/Gerência; autorização real permanece Host-side.

Pendente: regra editorial de nova revisão ainda carregando categoria arquivada.

## Equipamento

Equipamento é opcional e reutilizável entre Atendimentos.

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

`equipment_id` é identidade canônica. `equipment_code` usa formato inicial `EQP-000001`:

- gerado somente pelo Host;
- seis dígitos;
- sequência numérica simples por implantação/banco ativo;
- gaps permitidos;
- não editável pelo Client;
- não substitui `equipment_id`.

Múltiplos identificadores de rede ficam separados conceitualmente:

```text
equipment_network_identifiers
- network_identifier_id
- equipment_id
- kind
- normalized_value
- label NULL
```

MAC, serial e patrimônio não são identidade canônica exclusiva por inferência. Campos não aplicáveis permanecem nulos.

## Atendimento / Execução

`service_records` representa ocorrência real de trabalho.

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
- started_at
- completed_at NULL
- cancellation_reason NULL
- summary NULL
- observations NULL
- row_revision
- created_at / created_by_user_id
- updated_at / updated_by_user_id
```

Status:

```text
IN_PROGRESS   → Em andamento
COMPLETED     → Concluído
CANCELLED     → Cancelado
```

Nomes físicos de enum podem variar desde que a semântica permaneça estável.

`service_code` usa formato inicial `AT-000001`:

- gerado somente no primeiro save aceito;
- seis dígitos;
- sequência numérica simples por implantação/banco ativo;
- gaps permitidos;
- não editável;
- não reutilizado após cancelamento;
- não substitui `service_record_id`.

## Lifecycle do Atendimento

```text
rascunho Client
→ primeiro save Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Regras:

- abrir tela não cria registro;
- `started_at` nasce no Host no primeiro save;
- `completed_at` representa a conclusão atualmente aplicável quando o registro está concluído;
- cancelamento preserva código e dados;
- reabertura não apaga eventos históricos anteriores;
- conclusão/cancelamento/reabertura são mutações versionadas;
- não há exclusão física normal de Atendimento.

## Histórico de lifecycle e auditoria

Histórico relevante pode ser representado por `audit_events` e/ou estrutura operacional dedicada, desde que preserve pelo menos:

- criação;
- conclusão;
- reabertura;
- cancelamento + motivo;
- mudanças relevantes de responsável;
- vínculos relevantes de Equipamento/Procedimento;
- operações administrativas críticas quando aplicável.

Não é requisito persistir timeline de cada alteração trivial, campo ou checkbox.

Histórico oficial combina, conforme aplicabilidade:

1. snapshots imutáveis de Procedimentos;
2. vínculos/snapshots operacionais do Atendimento;
3. checklist operacional persistente;
4. observações de serviço por Etapa;
5. projeções finais de Equipamento por conclusão;
6. eventos append-only/auditoria proporcional.

Nunca guardar senha, token reutilizável ou segredo em auditoria.

## Procedimentos utilizados no Atendimento

Vínculo conceitual:

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

Regras:

- publicação futura não altera vínculo;
- Funcionário usa normalmente revisão publicada;
- ADM/Gerência podem selecionar explicitamente outra revisão que já possam ler;
- revisão histórica/não publicada nunca é escolhida silenciosamente;
- remoção só ocorre em Atendimento editável e preserva auditoria necessária.

## Checklist operacional persistente

Modelo conceitual possível:

```text
service_record_checklist_items
- execution_checklist_item_id
- service_record_process_id
- source_stage_id / source_block_id / source_item_key
- text_snapshot NULL
- is_checked
- checked_at NULL
- checked_by_user_id NULL
- row_revision
```

A implementação física deve preservar:

- vínculo à revisão/origem imutável;
- estado marcado/desmarcado;
- usuário/data quando aplicável;
- snapshot textual quando necessário à integridade histórica;
- controle concorrente por item/equivalente.

Checklist não pertence ao `process_revision` como estado mutável.

## Observações de serviço por Etapa

Modelo conceitual possível:

```text
service_record_stage_notes
- service_record_stage_note_id
- service_record_process_id
- source_stage_id
- stage_title_snapshot NULL
- note_text
- row_revision
- created_at / created_by_user_id
- updated_at / updated_by_user_id
```

Deve preservar:

- vínculo inequívoco ao Atendimento, `service_record_process` e Etapa da revisão exata;
- texto separado do Procedimento oficial;
- ausência de ruído persistente quando não houver texto relevante, conforme forma física escolhida;
- autoria/timestamps quando necessários;
- controle concorrente por Etapa/equivalente;
- edição somente quando Atendimento estiver editável/autorizado;
- somente leitura em `Concluído`/`Cancelado` até reabertura;
- nenhum autosave implícito;
- reprodução histórica do estado final aplicável.

A observação não é comentário social, chat, bloco documental ou item de checklist.

## Progresso operacional

Derivado, não armazenado como percentual arbitrário:

```text
checked_count / total_checklist_items
```

- Etapas visitadas não contam;
- observações não contam;
- revisão sem checklist não gera `0%` artificial;
- 100% não conclui Atendimento automaticamente.

Se cache de contagem for usado por desempenho, continua derivável do estado oficial.

## Concorrência granular

- Atendimento possui revisão otimista própria;
- Equipamento possui `row_revision`/equivalente;
- checklist possui revisão/controle por item;
- observação possui revisão/controle por Etapa;
- recursos independentes não devem conflitar globalmente;
- alteração concorrente do mesmo recurso recebe resultado determinístico/conflito apropriado;
- eventos só são emitidos pós-commit;
- evento remoto não sobrescreve edição local silenciosamente.

## Snapshot histórico do Equipamento

Mudanças posteriores no cadastro global não podem reescrever Atendimento concluído/Ficha histórica.

Ao concluir, preservar quando aplicável a projeção de:

```text
equipment_id
 equipment_code
 name
 equipment_type
 client_or_owner_name
 processor
 ram_summary
 storage_summary
 os_name / os_version
 serial_number
 asset_tag
 MACs relevantes
 battery_health_percent
 observations
```

A forma física pode ser snapshot estruturado, campos normalizados ou mecanismo versionado equivalente. O requisito é reprodução histórica determinística, não tabela de apresentação.

Cada nova conclusão após reabertura produz novo estado final aplicável sem apagar os anteriores da auditoria/histórico.

## Equipamento arquivado

- não aparece para novo vínculo normal;
- não pode ser arquivado enquanto vinculado a Atendimento `Em andamento`;
- histórico permanece;
- reativação depende de capacidade;
- arquivar não invalida snapshots de Atendimentos concluídos.

## Busca

Inicialmente por campos normalizados/índices SQLite, sem engine externo.

Procedimentos:

- código;
- título/termos;
- área/departamento;
- categoria.

Atendimentos/Equipamentos:

- `service_code`;
- `equipment_code`;
- OS/referência;
- cliente/solicitante/responsável;
- nome do Equipamento;
- serial;
- patrimônio;
- MAC normalizado;
- status;
- `started_at` para período/ordenação.

FTS5 só entra se necessidade real justificar.

## Ficha compacta

A Ficha é projeção resumida derivada do estado confirmado/histórico aplicável de Atendimento, Equipamento e observações.

- não criar tabela exclusiva apenas para apresentação;
- vínculos/revisões, checklist, progresso e auditoria permanecem fontes internas, mas não conteúdo padrão impresso;
- PDF e preview derivam do mesmo `PagedDocument`;
- resultado válido exige exatamente uma A4;
- soft limits 600/400/300/280 orientam densidade, não storage;
- `SHEET_OVERFLOW` não altera/trunca dado operacional.

## Arquivos persistentes

```text
data\
├── stepflow.sqlite
├── company\
└── avatars\
```

Backup inclui banco e arquivos administrados conforme contrato técnico final do Bloco 11.

## Arquivamento/desativação

- Procedimentos: arquivar;
- categorias: arquivar;
- Equipamentos: arquivar quando permitido;
- usuários: desativar;
- revisões/auditoria/Atendimentos históricos: não excluir por operação normal.

## Migrations

Usar migrations numeradas/versionadas e `schema_migrations` com identificador, nome, data e checksum.

- migration publicada é imutável;
- Host aplica pendências na inicialização;
- usar transação quando SQLite permitir;
- falha bloqueia readiness;
- correção exige nova migration;
- antes de migration incompatível, exigir backup consistente conforme política vigente.

## Rollback

- binário anterior só volta sem Restore quando suporta schema atual;
- caso contrário, restaurar backup correspondente;
- não usar down migration destrutiva automática por conveniência.

## Pendências antes da implementação física

- forma física final do snapshot de Equipamento/conclusão;
- forma física final para preservar observações em conclusões históricas após reabertura;
- nomes físicos finais de tabelas/colunas de checklist/observações/lifecycle;
- regra editorial de nova revisão com categoria arquivada;
- migrations oficiais, somente após gate de estrutura;
- integração precisa do pacote de Backup/Restore após Bloco 11.

Essas pendências não reabrem regras funcionais já consolidadas.

## Fora do schema inicial

- CRM completo;
- faturamento/financeiro;
- estoque;
- RMM/inventário automatizado;
- descoberta automática de hardware;
- chat social;
- workflow complexo/SLA;
- fila distribuída;
- banco externo.
