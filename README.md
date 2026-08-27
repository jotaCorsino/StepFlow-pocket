# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com foco em simplicidade operacional e implantação de baixo impacto.

## Painel de acompanhamento do projeto

**Atualização:** 2026-08-27  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Bloco atual:** Bloco 10 — Exportação / impressão / ficha compacta — Etapa 5 consolidada; Etapa 6 próxima  
**Implementação funcional oficial:** ainda não iniciada

Este painel é a visão rápida de andamento. Ele **não substitui** `AGENTS.md`, `docs/05-progresso/registro-de-decisoes.md`, documentos específicos e `docs/04-planejamento/plano-oficial-fase-1.md`.

### Andamento da Fase 1

| Bloco | Tema | Estado |
|---|---|---|
| 0 | Bootstrap do ambiente | ✅ Concluído |
| 1 | Client Windows / Tauri | ✅ Concluído |
| 2 | Host Pocket | ✅ Concluído |
| 3 | Launcher / distribuição | ✅ Concluído |
| 4 | Comunicação Client ↔ Host | ✅ Concluído |
| 5 | Autenticação / autorização | ✅ Núcleo concluído; parâmetros finais pendentes |
| 6 | Dados / schema / migrations | ✅ Núcleo + extensão operacional conceitual consolidados |
| 7 | Concorrência / fila / eventos | ✅ Núcleo concluído |
| 8 | UI/UX | ✅ Concluído |
| 9 | Atendimentos / execução / checklist | ✅ Concluído |
| 10 | Exportação / impressão / ficha compacta | 🟡 Em andamento — Etapa 5 consolidada |
| 11 | Backup / restauração | ⏳ Pendente |
| 12 | Estrutura oficial + plano da Fase 2 | ⏳ Pendente |

### Bloco 8 — telas e decisões

| Ordem | Superfície | Estado |
|---|---|---|
| 1 | Login | ✅ Consolidado |
| 2 | Shell / sidebar | ✅ Consolidado |
| 3 | Início / Dashboard | ✅ Consolidado |
| 4 | Lista / pesquisa de Processos | ✅ Consolidado |
| 5 | Leitor em formato livro | ✅ Consolidado |
| 6 | Editor de Processo + categorias | ✅ Consolidado |
| 7 | Histórico / revisões | ✅ Consolidado |
| 8 | Lista / pesquisa de Atendimentos | ✅ Consolidado |
| 9 | Atendimento / execução + equipamento | ✅ Consolidado |
| 10 | Usuários / permissões | ✅ Consolidado |
| 11 | Meu perfil | ✅ Consolidado |
| 12 | Configurações + categorias | ✅ Consolidado |
| 13 | Backup / restauração — UX | ✅ Consolidado |
| 14 | Exportação / impressão + ficha — UX | ✅ Consolidado |
| 15 | Estados transversais | ✅ Consolidado |

O Bloco 8 está concluído documentalmente. Nenhuma UI de produção foi criada.

Direção visual transversal consolidada:

- clareza com baixa densidade textual permanente;
- uso de cor, forma, símbolo, posição e ícones reconhecíveis quando isso simplificar sem gerar ambiguidade;
- detalhes secundários podem aparecer sob demanda;
- cor nunca é o único meio para comunicar estado importante;
- no Reader, `Visão geral` é a primeira página lógica e cada Etapa mantém página própria;
- o stepper do Reader é compacto, horizontal e navegável, com círculos/linhas e estados anteriores/atual/seguintes;
- o estado do stepper é navegação e não conclusão operacional/checklist.

### Bloco 9 — decisões operacionais consolidadas

Lifecycle mínimo:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho local
→ primeiro save aceito pelo Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Também estão aprovados:

- `AT-000001` para Atendimento;
- `EQP-000001` para Equipamento;
- códigos gerados somente pelo Host;
- Funcionário cria Atendimento inicialmente atribuído a si;
- Funcionário edita/conclui por preset somente Atendimento do qual é responsável;
- ADM/Gerência podem editar/concluir qualquer Atendimento acessível;
- cancelamento exige motivo e fica por preset para ADM/Gerência;
- reabertura fica por preset para ADM/Gerência;
- responsável + Resumo do trabalho são obrigatórios para concluir;
- checklist incompleto gera aviso, mas não bloqueia automaticamente;
- Reader standalone mantém checklist documental;
- Reader dentro de Atendimento persiste checklist;
- progresso deriva somente de itens de checklist marcados/total;
- 100% de checklist não conclui Atendimento automaticamente;
- Funcionário usa revisão publicada para execução;
- ADM/Gerência podem selecionar explicitamente revisão histórica/não publicada já autorizada;
- Funcionário pode criar/editar Equipamento;
- arquivar/reativar Equipamento fica ADM/Gerência;
- Equipamento ligado a Atendimento em andamento não pode ser arquivado;
- conclusão congela projeção histórica relevante do Equipamento;
- alterar cadastro global depois não reescreve ficha/histórico concluído;
- Gerência passa a gerir categorias por preset;
- gerar/reimprimir ficha fica disponível por preset para ADM/Gerência/Funcionário em Atendimento acessível;
- Em andamento pode gerar ficha de acompanhamento;
- Concluído pode reimprimir estado histórico;
- Cancelado precisa ficar inequivocamente identificado;
- checklist usa concorrência granular por item/equivalente;
- nenhuma fila de chamados/SLA/workflow complexo foi criada.

