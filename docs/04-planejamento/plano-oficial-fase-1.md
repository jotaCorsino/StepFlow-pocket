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
| 11 | Backup/restauração técnico | TECNICAMENTE CONCLUÍDO / GATE GIT PENDENTE | `bloco-11-backup-restauracao.md` |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | a abrir após gate remoto limpo |

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
- exigir Rust/Node/npm/Cargo/Office/LibreOffice/Adobe;
- exigir Internet para uso normal;
- exigir elevação administrativa;
- exigir configuração manual de dependência;
- executar permanentemente o Client pelo SMB.

WebView2 Evergreen existente é preferível quando compatível. Fixed Version não roda por UNC/SMB; fallback local só entra após PoC provar preparação automática sem instalação/elevação/manualidade.

## Blocos 8–10 — fechados

### Bloco 8 — UI/UX

Telas 01–15 consolidadas. Reader em formato livro/manual, `Visão geral`, uma Etapa por página lógica, stepper de navegação e baixa densidade textual.

### Bloco 9 — execução operacional

Lifecycle `Em andamento / Concluído / Cancelado`, reabertura explícita, checklist persistente, observação de serviço por Etapa, Equipamento opcional, revisão exata e reprodução histórica.

### Bloco 10 — geração documental

Geração Host-side, PDF Typst, DOCX OOXML Rust, impressão pelo mesmo PDF, Procedimento A4 multipágina, Ficha de exatamente uma A4, `SHEET_OVERFLOW`, naming/temporários e gates corporativos consolidados.

## Bloco 11 — Backup / Restore técnico

**TECNICAMENTE CONCLUÍDO em 2026-09-01.** O PR #26 ainda precisa cumprir o gate Git antes da entrada em `main`.

Fontes:

- `bloco-11-backup-restauracao.md` — mapa principal consolidado;
- `bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- `bloco-11-analise-4-restore-safety-compatibilidade.md`;
- `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`;
- `bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md`;
- `bloco-11-analise-7-validacao-tecnica-final.md`;
- `../02-telas/13-backup-restauracao.md`.

Decisões aprovadas: **D11.1–D11.116**.

Fechado:

- estado recuperável + `.stepflow-backup`;
- Online Backup API e consistência SQLite + arquivos;
- staging, verificação e promoção no-replace;
- catálogo reconstruível, retenção e coordenação administrativa;
- Restore com revalidação, migrations forward, safety backup e troca lógica de `data/`;
- journal/restart/rollback/`uncertain`;
- invalidação de sessões após fase destrutiva;
- disaster recovery local/transitório;
- Backup ADM/Gerência e Restore ADM-only;
- auditoria administrativa fora de `data/`;
- safety barrier contínuo do `pre_restore`;
- canonicalização Windows, `source_deployment_id` e parser bounded;
- baseline sem criptografia/assinatura application-level;
- gates Win32/filesystem/ACL/EDR/long paths/crash antes de produção.

Não existe bloqueador arquitetural conhecido para o Bloco 11.

## Bloco 12 — Estrutura oficial + Fase 2

**PENDENTE.**

Fechará:

- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- parâmetros finais ainda abertos;
- configuração/defaults/fixtures mensuráveis para limites técnicos;
- plano da Fase 2;
- sincronização segura do checkout local antes do primeiro trabalho de implementação.

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

## Gate atual

1. concluir revisão final do PR #26;
2. tornar PR ready;
3. squash merge em `main`;
4. remover branch remota;
5. verificar somente `main` e zero PRs abertos;
6. somente então abrir Bloco 12.

## Regras finais

- não criar scaffold/runtime definitivo, migration oficial ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO;
- gates Git consumidos não permanecem como estado em documentos técnicos estáveis.
