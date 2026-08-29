# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-29

Este arquivo registra decisões vigentes, pendências e gates. Propostas não aprovadas não podem ser tratadas como contrato. Detalhes extensos permanecem nos documentos específicos e em `docs/03-arquitetura/arquitetura-vigente.md`.

## 1. Governança

- GitHub é a fonte operacional de verdade durante o fechamento documental restante da Fase 1;
- checkout local `C:\dev\StepFlow` será sincronizado explicitamente antes do primeiro trabalho de implementação com Codex;
- alterações locais preexistentes do PO devem ser preservadas;
- uma tarefa lógica por vez, uma branch ativa e um PR;
- revisão/aprovação → squash merge → apagar branch → verificar remoto somente com `main` e zero PRs abertos → próximo trabalho;
- branch mergeada não está encerrada enquanto permanecer no remoto;
- `AGENTS.md` é a regra operacional superior;
- avanço consolidado de fase/bloco/etapa sincroniza indicadores documentais no mesmo checkpoint;
- Fase 1 não autoriza runtime/scaffold/código de negócio oficial antes do gate correspondente.

## 2. Produto e domínio

StepFlow é aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada.

Uso amplo: manutenção, TI, Service Desk/Help Desk, infraestrutura, redes, procedimentos internos e guias técnicos.

Não transformar por inferência em CRM, financeiro/faturamento, estoque, RMM ou sistema completo de chamados/SLA.

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

## 3. Procedimentos, categorias e revisões

Campos principais: Código, Título, Área/Departamento, Responsável, Status, Versão, Objetivo, Observações, Pré-requisitos, Categorias, Etapas e Histórico.

Categorias:

- configuráveis e múltiplas;
- simples, sem árvore inicial;
- pesquisáveis/filtráveis;
- arquivamento preserva histórico;
- gestão por preset ADM/Gerência;
- evitar nomes normalizados equivalentes.

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

## 4. Reader / manual e direção visual

- experiência em formato livro/manual;
- `Visão geral` antes da Etapa 1, não numerada;
- uma Etapa = uma página lógica;
- `Sumário` temporário, `Anterior`/`Próxima`, `Etapa X de Y` compacto;
- stepper horizontal compacto de círculos/linhas, navegável por clique/teclado;
- estados anterior/atual/seguinte usam preenchimento, contraste, forma/símbolo e cor, sem depender só de cor;
- nomes das Etapas permanecem no título/Sumário;
- estado anterior no stepper = percurso de navegação, nunca conclusão operacional;
- blocos iniciais: `paragraph`, `numbered_steps`, `checklist`, `note`, `warning`, `command`, `code`;
- sem HTML arbitrário;
- comando/código preserva whitespace, nunca executa e usa copiar icon-only acessível;
- revisão aberta permanece estável quando surge revisão nova;
- revisão histórica é identificada.

Princípio transversal de UI/UX:

- mostrar permanentemente apenas o necessário para entender e agir;
- usar cor, forma, símbolo, posição e ícones para reduzir texto repetitivo quando o significado continuar claro;
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
- lifecycle controla editabilidade;
- stepper continua navegação, separado do progresso operacional.

## 5. Atendimentos, checklist e observações

Lifecycle:

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

- primeiro save cria ID, `AT-000001` e `started_at`;
- Concluído/Cancelado são read-only até reabertura;
- conclusão exige responsável + `Resumo do trabalho` + estado confirmado;
- checklist incompleto avisa, não bloqueia automaticamente;
- cancelamento exige motivo curto;
- nova conclusão após reabertura cria novo estado final sem apagar o anterior;
- estado histórico necessário à reimpressão não pode ser reescrito silenciosamente.

Checklist:

- persistente apenas em Atendimento;
- controle concorrente por item/equivalente;
- 100% não conclui automaticamente;
- progresso deriva apenas de itens marcados/total.

Observação do serviço por Etapa:

