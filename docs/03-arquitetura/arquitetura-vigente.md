# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1, INCLUINDO BLOCOS 9 E 10 / ETAPAS 1–11  
**Atualização:** 2026-08-29

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
SQLite local + arquivos persistentes
```

## Contrato Pocket

A arquitetura deve permitir copiar/mover a pasta pronta para o servidor Windows e usar o sistema pelas estações autorizadas sem instalação individual do aplicativo.

Requisitos:

- Launcher é o ponto de entrada no compartilhamento;
- Client operacional fica em `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
- preparação/atualização local é automática;
- sem MSI/MSIX/NSIS obrigatório por estação;
- sem configuração manual de dependência;
- sem privilégio administrativo no uso normal;
- sem toolchain de desenvolvimento nas estações ou máquina central de produção;
- sem Internet obrigatória para uso normal;
- sem Client rodando permanentemente do SMB;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon StepFlow como baseline;
- Controller/Host sob demanda na máquina central;
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo.

Se uma dependência exigir instalação/elevação/manualidade por estação, a solução não atende ao contrato Pocket e deve ser redesenhada ou tratada como bloqueador.

## Client

Tecnologia: **Tauri 2 + HTML/CSS/JavaScript modular**.

Responsabilidades:

- UI/navegação;
- sessão em memória;
- consumir API do Host;
- receber eventos e reconsultar estado;
- apresentar conflitos/estados transversais;
- executar Atendimento sem abrir SQLite;
- solicitar geração documental;
- receber bytes de artefatos;
- salvar/preview/imprimir localmente conforme contratos específicos;
- realizar impressão física no contexto Windows da estação.

Direção visual transversal: clareza com baixa densidade textual permanente; usar cor, forma, símbolo, posição e ícones quando isso simplificar sem ambiguidade; detalhes secundários sob demanda; cor nunca é o único canal de estado importante.

Baseline: Windows 10/11 x64 + WebView2.

## WebView2

- Evergreen compatível já presente é preferível;
- disponibilidade real precisa ser detectada;
- não baixar/instalar runtime silenciosamente pela Internet em produção;
- Fixed Version não pode ser executado de localização de rede/UNC;
- eventual fallback Fixed/autocontido deve ser preparado localmente;
- fallback só entra em produção após PoC provar funcionamento em `%LOCALAPPDATA%` sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo;
- requisitos ACL/AppContainer de Fixed Runtime moderno no Windows 10 precisam ser automatizáveis sem enfraquecer o contrato Pocket.

## Launcher e distribuição

Launcher Rust x64 portátil/transitório:

