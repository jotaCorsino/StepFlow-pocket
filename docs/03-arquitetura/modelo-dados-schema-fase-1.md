# Modelo de Dados, Schema, Migrations e Histórico — StepFlow

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-20

## Princípios

- SQLite é acessado somente pelo Host;
- foreign keys habilitadas;
- transações em alterações compostas;
- timestamps persistidos em UTC;
- identificadores estáveis, preferencialmente UUID textual para entidades de negócio/histórico;
- arquivos como logo/avatar ficam no filesystem administrado pelo Host; o banco guarda referência/metadados;
- exclusão normal prefere arquivamento/desativação quando há histórico;
- nenhuma entidade de execução/checklist persistido é criada antes da decisão do Bloco 9.

## Usuários e sessões

`users` contém identidade estável, login único normalizado, `password_hash`, nome, cargo, avatar, preset, estado e timestamps.

`sessions` contém `session_id`, `token_hash`, `user_id`, criação/atividade/expiração e revogação. Token reutilizável não é persistido em texto puro.

## Empresa

`company_settings` contém identidade institucional e referência controlada ao logo. Bind/porta/paths operacionais permanecem em configuração da implantação, não como dado de negócio.

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

Não armazenar HTML arbitrário como fonte principal. O checklist aqui representa somente a definição documental.

## Histórico e auditoria

Histórico oficial combina:

1. snapshots em `process_revisions`;
2. eventos append-only em `audit_events`.

Auditoria registra ator, ação, entidade, data, `request_id` e contexto sanitizado quando necessário; não duplica snapshots completos nem guarda segredos.

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
- usuários: desativar por padrão;
- revisões/auditoria: não excluir por operação normal.

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

## Índices e busca

Criar apenas índices ligados a consultas reais: login, código do processo, revisão, ordenação de etapas/blocos, sessões e auditoria.

Busca inicial por campos normalizados; FTS5 somente se necessidade real justificar. Não introduzir motor externo de busca.

## Fora do schema inicial

- execução formal de processo;
- marcações persistidas de checklist;
- chat/comentários sociais;
- anexos genéricos sem requisito;
- workflow complexo;
- notificações persistentes;
- fila distribuída;
- banco externo.

## Relação com concorrência

Mutações de processo recebem `base_revision`; o writer do Host compara com a revisão atual antes de criar a próxima revisão. Detalhes em `concorrencia-fila-conflitos-eventos.md`.
