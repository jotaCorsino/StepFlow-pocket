# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1  
**Atualização:** 2026-09-01

Este arquivo é o **mapa arquitetural**. Contratos detalhados pertencem aos documentos específicos.

## Visão geral

```text
Pasta StepFlow publicada no servidor Windows
        ↓
StepFlow.exe na raiz do compartilhamento
        ↓
Launcher prepara/valida Client local automaticamente
        ↓
Client Tauri local por usuário
        ↓ HTTP/JSON + WebSocket autenticado quando houver sessão
StepFlow Host Pocket na máquina central
        ↓
SQLite local + arquivos persistentes administrados
```

Publicação aprovada:

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\
    │   ├── manifest.json
    │   ├── deployment.json
    │   └── releases\
    └── server\
        ├── app\
        ├── config\
        ├── data\
        │   ├── stepflow.sqlite
        │   ├── company\
        │   └── avatars\
        ├── logs\
        └── backups\
```

`_internal/server/` é a raiz lógica do estado central. `StepFlow.exe` é o Launcher empacotado e o único ponto de entrada normal do usuário.

## Componentes

### Client

**Tauri 2 + HTML/CSS/JavaScript modular em ES modules**, com `src-tauri` fino. Responsável por UI, sessão em memória, consumo da API, eventos/reconsulta, estados transversais, preview/save/impressão local e integrações Windows necessárias.

O Client nunca acessa SQLite diretamente e o baseline não usa Node/npm/Vite/bundler/framework.

### Launcher

Executável Rust x64 pequeno/transitório, empacotado como `StepFlow.exe`. Valida manifest/deployment/release, prepara versão local em `%LOCALAPPDATA%`, valida hashes/recursos, inicia o Client local e encerra.

### Controller / Host

**Rust + Tokio/Axum + `rusqlite` bundled.**

O Controller inicia/controla o Host sob demanda, garante exclusividade, aguarda readiness e coordena shutdown. O Host concentra autenticação/autorização, API, WebSocket, SQLite, concorrência, domínio, auditoria, Backup/Restore e geração documental.

Relaunch de Restore é bounded; Recovery é local/transitório. Nenhum fluxo cria watchdog geral, Windows Service ou daemon.

## Contrato Pocket

- pasta pronta na máquina central;
- zero instalador tradicional por estação;
- Client preparado localmente de forma automática;
- zero preparação manual de dependência;
- zero elevação administrativa no uso normal;
- nenhuma toolchain em produção;
- nenhuma Internet obrigatória no uso normal;
- Client não roda permanentemente do SMB;
- sem serviço StepFlow persistente, scheduler, watchdog, tray ou daemon baseline;
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece.

WebView2 Evergreen compatível já presente é preferível. Fixed Version não roda por UNC/SMB; fallback autocontido local exige PoC sem instalação/admin/manualidade.

## Domínio e persistência

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimentos usam revisões imutáveis;
- categorias são configuráveis/múltiplas;
- Atendimento preserva revisão exata usada;
- Equipamento é opcional/reutilizável;
- checklist e `Observação do serviço` persistem somente em Atendimento;
- histórico relevante precisa ser reproduzível.

Persistência central vive em `_internal/server/data/`:

- SQLite local ao Host;
- foreign keys + WAL;
- writer lógico coordenado + fila bounded;
- migrations versionadas/imutáveis;
- `revision_no` separado de `display_version`;
- auditoria proporcional;
- `config/`, `data/`, `logs/` e `backups/` não são substituídos com binários.

## Comunicação e concorrência

- HTTP/JSON versionado, inicialmente `/api/v1`;
- WebSocket autenticado para eventos quando sessão existir;
- `deployment.json` não contém segredo;
- handshake de compatibilidade antes do login;
- sem edição offline;
- eventos pós-commit sinalizam mudança e Client reconsulta;
- revisão otimista por recurso;
- `409`/erro equivalente para base obsoleta;
- timeout após mutação exige reconciliação, não retry cego.

Defaults D12 relevantes:

```text
connect_timeout = 5 s
common_request_timeout = 30 s
websocket_backoff = 1/2/4/8/15/30 s + jitter ±20%
```

## Autenticação e autorização

- Argon2id PHC: 64 MiB / 3 passes / 4 lanes, salt 16 bytes, output 32 bytes;
- senha: 15–128 caracteres Unicode após NFKC, sem composition rule/rotação periódica;
- blocklist offline ≥10.000 + throttling progressivo;
- token opaco CSPRNG de 32 bytes, somente em memória no Client e digest server-side;
- sessão: 30 min idle / 8 h absoluta;
- autorização Host-side por capacidade;
- ADM/Gerência/Funcionário como presets;
- Gerência pode alterar configuração da empresa;
- Backup = ADM/Gerência; Restore = ADM-only;
- Restore destrutivo invalida sessões anteriores.

## Geração documental

Geração pertence ao Host por snapshot consistente + `DocumentModel`. PDF usa Typst embutido, DOCX usa OOXML direto em Rust e impressão Windows usa o mesmo PDF via WebView2. Ficha válida possui exatamente uma A4; `2+` páginas = `SHEET_OVERFLOW`.

## Backup / Restore — D11.1–D11.116 + parâmetros D12

- estado recuperável = `stepflow.sqlite + company/** + avatars/**` relativos à raiz `data/`;
- pacote `.stepflow-backup`, ZIP `Stored`, manifesto versionado + SHA-256;
- SQLite via Online Backup API;
- Backup normal usa barrier curto e promoção same-volume/no-replace;
- Restore revalida pacote/paths/provenance/banco, prepara `data-next/`, exige safety backup e troca `data/` logicamente;
- journal fica fora de `data/`, fresh Host reconcilia antes de readiness;
- disaster recovery é local/transitório pelo Controller;
- `uncertain` bloqueia readiness/mutações/cleanup destrutivo.

Parâmetros iniciais aprovados:

```text
retention_max_confirmed_backups = 20 (faixa configurável 5–100)
max_entries = 10_000
max_total_payload = 8 GiB
max_managed_file = 16 MiB
min_free_space_reserve = 1 GiB
backup_capture_target <= 2 s
backup_capture_hard_limit = 10 s
pre_restore_no_progress = 120 s
pre_restore_total_before_destructive = 10 min
readiness_timeout_per_launch = 30 s
restore_relaunch_attempts = 3
```

Paths usam semântica Windows estrita e manifesto inclui `source_deployment_id`. Baseline sem criptografia/assinatura application-level; SHA-256 é integridade, não autenticidade.

## Fundação executável planejada — D12.80–D12.98

A Fase 2 segue:

```text
F2-T01 workspace/tooling + Host mínimo
→ F2-T02 Host runtime/readiness
→ F2-T03 SQLite + migrations runner
→ F2-T04 Controller lifecycle
→ F2-T05 Client Tauri + compatibilidade
→ F2-T06 Launcher Pocket
→ F2-T07 packaging Pocket
→ F2-T08 smoke integrado + gates Windows/Pocket
```

Cada tarefa usa branch/PR próprios e não antecipa funcionalidades das fases posteriores.

## Gates ainda reservados ao ambiente real

Windows/WebView2, Launcher pelo SMB, ACL/EDR/antivírus, filesystem/long paths, Word/impressoras, transporte e demais validações corporativas são executados no momento técnico apropriado. Fora do ambiente correspondente, registrar `NÃO APLICÁVEL NESTE AMBIENTE`, nunca PASS presumido.

Nenhum runtime/código funcional oficial foi criado durante a Fase 1.
