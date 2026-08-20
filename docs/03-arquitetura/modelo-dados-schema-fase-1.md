# Modelo de Dados, Schema, Migrations e Histórico — StepFlow

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL CONSOLIDADA PARA A FASE 1

## 1. Objetivo

Definir o modelo persistente inicial do StepFlow antes da implementação, preservando:

- SQLite acessado somente pelo Host;
- histórico confiável;
- revisão otimista;
- modelo de processo enxuto;
- migrations versionadas;
- dados separados de binários;
- capacidade de backup/restore;
- ausência de dados reais no repositório.

## 2. Princípios gerais

- SQLite é a fonte persistente principal do Host;
- Clients nunca abrem o arquivo `.sqlite` diretamente;
- foreign keys habilitadas;
- todas as alterações compostas ocorrem em transação;
- identificadores de negócio/histórico são estáveis;
- timestamps persistidos em UTC;
- exclusões relevantes preferem arquivamento/desativação para preservar histórico;
- arquivos binários grandes, como logo/avatar, ficam no filesystem administrado pelo Host; banco armazena somente metadados/referências;
- nenhum schema cria antecipadamente entidade de execução/checklist enquanto o Bloco 9 não decidir esse comportamento.

## 3. Identificadores

Direção inicial:

- entidades estáveis usam UUID textual gerado pelo Host;
- UUID não depende de nome, login ou código do processo;
- identificadores nunca são reciclados;
- tabelas internas auxiliares podem usar inteiro quando não houver benefício em UUID.

## 4. Timestamps

Persistir timestamps em UTC em formato canônico consistente, por exemplo RFC 3339.

Campos típicos:

```text
created_at
updated_at
published_at
archived_at
occurred_at
expires_at
```

Conversão para horário local ocorre na apresentação.

## 5. Usuários

Tabela conceitual `users`:

```text
user_id PK
login UNIQUE
password_hash
name
job_title
avatar_file
role_preset
is_active
is_primary_admin
created_at
updated_at
```

Regras:

- `user_id` imutável;
- `login` único por comparação normalizada definida pelo Host;
- senha somente como PHC Argon2id;
- desativação preserva referências históricas;
- pelo menos um ADM ativo deve existir após bootstrap.

Permissões granulares poderão usar tabela dedicada quando necessário, por exemplo `user_permissions` ou modelo equivalente. O schema final deve permitir presets + exceções sem duplicar toda a lógica de autorização em colunas booleanas de `users`.

## 6. Sessões

Tabela conceitual `sessions`:

```text
session_id PK
token_hash UNIQUE
user_id FK
created_at
last_seen_at
expires_at
absolute_expires_at
revoked_at
revocation_reason
```

Regras:

- token reutilizável não é persistido em texto puro;
- sessão revogada continua opcionalmente disponível até limpeza periódica para auditoria operacional;
- limpeza de sessões expiradas é manutenção interna, não funcionalidade de usuário.

## 7. Empresa/configuração

Tabela conceitual `company_settings` de uma única implantação:

```text
company_id PK
name
logo_file
created_at
updated_at
```

Configurações técnicas da implantação como bind/porta/path não precisam obrigatoriamente ficar nessa tabela; podem permanecer em `config/stepflow-host.toml` quando forem configuração operacional e não dado de negócio.

## 8. Identidade estável do processo

Tabela `processes` mantém a identidade do documento, não todo o conteúdo mutável.

Campos conceituais:

```text
process_id PK
code UNIQUE
current_revision_id FK
published_revision_id FK NULL
is_archived
created_at
created_by_user_id FK
archived_at NULL
archived_by_user_id FK NULL
```

Regras:

- `code` é único globalmente e não é reutilizado automaticamente após arquivamento;
- `process_id` nunca muda;
- conteúdo e metadados editáveis vivem em revisões imutáveis;
- `current_revision_id` aponta para o estado mais recente confirmado;
- `published_revision_id` permite distinguir revisão em trabalho de última publicação quando esse fluxo for necessário;
- `is_archived` representa remoção lógica do catálogo ativo.