- opcional;
- ligada ao vínculo da revisão + Etapa;
- separada de observações gerais do Atendimento/Equipamento;
- editável apenas em Atendimento editável/autorizado;
- controle concorrente por Etapa/equivalente;
- evento remoto não sobrescreve texto local em edição;
- somente leitura em Concluído/Cancelado;
- participa da reprodução histórica da Ficha aplicável;
- não é chat/comentário social nem conteúdo oficial do Procedimento;
- não introduz autosave por inferência.

## 6. Equipamento e matriz operacional

Código inicial: `EQP-000001`.

Campos conforme aplicável: nome, tipo, CPU, RAM, armazenamento, SO/versão, serial, patrimônio, múltiplos MACs, bateria, cliente/responsável e observações curtas.

- Equipamento é opcional/reutilizável;
- MAC/serial/patrimônio não são identidade canônica;
- bateria 0–100 quando informada e contextual;
- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- não arquivar se vinculado a Atendimento `Em andamento`;
- conclusão congela projeção histórica relevante.

Presets operacionais:

- ADM/Gerência operam Atendimentos acessíveis amplamente;
- Funcionário edita/conclui por padrão Atendimento do qual é responsável;
- Funcionário não cancela/reabre por preset;
- ADM/Gerência podem selecionar revisão histórica/não publicada já autorizada;
- Funcionário usa revisão publicada;
- checklist e observação de serviço por Etapa podem ser registrados pelo responsável autorizado;
- Ficha pode ser gerada/reimpressa por ADM/Gerência/Funcionário em Atendimento acessível;
- Gerência gere categorias por preset;
- autorização real permanece granular e Host-side.

Pendentes: Gerência × configuração da empresa e Gerência × Backup.

## 7. Segurança, concorrência e Pocket

- Client Tauri 2 + HTML/CSS/JS modular;
- Host Rust + Tokio/Axum + `rusqlite` bundled;
- SQLite Host-only, WAL, writer coordenado, fila bounded, revisão otimista e eventos pós-commit;
- checklist e observações de Etapa usam granularidade própria;
- sessão opaca server-side, token em memória, Argon2id;
- Gerência não administra ADM; pelo menos um ADM ativo;
- sem edição offline/autosave/fila local persistente;
- implantação central por pasta pronta, sem toolchain na produção;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon como padrão;
- Controller controla o ciclo central; fechar Client não encerra Host.

Parâmetros finais de Argon2id, senha, sessão e token permanecem pendentes.

## 8. Bloco 10 — geração documental

### Etapa 1 — arquitetura

- geração documental pertence ao Host;
- Client solicita fonte + revisão esperada e não envia documento montado;
- Host captura snapshot consistente, materializa `DocumentModel` e encerra leitura antes de renderizar;
- renderers não reconsultam SQLite nem usam DOM/HTML da UI;
- geração é leitura derivada, fora da fila de mutações;
- renderização possui limite próprio de concorrência/backpressure;
- sem `export_jobs` persistentes inicialmente;
- artefato retorna pela API autenticada;
- Host não escreve em path arbitrário do Client;
- runtime não depende de Office, LibreOffice, Adobe Reader, browser externo/headless ou cloud obrigatória.

### Etapa 2 — PDF de Procedimentos

- Typst embutido via crates Rust oficiais + adaptador interno;
- template interno confiável e conteúdo apenas como dados estruturados;
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
- geração direta Rust a partir do mesmo `DocumentModel`;
- `docx-rs` preferido sob adaptador interno;
- texto/numeração editáveis reais;
- sem Word/COM, LibreOffice, browser/headless ou cloud;
- sem template DOCX/DOTX fornecido pelo usuário na v1;
- Arial/Consolas referenciadas sem embedding na v1;
- pacote incompleto/corrompido nunca é sucesso.

### Etapa 4 — impressão Windows de Procedimentos

- impressão física acontece no Client Windows;
- usa o mesmo PDF oficial da Etapa 2;
- WebView2 transitória/dedicada + `ShowPrintUI(System)`;
- sem HTML da UI, DOCX, software externo, seletor próprio ou impressão silenciosa como baseline;
- sucesso = fluxo entregue ao Windows, não confirmação de papel impresso;
- recurso local transitório segue a Etapa 10.