### Bloco 10 — etapas e decisões

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | ✅ Consolidado |
| 2 | PDF de Procedimentos | ✅ Consolidado |
| 3 | DOCX de Procedimentos | ✅ Consolidado |
| 4 | Impressão Windows de Procedimentos | ✅ Consolidado |
| 5 | Template físico de Procedimentos | ✅ Consolidado |
| 6 | PDF + preview da Ficha compacta | 🟡 Próxima — ainda não em análise |
| 7 | Template físico A4 da Ficha | ⏳ Pendente |
| 8 | Limites textuais e densidade da Ficha | ⏳ Pendente |
| 9 | Múltiplos MACs / Procedimentos na Ficha | ⏳ Pendente |
| 10 | Nomes de arquivo + artefatos temporários | ⏳ Pendente |
| 11 | QR / barcode | ⏳ Pendente |
| 12 | Validação técnica final do Bloco 10 | ⏳ Pendente |

A tabela acima é o acompanhamento operacional do Bloco 10 e deve ser atualizada no mesmo checkpoint de cada avanço. Uma etapa só muda para `✅ Consolidado` após aprovação do PO e registro coerente nas fontes canônicas afetadas.

Consolidado na Etapa 1:

- geração documental é responsabilidade do Host;
- Client solicita por identidade da fonte e revisão esperada, sem enviar documento montado;
- Host captura snapshot consistente antes da renderização;
- a leitura/transação SQLite é encerrada antes do trabalho pesado de renderização;
- `DocumentModel` semântico separa regras de domínio dos renderers;
- renderers não reconsultam banco nem recebem DOM/HTML da UI;
- geração é leitura derivada e fica fora da fila de mutações;
- renderização usa limite próprio de concorrência/backpressure;
- primeira versão não cria job/fila persistente de exportações;
- artefato retorna pela API autenticada;
- Host não grava em path arbitrário do Client;
- runtime documental permanece autocontido, sem Office/LibreOffice/Adobe/Chrome externo/cloud obrigatório;
- artefatos gerados não viram histórico/backup por padrão.

Consolidado na Etapa 2:

- renderer PDF de Procedimentos usa Typst embutido como biblioteca Rust no Host, por crates oficiais + adaptador interno StepFlow;
- nenhuma dependência de `typst.exe`/CLI, browser ou processo conversor externo;
- template Typst interno, confiável e versionado;
- conteúdo originado do domínio entra somente como valores/dados estruturados e nunca participa da construção textual do source Typst;
- nenhum pacote/recurso remoto é resolvido em runtime e o renderer não recebe filesystem genérico, path arbitrário ou URL originados do conteúdo;
- PDF 1.7 é solicitado explicitamente ao exporter;
- Tagged PDF permanece explicitamente habilitado como baseline, sem prometer conformidade formal PDF/UA/PDF-A;
- texto permanece selecionável, pesquisável e copiável;
- fontes necessárias são empacotadas/incorporadas, sem depender das fontes instaladas no Windows;
- todos os blocos semânticos do Procedimento são representados sem descarte silencioso;
- comandos/código permanecem texto e preservam whitespace relevante;
- fluxo multipágina e quebra automática são obrigatórios e usam o template físico consolidado na Etapa 5;
- PNG/JPEG e SVG controlado são capacidades do renderer, recebendo somente assets já aceitos pelo Host;
- conteúdo visual não depende implicitamente do relógio/ambiente da máquina central;
- falha de renderer não produz artefato parcial tratado como sucesso;
- assinatura, senha, formulários, anexos, JavaScript, multimídia e conformidade formal PDF/A ou PDF/UA ficam fora da primeira versão.

Consolidado na Etapa 3:

- DOCX de Procedimentos é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem conversão de PDF/Typst;
- saída é `.docx` real em OOXML/WordprocessingML com baseline **OOXML Transitional**;
- `docx-rs` é a biblioteca Rust preferida, isolada por adaptador interno StepFlow;
- nenhuma dependência de Word/COM, LibreOffice, browser/headless, CLI conversor ou cloud;
- conteúdo do domínio entra somente como dados estruturados e nunca como XML/OOXML arbitrário;
- estilos/template são internos e versionados; nenhum `.docx`/`.dotx` fornecido pelo usuário em runtime;
- texto permanece real, selecionável, pesquisável, copiável e editável;
- todos os blocos semânticos conhecidos são representados sem descarte silencioso;
- passos/subpassos usam listas/numeração Word reais quando aplicável; checklist é documental, não formulário interativo;
- comandos/código permanecem texto e preservam whitespace relevante;
- PNG/JPEG são baseline; SVG não é requisito direto do DOCX v1 e não pode ser omitido silenciosamente;
- DOCX é refluível e não promete paginação idêntica ao PDF;
- DOCX v1 referencia Arial para texto e Consolas para comando/código, sem embedding/redistribuição dessas fontes pelo StepFlow;
- macros/VBA, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
- artefato incompleto/corrompido nunca é devolvido como sucesso.

