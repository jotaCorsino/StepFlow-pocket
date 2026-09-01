# StepFlow Host Pocket

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-09-01

## Tecnologia

O Host será implementado em **Rust**, usando Tokio, Axum e `rusqlite` com SQLite bundled.

A prova técnica descartável da Fase 1 confirmou build release e execução isolada sem toolchain no runtime.

## Requisito Pocket

Na máquina central, o StepFlow é implantado por pasta pronta. Nenhum serviço StepFlow persistente é instalado como baseline.

```text
StepFlow fechado
→ nenhum processo StepFlow

Controller iniciado sob demanda
→ Host iniciado como processo-filho
→ Clients utilizam o Host
→ shutdown controlado
→ Host termina
→ Controller termina
→ nenhum processo residual
```

## Estrutura publicada

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\
    └── server\
        ├── app\
        │   ├── StepFlowController.exe
        │   └── StepFlowHost.exe
        ├── config\
        │   └── stepflow-host.toml
        ├── data\
        │   ├── stepflow.sqlite
        │   ├── company\
        │   └── avatars\
        ├── logs\
        │   └── admin-audit.jsonl
        └── backups\
            └── .operations\
```

`_internal/server/` é a raiz lógica central. `app/` é substituível; `config/`, `data/`, `logs/` e `backups/` são preservados entre atualizações.

## Controller

Responsabilidades:

1. resolver raiz da implantação;
2. validar config/paths/permissões;
3. garantir exclusividade;
4. iniciar Host como processo-filho;
5. aguardar readiness;
6. registrar falhas de startup;
7. coordenar shutdown gracioso;
8. confirmar saída do Host;
9. coordenar relaunch bounded após Restore;
10. oferecer Recovery local/transitório quando necessário.

O Controller não instala serviço, altera PATH/registro, cria auto-start nem mantém watchdog residente.

Defaults D12 do lifecycle:

```text
readiness_timeout_per_launch = 30 s
restore_relaunch_attempts = 3 total
restore_relaunch_backoff = 1 s, 3 s
```

Falha das tentativas bounded de recovery exige intervenção local/controlada; não vira watchdog geral.

## Propriedade do ciclo de vida

- fechar Client não encerra Host;
- vários Clients podem entrar/sair no mesmo ciclo;
- encerramento central solicita shutdown gracioso;
- Controller só termina após confirmar saída do Host;
- nenhum processo StepFlow permanece após encerramento completo.

Não está consolidado auto-shutdown por “último Client” ou inatividade.

## Responsabilidades do Host

- autenticação/autorização;
- HTTP/JSON + WebSocket;
- SQLite e migrations;
- writer/fila/transações;
- revisão e conflitos;
- Procedimentos, Atendimentos e Equipamentos;
- checklist e observações de serviço;
- auditoria;
- Backup/Restore;
- geração documental.

## Readiness e instância única

O Host só fica ready quando as pré-condições disponíveis na fase estiverem válidas. No produto completo isso inclui, no mínimo:

- configuração carregada;
- exclusividade da implantação;
- recovery pendente reconciliado;
- `data/` acessível;
- schema/migrations válidos;
- SQLite aberto;
- listener disponível.

```text
resolver deployment + exclusividade
→ reconciliar Restore/Recovery pendente
→ seguir migrations/readiness normais
```

Enquanto `uncertain/RECOVERY_REQUIRED`:

- sem readiness normal;
- sem mutações de negócio;
- sem nova operação destrutiva normal;
- sem retenção/cleanup destrutivo;
- preservar source/safety backup, journal, old e staging relevantes.

Duas instâncias não podem coordenar o mesmo `data/`.

## Configuração

`_internal/server/config/stepflow-host.toml` contém somente parâmetros operacionais explicitamente configuráveis. Endpoint corporativo real não é hardcoded e alterar configuração não deve exigir recompilar.

Segredos reutilizáveis não ficam nesse arquivo em texto puro. Parâmetro de política que não foi declarado configurável não vira knob de operador por conveniência.

## Logs e diagnóstico

Logs técnicos em `_internal/server/logs/` com timestamp, nível, componente, mensagem e contexto sanitizado.

Baseline D12:

```text
log técnico: 20 MiB + 10 archives
admin audit: 50 MiB + 20 archives
```

Registrar início, configuração, data root efetivo, listener/readiness, falha de startup, shutdown, erro fatal, operações administrativas críticas e recovery. Nunca registrar senha, token reutilizável ou conteúdo sensível desnecessário.

### Auditoria administrativa

Backup/Restore/retention/Recovery emitem trilha estruturada fora de `data/`, baseline `logs/admin-audit.jsonl`/equivalente.

- append-only pela aplicação no fluxo normal;
- protegida por ACLs;
- não restaurada com `data/`;
- não declarada tamper-proof criptográfica;
- IDs lógicos, ator/origem, timestamps, resultado e códigos sanitizados;
- sem senha, token, dump SQL ou conteúdo integral do backup.

Journal, admin audit e log técnico têm funções/lifecycles distintos.

## Shutdown técnico

1. Controller inicia encerramento;
2. Host para novas mutações;
3. conclui/aborta transacionalmente trabalho em andamento;
4. trata fila ainda não iniciada;
5. coordena operações longas/administrativas;
6. encerra WebSockets;
7. fecha SQLite/handles;
8. Host termina;
9. Controller confirma saída e termina.

`kill` forçado não é mecanismo normal.

## Backup / Restore — D11.1–D11.116 + D12.66–D12.75

### Invariantes

- Client nunca copia SQLite diretamente;
- backup representa SQLite + arquivos administrados como conjunto coerente;
- pacote `.stepflow-backup`, ZIP `Stored`, manifesto versionado e SHA-256;
- SQLite usa Online Backup API;
- Backup normal usa barrier curto;
- promoção same-volume/no-replace;
- Restore revalida provenance, paths, hashes e banco;
- schema antigo somente com migrations forward completas;
- sem down migration automática;
- safety backup confirmado obrigatório;
- ativação troca logicamente `data/`, preservando `old`;
- journal fora de `data/` e fresh Host após fase destrutiva;
- Restore destrutivo invalida sessões anteriores;
- `uncertain` bloqueia readiness;
- disaster recovery local/transitório;
- operações administrativas críticas são coordenadas.

### Parâmetros iniciais

```text
retention_max_confirmed_backups = 20
configurável = 5..100
max_entries = 10_000
max_total_payload = 8 GiB
max_managed_file = 16 MiB
max_logical_path = 512 UTF-16
max_path_component = 120 UTF-16
max_path_depth = 8
min_free_space_reserve = 1 GiB
backup_capture_target <= 2 s
backup_capture_hard_limit = 10 s
pre_restore_no_progress = 120 s
pre_restore_total_before_destructive = 10 min
```

Valor de retenção ausente usa 20; valor explicitamente inválido não deve sofrer clamp/fallback silencioso, conforme validação final do Bloco 12.

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

Depois da captura nenhuma mutação em SQLite/company/avatars é aceita até o primeiro rename ou abort seguro pré-destrutivo.

### Paths e provenance

- canonicalização Windows estrita;
- bloquear drive/UNC/device/ADS/reserved names/trailing dot-space/case collision/reparse/non-regular/escape;
- `manifest.json` inclui `source_deployment_id`;
- deployment diferente gera `source_mismatch` no baseline.

### Confiança e limites

- baseline sem criptografia application-level;
- baseline sem assinatura criptográfica application-level;
- SHA-256 é integridade, não autenticidade;
- trust boundary = root administrado + ACLs + infraestrutura de volume + deployment ID + auditoria.

Backup local não promete proteção contra perda física total, ransomware com acesso ao mesmo storage ou site loss. Offsite/cópia corporativa é responsabilidade operacional externa.

## Atualização e rollback

```text
encerrar Controller/Host
→ confirmar ausência de processos
→ backup quando exigido
→ substituir _internal/server/app/
→ preservar config/data/logs/backups
→ iniciar Controller
→ validar readiness
```

Rollback de binário só quando schema atual for compatível; caso contrário exige Restore correspondente.

## Limitação deliberada

Executar `.exe` por SMB na estação executa-o na estação; não cria processo remoto no servidor. Sem componente residente, o Controller precisa ser iniciado na máquina central ou por mecanismo corporativo já existente/aprovado.

## Fundação da Fase 2

D12.80–D12.98 implementam a fundação em tarefas separadas: Host mínimo, runtime/readiness, SQLite/migrations, Controller, Client, Launcher, packaging e smoke integrado.

Antes de produção permanecem gates reais de Windows/WebView2, SMB, filesystem/ACL/EDR/antivírus, long paths e crash/restart injection conforme fase aplicável.