### Etapa 5 — template físico de Procedimentos

- Reader diário não possui geometria A4;
- Procedimento exportado usa A4 retrato multipágina, margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- paginação automática sem truncamento/redução silenciosa;
- PDF: Noto Sans/Noto Sans Mono incorporadas;
- DOCX: Arial/Consolas referenciadas;
- PDF é referência física, DOCX é refluível;
- limite de uma A4 pertence somente à Ficha.

### Etapa 6 — PDF + preview da Ficha

- Ficha é **prestação de contas resumida ao cliente**;
- prioriza identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações;
- dados do Equipamento vêm do cadastro/snapshot existente;
- observações de serviço por Etapa entram quando preenchidas;
- checklist, progresso, passos, comandos, timeline, IDs internos e revisões detalhadas não aparecem por padrão;
- PDF próprio/canônico via template Typst da Ficha;
- PDF e preview SVG derivam do mesmo `PagedDocument`;
- `2+ páginas` = `SHEET_OVERFLOW`, sem corte/segunda página/redução silenciosa;
- preview em modal/overlay simples;
- Salvar/Imprimir reutilizam os mesmos bytes PDF da prévia;
- impressão reutiliza WebView2 + `ShowPrintUI(System)`;
- PDF/SVG são transitórios e presos à `source_version`.

### Etapa 7 — template físico A4 da Ficha

- papel A4 retrato, **exatamente uma página**, margens de **15 mm** e sem bleed;
- composição predominantemente vertical/uma coluna, com microagrupamentos horizontais apenas para dados curtos;
- cabeçalho institucional compacto: logo opcional, empresa/contato e identificação/data do Atendimento; sem título gigante e sem footer obrigatório;
- status aparece proporcionalmente: `CANCELADO` sempre textual/inequívoco; acompanhamento/em andamento discreto; sem watermark grande;
- identificação de cliente/OS/técnico em linha curta, omitindo campos vazios;
- Equipamento usa ficha técnica resumida **sem tabela gradeada**, com dados existentes e aplicáveis;
- `SERVIÇO REALIZADO` é a área narrativa principal, baseada no `Resumo do trabalho`;
- uma única seção `OBSERVAÇÕES` reúne observações relevantes do Atendimento, Equipamento e Etapas, em lista simples e sem subcards;
- nome curto da Etapa só aparece quando necessário para contextualizar a observação;
- seções vazias colapsam completamente; não imprimir `Sem observações`, `Nenhum equipamento` ou placeholders;
- PDF da Ficha usa **Noto Sans**; baseline tipográfico: 14 pt identificação principal, 10,5 pt títulos de seção, 10 pt corpo/resumo, 9 pt ficha técnica e 8,5 pt metadados;
- divisórias são discretas, contraste neutro e leitura precisa funcionar em monocromático; não congelar paleta RGB/hex nesta fase;
- não reservar caixas para escrita manual e não adicionar assinatura, financeiro, garantia, checklist, progresso, timeline, lista detalhada de Procedimentos, página 2 ou footer promocional;
- nenhuma redução dinâmica de fonte para caber; overflow continua `SHEET_OVERFLOW`.

### Etapa 8 — limites textuais e densidade da Ficha

- a restrição de uma A4 pertence ao artefato e não pode apagar/truncar o dado operacional;
- soft limits recomendados: `Resumo do trabalho` **600**, observação geral do Atendimento **400**, observação do Equipamento **300** e observação de serviço por Etapa **280 caracteres**;
- contador/aviso aparece apenas próximo de aproximadamente **80%** da faixa recomendada;
- soft limit não bloqueia save nem conclusão do Atendimento;
- o layout real do Typst é a autoridade final para o encaixe físico;
- `SHEET_OVERFLOW` bloqueia somente a geração da Ficha e mantém o Atendimento íntegro;
- Host devolve diagnóstico semântico dos principais campos que pressionam a folha, sem necessidade de percentual visual exato;
- Client orienta revisão dos campos reais, sem editor paralelo exclusivo da Ficha;
- observações seguem ordem Atendimento → Equipamento → Etapas na ordem executada;
- não deduplicar automaticamente textos semelhantes;
- não usar IA/resumo automático, truncamento, reticências, modo compacto ou redução automática de fonte/margem/espaçamento;
- normalização de apresentação só pode remover ruído de whitespace sem alterar significado;
- hard limits técnicos de storage/API são independentes da geometria A4.

