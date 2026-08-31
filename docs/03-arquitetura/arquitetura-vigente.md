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
- timeout após mutação exige reconciliação, não retry cego.

Fontes: `comunicacao-client-host.md` e `concorrencia-fila-conflitos-eventos.md`.

## Autenticação e autorização

- Argon2id;
- sessão opaca server-side;
- token em memória;
- autorização Host-side por capacidade;
- ADM/Gerência/Funcionário como presets;
- bootstrap do primeiro ADM local/controlado;
- sessão expirada exige nova autenticação;
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

Fontes:

- `../04-planejamento/bloco-10-exportacao-impressao-ficha.md`;
- `../04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`;
- `../02-telas/14-exportacao-impressao-ficha.md`.

## Backup / Restore — decisões aprovadas no Bloco 11

UX continua em `../02-telas/13-backup-restauracao.md`.

Estado recuperável inicial:

```text
stepflow.sqlite
company/**
avatars/**
```

Ficam fora do Backup normal: binários, configuração de implantação, logs, backups anteriores, exportações e temporários.

Envelope:

- um pacote imutável `.stepflow-backup`;
- ZIP `Stored` no baseline;
- `manifest.json` versionado;
- SHA-256 por entrada;
- paths lógicos allowlisted;
- staging antes de promoção;
- pacote parcial nunca é válido.

Consistência:

```text
Host entra em BACKUP_CAPTURE
→ drena mutações aceitas
→ ponto quiescente
→ Online Backup API para SQLite novo em staging
→ copia company/** + avatars/** no mesmo barrier
→ libera mutações
→ hash/ZIP/verificação/promoção fora do barrier
```

- `-wal`/`-shm` não entram no pacote;
- criação exige `quick_check = ok` e `foreign_key_check` vazio;
- candidato recebe flush explícito;
- promoção final é same-volume e no-replace;
- sucesso somente após arquivo final reaberto/confirmado;
- crash/falha nunca transforma staging/parcial em válido.

Catálogo/retenção/coordenação:

- catálogo é reconstruído dos pacotes finais e não depende do banco ativo;
- `backup_id` é identidade canônica;
- cache de verificação é somente em memória e Restore sempre revalida o pacote;
- retenção não usa scheduler e é inicialmente por quantidade;
- source/safety/pre-migration em uso ou resultado incerto ficam protegidos;
- pacote inválido/corrompido não é apagado silenciosamente pela retenção;
- Host possui lease administrativo exclusivo para `BACKUP`, `RESTORE` e `MIGRATION`;
- safety/pre-migration backup reutilizam a pipeline como suboperações do lease raiz;
- `uncertain` suspende cleanup destrutivo/retenção.

Restore normal, compatibilidade, sessões, disaster recovery e capacidades ainda estão sendo fechados. A Análise 4 propõe preparar `data-next/`, validar `integrity_check + foreign_key_check`, permitir somente migrations forward completas em staging, criar safety backup confirmado e ativar o conjunto `data/` por troca controlada same-volume; essa proposta ainda não é contrato.

Fontes:

- `../04-planejamento/bloco-11-backup-restauracao.md`;
- `../04-planejamento/bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- `../04-planejamento/bloco-11-analise-4-restore-safety-compatibilidade.md` — proposta em revisão.

## Pendências arquiteturais ainda abertas

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- restante do contrato técnico do Bloco 11: Restore/compatibilidade, restart/sessões/falhas, disaster recovery, capacidades/auditoria e validação final;
- estrutura oficial da implementação e plano da Fase 2;
- gates de ambiente real: Windows/WebView2, Launcher/SMB, Word, impressoras e EDR.

Nenhum runtime/código funcional oficial foi criado durante a Fase 1.
