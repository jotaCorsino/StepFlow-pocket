# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1  
**Atualização:** 2026-09-01

Este arquivo é o **mapa arquitetural**. Contratos detalhados pertencem aos documentos específicos e não devem ser duplicados integralmente aqui.

## Visão geral

```text
Pasta StepFlow publicada no servidor Windows
        ↓
StepFlowLauncher.exe no compartilhamento
        ↓
preparação/validação local automática
        ↓
Client Tauri local por usuário
        ↓ HTTP/JSON + WebSocket
StepFlow Host Pocket na máquina central
        ↓
SQLite local + arquivos persistentes administrados
```

## Componentes

### Client

**Tauri 2 + HTML/CSS/JavaScript modular.**

Responsável por UI, sessão em memória, consumo da API, eventos/reconsulta, estados transversais, execução de Atendimento, preview/save/impressão local e integração Windows necessária.

O Client nunca acessa SQLite diretamente.

### Launcher

Executável Rust x64 pequeno/transitório no compartilhamento.

Responsável por validar manifesto/deployment, preparar versão local em `%LOCALAPPDATA%`, validar hashes/recursos, iniciar o Client local e encerrar.

Fonte: `launcher-distribuicao-client.md`.

### Controller / Host

**Rust + Tokio/Axum + `rusqlite` bundled.**

O Controller inicia/controla o Host sob demanda na máquina central. O Host concentra autenticação/autorização, API, WebSocket, SQLite, concorrência, domínio, auditoria, Backup/Restore e geração documental.

O Controller também pode:

- coordenar relaunch **bounded** do Host quando Restore destrutivo exige fresh Host;
- entrar em modo Recovery local/transitório quando o Host normal não consegue atingir readiness.

Nenhum desses fluxos cria watchdog geral, Windows Service ou daemon.

Fonte: `host-pocket.md`.

## Contrato Pocket

- pasta pronta na máquina central;
- zero instalador tradicional obrigatório por estação;
- preparação local automática do Client;
- zero preparação manual de dependência;
- zero elevação administrativa no uso normal;
- nenhuma toolchain em produção;
- nenhuma Internet obrigatória no uso normal;
- Client não roda permanentemente do SMB;
- sem serviço StepFlow persistente, scheduler, watchdog, tray ou daemon baseline;
- Controller/Host sob demanda;
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo.

WebView2 Evergreen compatível já presente é preferível. Fixed Version não roda por UNC/SMB; fallback autocontido local exige PoC sem instalação/admin/manualidade.

Fontes: `implantacao-pocket.md`, `launcher-distribuicao-client.md`, `compatibilidade-windows-client.md`.

## Domínio funcional

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimentos possuem revisões imutáveis;
- categorias são configuráveis/múltiplas;
- Atendimento é ocorrência real;
- Equipamento é opcional/reutilizável;
- vínculo preserva revisão exata;
- checklist persistente existe somente em Atendimento;
- `Observação do serviço` por Etapa existe somente em Atendimento;
- conclusão/reabertura precisa preservar reprodução histórica suficiente;
- Ficha compacta pode existir com ou sem Equipamento.

Fontes: `../01-produto/categorizacao-atendimentos-equipamentos.md` e `modelo-dados-schema-fase-1.md`.

## Lifecycle operacional

