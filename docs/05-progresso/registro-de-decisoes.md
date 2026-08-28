# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-28

Este arquivo registra decisões vigentes, pendências e gates. Propostas não aprovadas não podem ser tratadas como contrato. Detalhes extensos permanecem nos documentos específicos e em `docs/03-arquitetura/arquitetura-vigente.md`.

## 1. Governança

- GitHub é a fonte operacional de verdade durante o fechamento documental restante da Fase 1;
- checkout local `C:\dev\StepFlow` será sincronizado explicitamente antes do primeiro trabalho de implementação com Codex;
- alterações locais preexistentes do PO devem ser preservadas;
- uma tarefa lógica por vez, uma branch ativa, um PR;
- revisão/aprovação → squash merge → apagar branch → verificar remoto somente com `main` e zero PRs abertos → próximo trabalho;
- branch mergeada não está encerrada enquanto permanecer no remoto;
- `AGENTS.md` é a regra operacional superior;
- avanço consolidado de fase/bloco/etapa sincroniza indicadores documentais no mesmo checkpoint;
- Fase 1 não autoriza runtime/scaffold/código de negócio oficial antes do gate correspondente.

## 2. Papéis

- PO: requisitos, prioridade, regra de negócio e aprovação final;
- Assistente: análise, arquitetura, coerência documental e gates;
- Codex: implementação pequena/aprovada, sem inventar produto.

## 3. Produto e domínio

StepFlow é aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada.

Uso amplo: manutenção, TI, Service Desk/Help Desk, infraestrutura, redes, procedimentos internos e guias técnicos.

Não transformar por inferência em CRM, financeiro/faturamento, estoque, RMM ou sistema completo de chamados/SLA.

Modelo:

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação/modelo oficial;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode existir sem Equipamento e usar zero, um ou vários Procedimentos;
- vínculo preserva revisão exata realmente utilizada;
- alteração futura do Procedimento/Equipamento não reescreve histórico concluído.

## 4. Procedimentos, categorias e revisões

Campos principais:

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Categorias;
- Etapas;
- Histórico.

Categorias:

- configuráveis pela empresa;
- múltiplas por Procedimento;
- simples, sem árvore hierárquica inicial;
- pesquisáveis/filtráveis;
- arquivamento preserva histórico;
- gestão por preset ADM/Gerência;
- nomes normalizados equivalentes devem ser evitados.

Editor/revisões:

- Editor = `Informações` + `Etapas`;
- painel local `Estrutura`, sem segunda sidebar global;
- blocos tipados;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- `base_revision`/controle otimista e `409` preservando alterações locais;
- sem merge automático;
- publicar é separado de salvar;
- `revision_no` técnico separado de `display_version` editorial.

Pendente: regra editorial de nova revisão ainda referenciando categoria arquivada.

## 5. Reader / manual e direção visual

- experiência principal em formato livro/manual;
- `Visão geral` antes da Etapa 1 como página lógica não numerada;
- uma Etapa = uma página lógica;
- `Sumário` temporário;
- `Anterior`/`Próxima`;
- `Etapa X de Y` textual compacto;
- stepper horizontal compacto de círculos/linhas, navegável por clique/teclado;
- estados anterior/atual/seguinte usam preenchimento, contraste, forma/símbolo e cor, sem depender só de cor;
- nomes das Etapas permanecem no título/Sumário, não repetidos permanentemente no stepper;
- estado anterior no stepper = percurso de navegação, nunca conclusão operacional;
- blocos iniciais: `paragraph`, `numbered_steps`, `checklist`, `note`, `warning`, `command`, `code`;
- sem HTML arbitrário;
- comando/código preserva whitespace, nunca executa e usa copiar icon-only acessível;
- revisão aberta permanece estável quando surge revisão nova;
- revisão histórica é identificada.

Princípio transversal de UI/UX:

- mostrar permanentemente apenas o necessário para entender e agir;
- usar cor, forma, símbolo, posição e ícones reconhecíveis para reduzir texto repetitivo quando o significado continuar claro;
- detalhes secundários podem aparecer sob demanda;
- evitar badges/cards/labels sem ganho de leitura;
- cor nunca é o único meio para estado importante;
- quando retirar texto criar ambiguidade, o texto permanece.