Consolidado na Etapa 4:

- impressão física de Procedimentos acontece no Client Windows, não no Host central;
- o mesmo PDF do renderer da Etapa 2 é o artefato canônico de impressão para a revisão exata selecionada;
- não existe renderer separado de impressão e o Client não imprime HTML da UI nem DOCX;
- o Client usa WebView2 transitória/dedicada para carregar somente recurso PDF local controlado;
- baseline usa WebView2 `ShowPrintUI(System)` por adaptador Windows isolado sob Tauri `with_webview`;
- o diálogo padrão é o diálogo de impressão do Windows; impressão silenciosa e seletor próprio de impressoras não são requisitos iniciais;
- StepFlow não enumera/persiste impressoras no Host e não gerencia drivers/spooler corporativo;
- `ShellExecute`/visualizador PDF externo, Word/COM, LibreOffice, browser externo e spool PDF bruto não são baseline;
- o recurso local de impressão é transitório e seus nomes/paths/limpeza concreta ficam para a Etapa 10;
- abrir o diálogo não confirma impressão física; a UI não exibe falso `Impresso com sucesso` nem grava `printed=true` por inferência;
- cancelamento/fechamento do diálogo não é erro funcional;
- falhas de geração, preparação local, compatibilidade WebView2 e abertura do diálogo permanecem distinguíveis;
- gate técnico posterior valida Windows 10/11 x64, WebView2, PDF multipágina, impressoras locais/de rede e operação offline;
- impressão usa o template físico consolidado da Etapa 5.

Consolidado na Etapa 5:

- Reader diário e documento físico são superfícies distintas; A4 não participa da geometria/navegação do app;
- `Visão geral` é a primeira página lógica do Reader e cada Etapa mantém página lógica própria;
- stepper superior é compacto, horizontal e navegável por círculos/linhas, com estados anteriores/atual/seguintes sem rótulos permanentes repetitivos;
- estado anterior no stepper representa percurso de navegação, nunca conclusão operacional ou checklist persistido;
- Procedimento exportado usa **A4 retrato multipágina**, com margens-base de **18 mm**;
- não há capa exclusiva e o conteúdo útil começa na primeira página;
- v1 não exige sumário físico por padrão;
- títulos de Etapa são enxutos, como `01 · Preparação`, e não forçam nova folha automaticamente;
- sem header repetitivo; rodapé compacto identifica código/revisão e paginação;
- paginação é automática, evitando títulos/linhas órfãs e sem truncar ou reduzir fonte silenciosamente;
- blocos físicos seguem baixa densidade visual e preservam significado sem depender apenas de cor;
- PDF usa **Noto Sans + Noto Sans Mono** empacotadas/incorporadas/subsetadas;
- DOCX usa **Arial + Consolas** referenciadas, sem embedding na v1;
- baseline tipográfico: título 18 pt, Etapa 14 pt, corpo 10,5 pt, comando/código 9 pt e rodapé 8 pt;
- PDF continua referência física de impressão; DOCX mantém hierarquia/semântica mas permanece refluível;
- o limite rígido de **uma A4** pertence somente à Ficha compacta de Atendimento.

A Etapa 6 — PDF + preview da Ficha compacta é a próxima e ainda não está em análise. Etapas 7–12 permanecem pendentes.

A fonte técnica é `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md`.

O Bloco 11 permanece **não iniciado**.

### Extensão de produto consolidada

O StepFlow deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias e outros procedimentos internos.

Permanecem consolidados:

- categorias configuráveis e múltiplas;
- separação `Procedimento × Atendimento/Execução × Equipamento`;
- Atendimentos como área operacional própria;
- Equipamento opcional/reutilizável;
- múltiplos Procedimentos por Atendimento;
- revisão exata utilizada preservada;
- ficha compacta com ou sem Equipamento;
- tipos mínimos `Servidor`, `Desktop`, `Notebook`;
- bateria contextual para Notebook;
- identidade central da empresa;
- Backup/Restauração em Configurações;
- safety backup antes de Restore normal;
- PDF/DOCX/impressão de Procedimentos pela revisão selecionada;
- baixa densidade visual como princípio transversal, com uso responsável de cor/forma/símbolo e detalhes secundários sob demanda;
- estados transversais comuns.

### Pendências ainda vigentes

- custo final Argon2id;
- senha mínima final;
- duração de sessão;
- entropia/tamanho final de token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de nova revisão ainda referenciando categoria arquivada;
- Etapas 6–12 do Bloco 10;
- mecanismo técnico do Bloco 11;
- validações reais do ambiente corporativo.

### Regra de atualização deste painel

Todo avanço consolidado de **fase, bloco, tela ou etapa do bloco atual** deve atualizar este README **no mesmo checkpoint documental**. Um avanço não está documentalmente encerrado se este painel ficar atrasado.