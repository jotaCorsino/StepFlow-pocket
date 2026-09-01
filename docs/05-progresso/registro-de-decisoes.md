# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-09-01

Este arquivo é o **digest ativo de decisões, pendências reais e gates vigentes**. Detalhes pertencem aos documentos específicos. Proposta não aprovada não é contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade;
- fluxo: branch → draft PR → discussão/refino → aprovação PO → consolidação → validação → ready → squash merge → remover branch → verificar remoto limpo;
- merge não encerra trabalho enquanto a branch remota existir;
- checkout local previsto em `C:\dev\StepFlow` só é sincronizado antes da implementação, preservando alterações locais do PO;
- Fase 1 não autoriza scaffold/runtime/migrations oficiais/código de negócio antes do gate final do Bloco 12;
- `AGENTS.md` é a regra operacional superior.

## 2. Contrato Pocket

```text
pasta publicada no servidor Windows
→ usuário acessa compartilhamento
→ executa StepFlow.exe na raiz
→ Launcher prepara/valida Client local em %LOCALAPPDATA%
→ Client abre localmente
→ Launcher encerra
→ Client comunica com Host central
```

`StepFlow.exe` é o Launcher amigável e único ponto de entrada normal. Artefatos técnicos ficam sob `_internal/`; `.lnk` não é requisito baseline.

Obrigatório: sem instalador por estação, preparação manual, admin, toolchain em produção, Internet obrigatória ou Client permanente em SMB; Controller/Host sob demanda e sem Service/Task Scheduler/watchdog/tray/daemon baseline.

WebView2 Evergreen compatível já presente é preferível. Fixed Version não roda por UNC/SMB; fallback local exige PoC automática em `%LOCALAPPDATA%` sem instalação/admin/manualidade.

## 3. Produto e domínio

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação oficial versionada;
- Atendimento = ocorrência real;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode existir sem Equipamento e usar vários Procedimentos;
- vínculo preserva revisão exata;
- histórico concluído não é reescrito;
- StepFlow não vira CRM/financeiro/estoque/RMM/help desk completo por inferência.

## 4. Procedimentos, categorias e revisões

Campos principais: Código, Título, Área/Departamento, Responsável, Status, Versão, Objetivo, Observações, Pré-requisitos, Categorias, Etapas e Histórico.

- categorias configuráveis/múltiplas, sem árvore inicial;
- salvamento explícito e revisões imutáveis;
- controle otimista e publicação separada de save;
- `revision_no` técnico separado de `display_version`;
- categoria arquivada não entra em associação nova;
- categoria arquivada herdada permanece com aviso, pode ser removida e não pode ser re-adicionada enquanto arquivada — D12.65.

## 5. Reader e direção visual

- experiência livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa por página lógica;
- stepper = navegação, nunca conclusão;
- comandos preservam whitespace e usam copiar icon-only acessível;
- baixa densidade textual;
- Reader standalone não persiste execução;
- Reader em Atendimento persiste checklist e `Observação do serviço` conforme lifecycle/autorização.

## 6. Atendimentos e Equipamentos

Lifecycle `Em andamento / Concluído / Cancelado`, com reabertura explícita. Primeiro save gera `AT-000001`; Equipamento usa `EQP-000001`. Checklist incompleto avisa, progresso deriva somente dele e 100% não conclui automaticamente. Equipamento é opcional/reutilizável; histórico relevante deve ser reproduzível.

## 7. Autenticação e capacidades

- Argon2id PHC: 64 MiB / 3 passes / 4 lanes, salt 16 bytes, output 32 bytes — D12.56;
- senha 15–128 Unicode após NFKC, sem composition rule/rotação periódica — D12.57;
- blocklist offline ≥10.000 — D12.58;
- throttling progressivo + cooldown temporário — D12.59;
- token opaco CSPRNG 32 bytes, memória do Client + digest server-side — D12.60;
- sessão 30 min idle / 8 h absoluta — D12.61;
- autorização Host-side por capacidades;
- ADM/Gerência/Funcionário são presets;
- Gerência não administra ADM;
- Backup = ADM/Gerência; Restore = ADM-only;
- configuração da empresa = ADM/Gerência; Funcionário não — D12.62;
- Restore destrutivo invalida sessões anteriores e exige novo login após fresh Host.

## 8. Arquitetura técnica

- Client Tauri 2 + HTML/CSS/JavaScript modular;
- Host Rust + Tokio/Axum + `rusqlite` bundled;
- HTTP/JSON + WebSocket autenticado quando houver sessão;
- SQLite somente pelo Host;
- WAL + writer lógico + fila bounded + revisão otimista;
- dados/config/logs/backups separados de binários substituíveis;
- publicação `StepFlow.exe + _internal/client|server`;
- nenhuma toolchain em produção.

