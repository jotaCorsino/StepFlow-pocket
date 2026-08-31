# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-31

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
| 11 | Backup/restauração técnico | EM ANÁLISE — ANÁLISE 6 | `bloco-11-backup-restauracao.md` |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | a abrir |

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

**EM ANÁLISE.**

Fontes:

- `bloco-11-backup-restauracao.md` — mapa principal;
- `bloco-11-analise-3-catalogo-retencao-coordenacao.md` — aprovada;
- `bloco-11-analise-4-restore-safety-compatibilidade.md` — aprovada;
- `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md` — aprovada;
- `bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md` — proposta atual;
- `../02-telas/13-backup-restauracao.md` — UX consolidada.

Estado das análises:

- Análise 1 — estado recuperável + envelope: **APROVADA**;
- Análise 2 — consistência + escrita/promoção/verificação: **APROVADA**;
- Análise 3 — catálogo + retenção + coordenação: **APROVADA**;
- Análise 4 — Restore + safety backup + compatibilidade: **APROVADA**;
- Análise 5 — restart + sessões + reconexão + falhas: **APROVADA**;
- Análise 6 — disaster recovery + capacidades + auditoria: **EM REVISÃO**;
- Análise 7 — validação técnica final: **PENDENTE**.

Já consolidado até D11.82:

- backup protege `stepflow.sqlite + company/** + avatars/**`;
- pacote `.stepflow-backup`, manifesto, hashes e Online Backup API;
- consistência conjunta de banco + arquivos e barrier curto de mutações;
- staging/verificação/promoção same-volume/no-replace;
- catálogo reconstruível e retenção sem scheduler/por quantidade;
- coordinator administrativo para Backup/Restore/migration;
- Restore revalidado em `data-next/`, migrations somente forward e safety backup obrigatório;
- troca lógica de `data/`, rollback conhecido ou `uncertain`;
- journal fora de `data/`, recovery antes de readiness e fresh Host após fase destrutiva;
- sessões anteriores invalidadas após Restore destrutivo;
- `uncertain` bloqueia readiness/mutações/cleanup destrutivo.

Em revisão P11.83–P11.103:

- disaster recovery local do Controller;
- autoridade local/ACL quando banco não autentica;
- Gerência × Backup;
- auditoria administrativa fora de `data/`.

Aprovação da Análise 6 não encerra o bloco: ainda é obrigatória a Análise 7 de validação técnica final.

## Bloco 12 — Estrutura oficial + Fase 2

**PENDENTE.**

Fechará:

- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- parâmetros finais ainda abertos;
- plano da Fase 2;
- sincronização segura do checkout local antes do primeiro trabalho de implementação.

## Pendências restantes da Fase 1

### Segurança/configuração

- custo Argon2id final;
- senha mínima final;
- duração/expiração de sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup — proposta em revisão na Análise 6;
- regra editorial de categoria arquivada;
- valor/default final de `retention_max_confirmed_backups`.

### Bloco 11

- revisão P11.83–P11.103;
- validação técnica final;
- sincronização documental final do bloco.

### Ambiente real

- Windows/WebView2 nas estações reais;
- PoC do fallback Pocket WebView2;
- Launcher pelo share corporativo;
- Word/impressoras;
- SMB real;
- ACL/filesystem de recovery;
- EDR/firewall/políticas.

## Regras finais

- não criar scaffold/runtime definitivo, migration oficial ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO;
- gates Git consumidos não permanecem como estado em documentos técnicos estáveis.