## 9. Revisão técnica versus versão exibida

Esses conceitos são diferentes.

### `revision_no`

Número técnico monotônico por processo:

```text
1, 2, 3, 4...
```

Usado para:

- concorrência otimista;
- detecção de edição obsoleta;
- ordenação de histórico;
- referência técnica interna.

### `display_version`

Texto de versão exibido ao usuário, por exemplo:

```text
1.0
1.1
2.0
```

Pode seguir regra editorial aprovada e não precisa mudar em todo salvamento técnico.

O Host nunca usa `display_version` como mecanismo de concorrência.

## 10. Revisões imutáveis de processo

Tabela conceitual `process_revisions`:

```text
revision_id PK
process_id FK
revision_no
base_revision_no NULL
code_snapshot
title
area_department
responsible_user_id FK NULL
status
display_version
objective
observations
prerequisites
change_summary NULL
created_at
created_by_user_id FK
published_at NULL
published_by_user_id FK NULL
```

Constraint:

```text
UNIQUE(process_id, revision_no)
```

Regras:

- revisão confirmada nunca é editada in-place;
- salvar nova alteração cria nova revisão;
- criação da revisão + filhos + atualização de `processes.current_revision_id` ocorre em uma transação;
- `base_revision_no` registra a base informada pelo Client para fins de diagnóstico/histórico;
- estado anterior permanece consultável.

## 11. Etapas

Tabela conceitual `process_stages`:

```text
stage_id PK
revision_id FK
position
title
introduction NULL
```

Constraint:

```text
UNIQUE(revision_id, position)
```

Cada revisão possui seu próprio snapshot das etapas. Alterar uma etapa cria uma nova revisão do processo, não modifica retrospectivamente histórico publicado.

## 12. Conteúdo estruturado da etapa

Para evitar dezenas de tabelas frágeis para cada elemento visual, usar blocos ordenados com tipo explícito e payload validado pelo Host.

Tabela conceitual `stage_blocks`:

```text
block_id PK
stage_id FK
position
block_type
payload_json
```

Tipos iniciais possíveis, definidos por contrato e não livres:

```text
paragraph
numbered_steps
checklist
note
warning
command
code
```

Regras:

- `payload_json` precisa ser validado contra o tipo antes de persistir;
- não armazenar HTML arbitrário como fonte principal;
- UI renderiza modelo estruturado;
- tipos novos exigem migration/compatibilidade de contrato quando necessário;
- checklist aqui descreve a **definição documental**; não persiste marcação de execução do usuário.

## 13. Passos e subpassos

`numbered_steps` pode armazenar estrutura hierárquica validada no payload do bloco, com identificadores internos estáveis dentro daquela revisão quando necessário.

Não criar tabela separada para cada nível de subpasso sem necessidade comprovada.

## 14. Histórico de alterações

O histórico oficial combina duas fontes:

1. `process_revisions` — snapshot imutável do documento;
2. `audit_events` — quem executou ações relevantes e em qual contexto.

A UI de histórico pode apresentar:

- revisão técnica;
- versão exibida;
- autor;
- data;
- resumo de mudança;
- publicação/arquivamento;
- comparação futura entre revisões quando implementada.

Não duplicar snapshots completos dentro do audit log.

## 15. Auditoria

Tabela conceitual `audit_events`:

```text
audit_id PK
occurred_at
actor_user_id FK NULL
action
entity_type
entity_id
request_id NULL
summary_json NULL
```

Princípios:

- append-only pela aplicação;
- nenhuma senha/token reutilizável;
- nenhuma stack trace como dado de negócio;
- `actor_user_id` pode ser nulo para operação interna/sistema claramente identificada;
- `summary_json` contém somente contexto sanitizado necessário;
- auditoria não substitui logs técnicos.

## 16. Arquivos administrados

Estrutura conceitual persistente:

```text
StepFlow\data\
├── stepflow.sqlite
├── company\
│   └── <logo-gerenciado>
└── avatars\
    └── <arquivos-gerenciados>
```

Regras:

