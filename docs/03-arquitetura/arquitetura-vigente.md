# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1, INCLUINDO BLOCO 9 E BLOCO 10 / ETAPA 1  
**Atualização:** 2026-08-25

## Visão geral

```text
Ponto de entrada interno
        ↓
StepFlowLauncher.exe (transitório)
        ↓
Client Tauri local por usuário
        ↓ HTTP/JSON + WebSocket
StepFlow Host Pocket na máquina central
        ↓
SQLite local + arquivos persistentes
```

## Client

Tecnologia: **Tauri 2 + HTML/CSS/JavaScript modular**.

Responsabilidades:

- UI/navegação;
- sessão em memória;
- consumir API do Host;
- receber eventos e reconsultar estado;
- apresentar conflitos/estados transversais;
- executar contexto operacional de Atendimento sem abrir SQLite diretamente;
- iniciar geração documental e receber artefatos produzidos pelo Host;
- encaminhar artefatos para fluxos locais de salvar/preview/impressão conforme os contratos específicos.

Baseline inicial: Windows 10/11 x64 + WebView2. Validação corporativa ainda pendente.

## Launcher

Launcher Rust x64 portátil/transitório:

1. lê manifesto/configuração;
2. compara versão;
3. prepara `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
4. valida SHA-256;
5. inicia Client local;
6. encerra.

Sem updater residente.

## Host Pocket

Tecnologia: Rust + Tokio/Axum + `rusqlite` bundled.

- Controller: lifecycle central, paths/config, instância única, readiness/shutdown;
- Host: autenticação, autorização, API, eventos, SQLite, writer/fila, revisões, Atendimentos, Equipamentos, checklist, auditoria, backup/restore e geração documental.

Sem Windows Service, Task Scheduler, auto-start, watchdog ou daemon StepFlow como padrão.

Fechar um Client individual não encerra o Host. Encerrado o ciclo central, nenhum processo StepFlow permanece ativo.

## Domínio funcional

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Consolidado:

- Procedimentos oficiais/revisões imutáveis;
- categorias configuráveis/múltiplas;
- Atendimentos como ocorrências reais;
- Equipamento opcional/reutilizável;
- busca documental separada da operacional;
- múltiplas revisões de Procedimento por Atendimento;
- revisão exata usada preservada;
- checklist persistente somente em contexto de Atendimento;
- ficha compacta com ou sem Equipamento;
- identidade da empresa centralizada;
- Backup/Restore administrativo;
- exportação contextual pela revisão selecionada.

## Lifecycle operacional — Bloco 9

Estados iniciais:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho Client
→ primeiro save aceito pelo Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Regras:

- rascunho inicial não persiste;
- Host gera `AT-000001` no primeiro save;
- conclusão exige responsável + resumo do trabalho;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo curto;
- Concluído/Cancelado não aceitam edição direta;
- mudança posterior exige reabertura;
- lifecycle é auditável/versionado.

## Equipamento operacional

Código legível inicial:

```text
EQP-000001
```

- Host-only;
- ID interno continua canônico;
- serial/MAC/patrimônio são atributos de busca;
- múltiplos MACs;
- não arquivar Equipamento ligado a Atendimento `Em andamento`;
- Funcionário pode criar/editar Equipamento por preset;
- arquivar/reativar: ADM/Gerência.

Ao concluir Atendimento, o Host preserva uma projeção histórica relevante do Equipamento. Alteração futura do cadastro global não reescreve ficha/histórico concluído.

## Checklist operacional

A definição continua em `process_revision`/blocos imutáveis.

Estado de execução é separado e ligado a `service_record_process`.

- somente contexto de Atendimento persiste marcações;
- Reader standalone continua documental;
- progresso = itens marcados / total;
- etapa visitada não é progresso;
- 100% não conclui Atendimento automaticamente;
- checklist concluído/cancelado é somente leitura até reabertura;
- concorrência por item/equivalente evita conflito global desnecessário.

## Responsabilidade e permissões operacionais

Autorização continua Host-side e por capacidade.

Preset:

- ADM/Gerência: podem criar/editar/concluir/cancelar/reabrir Atendimentos acessíveis;
- Funcionário: cria e opera Atendimento do qual é responsável, podendo concluir o próprio;
- Funcionário não cancela/reabre por preset;
- Gerência gere categorias por preset;
- Funcionário seleciona revisão publicada para execução;
- ADM/Gerência podem selecionar explicitamente outras revisões autorizadas;
- gerar/reimprimir ficha: sim para os três presets em Atendimento acessível.

Continuam pendentes:

- Gerência × configuração da empresa;
- Gerência × Backup.

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
- foreign keys;
- WAL;
- migrations versionadas;
- revisões de Procedimento imutáveis;
- `revision_no` separado de `display_version`;
- auditoria append-only;
- categorias/Equipamentos/Atendimentos no schema conceitual;
- checklist operacional separado do snapshot documental;
- logo/avatar como arquivos controlados pelo Host;
- dados/config não são substituídos com binários.

## Comunicação

- HTTP/JSON versionado, inicialmente `/api/v1`;
- WebSocket autenticado para eventos;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- sem edição offline;
- evento = sinal de mudança, não estado oficial completo;
- WebSocket degradado com HTTP saudável não implica Host totalmente indisponível;
- reconexão faz reconsulta/reconciliação.

## Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso;
- `409` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir;
- checklist usa granularidade por item/equivalente;
- Atendimento/Equipamento têm revisões separadas;
- timeout após mutação exige reconciliação, não retry cego;
- geração documental é leitura derivada e não passa pela fila de mutações;
- renderização documental usa limite próprio de concorrência/backpressure.

## Autenticação

- Argon2id;
- sessão opaca server-side;
- token em memória;
- capacidade Host-side;
- ADM/Gerência/Funcionário como presets;
- Gerência não administra ADM;
- bootstrap do primeiro ADM é local/controlado;
- sessão expirada exige nova autenticação;
- troca da própria senha mantém sessão atual e revoga demais sessões da conta.

Parâmetros numéricos finais ainda pendentes antes da implementação.

## Exportação e impressão

### Arquitetura de geração documental — Bloco 10 / Etapa 1

Consolidado:

```text
Client
  ↓ solicita por identidade da fonte + revisão esperada
