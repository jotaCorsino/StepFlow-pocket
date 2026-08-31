# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1 — BLOCO 11 EM ANÁLISE  
**Atualização:** 2026-08-31

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

O Controller também pode coordenar um relaunch **bounded** do Host quando um Restore já entrou na fase destrutiva e exige fresh Host para recovery/finalização; isso não é watchdog geral.

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

Fontes:

- `implantacao-pocket.md`;
- `launcher-distribuicao-client.md`;
- `compatibilidade-windows-client.md`.

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

Estrutura conceitual:

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
- checklist e observações usam granularidade própria;
- timeout após mutação exige reconciliação, não retry cego;
- manutenção de Restore pode encerrar WebSockets; desconexão nunca prova resultado;
- após fresh Host de Restore, Client precisa autenticar novamente e reconsultar estado.

Fontes: `comunicacao-client-host.md` e `concorrencia-fila-conflitos-eventos.md`.

## Autenticação e autorização

- Argon2id;
- sessão opaca server-side;
- token em memória;
- autorização Host-side por capacidade;
- ADM/Gerência/Funcionário como presets;
- bootstrap do primeiro ADM local/controlado;
- sessão expirada exige nova autenticação;
- Restore que entra na fase destrutiva invalida todas as sessões/tokens anteriores, inclusive se houver rollback;
- conteúdo restaurado nunca ressuscita token antigo;
- parâmetros numéricos finais permanecem pendentes;
- Gerência × Backup continua pendente até aprovação da Análise 6.

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

Fontes:

- `../04-planejamento/bloco-10-exportacao-impressao-ficha.md`;
- `../04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`;
- `../02-telas/14-exportacao-impressao-ficha.md`.

## Backup / Restore — decisões aprovadas no Bloco 11

UX continua em `../02-telas/13-backup-restauracao.md`.

### Estado, envelope e consistência

- estado recuperável = `stepflow.sqlite + company/** + avatars/**`;
- binários/config/logs/backups/exportações/temporários ficam fora;
- pacote único imutável `.stepflow-backup`, ZIP `Stored`, manifesto versionado e SHA-256;
- SQLite via Online Backup API;
- barrier curto sobre mutações captura banco + arquivos no mesmo ponto lógico;
- `-wal`/`-shm` não entram no pacote;
- criação exige `quick_check = ok` + `foreign_key_check` vazio;
- hash/ZIP/verificação/promoção ficam fora do barrier;
- promoção final same-volume/no-replace;
- parcial/crash nunca vira backup válido.

### Catálogo, retenção e coordenação

- catálogo reconstruído dos pacotes finais e independente do banco ativo;
- `backup_id` é identidade canônica;
- cache de verificação somente em memória; Restore sempre revalida;
- retenção sem scheduler e por quantidade;
- source/safety/pre-migration em uso ou resultado incerto ficam protegidos;
- pacote inválido/corrompido não é apagado silenciosamente;
- lease exclusivo coordena `BACKUP`, `RESTORE`, `MIGRATION`;
- safety/pre-migration backup são suboperações do lease raiz;
- `uncertain` suspende cleanup destrutivo/retenção.

### Restore e compatibilidade

- Restore revalida envelope, hashes e SQLite;
- prepara `data-next/` same-volume, nunca sobre `data/` ativo;
- exige `integrity_check = ok` + `foreign_key_check` vazio;
- compatibilidade = `format_version + schema/migration path`;
- schema antigo somente com migrations forward completas no staging;
- schema mais novo ou cadeia incompleta é incompatível;
- sem down migration automática;
- safety backup confirmado antes da fase destrutiva;
- cancelamento termina antes da primeira alteração física do `data/`;
- ativação usa `data → .restore-old-<id>` e `data-next → data`;
- `old` permanece até validação;
- falha reversível retorna ao estado anterior; estado não comprovável = `uncertain`.

### Restart, sessões, reconexão e falhas

- journal de Restore vive fora de `data/`, baseline `backups/.operations/restore-active.json`;
- journal registra fase/IDs/schema/digest sem segredos e é atualizado antes da ação física correspondente;
- fresh Host reconcilia Restore **antes** de migrations/readiness normais;
- digest determinístico identifica o candidato preparado;
- queda antes da primeira troca preserva estado original;
- queda entre `data→old` e `data-next→data` causa rollback para `old`;
- combinação journal/filesystem não comprovável = `RECOVERY_REQUIRED/uncertain`;
- Restore aplicado ou rollback após fase destrutiva exige fresh Host;
- Controller pode executar relaunch bounded, sem watchdog geral;
- fase destrutiva invalida sessões anteriores;
- WebSocket de manutenção é best-effort e Clients fazem novo login após fresh Host;
- `restore-last.json`/equivalente preserva resultado terminal mínimo;
- `uncertain` bloqueia readiness, mutações, nova operação destrutiva, retenção e cleanup.

Fontes:

- `../04-planejamento/bloco-11-backup-restauracao.md`;
- `../04-planejamento/bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- `../04-planejamento/bloco-11-analise-4-restore-safety-compatibilidade.md`;
- `../04-planejamento/bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`.

### Em análise — não contrato

A Análise 6 propõe fechar disaster recovery local, Gerência × Backup e auditoria administrativa que atravesse Restore. Detalhe em `../04-planejamento/bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md`.

## Pendências arquiteturais ainda abertas

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup — em revisão na Análise 6;
- regra editorial de categoria arquivada;
- restante do Bloco 11: disaster recovery, capacidades/auditoria e validação técnica final;
- estrutura oficial da implementação e plano da Fase 2;
- gates de ambiente real: Windows/WebView2, Launcher/SMB, Word, impressoras, ACL/filesystem e EDR.

Nenhum runtime/código funcional oficial foi criado durante a Fase 1.