```text
rascunho Client
→ primeiro save Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Regras detalhadas ficam no Bloco 9 e nas Telas 05/08/09.

## Persistência

```text
StepFlow\
├── app\
├── config\
├── data\
│   ├── stepflow.sqlite
│   ├── company\
│   └── avatars\
├── logs\
└── backups\
```

Princípios:

- SQLite local ao Host;
- foreign keys + WAL;
- migrations versionadas;
- revisões imutáveis;
- `revision_no` separado de `display_version`;
- auditoria proporcional/append-only;
- dados/config não são substituídos com binários;
- backups e arquivos administrados permanecem separados dos artefatos substituíveis.

## Comunicação e concorrência

- HTTP/JSON versionado, inicialmente `/api/v1`;
- WebSocket autenticado para eventos;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- sem edição offline;
- evento sinaliza mudança; Client reconsulta;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso;
- `409` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- timeout após mutação exige reconciliação, não retry cego;
- manutenção de Restore pode encerrar WebSockets; desconexão nunca prova resultado;
- após fresh Host de Restore, Client autentica novamente e reconsulta estado.

Fontes: `comunicacao-client-host.md` e `concorrencia-fila-conflitos-eventos.md`.

## Autenticação e autorização

- Argon2id;
- sessão opaca server-side;
- token em memória;
- autorização Host-side por capacidade;
- ADM/Gerência/Funcionário como presets;
- bootstrap do primeiro ADM local/controlado;
- sessão expirada exige nova autenticação;
- Backup = ADM/Gerência;
- Restore = ADM-only;
- Restore destrutivo invalida todas as sessões/tokens anteriores, inclusive em rollback;
- conteúdo restaurado nunca ressuscita token antigo;
- parâmetros numéricos finais permanecem pendentes.

Fonte: `autenticacao-sessao-autorizacao.md`.

## Geração documental

```text
Client solicita fonte + revisão esperada
→ Host autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ renderiza em capacidade bounded
→ devolve bytes
→ Client salva / pré-visualiza / imprime
```

Contratos consolidados:

- PDF de Procedimentos via Typst embutido;
- DOCX OOXML direto em Rust sob adaptador;
- impressão Windows usa o mesmo PDF oficial via WebView2;
- Procedimento físico A4 multipágina;
- Ficha compacta deriva PDF + preview do mesmo `PagedDocument`;
- Ficha válida possui exatamente uma A4; `2+` páginas = `SHEET_OVERFLOW`;
- naming persistente e temporários têm lifecycles separados;
- artefatos gerados não entram em histórico/backup por padrão.

Fontes: Bloco 10 e Tela 14.

## Backup / Restore — D11.1–D11.116

UX: `../02-telas/13-backup-restauracao.md`.  
Mapa técnico: `../04-planejamento/bloco-11-backup-restauracao.md`.

### Estado e envelope

- estado recuperável = `stepflow.sqlite + company/** + avatars/**`;
- binários/config/logs/backups/exportações/temporários ficam fora;
- pacote único imutável `.stepflow-backup`, ZIP `Stored`;
- manifesto versionado + SHA-256 por entrada;
- SQLite via Online Backup API;
- staging antes da promoção;
- pacote parcial nunca é válido.

### Consistência

Backup normal usa barrier curto:

```text
suspender/drain mutações
→ capturar SQLite + arquivos administrados
→ liberar mutações
→ hash/ZIP/verificação/promoção
```

`-wal`/`-shm` não entram no pacote; criação exige `quick_check = ok` + `foreign_key_check` vazio; promoção final é same-volume/no-replace.

### Catálogo/retention/coordinator

- catálogo reconstruível sem depender do banco ativo;
- `backup_id` é identidade;
- Restore sempre revalida integralmente;
- retenção sem scheduler e por quantidade;
- lease exclusivo coordena Backup/Restore/Migration;
- backups protegidos/`uncertain` não sofrem cleanup destrutivo.

### Restore

- candidato em `data-next/` same-volume;
- `integrity_check = ok` + `foreign_key_check` vazio;
- schema antigo somente com migrations forward completas no staging;
- schema novo/cadeia incompleta = incompatível;
- sem down migration automática;
- safety backup confirmado obrigatório;
- ativação = `data → old`, `data-next → data`;
- `old` permanece até validação;
- rollback conhecido ou `uncertain`.

### Safety barrier final

No `pre_restore`, o barrier permanece desde a captura do safety backup até o primeiro rename. Nenhuma mutação em `data/` ocorre nesse intervalo; journal/admin-audit externos permanecem permitidos.

Antes de `DESTRUCTIVE_STARTED`, o digest/schema/root de `data-next/` é revalidado.

### Restart e recovery

- journal vive fora de `data/`;
- fresh Host reconcilia antes de migrations/readiness;
- queda entre renames retorna para `old` quando comprovável;
- estado ambíguo = `RECOVERY_REQUIRED/uncertain`;
- relaunch de Restore é bounded;
- fase destrutiva invalida sessões antigas;
- `uncertain` bloqueia readiness/mutações/cleanup.

### Disaster recovery

- modo Recovery local/transitório do Controller;
- sem listener HTTP/WebSocket normal;
- exclusividade da implantação;
- autoridade por acesso local/ACL quando o banco não autentica;
- mesma integridade/compatibilidade do Restore normal;
- ausência de safety backup só é aceita em disaster recovery real;
- fresh Host normal precisa atingir readiness antes de retorno ao uso.

### Auditoria

- auditoria funcional quando possível;
- trilha administrativa estruturada fora de `data/`, baseline `logs/admin-audit.jsonl`/equivalente;
- journal, admin audit e logs técnicos são mecanismos distintos;
- não registrar senha, token reutilizável, dump SQL ou conteúdo de negócio desnecessário.

### Paths, provenance e limites

- paths lógicos seguem canonicalização Windows estrita;
- bloquear drive/UNC/device/ADS/reserved names/trailing dot-space/case collision/reparse/non-regular/escape do root;
- criação do Backup aplica a mesma disciplina aos arquivos administrados;
- manifesto inclui `source_deployment_id`;
- pacote de deployment diferente é `source_mismatch` no baseline;
- parser/extração são bounded e fazem preflight de espaço;
- números finais ficam no Bloco 12.

### Criptografia/autenticidade

- baseline sem criptografia application-level;
- baseline sem assinatura criptográfica application-level;
- SHA-256 representa integridade, não autenticidade;
- trust boundary = root administrado + ACLs + infraestrutura de volume + deployment ID + auditoria.

### Limite operacional e gates

Backup local não promete proteção contra perda física total/ransomware/site loss; offsite/cópia corporativa é responsabilidade operacional externa.

Antes de produção são gates obrigatórios: adapter Win32, filesystem real, ACLs, EDR/antivírus, long paths, espaço, performance e crash injection.

**Não existe bloqueador arquitetural conhecido para o Bloco 11.**

## Pendências arquiteturais ainda abertas

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- regra editorial de categoria arquivada;
- parâmetros numéricos reservados ao Bloco 12;
- estrutura oficial da implementação e plano da Fase 2;
- gates de ambiente real: Windows/WebView2, Launcher/SMB, Word, impressoras, filesystem/ACL/EDR.

Nenhum runtime/código funcional oficial foi criado durante a Fase 1.