Host
  ↓ autentica/autoriza
  ↓ captura snapshot consistente
  ↓ materializa DocumentModel semântico
  ↓ encerra leitura/transação SQLite
  ↓ renderiza fora da fila de mutações
  ↓ devolve artefato pela API autenticada
Client
  ↓ recebe
  └─→ destino local/preview/impressão conforme etapas específicas
```

Regras:

- geração é responsabilidade do Host;
- Client não envia documento montado nem usa DOM/HTML como fonte;
- fonte mutável usa revisão esperada; estado mais novo não substitui silenciosamente o confirmado pelo Client;
- `DocumentModel` semântico separa domínio de renderers;
- renderers não reconsultam SQLite nem reconstruem regras de negócio;
- captura consistente termina antes do trabalho pesado de renderização;
- geração não cria revisão, não altera Atendimento/checklist e não muda `updated_at` funcional;
- limite de renderização é separado do writer;
- primeira versão não cria `export_jobs`, scheduler ou fila persistente documental;
- artefato retorna pela API autenticada; Host não escreve em path arbitrário do Client;
- artefatos gerados não viram histórico/backup por padrão;
- runtime documental não depende de Office, LibreOffice, Adobe Reader, Chrome/Chromium externo headless, `wkhtmltopdf` ou serviço cloud obrigatório;
- bibliotecas compiladas com o Host podem ser usadas;
- endpoints, engines e parâmetros exatos pertencem às etapas/implementação correspondentes.

### Procedimentos

- PDF, DOCX e impressão obrigatórios;
- fonte = revisão selecionada;
- revisão nova não substitui geração em andamento;
- documento próprio, não screenshot;
- histórico/draft autorizado recebe identificação inequívoca;
- identidade central da empresa.

### Ficha de Atendimento

- estado confirmado do Host;
- com ou sem Equipamento;
- `Em andamento`: geração para acompanhamento;
- `Concluído`: reimpressão do estado histórico aplicável;
- `Cancelado`: identificação inequívoca do status;
- máximo uma página A4;
- conteúdo excessivo bloqueia saída em vez de segunda página/truncamento silencioso;
- impressão é requisito;
- DOCX específico não é requisito inicial.

Bloco 10 ainda fecha, uma etapa por vez, engine PDF, DOCX, impressão Windows, templates, limites, preview, PDF específico da ficha, tratamento de muitos MACs/Procedimentos, nomes de arquivo/temporários e QR/barcode.

## Backup / Restore

UX normal já consolidada:

- dentro de Configurações;
- Client não copia SQLite nem escolhe path;
- backups conhecidos pelo Host;
- Restore exige autorização + backup elegível + confirmação reforçada;
- Restore normal exige safety backup do estado atual antes da etapa destrutiva;
- disaster recovery sem Host funcional fica fora da UI normal.

Bloco 11 ainda fecha pacote, atomicidade, checksums, retenção, restart/reconexão, sessões e recuperação local.

## Estados transversais

- menor superfície adequada: campo → seção → página → Shell;
- sem indicador permanente de conexão saudável;
- loading não apresenta cache antigo como atual;
- `sem registros` ≠ `sem resultados`;
- perda de permissão limpa conteúdo protegido;
- conflito preserva edição local;
- incompatibilidade Client↔Host bloqueia uso;
- sem offline queue/autosave/draft persistente.

## Ambiente corporativo pendente

- hostname/IP/paths reais;
- SMB/permissões;
- Windows/WebView2 reais;
- HTTP/HTTPS;
- antivírus/EDR/firewall;
- mecanismo real de start do Controller.

Exemplos históricos não podem virar hardcode.

## Estado da Fase 1

- Blocos 0–4: concluídos;
- Bloco 5: núcleo concluído, parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Bloco 8: concluído;
- Bloco 9: concluído documentalmente;
- **Bloco 10: em andamento — Etapa 1 consolidada; Etapa 2 próxima, ainda não aberta**;
- Blocos 11–12: pendentes.

Nenhum runtime/código funcional oficial foi criado durante esse fechamento documental.