## 9. Geração documental — Bloco 10 concluído

Host-side por snapshot consistente + `DocumentModel`; PDF Typst; DOCX OOXML Rust; impressão pelo mesmo PDF via WebView2; Procedimento A4 multipágina; Ficha exatamente uma A4 com `SHEET_OVERFLOW`; gates reais de Word/impressoras/SMB/Windows/WebView2/EDR permanecem para o ambiente apropriado.

## 10. Backup / Restore — Bloco 11 concluído

Decisões vigentes: **D11.1–D11.116**.

- estado recuperável = `stepflow.sqlite + company/** + avatars/**`;
- `.stepflow-backup`, ZIP `Stored`, manifesto + SHA-256;
- SQLite via Online Backup API;
- consistency barrier + staging/flush/promoção no-replace;
- catálogo reconstruível e retenção por quantidade;
- lease coordena Backup/Restore/Migration;
- Restore usa migrations forward no staging, safety backup obrigatório e nunca down migration automática;
- journal fora de `data/`, fresh Host e invalidação de sessões;
- disaster recovery local/transitório;
- paths Windows/provenance estritos;
- `uncertain` bloqueia readiness/mutações/cleanup.

Parâmetros D12:

```text
retention default 20; faixa 5–100
10.000 entradas / 8 GiB / 16 MiB por managed file
reserva mínima 1 GiB
Backup capture alvo <=2 s; hard limit 10 s
pre_restore 120 s sem progresso / 10 min pré-destrutivo
readiness 30 s; relaunch Restore 3 tentativas
Client connect 5 s; request comum 30 s
WebSocket 1/2/4/8/15/30 s + jitter
log técnico 20 MiB + 10 archives
admin audit 50 MiB + 20 archives
```

## 11. Estrutura oficial / Bloco 12

Decisões vigentes: **D12.1–D12.98**.

### D12.1–D12.18 — estrutura/publicação

Workspace `apps/`/`crates/`, Client modular, source tree distinta da publicação, `StepFlow.exe` na raiz e `_internal/` técnico; sem crates vazios por antecipação.

### D12.19–D12.34 — workspace/build

Rust 1.98.0, Edition 2024, resolver 3, Windows x64 MSVC, toolchain/lockfile versionados, build lockfile-aware, Client sem Node/bundler, configuração separada e packaging como fonte da produção.

### D12.35–D12.55 — migrations/testes/fixtures

Migrations Host-side imutáveis/embutidas, `schema_migrations` com checksum, lote transacional, `quick_check` + `foreign_key_check`, sem down migration, testes em SQLite temporário real, fixtures sintéticas e scripts finos.

### D12.56–D12.79 — parâmetros finais

Autenticação/sessão, Gerência × Empresa, limites de identidade/logo, categoria arquivada, parâmetros de Backup/Restore, readiness/reconexão e rotação de logs estão fechados nas fontes específicas.

### D12.80–D12.98 — plano da Fase 2

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

Cada tarefa usa branch/PR próprios, parte de `main` consolidada e recebe pré-flight separado. Não antecipar autenticação, domínio, documentos ou Backup/Restore funcional.

### Em revisão — P12.99–P12.108

Validação final em `bloco-12-analise-6-validacao-final-fase-1.md` propõe:

- retenção ausente usa 20; valor explicitamente inválido gera erro, sem clamp/fallback;
- owner único para cada parâmetro e somente knobs explicitamente configuráveis;
- `deployment.json` real materializado por input de implantação, nunca por placeholder silencioso;
- sync local limpo via `fetch --prune` + `merge --ff-only`, parando diante de alteração/divergência;
- gates corporativos podem permanecer reservados ao encerrar Fase 1, mas bloqueiam a etapa/produção quando aplicáveis;
- merge do Bloco 12 não autoriza scaffold automaticamente.

## 12. Pendências vigentes

### Bloco 12

- decidir P12.99–P12.108;
- consolidar validação final;
- gate Git do PR #27;
- sincronização segura do checkout local;
- autorização explícita da F2-T01.

### Ambiente corporativo

- Windows/WebView2 real e fallback quando necessário;
- Launcher pelo compartilhamento;
- Word/impressoras;
- SMB/permissões/falhas;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Win32 e crash injection para Backup/Restore.

## 13. Precedência

1. `AGENTS.md`;
2. este registro;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão pelo executor.