### Reader standalone

- checklist é documental;
- não persiste execução;
- não existe `Observação do serviço` persistente;
- navegação/stepper não grava progresso.

### Reader em Atendimento

- identifica `Executando no atendimento AT-...`;
- revisão fica presa ao vínculo;
- checklist é persistente;
- cada Etapa pode receber `Observação do serviço` opcional e persistente;
- observação pertence ao Atendimento + revisão vinculada + Etapa, nunca ao Procedimento oficial;
- voltar retorna ao Atendimento;
- lifecycle controla editabilidade;
- stepper continua navegação, separado do progresso operacional.

## 6. Atendimentos — lifecycle

Estados:

```text
Em andamento
Concluído
Cancelado
```

```text
rascunho Client
→ primeiro save aceito
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- abrir tela nova não persiste registro;
- primeiro save cria ID, `AT-000001` e `started_at`;
- não existe draft persistente inicial;
- Concluído/Cancelado são read-only operacionalmente;
- correção posterior exige reabertura;
- lifecycle é auditável/versionado;
- estado final histórico necessário para reimpressão não pode ser reescrito silenciosamente após reabertura.

Responsável:

- obrigatório para conclusão;
- Funcionário cria inicialmente para si;
- Funcionário padrão edita/conclui apenas Atendimento do qual é responsável;
- ADM/Gerência podem atribuir/reatribuir.

Conclusão:

- exige `Em andamento`, capacidade, responsável, `Resumo do trabalho`, estado confirmado e sem alterações locais relevantes não salvas;
- checklist incompleto avisa, não bloqueia automaticamente;
- observação de serviço por Etapa é opcional;
- preserva revisões/checklist/observações de serviço e congela projeção relevante do Equipamento;
- nova conclusão após reabertura cria novo estado final sem apagar o anterior.

Cancelamento:

- somente `Em andamento`;
- ADM/Gerência por preset;
- motivo curto obrigatório;
- não exclui registro;
- pode ser reaberto quando autorizado.

## 7. Checklist, observações de serviço e progresso

Checklist operacional:

- estado separado da revisão documental;
- persistente apenas em Atendimento;
- controle concorrente por item/equivalente;
- Concluído/Cancelado = somente leitura até reabertura;
- 100% não conclui automaticamente.

Observação do serviço por Etapa:

- opcional;
- vinculada ao `service_record_process` + Etapa da revisão exata;
- separada das observações gerais do Atendimento/Equipamento;
- editável apenas em Atendimento editável e autorizado;
- controle concorrente por Etapa/equivalente;
- evento remoto não sobrescreve texto local em edição;
- fica somente leitura em Concluído/Cancelado;
- precisa participar da reprodução histórica da Ficha aplicável;
- não é chat/comentário social nem bloco do Procedimento;
- não introduz autosave por inferência.

Progresso:

```text
checked_count / total_checklist_items
```

- deriva somente do checklist;
- páginas visitadas/stepper/observações não contam;
- revisão sem checklist não mostra `0%` artificial.

## 8. Equipamento

Código inicial: `EQP-000001`.

Campos conforme aplicável: nome, tipo, CPU, RAM, armazenamento, SO/versão, serial, patrimônio, múltiplos MACs, bateria, cliente/responsável e observações curtas.

- Equipamento é opcional/reutilizável;
- MAC/serial/patrimônio não são identidade canônica;
- múltiplos MACs com label opcional;
- bateria 0–100 quando informada e contextual;
- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- não arquivar se vinculado a Atendimento `Em andamento`;
- conclusão congela projeção histórica relevante para que cadastro futuro não reescreva ficha/histórico.

## 9. Matriz operacional — presets

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar/criar Atendimento | sim | sim | sim |
| Editar/concluir próprio Atendimento em andamento | sim | sim | sim |
| Editar/concluir qualquer Atendimento | sim | sim | não |
| Cancelar / Reabrir | sim | sim | não |
| Vincular/trocar Equipamento editável | sim | sim | sim, quando responsável |
| Criar/editar Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica | sim | sim | não |
| Marcar/desmarcar checklist | sim | sim | sim, quando responsável |
| Registrar observação de serviço por Etapa | sim | sim | sim, quando responsável |
| Gerar/reimprimir Ficha acessível | sim | sim | sim |
| Gerir categorias | sim | sim | não |

Presets são defaults; autorização real é granular e Host-side.

Pendentes: Gerência × configuração da empresa e Gerência × Backup.

## 10. Segurança e concorrência

- Gerência não administra ADM;
- pelo menos um ADM ativo;
- Argon2id;
- sessão opaca server-side;
- token em memória;
- troca da própria senha mantém sessão corrente e revoga demais sessões da conta;
- sessão expirada exige nova autenticação.

Concorrência:

- SQLite WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista e `409` para base obsoleta;
- eventos pós-commit;
- sem soft/hard lock inicial;
- Atendimento/Equipamento têm revisões independentes;
- checklist e observações de Etapa têm granularidade própria;
- timeout após mutação exige reconciliação, não retry cego;
- evento remoto não sobrescreve edição local silenciosamente.

Parâmetros finais de Argon2id, senha, duração da sessão e token permanecem pendentes antes da implementação.

## 11. Bloco 10 — geração documental

### Etapa 1 — arquitetura

- geração documental pertence ao Host;
- Client solicita fonte + revisão esperada e não envia documento montado;
- Host captura snapshot consistente, materializa `DocumentModel` semântico e encerra leitura/transação antes de renderizar;
- renderers não reconsultam SQLite nem usam DOM/HTML da UI;
- geração é leitura derivada, fora da fila de mutações;
- renderização possui limite próprio de concorrência/backpressure;
- sem `export_jobs` persistentes inicialmente;
- artefato retorna pela API autenticada;
- Host não escreve em path arbitrário do Client;
- artefatos não viram histórico/backup por padrão;
- runtime não depende de Office, LibreOffice, Adobe Reader, browser externo/headless, `wkhtmltopdf` ou cloud obrigatória.

### Etapa 2 — PDF de Procedimentos

- Typst embutido via crates Rust oficiais + adaptador interno;
- template interno confiável; conteúdo entra apenas como dados estruturados;
- mundo virtual restringe imports/filesystem/fontes/assets; sem recursos remotos;
- PDF 1.7 + Tagged PDF como baseline, sem promessa formal PDF/A ou PDF/UA;
- texto real selecionável/pesquisável;
- fontes incorporadas/subsetadas;
- todos os blocos conhecidos são representados ou falham explicitamente;
- multipágina automático, sem truncamento;
- PNG/JPEG/SVG controlados;
- falha nunca retorna artefato parcial como sucesso.

### Etapa 3 — DOCX de Procedimentos

- DOCX real OOXML/WordprocessingML/OPC, OOXML Transitional;
- geração direta Rust a partir do mesmo `DocumentModel`, sem PDF → DOCX;
- `docx-rs` preferido sob adaptador interno;
- texto/numeração editáveis reais;
- sem Word/COM, LibreOffice, browser/headless ou cloud;
- sem template DOCX/DOTX fornecido pelo usuário na v1;
- PNG/JPEG baseline; SVG direto não obrigatório;
- Arial/Consolas referenciadas sem embedding na v1;
- pacote incompleto/corrompido nunca é sucesso.

### Etapa 4 — impressão Windows de Procedimentos

- impressão física acontece no Client Windows;
- usa o mesmo PDF oficial da Etapa 2;
- WebView2 transitória/dedicada + `ShowPrintUI(System)`;
- sem HTML da UI, DOCX, software externo, seletor próprio ou impressão silenciosa como baseline;
- recurso local de impressão é transitório;
- sucesso = fluxo entregue ao Windows, não confirmação de papel impresso;
- estratégia concreta de temporário fica para Etapa 10.

### Etapa 5 — template físico de Procedimentos

- Reader diário não possui geometria A4;
- Procedimento exportado usa A4 retrato multipágina, margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- título de Etapa fica com conteúdo quando possível;
- paginação automática sem truncamento/redução silenciosa;
- PDF: Noto Sans/Noto Sans Mono incorporadas;
- DOCX: Arial/Consolas referenciadas;
- PDF é referência física, DOCX é refluível;
- limite de uma A4 pertence somente à Ficha.

### Etapa 6 — PDF + preview da Ficha compacta

Finalidade:

- Ficha é **prestação de contas resumida ao cliente**;
- prioriza identificação do serviço, identificação/características do dispositivo, `Resumo do trabalho` e observações relevantes;
- dados do Equipamento vêm do cadastro/snapshot já existente;
- observações de serviço por Etapa entram quando preenchidas;
- checklist, progresso, passos, comandos, timeline, IDs internos e lista detalhada de revisões não aparecem por padrão.

Geração:

```text
Atendimento confirmado + source_version esperada
→ DocumentModel document_kind = service_sheet
→ template Typst próprio
→ PagedDocument
→ exatamente 1 página
   ├─→ PDF canônico
   └─→ SVG de preview
