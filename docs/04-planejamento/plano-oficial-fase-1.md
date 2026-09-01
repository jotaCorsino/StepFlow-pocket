# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-09-01

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e PoCs descartáveis quando necessárias. Não autoriza scaffold/runtime oficial, migrations oficiais ou código de negócio definitivo antes do gate do Bloco 12/Fase 2.

## Estado dos blocos

| Bloco | Tema | Estado | Fonte principal |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório |
| 1 | Client Windows/Tauri | CONCLUÍDO | `../03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `../03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `../03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `../03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | NÚCLEO CONCLUÍDO / PARÂMETROS FINAIS PENDENTES | `../03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS | `../03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | NÚCLEO CONCLUÍDO | `../03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | CONCLUÍDO | `../02-telas/README.md` |
| 9 | Execução operacional/Atendimentos | CONCLUÍDO | `bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão + Ficha compacta | CONCLUÍDO | `bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração técnico | CONCLUÍDO | `bloco-11-backup-restauracao.md` |
| 12 | Estrutura oficial + Fase 2 | EM ANÁLISE — ANÁLISES 1–2 APROVADAS | `bloco-12-estrutura-oficial-plano-fase-2.md` |

## Bloco 12 — Estrutura oficial + Fase 2

**EM ANÁLISE desde 2026-09-01.**

### Análises 1–2 — aprovadas

Decisões vigentes: **D12.1–D12.34**.

Fechado até aqui:

- workspace Rust modular na raiz;
- `apps/` para Client, Launcher, Controller e Host;
- frontend Client modular em ES modules;
- crates compartilhados somente para responsabilidades concretas;
- source tree separado da publicação Pocket;
- `StepFlow.exe` na raiz publicada como único ponto de entrada normal;
- artefatos Client/Server encapsulados sob `_internal/`;
- Rust 1.98.0, Edition 2024, resolver 3 e target Windows x64 MSVC;
- `rust-toolchain.toml` e `Cargo.lock` versionados;
- política lockfile-aware para build/test/package e atualização explícita de dependências;
- Client vanilla sem Node/npm/Vite/bundler/framework no baseline;
- configuração separada entre build/dev, deployment e runtime central;
- `target/`/`dist/` descartáveis e produção montada por packaging;
- scripts PowerShell finos; sem LTO/strip/tuning prematuro.

### Análise 3 — em revisão

Proposta **P12.35–P12.55** cobre:

- migrations Host-side em `apps/host/migrations/` com IDs imutáveis;
- registry compilado e SQL embutido no Host;
- `schema_migrations` com checksum SHA-256;
- bloqueio por checksum/schema incompatível;
- `pre_migration` backup único antes de lote pendente em banco existente;
- lote de migrations transacional no baseline;
- `quick_check` + `foreign_key_check` antes de readiness;
- sem down migration automática ou `writable_schema` como atalho;
- testes de migration em SQLite temporário real;
- fixtures sintéticas e sem dados/segredos reais;
- scripts iniciais `check/test/build/package`;
- baseline sem adicionar Node apenas por test runner frontend.

Fontes:

- `bloco-12-estrutura-oficial-plano-fase-2.md`;
- `bloco-12-analise-2-workspace-build-dependencias.md`;
- `bloco-12-analise-3-migrations-testes-fixtures.md`.

## O Bloco 12 ainda deve fechar

1. decisão da Análise 3;
2. parâmetros finais ainda pendentes;
3. plano detalhado da Fase 2 e sequência de tarefas Codex;
4. sincronização segura do checkout local;
5. validação final da Fase 1 e autorização explícita do primeiro scaffold.

## Pendências restantes da Fase 1

### Segurança/configuração

- custo Argon2id final;
- senha mínima final;
- duração/expiração de sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- regra editorial de categoria arquivada.

### Parâmetros técnicos para Bloco 12

- `retention_max_confirmed_backups`;
- limites de pacote/entradas/path;
- margem mínima de espaço;
- timeouts;
- duração alvo de barrier/manutenção;
- backoff/reconexão;
- rotação física de logs/admin audit;
- parâmetros de dependências/adapters não cobertos pela fundação inicial.

### Ambiente real

- Windows/WebView2 nas estações reais;
- PoC do fallback Pocket WebView2;
- Launcher pelo share corporativo;
- Word/impressoras;
- SMB real;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Windows e crash injection de Backup/Restore.

## Gate atual

1. concluir Análises do Bloco 12;
2. aprovar parâmetros/estrutura/plano;
3. sincronizar documentação estável;
4. realizar validação final da Fase 1;
5. cumprir gate Git do PR do Bloco 12;
6. sincronizar explicitamente o checkout local preservando alterações preexistentes do PO;
7. somente então iniciar o primeiro scaffold/runtime oficial da Fase 2.

## Regras finais

- não criar scaffold/runtime definitivo, migration oficial ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO;
- gates Git consumidos não permanecem como estado em documentos técnicos estáveis.
