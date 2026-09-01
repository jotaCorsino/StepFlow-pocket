# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO — BLOCO 12 EM ANÁLISE  
**Início:** 2026-08-19  
**Atualização:** 2026-09-01

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e PoCs descartáveis quando necessárias. Não autoriza scaffold/runtime oficial, migrations oficiais ou código de negócio definitivo antes do gate final do Bloco 12/Fase 2.

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
| 12 | Estrutura oficial + Fase 2 | EM ANÁLISE | `bloco-12-estrutura-oficial-plano-fase-2.md` |

## Contrato Pocket da Fase 1

```text
pasta pronta no servidor Windows
→ usuário acessa o compartilhamento
→ executa StepFlowLauncher.exe
→ Launcher prepara versão local validada
→ Client abre localmente
```

Não é aceitável como baseline:

- instalar StepFlow em cada estação;
- exigir MSI/MSIX/NSIS;
- exigir Rust/Node/npm/Cargo/Office/LibreOffice/Adobe em produção;
- exigir Internet para uso normal;
- exigir elevação administrativa;
- exigir configuração manual de dependência;
- executar permanentemente o Client pelo SMB.

WebView2 Evergreen existente é preferível quando compatível. Fixed Version não roda por UNC/SMB; fallback local só entra após PoC provar preparação automática sem instalação/elevação/manualidade.

## Blocos 8–11 — fechados

- **Bloco 8:** Telas 01–15 e experiência Reader/manual consolidadas;
- **Bloco 9:** Atendimento/Execução, checklist, observações, Equipamento e histórico consolidados;
- **Bloco 10:** PDF/DOCX/impressão/Ficha/naming/temporários consolidados;
- **Bloco 11:** Backup/Restore, recovery, capacidades, auditoria e validação Windows consolidados em D11.1–D11.116.

Os gates Git desses blocos foram consumidos; não permanecem como pendência da Fase 1.

## Bloco 12 — Estrutura oficial + plano da Fase 2

**EM ANÁLISE desde 2026-09-01.**

Fonte: `bloco-12-estrutura-oficial-plano-fase-2.md`.

O bloco fecha o último gate da Fase 1. Ele deve produzir decisões implementáveis, mas não antecipar implementação funcional no próprio PR documental.

### Trilhas obrigatórias

1. **Árvore oficial e ownership**
   - source tree;
   - executáveis e crates;
   - separação entre source tree e pacote Pocket publicado;
   - modularidade do frontend e fronteiras Rust.

2. **Workspace/build/configuração**
   - workspace Rust/Tauri;
   - versões pinadas e política de dependências;
   - configuração de desenvolvimento e exemplos não secretos;
   - outputs/build/package não versionados.

3. **Migrations/scripts/testes/fixtures**
   - localização/naming/imutabilidade das migrations;
   - scripts apenas de tooling;
   - testes unitários, integração e e2e;
   - fixtures determinísticas sem dados reais.

4. **Parâmetros finais**
   - autenticação/sessão;
   - Backup/Restore;
   - configuração da empresa;
   - categoria arquivada;
   - limites/timeouts/backoff/log rotation e outros números ainda pendentes.

5. **Fase 2 e gate de implementação**
   - sequência executável de fundação;
   - critérios de aceite;
   - tarefas Codex;
   - sincronização segura do checkout local;
   - autorização explícita do primeiro scaffold/runtime oficial.

### Checkpoint atual — Análise 1

Proposta em revisão: **P12.1–P12.14 — árvore fonte e fronteiras de responsabilidade**.

Direção proposta:

```text
StepFlow/
├── Cargo.toml
├── apps/
│   ├── client/
│   ├── launcher/
│   ├── controller/
│   └── host/
├── crates/
│   ├── protocol/
│   ├── domain/
│   ├── documents/
│   └── platform-windows/
├── scripts/
├── tests/e2e/
└── docs/
```

A aprovação da árvore não cria autorização automática para materializá-la antes do gate final do Bloco 12.

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
- versões pinadas de crates/adapters.

### Ambiente real

- Windows/WebView2 nas estações reais;
- PoC do fallback Pocket WebView2;
- Launcher pelo share corporativo;
- Word/impressoras;
- SMB real;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Windows e crash injection de Backup/Restore.

Esses gates de ambiente podem permanecer para execução/validação na fase apropriada, desde que o Bloco 12 deixe claro em qual etapa da Fase 2 eles são obrigatórios.

## Gate atual

1. revisar/aprovar a arquitetura de source tree;
2. fechar workspace/build/configuração;
3. fechar migrations/scripts/testes/fixtures;
4. fechar parâmetros finais ainda pendentes;
5. aprovar plano executável da Fase 2;
6. definir procedimento de sincronização segura do checkout local;
7. executar validação cruzada final da Fase 1;
8. cumprir gate Git do Bloco 12;
9. somente então autorizar o primeiro scaffold/runtime oficial da Fase 2.

## Regras finais

- não criar scaffold/runtime definitivo, migration oficial ou código de negócio antes do gate explícito;
- toda tarefa Codex que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO;
- gates Git consumidos não permanecem como estado em documentos técnicos estáveis.