- nomes físicos não vêm diretamente do nome enviado pelo usuário;
- Host valida tipo/tamanho antes de aceitar;
- banco referencia caminho/nome relativo controlado;
- substituir avatar/logo deve tratar limpeza do arquivo anterior de forma segura;
- backup precisa considerar banco + arquivos administrados.

## 17. Exclusão e arquivamento

### Processos

A ação de “Excluir” no produto deve, por padrão, **arquivar** o processo e preservar revisões/histórico.

Restauração futura de arquivo pode ser permitida sem reutilizar identidade/código de forma ambígua.

### Usuários

Desativar, não apagar fisicamente quando houver referências.

### Auditoria/revisões

Não excluir por operações normais de usuário.

Políticas de retenção excepcional, se algum dia necessárias, exigem decisão explícita.

## 18. Concorrência na persistência

Ao salvar processo:

```text
Client envia base_revision = 41
Host lê current revision = 41
   ↓
transação
   ↓
cria revision 42 + filhos
   ↓
atualiza current_revision_id
   ↓
commit
```

Se `current revision != base_revision`, não cria revisão e retorna conflito.

A fila/serialização de escritas será detalhada no Bloco 7.

## 19. Migrations

Manter tabela interna `schema_migrations`:

```text
migration_id PK
name
applied_at
checksum
```

Arquivos de migration oficiais ficam versionados no repositório e são incorporados/empacotados com o Host.

Direção:

- migrations numeradas e imutáveis após publicação;
- Host verifica versão do schema na inicialização;
- aplica migrations pendentes em ordem;
- migration deve ser transacional sempre que o SQLite permitir;
- falha interrompe inicialização operacional e gera diagnóstico claro;
- nunca editar silenciosamente migration já aplicada; criar nova migration corretiva.

## 20. Atualização e rollback com migrations

Binário e banco têm ciclos distintos.

Antes de migration que possa impedir rollback simples:

- criar backup consistente;
- registrar versão de schema;
- aplicar migration;
- só considerar atualização ativa após sucesso.

Rollback de binário:

- se versão anterior suportar o schema atual, pode voltar apenas o binário;
- se não suportar, rollback exige restauração do backup correspondente;
- não implementar “down migration” automática destrutiva apenas por conveniência.

## 21. Índices iniciais

Criar apenas índices ligados a consultas reais previstas:

- `users.login` único;
- `processes.code` único;
- `process_revisions(process_id, revision_no)` único;
- `process_stages(revision_id, position)`;
- `stage_blocks(stage_id, position)`;
- sessões por token hash/usuário/expiração;
- auditoria por entidade/data e ator/data quando necessário.

Evitar indexação especulativa excessiva.

## 22. Busca de processos

A primeira versão pode começar com busca convencional por campos normalizados (`code`, `title`, `area_department`, responsável/status) adequada ao volume real.

FTS5 ou mecanismo equivalente só deve ser introduzido se a necessidade de busca textual ampla for confirmada e testada.

Não adicionar motor externo de busca.

## 23. Dados que NÃO entram no schema agora

Não criar ainda:

- execução formal de processo;
- marcações persistidas de checklist;
- comentários sociais/chat;
- anexos genéricos sem requisito;
- workflow de aprovação complexo;
- notificações persistentes;
- motor de busca externo;
- fila distribuída;
- banco externo.

## 24. Gate do Bloco 6

Bloco 6 arquiteturalmente fechado com:

1. identidade estável de processo;
2. revisões imutáveis;
3. `revision_no` técnico separado de `display_version`;
4. etapas e blocos estruturados por revisão;
5. checklist documental separado de estado de execução;
6. arquivamento/desativação em vez de exclusão destrutiva normal;
7. auditoria append-only separada dos snapshots;
8. migrations numeradas e versionadas;
9. backup obrigatório antes de migrations incompatíveis com rollback;
10. SQLite e arquivos persistentes exclusivamente sob coordenação do Host.

Próximo bloco: **Bloco 7 — Concorrência, fila de escrita, conflitos e atualização multiusuário**.