### Etapa 9 — dados excepcionais e multiplicidade

- a Ficha permanece projeção client-facing resumida e não dump integral do domínio;
- Procedimentos vinculados permanecem fora da Ficha por padrão, independentemente da quantidade;
- MACs: 0 omite; 1–2 exibem valores compactos; 3+ exibem apenas a quantidade cadastrada;
- labels existentes podem contextualizar MACs; não inventar `MAC principal`;
- observações legítimas não recebem cap/descarte automático;
- multiplicidade real pode produzir `SHEET_OVERFLOW` e exigir revisão humana consciente dos textos reais;
- campos estruturados longos quebram linha quando possível, sem truncamento, reticências ou abreviação inventada;
- diagnóstico pode indicar quantidade de observações, Etapa específica ou campo estruturado longo;
- sem `include_in_sheet`, `sheet_priority`, seleção transitória, editor paralelo, segunda página ou compactação automática;
- limites técnicos finais de quantidade/payload/recursos permanecem para a Etapa 11.

### Etapa 10 — nomes de arquivo e artefatos temporários

- arquivo persistente escolhido pelo usuário e temporário interno são lifecycles distintos;
- Procedimento sugere `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`; sem `display_version`, omite esse segmento;
- Ficha sugere `{service_code} - Ficha.pdf`, sem dados pessoais/operacionais adicionais no filename por padrão;
- sanitização segue regras Windows, impede injeção de path e não altera conteúdo documental;
- conflito de nome não causa overwrite silencioso;
- save só é sucesso após gravação integral; auxiliar opaco no mesmo destino + promoção/replace seguro são preferidos quando suportados;
- temporário só é materializado no Client se uma integração local precisar de filesystem;
- raiz transitória é por usuário, resolvida por API do sistema/Tauri, sob namespace StepFlow e subdiretório opaco por instância;
- filenames temporários são opacos e não contêm cliente, título, equipamento, serial/MAC, resumo/observações ou técnico;
- cleanup ocorre best-effort após liberação e no encerramento normal; lock não autoriza kill/unlock forçado/alteração de ACL;
- crash pode deixar órfãos para scavenging posterior restrito ao namespace StepFlow, sem seguir reparse point para fora e sem apagar instância possivelmente ativa;
- sem Windows Service, Task Scheduler, daemon ou watchdog para limpeza;
- falha de cleanup não altera retroativamente save/preview/print já concluído;
- temporários/exportações não entram em SQLite, histórico ou backup por padrão;
- API concreta, NTFS/SMB, WebView2, memória, Unicode/path longo, concorrência e EDR ficam para a Etapa 11.

## 9. Estado da Fase 1

- Blocos 0–4: concluídos;
- Bloco 5: núcleo concluído / parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Blocos 8–9: concluídos;
- **Bloco 10: em andamento — Etapas 1–10 consolidadas; Etapa 11 — Validação técnica final é a próxima, ainda não aberta**;
- Blocos 11–12: pendentes.

## 10. Pendências vigentes

### Bloco 10

- Etapa 11: validação técnica final/matriz real/limites de recursos.

**Gate:** Etapa 11 — Validação técnica final só pode ser aberta após squash merge da limpeza documental atual, remoção da branch correspondente e verificação de remoto somente com `main` e zero PRs abertos.

### Bloco 11

- pacote/atomicidade/checksums/retenção de Backup/Restore;
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

## 11. Precedência

Em divergência:

1. `AGENTS.md`;
2. este registro de decisões;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.