1. lê/valida manifesto/configuração;
2. compara versão;
3. prepara `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
4. valida SHA-256;
5. preserva versão anterior válida;
6. valida recursos locais necessários;
7. inicia Client local;
8. encerra.

Sem updater residente. A máquina central recebe pasta pronta; não exige toolchain de desenvolvimento.

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

O Controller inicia/controla o Host. Não criar serviço persistente para contornar limitações de SMB/execução remota.

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
- conclusão exige responsável + `Resumo do trabalho`;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo curto;
- Concluído/Cancelado não aceitam edição direta;
- mudança posterior exige reabertura;
- lifecycle é auditável/versionado;
- estado final necessário à reimpressão histórica não pode ser reescrito silenciosamente após reabertura.

## Equipamento, checklist e observações

Equipamento:

- código `EQP-000001`;
- ID interno canônico;
- serial/MAC/patrimônio são atributos de busca;
- múltiplos MACs;
- não arquivar se ligado a Atendimento `Em andamento`;
- Funcionário pode criar/editar por preset; arquivar/reativar fica ADM/Gerência;
- conclusão preserva projeção histórica relevante.

Checklist:

- definição fica no Procedimento imutável;
- estado de execução fica separado e ligado ao Atendimento/vínculo;
- somente Atendimento persiste marcações;
- progresso = itens marcados / total;
- etapa visitada não é progresso;
- 100% não conclui automaticamente;
- concorrência granular por item/equivalente.

Observação do serviço por Etapa:

- texto opcional de execução ligado ao Atendimento + revisão + Etapa;
- não altera Procedimento oficial;
- Reader standalone não persiste;
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
- checklist usa granularidade por item/equivalente;
- observação por Etapa usa granularidade própria;
- Atendimento/Equipamento têm revisões separadas;
- timeout após mutação exige reconciliação, não retry cego;
- geração documental é leitura derivada fora da fila de mutações;
- renderização documental possui admissão/limite próprio bounded.

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

## Geração documental

```text
Client
→ solicita identidade da fonte + revisão esperada
Host
→ autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ admite renderização bounded
→ renderiza fora da fila de mutações
→ devolve bytes
Client
→ salva / preview / imprime
```

- sem `export_jobs` persistente inicialmente;
- artefato retorna pela API autenticada;
- Host não grava em path arbitrário do Client;
- artefato não vira histórico/backup por padrão.

## PDF de Procedimentos

- Typst embutido via crates oficiais + adaptador;
- `World` restrito;
- PDF 1.7 + Tagged PDF baseline;
- fontes Noto Sans/Noto Sans Mono incorporadas;
- texto selecionável/pesquisável;
- multipágina automático;
- PNG/JPEG/SVG controlados;
- sem recurso remoto arbitrário;
- parcial nunca é sucesso.

## DOCX de Procedimentos

- OOXML Transitional direto em Rust;
- `docx-rs` preferido sob adaptador;
- sem Word/COM, LibreOffice ou conversão do PDF;
- texto/listas/numeração editáveis;
- Arial/Consolas referenciadas;
- DOCX refluível, sem promessa de paginação do PDF.

## Impressão Windows

```text
PDF oficial
→ Client Windows
→ recurso local transitório quando necessário
→ WebView2 dedicada/transitória
→ adapter nativo
→ ShowPrintUI(System)
```

- sem software externo ou seletor próprio baseline;
- sucesso = fluxo entregue ao Windows;
- lifetime exato do arquivo e drivers permanecem teste Windows real.

## Procedimento físico

- A4 retrato multipágina;
- margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- sem truncamento/redução silenciosa;
- PDF é referência física; DOCX é refluível.

## Ficha compacta

- prestação de contas resumida ao cliente;
- PDF próprio/canônico + preview SVG do mesmo `PagedDocument`;
- exatamente uma página A4;
- margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- sem corte/segunda página/redução automática de fonte;
- Salvar/Imprimir reutilizam os mesmos bytes PDF da prévia;
- seções vazias colapsam;
- soft limits: 600 / 400 / 300 / 280;
- Typst real é autoridade de encaixe;
- Procedimentos vinculados ficam fora da Ficha por padrão;
- MACs: 0 omite; 1–2 valores; 3+ quantidade;
- observações legítimas não recebem cap automático.

## Naming e temporários

- Procedimento: `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`;
- sem `display_version`: omitir apenas esse segmento;
- Ficha: `{service_code} - Ficha.pdf`;
- sanitização de filename Windows; sem path injection;
- conflito não sobrescreve silenciosamente;
- save só é sucesso após gravação integral;
- temporário somente quando integração local precisa de filesystem;
- raiz por usuário sob namespace StepFlow + diretório opaco por Client;
- cleanup/scavenging best-effort;
- reparse point não atravessa a raiz gerenciada;
- sem serviço/daemon/tarefa agendada de limpeza.

## Validação técnica final — Etapa 11

Fonte: `../04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`.

Resultado:

- nenhum bloqueador arquitetural identificado;
- Typst/PDF/PagedDocument validados;
- DOCX direto validado com teste de Word real pendente;
- impressão Windows validada arquiteturalmente via WebView2 nativo;
- adapter Tauri/Wry/WebView2 deve ser pinado/testado;
- save local/naming/temp/scavenging viáveis com limites;
- SMB/Word/impressoras/EDR pendentes de ambiente real;
- performance e backpressure serão definidos por benchmark;
- requisito Pocket é gate superior;
- Fixed WebView2 por UNC/SMB não utilizar;
- fallback local exige PoC sem instalação/elevação/manualidade.

## Backup / Restore

UX consolidada:

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
- **Bloco 10: Etapas 1–11 consolidadas; fechamento operacional do PR/branch pendente**;
- Blocos 11–12: pendentes.

Nenhum runtime/código funcional oficial foi criado durante esse fechamento documental.