```

- mesma infraestrutura Typst embutida, template próprio da Ficha;
- PDF 1.7 + Tagged PDF como baseline;
- `2+ páginas` = `SHEET_OVERFLOW`, sem corte/segunda página/redução silenciosa;
- PDF e preview SVG derivam do mesmo `PagedDocument`;
- preview abre em modal/overlay simples, folha A4 centralizada, sem nova sidebar/toolbar extensa;
- `Salvar PDF` e `Imprimir` reutilizam os mesmos bytes PDF da prévia;
- impressão reutiliza WebView2 + `ShowPrintUI(System)`;
- PDF/SVG são transitórios, sem job/histórico/backup automático;
- preview fica preso à `source_version`; atualização remota exige regeneração antes de nova saída atual;
- Concluído reimprime estado histórico aplicável; reabertura não pode reescrever silently a Ficha anterior.

## 12. Backup/Restore e implantação Pocket

Backup/Restore UX:

- dentro de Configurações;
- Host coordena;
- Client não escolhe SQLite/path;
- Restore exige capacidade + backup elegível + confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva normal;
- disaster recovery sem Host funcional fica no Bloco 11.

Implantação:

- máquina central recebe pasta pronta;
- sem instalação normal/toolchain;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon;
- Controller inicia Host como filho;
- fechar Client individual não encerra Host;
- encerrado ciclo central, zero processo StepFlow residual;
- launcher da workstation não inicia remotamente o Host central por si só.

Tecnologias:

- Client: Tauri 2 + HTML/CSS/JS modular + Windows 10/11 x64 + WebView2;
- Host: Rust + Tokio/Axum + `rusqlite` bundled + SQLite Host-only;
- comunicação: HTTP/JSON `/api/v1` inicialmente + WebSocket autenticado + handshake de compatibilidade;
- sem edição offline.

## 13. Estado da Fase 1

- Blocos 0–4: concluídos;
- Bloco 5: núcleo concluído / parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Bloco 8: concluído;
- Bloco 9: concluído;
- **Bloco 10: em andamento — Etapas 1–6 consolidadas; Etapa 7 é a próxima, ainda não aberta**;
- Bloco 11: pendente;
- Bloco 12: pendente.

## 14. Pendências vigentes

### Bloco 10

- Etapa 7: template físico A4 da Ficha;
- Etapa 8: limites textuais/priorização/densidade;
- Etapa 9: muitos MACs/Procedimentos/dados excepcionais;
- Etapa 10: nomes de arquivo + temporários concretos;
- Etapa 11: QR/barcode somente se benefício aprovado;
- Etapa 12: validação técnica final/matriz real/limites de recursos.

A Etapa 7 só pode ser aberta após squash merge da Etapa 6, remoção da branch correspondente e verificação de remoto somente com `main` e zero PRs abertos.

### Bloco 11

- pacote/atomicidade/checksums/retensão de Backup/Restore;
- restart/reconexão/sessões;
- disaster recovery local.

### Bloco 12 / implementação

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão;
- forma física final de snapshots históricos necessários;
- árvore oficial/migrations/scripts/testes;
- plano Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

### Ambiente corporativo

- hostname/IP/path reais;
- SMB/permissões;
- WebView2/Windows reais;
- HTTP/HTTPS;
- EDR/firewall;
- mecanismo real de start central.

## 15. Regra de precedência

Em divergência:

1. `AGENTS.md`;
2. este registro de decisões, na versão mais recente;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.