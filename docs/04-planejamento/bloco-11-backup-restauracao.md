# Bloco 11 — Backup / Restauração técnico

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Consolidação técnica:** 2026-09-01

## Objetivo

Contrato técnico de Backup/Restore. A UX permanece em `../02-telas/13-backup-restauracao.md`.

Decisões vigentes: **D11.1–D11.116**. Parâmetros numéricos iniciais complementares: **D12.66–D12.75**.

## Estado recuperável e envelope — D11.1–D11.10

Backup protege, relativamente à raiz central `data/`:

```text
stepflow.sqlite
company/**
avatars/**
```

Ficam fora binários, config, logs, backups, exportações, temporários e Client local.

- pacote único `.stepflow-backup`;
- ZIP `Stored`;
- `manifest.json` versionado;
- payload allowlisted;
- SHA-256 por entrada;
- SQLite via Online Backup API;
- staging antes da promoção;
- parcial nunca é backup válido.

## Consistência e promoção — D11.11–D11.25

```text
BACKUP_CAPTURE
→ suspender novas mutações
→ drenar mutações aceitas
→ capturar SQLite + company/** + avatars/**
→ liberar mutações
→ hash/ZIP/verificação/promoção
```

- leituras seguras podem continuar;
- `-wal`/`-shm` não entram;
- `quick_check = ok` + `foreign_key_check` vazio;
- flush explícito;
- promoção same-volume/no-replace;
- crash/falha nunca promove parcial.

## Catálogo, retenção e coordenação — D11.26–D11.42

- catálogo reconstruível dos pacotes finais;
- `backup_id` é identidade canônica;
- Restore revalida integralmente;
- cache de verificação somente em memória;
- retenção sem scheduler e por quantidade;
- não apagar backup antigo antes de confirmar o novo para abrir espaço;
- source/safety/pre-migration protegidos enquanto necessários;
- lease exclusivo coordena `BACKUP`, `RESTORE` e `MIGRATION`;
- `uncertain` suspende retenção e cleanup destrutivo.

Parâmetro D12:

```text
retention_max_confirmed_backups = 20
faixa configurável = 5..100
```

Campo ausente usa 20. Valor explicitamente inválido/fora da faixa não sofre clamp/fallback silencioso; a validação final do Bloco 12 fecha erro explícito de configuração.

## Restore, safety backup e compatibilidade — D11.43–D11.61

- revalidar envelope, hashes, paths, provenance e SQLite;
- preparar `data-next/` same-volume;
- `integrity_check = ok` + `foreign_key_check` vazio;
- schema antigo somente com migrations forward completas;
- schema mais novo/cadeia incompleta = incompatível;
- nenhuma down migration automática;
- safety backup confirmado obrigatório;
- ativação `data → old`, `data-next → data`;
- `old` preservado até validação final;
- rollback conhecido ou `uncertain`.

### Safety barrier

```text
suspender/drain mutações
→ capturar safety backup
→ manter barrier
→ finalizar/verificar/promover
→ revalidar data-next
→ DESTRUCTIVE_STARTED
→ primeiro rename
```

Nenhuma mutação em `data/` entre a captura do safety backup e o primeiro rename.

## Restart, sessões, reconexão e falhas — D11.62–D11.82

- journal fora de `data/`, baseline `backups/.operations/restore-active.json`;
- fresh Host reconcilia antes de migrations/readiness;
- queda entre renames retorna para `old` quando comprovável;
- estado não comprovável = `RECOVERY_REQUIRED/uncertain`;
- relaunch bounded, sem watchdog geral;
- fase destrutiva invalida sessões/tokens anteriores;
- `restore-last.json`/equivalente preserva resultado terminal;
- `uncertain` bloqueia readiness/mutações/cleanup.

## Disaster recovery, capacidades e auditoria — D11.83–D11.103

- Recovery excepcional, local/transitório pelo Controller;
- sem listener normal HTTP/WebSocket;
- autoridade por acesso local/ACL/exclusividade quando o banco não autentica;
- mesma integridade/compatibilidade/migrations forward do Restore normal;
- ausência de safety backup somente em disaster recovery real;
- Backup = ADM/Gerência; Restore = ADM-only;
- auditoria funcional quando disponível + trilha administrativa fora de `data/`;
- journal, admin audit e logs técnicos têm funções distintas.

## Validação técnica final — D11.104–D11.116

### Revalidação

Antes de `DESTRUCTIVE_STARTED`, verificar novamente digest/schema/root/volume de `data-next/`.

### Paths Windows

Rejeitar path absoluto/drive/UNC/device, `.`/`..`, ADS/`:`, NUL/controles, nomes reservados, trailing dot/space, colisões Windows-equivalent, duplicidade pós-canonicalização, reparse/non-regular e escape do root.

Criação do backup aplica a mesma disciplina a `company/**` e `avatars/**`.

### Provenance

`manifest.json` inclui `source_deployment_id`; pacote de deployment diferente é `source_mismatch` no baseline.

### Limites estruturais — D12.67–D12.70

```text
max_entries = 10_000
max_total_payload = 8 GiB
max_managed_file = 16 MiB
max_logical_path = 512 UTF-16 code units
max_path_component = 120 UTF-16 code units
max_path_depth = 8
min_free_space_reserve = 1 GiB
backup_capture_target <= 2 s
backup_capture_hard_limit = 10 s
pre_restore_no_progress = 120 s
pre_restore_total_before_destructive = 10 min
```

Ultrapassar limite nunca trunca dados; a operação falha com diagnóstico apropriado. Depois de `DESTRUCTIVE_STARTED`, timeout genérico não mata Restore.

### Readiness/reconexão — D12.71–D12.73

```text
readiness_timeout_per_launch = 30 s
restore_relaunch_attempts = 3
connect_timeout = 5 s
common_request_timeout = 30 s
websocket_backoff = 1/2/4/8/15/30 s + jitter ±20%
```

### Logs — D12.74–D12.75

```text
log técnico = 20 MiB + 10 archives
admin audit = 50 MiB + 20 archives
```

### Criptografia/autenticidade

- sem criptografia application-level baseline;
- sem assinatura criptográfica application-level baseline;
- SHA-256 = integridade/corrupção, não autenticidade;
- trust boundary = root administrado + ACLs + infraestrutura de volume + deployment ID + auditoria.

## Limite operacional e gates

Backup local não promete proteção contra perda física total, ransomware com acesso ao mesmo storage ou site loss. Offsite/cópia corporativa é responsabilidade operacional externa.

Antes de produção, quando aplicável: adapter Win32, filesystem real, ACLs, EDR/antivírus, long paths, espaço, performance e crash/restart injection.

## Critérios de fechamento

- [x] decisões D11 permitem implementação sem escolha arquitetural crítica aberta;
- [x] UX da Tela 13 permanece coerente;
- [x] Restore possui safety backup/recovery/uncertain;
- [x] disaster recovery está separado do Restore normal;
- [x] capacidades e auditoria estão fechadas;
- [x] parâmetros numéricos foram fechados em D12.66–D12.75;
- [x] não existe bloqueador arquitetural conhecido no contrato de Backup/Restore.

## Fontes detalhadas

- `bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- `bloco-11-analise-4-restore-safety-compatibilidade.md`;
- `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`;
- `bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md`;
- `bloco-11-analise-7-validacao-tecnica-final.md`;
- `../02-telas/13-backup-restauracao.md`.

## Conclusão

**Bloco 11 consolidado e operacionalmente encerrado.** O PR #26 foi squash-merged, sua branch foi removida e o remoto foi verificado limpo antes da abertura do Bloco 12.
