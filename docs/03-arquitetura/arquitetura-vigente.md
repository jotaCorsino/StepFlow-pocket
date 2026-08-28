# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1, INCLUINDO BLOCO 9 E BLOCO 10 / ETAPAS 1–7  
**Atualização:** 2026-08-28

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
- executar Atendimento sem abrir SQLite diretamente;
- iniciar geração documental e receber artefatos produzidos pelo Host;
- salvar/preview/imprimir localmente conforme contratos específicos;
- realizar impressão física no contexto Windows da estação usando o PDF oficial recebido do Host.

Direção visual transversal: clareza com baixa densidade textual permanente, usando cor, forma, símbolo, posição e ícones reconhecíveis quando isso simplificar sem gerar ambiguidade; detalhes secundários podem aparecer sob demanda e cor nunca é o único canal de um estado importante.

Baseline inicial: Windows 10/11 x64 + WebView2. Validação corporativa real ainda pendente.

## Launcher e ciclo Pocket

Launcher Rust x64 portátil/transitório:

1. lê manifesto/configuração;
2. compara versão;
3. prepara `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
4. valida SHA-256;
5. inicia Client local;
6. encerra.

Sem updater residente. A máquina central recebe pasta pronta; não exige toolchain de desenvolvimento.

O Controller inicia/controla o Host. Sem Windows Service, Task Scheduler, auto-start, watchdog, tray agent ou daemon como padrão. Fechar um Client individual não encerra o Host; encerrado o ciclo central, nenhum processo StepFlow permanece ativo.

## Host Pocket

Tecnologia: **Rust + Tokio/Axum + `rusqlite` bundled**.

Responsabilidades:

- autenticação/autorização;
- API HTTP/JSON + WebSocket;
- SQLite Host-only;
- writer/fila/backpressure;
- Procedimentos/revisões;
- Atendimentos e Equipamentos;
- checklist operacional;
- observações de serviço por Etapa;
- auditoria;
- Backup/Restore;
- geração documental.

## Domínio funcional

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Consolidado:

- Procedimentos oficiais com revisões imutáveis;
- categorias configuráveis/múltiplas;
- Atendimentos como ocorrências reais;
- Equipamento opcional/reutilizável;
- busca documental separada da operacional;
- revisão exata usada preservada;
- checklist persistente apenas em Atendimento;
- observação de serviço opcional por Etapa apenas em Atendimento;
- Ficha compacta com ou sem Equipamento;
- Ficha como prestação de contas resumida ao cliente;
- identidade da empresa centralizada;
- Backup/Restore administrativo;
- exportação contextual pela revisão/estado selecionado.

## Lifecycle operacional

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

- rascunho inicial não persiste;
- Host gera `AT-000001` no primeiro save;
- conclusão exige responsável + resumo do trabalho;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo curto;
- Concluído/Cancelado não aceitam edição direta;
- mudança posterior exige reabertura;
- lifecycle é auditável/versionado;
- estado final necessário à reimpressão histórica não pode ser reescrito silenciosamente após reabertura.

## Equipamento, checklist e observações de serviço

Equipamento:

- código legível `EQP-000001`;
- ID interno continua canônico;
- serial/MAC/patrimônio são atributos de busca;
- múltiplos MACs;
- não arquivar Equipamento ligado a Atendimento `Em andamento`;
- Funcionário pode criar/editar por preset; arquivar/reativar fica ADM/Gerência;
- conclusão preserva projeção histórica relevante do Equipamento.

Checklist:

- definição permanece no Procedimento imutável;
- estado de execução fica separado e ligado a `service_record_process`;
- somente Atendimento persiste marcações;
- progresso = itens marcados / total;
- etapa visitada não é progresso;
- 100% não conclui automaticamente;
- concorrência granular por item/equivalente.

Observação do serviço por Etapa:

- texto opcional de execução ligado ao Atendimento + vínculo da revisão + Etapa;
- não altera o Procedimento oficial;
- Reader standalone não persiste esse dado;
- editável somente enquanto Atendimento estiver editável/autorizado;
- Concluído/Cancelado = somente leitura até reabertura;
- concorrência granular por Etapa/equivalente;
- conclusão preserva estado histórico suficiente para reimpressão da Ficha;
- sem autosave por inferência.

## Permissões operacionais

Autorização continua Host-side por capacidade.

Preset:

- ADM/Gerência operam amplamente Atendimentos acessíveis;
- Funcionário cria/opera Atendimento do qual é responsável e pode concluir o próprio;
- Funcionário não cancela/reabre por preset;
- Gerência gere categorias;
- Funcionário seleciona revisão publicada;
- ADM/Gerência podem selecionar explicitamente outras revisões autorizadas;
- gerar/reimprimir Ficha: sim para os três presets em Atendimento acessível.

Pendentes: Gerência × configuração da empresa e Gerência × Backup.

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
- revisões de Procedimento imutáveis;
- `revision_no` separado de `display_version`;
- auditoria append-only;
- checklist e observações operacionais separados do snapshot documental;
- dados/config não são substituídos com binários.

## Comunicação e concorrência

- HTTP/JSON versionado, inicialmente `/api/v1`;
- WebSocket autenticado para eventos;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- sem edição offline;
- evento = sinal de mudança, não estado oficial completo;
- reconexão faz reconsulta/reconciliação.

Concorrência:

- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso;
- `409` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- sem soft/hard lock inicial;
- checklist usa granularidade por item/equivalente;
- observação de serviço usa granularidade por Etapa/equivalente;
- Atendimento/Equipamento têm revisões separadas;
- timeout após mutação exige reconciliação, não retry cego;
- geração documental é leitura derivada fora da fila de mutações;
- renderização documental usa limite próprio de concorrência/backpressure.

## Autenticação

- Argon2id;
- sessão opaca server-side;
- token em memória;
- capacidade Host-side;
- ADM/Gerência/Funcionário como presets;
- Gerência não administra ADM;
- bootstrap do primeiro ADM local/controlado;
- sessão expirada exige nova autenticação;
- troca da própria senha mantém sessão atual e revoga demais sessões da conta.

Parâmetros numéricos finais permanecem pendentes antes da implementação.

# Exportação e impressão — Bloco 10

## Etapa 1 — geração documental

```text
Client
→ solicita identidade da fonte + revisão esperada
Host
→ autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ renderiza fora da fila de mutações
→ devolve artefato
Client
→ salva / preview / imprime
```

- Client não envia documento montado nem DOM/HTML;
- renderers não reconsultam SQLite;
- sem `export_jobs` persistentes inicialmente;
- artefato retorna pela API autenticada;
- Host não grava em path arbitrário do Client;
- artefatos não viram histórico/backup por padrão;
- runtime não depende operacionalmente de Office, LibreOffice, Adobe Reader, browser externo/headless ou cloud obrigatória.

## Etapa 2 — PDF de Procedimentos

- Typst embutido como biblioteca Rust via crates oficiais + adaptador interno;
- template interno, conteúdo apenas como dados estruturados;
- mundo virtual restringe imports/filesystem/fontes/assets e recursos remotos;
- PDF 1.7 + Tagged PDF baseline, sem promessa formal PDF/A ou PDF/UA;
- fontes incorporadas/subsetadas;
- texto real selecionável/pesquisável;
- todos os blocos conhecidos são representados ou falham explicitamente;
- fluxo multipágina automático, sem truncamento;
- PNG/JPEG/SVG controlados;
- falha nunca devolve artefato parcial como sucesso.

## Etapa 3 — DOCX de Procedimentos

- DOCX real OOXML/WordprocessingML/OPC, baseline OOXML Transitional;
- geração direta Rust a partir do mesmo `DocumentModel`;
- `docx-rs` preferido sob adaptador interno;
- sem Word/COM, LibreOffice, browser/headless ou cloud;
- sem XML/OOXML/relationships arbitrários nem template externo v1;
- texto e numeração permanecem editáveis;
- Arial/Consolas referenciadas sem embedding v1;
- DOCX é refluível e não promete paginação idêntica ao PDF;
- pacote incompleto/corrompido nunca é sucesso.

## Etapa 4 — impressão Windows de Procedimentos

```text
PDF oficial
→ Client Windows
→ recurso local transitório
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo Windows
```

- usa o mesmo PDF oficial;
- não imprime HTML da UI nem DOCX;
- sem software externo, seletor próprio ou impressão silenciosa como baseline;
- StepFlow não gerencia drivers/spooler;
- sucesso = entrega ao Windows, não confirmação física de papel;
- detalhes do temporário ficam para Etapa 10.

## Etapa 5 — template físico de Procedimentos

- Reader diário não possui geometria A4;
- Procedimento físico usa A4 retrato multipágina com margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- paginação automática sem truncamento/redução silenciosa;
- PDF usa Noto Sans/Noto Sans Mono incorporadas;
- DOCX referencia Arial/Consolas;
- limite de uma A4 pertence somente à Ficha.

## Etapa 6 — PDF + preview da Ficha

- Ficha = prestação de contas resumida ao cliente;
- prioriza identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações;
- PDF próprio/canônico via template Typst da Ficha;
- PDF e preview SVG derivam do mesmo `PagedDocument`;
- resultado válido exige exatamente uma página;
- `2+ páginas` = `SHEET_OVERFLOW`, sem corte/segunda folha/redução silenciosa;
- preview em modal/overlay simples;
- Salvar/Imprimir reutilizam os mesmos bytes PDF;
- impressão reutiliza WebView2 + `ShowPrintUI(System)`;
- PDF/SVG são transitórios e presos à `source_version`.

## Etapa 7 — template físico A4 da Ficha

Geometria:

```text
A4 retrato
exatamente 1 página
margens 15 mm
sem bleed
```

Ordem física:

```text
1. identidade da empresa + Atendimento
2. identificação curta do serviço
3. Equipamento, quando houver
4. Serviço realizado
5. Observações, quando houver
```

Contrato:

- composição predominantemente vertical/uma coluna;
- cabeçalho institucional compacto, logo opcional, sem título gigante e sem footer obrigatório;
- status proporcional: `CANCELADO` sempre textual/inequívoco; acompanhamento discreto;
- cliente/OS/técnico em linha curta, omitindo vazios;
- Equipamento em ficha técnica resumida sem grade/tabela pesada;
- `SERVIÇO REALIZADO` usa `Resumo do trabalho` como área narrativa principal;
- `OBSERVAÇÕES` unifica observações relevantes do Atendimento, Equipamento e Etapas em lista simples;
- nome curto da Etapa só aparece quando necessário para contexto;
- seções vazias colapsam completamente;
- PDF usa Noto Sans: 14 pt identificação principal, 10,5 pt seção, 10 pt corpo, 9 pt ficha técnica, 8,5 pt metadados;
- divisórias discretas e contraste neutro legível em monocromático;
- sem caixas vazias para escrita manual, assinatura, financeiro, garantia, checklist, progresso, timeline, QR/barcode, lista detalhada de Procedimentos, página 2 ou footer promocional;
- nenhuma redução dinâmica de fonte para caber; overflow continua `SHEET_OVERFLOW`.

Etapa 8 definirá limites/priorização/densidade textual. Etapa 9 tratará dados multiplicativos/excepcionais.

## Backup / Restore

UX já consolidada:

- dentro de Configurações;
- Host coordena;
- Client não escolhe SQLite/path;
- Restore exige autorização + backup elegível + confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva normal;
- disaster recovery sem Host funcional fica no Bloco 11.

## Estado da Fase 1

- Blocos 0–4: concluídos;
- Bloco 5: núcleo concluído, parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Blocos 8–9: concluídos;
- **Bloco 10: em andamento — Etapas 1–7 consolidadas; Etapa 8 próxima, ainda não aberta**;
- Blocos 11–12: pendentes.

Nenhum runtime/código funcional oficial foi criado durante esse fechamento documental.
