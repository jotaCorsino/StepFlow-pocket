# Modelo de Dados, Schema, Migrations e Histórico — StepFlow

**Status:** NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS CONCEITUALMENTE PARA A FASE 1  
**Atualização:** 2026-08-25

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
- nenhuma migration é criada durante esta fase documental.

## Usuários, sessões e empresa

`users`, `sessions` e `company_settings` seguem os contratos vigentes de autenticação, Tela 10/11/12 e configuração da empresa.

## Processos / Procedimentos

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

Checklist aqui é **definição documental**. Estado de execução fica separado no domínio do Atendimento.

## Categorias de Procedimentos

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

Relação múltipla:

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
- preset de gestão: ADM/Gerência; autorização real por capacidade.

Permanece pendente antes da implementação editorial a regra exata de nova revisão de Procedimento que ainda carregue categoria arquivada.

## Equipamentos

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

`equipment_id` é identidade canônica. `equipment_code` é referência legível gerada pelo Host no formato inicial:

```text
EQP-000001
```

- seis dígitos;
- gaps permitidos;
- sequência por implantação/banco ativo;
- não editável pelo Client.

Múltiplos identificadores de rede ficam separados:

```text
equipment_network_identifiers
- network_identifier_id
- equipment_id
- kind
- normalized_value
- label NULL
```

MAC não é chave canônica. Serial/patrimônio também não se tornam identidade exclusiva por inferência.

Campos não aplicáveis permanecem nulos.

## Atendimento / Execução

`service_records` representa ocorrência real de trabalho.

Modelo conceitual atualizado:

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

Status inicial consolidado:

```text
IN_PROGRESS   → Em andamento
COMPLETED     → Concluído
CANCELLED     → Cancelado
```

Os nomes físicos/enum internos podem variar na implementação, desde que a semântica permaneça estável.

Código inicial:

```text
AT-000001
```

- gerado somente no primeiro save aceito;
- seis dígitos;
- gaps permitidos;
- não editável;
- não substitui `service_record_id`.

## Lifecycle do Atendimento

```text
novo rascunho Client
→ primeiro save Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Regras de persistência:

- abrir tela não cria registro;
- `started_at` nasce no Host no primeiro save;
- `completed_at` representa a conclusão atualmente aplicável quando o registro está concluído;
- cancelamento preserva código e dados;
- reabertura não apaga eventos históricos anteriores;
- conclusão/cancelamento/reabertura incrementam/revalidam revisão do recurso;
- não há exclusão física normal de Atendimento.

## Histórico de lifecycle

Além do estado atual, lifecycle relevante precisa ficar auditável.

Pode ser representado por `audit_events` e/ou estrutura operacional dedicada, desde que preserve pelo menos:

- criação;
- conclusão;
- reabertura;
- cancelamento;
- motivo de cancelamento;
- mudanças relevantes de responsável;
- vínculos relevantes de Equipamento/Procedimento.

Não é requisito criar uma timeline persistente de cada alteração trivial de campo ou checkbox.

## Procedimentos utilizados em Atendimento

O vínculo preserva a revisão exata:

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
- Funcionário seleciona normalmente revisão publicada;
- ADM/Gerência podem selecionar explicitamente outra revisão que já possam ler;
- revisão histórica/não publicada nunca é escolhida silenciosamente;
- remoção de vínculo só ocorre em Atendimento editável e é auditável.

## Checklist operacional persistente

O estado operacional é separado da revisão documental.

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

A implementação pode ajustar nomes/normalização, mas deve preservar:

- vínculo com a revisão/origem imutável;
- estado marcado/desmarcado;
- usuário/data quando aplicável;
- snapshot textual quando necessário para integridade histórica;
- controle concorrente por item ou equivalente.

Checklist não pertence ao `process_revision` como estado mutável.

## Progresso operacional

Progresso é derivado, não armazenado como percentual arbitrário:

```text
checked_count / total_checklist_items
```

- etapas visitadas não contam;
- revisão sem checklist não gera `0%` artificial;
- 100% não conclui Atendimento automaticamente.

Se cache de contagem for usado futuramente por desempenho, continua derivável do estado oficial.

## Concorrência do checklist

Checklist não deve depender da revisão global do Atendimento para cada clique.

Direção consolidada:

- item possui `row_revision` próprio ou controle atômico equivalente;
- usuários marcando itens diferentes não devem conflitar globalmente;
- dois usuários alterando o mesmo item recebem resultado determinístico/conflito apropriado;
- eventos só são emitidos pós-commit.

## Snapshot histórico do Equipamento na conclusão

Mudanças posteriores do cadastro global não podem reescrever Atendimento concluído/ficha final.

A conclusão precisa congelar a projeção de Equipamento relevante ao estado final. Conceitualmente, o snapshot deve poder preservar quando aplicável:

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

A persistência física pode ser:

- snapshot estruturado ligado ao evento/conclusão;
- campos snapshot normalizados necessários;
- outra forma versionada/documentada.

Não criar tabela meramente para apresentação. O requisito é **reprodução histórica determinística**, não uma forma física específica.

Cada nova conclusão após reabertura gera novo estado final aplicável sem apagar conclusões anteriores da auditoria/histórico.

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
- título/termos compatíveis;
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

FTS5 só entra se a necessidade real justificar.

## Ficha compacta

A ficha deriva do estado confirmado de:

- empresa;
- Atendimento;
- Equipamento ou snapshot histórico aplicável;
- Procedimentos/revisões utilizados;
- responsável;
- checklist apenas quando/como o template futuro decidir exibir;
- resumo/observações.

Não criar tabela exclusiva apenas para apresentação.

Bloco 10 ainda decidirá template físico, limites textuais, preview, impressão e PDF específico.

## Histórico e auditoria

Histórico oficial combina:

1. snapshots imutáveis de Procedimentos;
2. vínculos/snapshots operacionais do Atendimento;
3. checklist operacional persistente;
4. projeções finais de Equipamento quando concluído;
5. eventos append-only em `audit_events`.

Nunca guardar senha, token reutilizável ou segredo em auditoria.

## Arquivos persistentes

```text
data\
├── stepflow.sqlite
├── company\
└── avatars\
```

Backup inclui banco e arquivos administrados.

## Arquivamento/desativação

- Processos: arquivar;
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
- correção exige nova migration.

Antes de migration incompatível, backup consistente.

## Rollback

- binário anterior só volta sem Restore quando suporta schema atual;
- caso contrário, restaurar backup correspondente;
- não usar down migration destrutiva automática por conveniência.

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

## Pendências antes da implementação correspondente

- forma física final do snapshot de Equipamento/conclusão;
- nomes físicos finais de tabelas/colunas de checklist/lifecycle;
- limites numéricos dos textos destinados à ficha, no Bloco 10;
- regra editorial de nova revisão com categoria arquivada;
- migrations oficiais, somente após o gate de estrutura da Fase 1.

Essas pendências não reabrem as regras funcionais já consolidadas no Bloco 9.