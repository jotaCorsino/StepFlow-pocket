# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-09-01

Este arquivo é o **digest ativo de decisões, pendências reais e gates vigentes**. Detalhes técnicos pertencem aos documentos específicos. Proposta não aprovada não é contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade durante a Fase 1;
- fluxo: branch → draft PR → discussão/refino → aprovação PO → consolidação → validação → ready → squash merge → remover branch → verificar remoto limpo;
- merge não encerra etapa/bloco sem remoção da branch e verificação do gate;
- checkout local previsto em `C:\dev\StepFlow` só deve ser sincronizado antes da implementação, preservando alterações locais do PO;
- Fase 1 não autoriza scaffold/runtime/migrations oficiais/código de negócio antes do gate final do Bloco 12/Fase 2;
- `AGENTS.md` é a regra operacional superior;
- documentos estáveis não carregam gates Git consumidos nem anunciam próximo bloco.

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

`StepFlow.exe` é o Launcher com nome/ícone amigáveis e o único ponto de entrada normal. Artefatos técnicos ficam sob `_internal/`; `.lnk` não é requisito baseline.

Obrigatório:

- sem instalador tradicional por estação;
- sem preparação manual de dependências;
- sem elevação administrativa no uso normal;
- sem toolchain em produção;
- sem Office/LibreOffice/Adobe como dependência operacional;
- sem Internet obrigatória;
- Client não roda permanentemente do SMB;
- Controller/Host sob demanda;
- fechar Client individual não encerra Host;
- sem Windows Service, Task Scheduler, watchdog, tray ou daemon baseline.

WebView2 Evergreen compatível já presente é preferível. Fixed Version não roda por UNC/SMB; fallback local só entra após PoC provar preparação automática em `%LOCALAPPDATA%` sem instalação/admin/manualidade.

## 3. Produto e domínio

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação oficial versionada;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode existir sem Equipamento e usar vários Procedimentos;
- vínculo preserva revisão exata utilizada;
- histórico concluído não é reescrito por alterações posteriores;
- StepFlow não vira CRM, financeiro, estoque, RMM ou help desk/SLA completo por inferência.

## 4. Procedimentos, categorias e revisões

Campos principais: Código, Título, Área/Departamento, Responsável, Status, Versão, Objetivo, Observações, Pré-requisitos, Categorias, Etapas e Histórico.

- categorias configuráveis, múltiplas e sem árvore inicial;
- arquivamento preserva histórico;
- salvamento explícito e revisões imutáveis;
- controle otimista com conflito explícito;
- publicar é separado de salvar;
- `revision_no` técnico separado de `display_version` editorial;
- categoria arquivada não pode ser adicionada como associação nova;
- categoria arquivada já herdada pela revisão-base permanece na nova revisão com aviso, pode ser removida, mas não pode ser re-adicionada enquanto arquivada — D12.65.

## 5. Reader e direção visual

- experiência livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa por página lógica;
- stepper horizontal representa navegação, não conclusão;
- comandos preservam whitespace e usam copiar icon-only acessível;
- baixa densidade textual e informação secundária sob demanda;
- Reader standalone não persiste execução;
- Reader em Atendimento persiste checklist e `Observação do serviço` por Etapa conforme lifecycle/autorização.

## 6. Atendimentos e Equipamentos

```text
rascunho Client
→ primeiro save
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- primeiro save cria `AT-000001`;
- conclusão exige responsável + `Resumo do trabalho`;
- checklist incompleto avisa, não bloqueia automaticamente;
- progresso deriva somente do checklist;
- 100% não conclui automaticamente;
- observação por Etapa é opcional e operacional;
- Equipamento usa `EQP-000001`, é opcional e reutilizável;
- serial/MAC/patrimônio não são identidade canônica;
- reprodução histórica suficiente é obrigatória.

## 7. Autenticação e capacidades

- Argon2id em PHC;
- baseline D12.56: 64 MiB, 3 passes, 4 lanes, salt 16 bytes, output 32 bytes;
- senha D12.57: 15–128 caracteres Unicode após NFKC, sem composition rule/rotação periódica, sem truncamento;
- blocklist offline ≥10.000 — D12.58;
- throttling progressivo e cooldown de 15 min na décima falha — D12.59;
- token opaco CSPRNG de 32 bytes, Client-memory-only e digest server-side — D12.60;
- sessão: 30 min idle / 8 h absoluta; heartbeat/background não renovam idle — D12.61;
- autorização Host-side por capacidade;
- `ADM`, `GERENCIA`, `FUNCIONARIO` são presets;
- pelo menos um ADM ativo;
- bootstrap do primeiro ADM é local/controlado;
- Gerência não administra ADM;
- Backup = ADM/Gerência; Restore = ADM-only;
- alterar configuração da empresa = ADM/Gerência; Funcionário não — D12.62.

### Restore e sessões

- Restore que entra na fase destrutiva invalida todas as sessões/tokens anteriores;
- isso vale também em rollback;
- backup restaurado nunca ressuscita token reutilizável antigo;
- fresh Host exige novo login.

## 8. Arquitetura técnica

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite` bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- WAL + writer lógico coordenado + fila bounded + revisão otimista;
- eventos pós-commit;
- implantação central por pasta pronta;
- dados/config/logs/backups separados de binários substituíveis;
- nenhuma toolchain na produção.

