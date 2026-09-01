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
| 5 | Autenticação/autorização | CONSOLIDADO, INCLUINDO D12.56–D12.62 | `../03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | NÚCLEO CONCEITUAL + DISCIPLINA D12.35–D12.55 CONSOLIDADOS | `../03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | NÚCLEO CONCLUÍDO | `../03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | CONCLUÍDO | `../02-telas/README.md` |
| 9 | Execução operacional/Atendimentos | CONCLUÍDO | `bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão + Ficha compacta | CONCLUÍDO | `bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração técnico | CONCLUÍDO | `bloco-11-backup-restauracao.md` |
| 12 | Estrutura oficial + Fase 2 | EM ANÁLISE — ANÁLISES 1–4 APROVADAS | `bloco-12-estrutura-oficial-plano-fase-2.md` |

## Bloco 12 — Estrutura oficial + Fase 2

**EM ANÁLISE desde 2026-09-01.**

### Análises 1–4 — aprovadas

Decisões vigentes: **D12.1–D12.79**.

Fechado:

- source tree modular `apps/` + `crates/`;
- `StepFlow.exe` como único ponto de entrada e `_internal/` técnico;
- Rust 1.98.0, Edition 2024, resolver 3, Windows x64 MSVC;
- toolchain/lockfile versionados e dependências lockfile-aware;
- Client vanilla modular sem Node/npm/Vite/bundler/framework;
- configuração build/dev × deployment × runtime e packaging separado de `target/`;
- migrations Host-side imutáveis/embutidas, runner/checksum e testes em SQLite temporário real;
- fixtures sintéticas e scripts finos `check/test/build/package.ps1`;
- Argon2id/senha/blocklist/throttling/token/sessão fechados;
- Gerência pode alterar configuração da empresa;
- limites de identidade/logo e categoria arquivada fechados;
- retenção/limites/espaço/timeouts de Backup/Restore, readiness/reconexão e rotação de logs fechados.

Fontes:

- `bloco-12-estrutura-oficial-plano-fase-2.md`;
- `bloco-12-analise-2-workspace-build-dependencias.md`;
- `bloco-12-analise-3-migrations-testes-fixtures.md`;
- `bloco-12-analise-4-parametros-finais.md`.

### Análise 5 — em revisão

Proposta **P12.80–P12.98** define a sequência da Fase 2:

```text
Gate Fase 1 + sync local seguro
→ F2-T01 workspace/tooling + Host mínimo
→ F2-T02 Host runtime/readiness
→ F2-T03 SQLite + migrations runner
→ F2-T04 Controller lifecycle
→ F2-T05 Client Tauri + compatibilidade
→ F2-T06 Launcher Pocket
→ F2-T07 packaging Pocket
→ F2-T08 smoke integrado + gates Windows/Pocket
→ Gate Fase 2
```

Fonte: `bloco-12-analise-5-plano-fase-2.md`.

## O Bloco 12 ainda deve fechar

1. decisão da Análise 5;
2. validação técnica/documental final da Fase 1;
3. gate Git do PR do Bloco 12;
4. sincronização segura do checkout local;
5. autorização explícita do primeiro scaffold/runtime da Fase 2.

## Pendências restantes da Fase 1

### Em decisão no Bloco 12

- P12.80–P12.98;
- validação final e gate do primeiro scaffold.

### Ambiente real

- Windows/WebView2 nas estações reais;
- PoC do fallback Pocket WebView2 quando necessária;
- Launcher pelo share corporativo;
- Word/impressoras;
- SMB real;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Windows e crash injection de Backup/Restore.

## Gate atual

1. concluir Análise 5;
2. realizar validação final da Fase 1;
3. sincronizar documentação estável;
4. cumprir gate Git do PR do Bloco 12;
5. sincronizar explicitamente o checkout local preservando alterações preexistentes do PO;
6. somente então criar o primeiro branch/prompt Codex de implementação da Fase 2.

## Regras finais

- não criar scaffold/runtime definitivo, migration oficial ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO;
- gates Git consumidos não permanecem como estado em documentos técnicos estáveis.