## 9. Geração documental — Bloco 10 concluído

- geração Host-side por snapshot consistente + `DocumentModel`;
- PDF via Typst embutido;
- DOCX OOXML Transitional direto em Rust;
- impressão Windows pelo mesmo PDF oficial via WebView2;
- Procedimento A4 multipágina;
- Ficha PDF + preview do mesmo `PagedDocument`, exatamente uma A4;
- `SHEET_OVERFLOW` sem truncamento;
- soft limits 600/400/300/280;
- Word/impressoras/SMB/Windows/WebView2/EDR são gates de ambiente real.

## 10. Backup / Restore — Bloco 11 concluído

Decisões vigentes: **D11.1–D11.116**.

Resumo:

- Backup protege `stepflow.sqlite + company/** + avatars/**`;
- pacote único `.stepflow-backup`, ZIP `Stored`, manifesto + SHA-256;
- SQLite via Online Backup API;
- consistência cobre banco + arquivos administrados;
- staging, flush e promoção same-volume/no-replace;
- catálogo reconstruível e retenção por quantidade sem scheduler;
- lease coordena `BACKUP`, `RESTORE`, `MIGRATION`;
- Restore revalida pacote/banco, aceita schema antigo somente com migrations forward completas e nunca usa down migration automática;
- safety backup obrigatório no Restore normal;
- journal fora de `data/`, fresh Host e invalidação de sessões após fase destrutiva;
- disaster recovery é local/transitório pelo Controller;
- paths Windows e provenance são validados estritamente;
- `uncertain/RECOVERY_REQUIRED` bloqueia readiness/mutações/cleanup destrutivo.

Parâmetros numéricos fechados no Bloco 12:

- retenção default 20, faixa 5–100 — D12.66;
- envelope 10.000 entradas / 8 GiB / 16 MiB por managed file + limites de path — D12.67;
- reserva final mínima 1 GiB — D12.68;
- Backup capture alvo ≤2 s, hard limit 10 s — D12.69;
- pre_restore 120 s sem progresso / 10 min antes da fase destrutiva — D12.70;
- readiness 30 s e relaunch Restore 3 tentativas — D12.71;
- Client connect 5 s / request comum 30 s — D12.72;
- WebSocket backoff 1/2/4/8/15/30 s + jitter — D12.73;
- log técnico 20 MiB + 10 archives — D12.74;
- admin audit 50 MiB + 20 archives — D12.75.

## 11. Estrutura oficial / Bloco 12

Decisões vigentes: **D12.1–D12.79**.

### Estrutura e publicação — D12.1–D12.18

- workspace Rust virtual na raiz;
- `apps/` contém `client`, `launcher`, `controller`, `host`;
- `crates/` somente para responsabilidades reutilizáveis concretas;
- Client usa ES modules e `src-tauri` fino;
- source tree e pasta publicada são distintos;
- `StepFlow.exe` na raiz é Launcher amigável e único ponto de entrada;
- `_internal/` encapsula artefatos técnicos;
- `.lnk` não é requisito.

### Workspace/build — D12.19–D12.34

- Rust 1.98.0, Edition 2024, resolver 3, Windows x64 MSVC;
- toolchain e `Cargo.lock` versionados;
- build/test/package lockfile-aware;
- dependências só entram com uso real;
- Client vanilla sem Node/npm/Vite/bundler;
- configuração build/dev × deployment × runtime;
- produção vem de packaging, não `target/release` direto;
- scripts PowerShell finos.

### Migrations/testes/fixtures — D12.35–D12.55

- migrations Host-side `NNNNNN_<slug>.sql`, imutáveis e embutidas;
- runner próprio + `schema_migrations` com checksum;
- lote transacional, sem down migration automática ou `writable_schema` como atalho;
- `quick_check` + `foreign_key_check` antes de readiness;
- testes em SQLite temporário real;
- fixtures sintéticas, nunca seed de produção;
- scripts `check/test/build/package.ps1` como wrappers finos.

### Parâmetros finais — D12.56–D12.79

- autenticação/senha/sessão/token conforme seção 7;
- GERENCIA pode alterar configuração da empresa;
- identidade da empresa 120/160/200/254 e logo PNG/JPEG até 2 MiB / 2048×2048;
- categoria arquivada herdada segue D12.65;
- Backup/Restore/reconexão/logs seguem os números da seção 10;
- parâmetros ficam centralizados e configuração crítica inválida nunca cai silenciosamente em default inseguro.

### Em análise

**P12.80–P12.98 — plano detalhado da Fase 2**, em `bloco-12-analise-5-plano-fase-2.md`:

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
```

P12.80–P12.98 ainda não são contrato.

## 12. Pendências vigentes

### Bloco 12

- decidir P12.80–P12.98;
- validação final da Fase 1;
- gate Git do Bloco 12;
- sincronização segura do checkout local;
- autorização explícita do primeiro scaffold/runtime da Fase 2.

### Ambiente corporativo

- Windows/WebView2 real e fallback Pocket quando necessário;
- Launcher pelo compartilhamento;
- Word/impressoras;
- SMB/permissões/falhas;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Win32 e crash injection para Backup/Restore.

## 13. Precedência

Em divergência:

1. `AGENTS.md`;
2. este registro;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão pelo executor.
