# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-26

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e provas descartáveis quando necessárias. Não autoriza scaffold/runtime oficial nem código de negócio definitivo antes do Bloco 12/Fase 2.

## Estado dos blocos

| Bloco | Tema | Status | Fonte vigente |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório validado |
| 1 | Client Windows/Tauri | CONCLUÍDO | `03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | NÚCLEO CONCLUÍDO / PARÂMETROS FINAIS PENDENTES | `03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS CONCEITUALMENTE | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | NÚCLEO CONCLUÍDO | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` + Bloco 9 |
| 8 | UI/UX | CONCLUÍDO | `02-telas/README.md` |
| 9 | Execução operacional/Atendimentos + checklist | **CONCLUÍDO** | `04-planejamento/bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão + ficha compacta | **EM ANDAMENTO — ETAPAS 1–4 CONSOLIDADAS / ETAPA 5 PRÓXIMA** | `04-planejamento/bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Extensão de produto consolidada

Fazem parte da Fase 1:

- categorias configuráveis/múltiplas;
- `Procedimento × Atendimento/Execução × Equipamento`;
- Atendimentos como área própria;
- Equipamento opcional/reutilizável;
- múltiplos Procedimentos por Atendimento;
- revisão exata utilizada preservada;
- ficha compacta com ou sem Equipamento;
- identidade central da empresa;
- Backup/Restauração administrativo;
- PDF/DOCX/impressão contextual de Procedimentos;
- estados transversais;
- lifecycle operacional de Atendimentos;
- checklist persistente em contexto de execução;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`.

## Bloco 8 — UI/UX — concluído

Telas 01–15 estão consolidadas/aprovadas:

1. Login;
2. Shell/sidebar;
3. Dashboard;
4. Lista/Pesquisa de Processos;
5. Reader em formato livro;
6. Editor de Processo + categorias;
7. Histórico/Revisões;
8. Lista/Pesquisa de Atendimentos;
9. Atendimento/Execução + Equipamento;
10. Usuários/Permissões;
11. Meu perfil;
12. Configurações + Categorias;
13. Backup/Restauração — UX;
14. Exportação/Impressão + ficha — UX;
15. Estados transversais.

Nenhuma UI de produção foi criada.

## Bloco 9 — Execução operacional / Atendimentos + checklist — concluído

Fonte canônica: `bloco-9-atendimentos-execucao-checklist.md`.

### Lifecycle

```text
Em andamento
Concluído
Cancelado
```

- primeiro save cria o Atendimento;
- `Resumo do trabalho` + responsável são obrigatórios para concluir;
- checklist incompleto avisa, mas não bloqueia automaticamente;
- cancelamento exige motivo;
- concluído/cancelado são read-only até reabertura;
- ADM/Gerência reabrem por preset.

### Responsabilidade

- Funcionário cria inicialmente para si;
- Funcionário opera/conclui o Atendimento do qual é responsável;
- ADM/Gerência podem atribuir/editar qualquer Atendimento acessível;
- usuário desativado permanece no histórico.

### Procedimentos e checklist

- Funcionário seleciona revisão publicada;
- ADM/Gerência podem selecionar explicitamente outras revisões autorizadas;
- Reader standalone = checklist documental;
- Reader no Atendimento = checklist persistente;
- progresso deriva de marcados/total;
- 100% não conclui automaticamente;
- concorrência granular por item/equivalente.

### Equipamento

- Funcionário cria/edita;
- ADM/Gerência arquivam/reativam;
- não arquivar Equipamento ligado a Atendimento em andamento;
- conclusão congela projeção histórica relevante do Equipamento.

### Códigos

```text
AT-000001
EQP-000001
```

Host-only, seis dígitos, gaps permitidos.

### Categorias

- gerir categorias: ADM/Gerência por preset;
- Funcionário não;
- regra editorial de nova revisão ainda referenciando categoria arquivada permanece pendente antes da implementação editorial.

### Ficha

- capacidade por preset para ADM/Gerência/Funcionário em Atendimento acessível;
- Em andamento: geração para acompanhamento;
- Concluído: reimpressão do estado histórico;
- Cancelado: identificação inequívoca;
- tecnologia física permanece no Bloco 10.

## Bloco 10 — Exportação e impressão

**Status: EM ANDAMENTO — Etapas 1–4 consolidadas; Etapa 5 próxima, ainda não aberta.**

Fonte ativa: `bloco-10-exportacao-impressao-ficha.md`.

A UX já foi consolidada no Bloco 8. O fechamento técnico do Bloco 10 segue uma etapa por vez.

### Etapas

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 3 | DOCX de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 4 | Impressão Windows de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 5 | Template físico de Procedimentos | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 6 | PDF + preview da Ficha compacta | PENDENTE |
| 7 | Template físico A4 da Ficha | PENDENTE |
| 8 | Limites textuais e densidade da Ficha | PENDENTE |
| 9 | Múltiplos MACs / Procedimentos na Ficha | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

### Etapa 1 — consolidado

Contrato aprovado:

- geração documental é responsabilidade do Host;
- Client solicita por identidade da fonte/revisão esperada, sem enviar documento montado;
- fonte mutável não é substituída silenciosamente por revisão mais nova;
- Host captura snapshot consistente antes da renderização;
- leitura/transação SQLite é encerrada antes do trabalho pesado de renderização;
- `DocumentModel` semântico separa regras de domínio dos renderers;
- renderers não reconsultam o banco nem recebem DOM/HTML da UI;
- geração é leitura derivada e não passa pela fila de mutações;
- renderização tem limite próprio de concorrência/backpressure;
- primeira versão não cria `export_jobs`, scheduler ou fila persistente;
- artefato retorna pela API autenticada;
- Host não escreve em path arbitrário do Client;
- runtime documental é autocontido, sem Office/LibreOffice/Adobe/Chrome externo/cloud obrigatório;
- artefatos gerados não viram histórico/backup por padrão.

### Etapa 2 — consolidada

Contrato aprovado:

- renderer PDF de Procedimentos baseado em **Typst embutido como biblioteca Rust no Host**, usando crates oficiais com adaptador interno StepFlow;
- nenhum `typst.exe`/CLI, browser ou processo conversor externo;
- template Typst interno, confiável e versionado com o produto;
- conteúdo originado do domínio entra somente como valores/dados estruturados e nunca participa da construção textual do source Typst, mesmo após escaping;
- nenhum pacote/recurso remoto é resolvido durante geração; filesystem/imports ficam restritos ao mundo virtual, templates, fontes e assets controlados pelo Host;
- PDF 1.7 é solicitado explicitamente ao exporter;
- Tagged PDF permanece explicitamente habilitado como baseline, sem prometer conformidade formal PDF/UA/PDF-A;
- texto textual permanece selecionável/pesquisável/copiável;
- fontes necessárias são empacotadas e incorporadas/subsetadas, sem depender das fontes instaladas no Windows;
- todos os blocos semânticos do Procedimento devem ser representados sem descarte silencioso; incompatibilidade deve falhar explicitamente;
- comandos/código permanecem texto e preservam whitespace relevante;
- engine suporta fluxo multipágina e quebra automática, sem definir ainda margens, A4, tipografia, cabeçalho, rodapé ou paginação visual;
- PNG/JPEG e SVG controlado são suportados somente a partir de assets previamente aceitos/resolvidos pelo Host;
- conteúdo visual não depende implicitamente do relógio/ambiente da máquina central; data/hora visível vem de dados explícitos do `DocumentModel`/`generation_metadata`;
- estabilidade visual/semântica é exigida sob mesma versão do Host/template/fontes/assets/modelo, sem exigir bytes idênticos quando metadados técnicos variarem;
- falha do renderer não produz artefato parcial tratado como sucesso;
- assinatura digital, senha, formulários, anexos, JavaScript, multimídia, PDF/A formal e PDF/UA formal ficam fora da primeira versão;
- versão exata das crates e limites numéricos de memória/tamanho/tempo não são congelados na Fase 1; ficam para implementação/medição e validação técnica posterior.

### Etapa 3 — consolidada

Contrato aprovado:

- DOCX de Procedimentos é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem converter PDF/Typst;
- saída é `.docx` real em OOXML/WordprocessingML e **OOXML Transitional** é o baseline inicial de compatibilidade; Strict não é baseline da primeira versão;
- `docx-rs` é a biblioteca preferida, encapsulada por adaptador interno StepFlow;
- não depende de Microsoft Word/COM, LibreOffice, browser/headless, CLI conversor ou serviço cloud;
- conteúdo do domínio entra apenas como dados estruturados e não pode injetar XML/OOXML, relationships, partes OPC, paths ou URLs arbitrários;
- estilos/template são internos e versionados; nenhum `.docx`/`.dotx` fornecido pelo usuário é carregado como template em runtime na primeira versão;
- texto textual permanece texto Word real, selecionável, pesquisável, copiável e editável;
- todos os blocos semânticos conhecidos são representados sem descarte silencioso; incompatibilidade falha explicitamente;
- passos/subpassos usam numeração/lista Word real quando aplicável;
- checklist exportado é documental e não vira formulário/content control interativo;
- comandos/código permanecem texto e preservam whitespace relevante;
- PNG/JPEG são baseline de imagem; SVG não é requisito direto do DOCX v1 e exige representação interna compatível ou falha explícita, nunca omissão silenciosa;
- DOCX é refluível: não há promessa de paginação idêntica ao PDF ou entre versões/consumidores Word;
- política de fontes/embedding do DOCX não é herdada automaticamente do PDF e permanece para Etapa 5/gate técnico;
- macros/VBA/`.docm`, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura digital, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
- geração só é sucesso com pacote OPC/ZIP e OOXML coerentes; artefato incompleto/corrompido não é devolvido como sucesso;
- versão exata da crate, limites numéricos e matriz real de compatibilidade ficam para implementação/Etapa 12.

### Etapa 4 — consolidada

Contrato aprovado:

- impressão física de Procedimentos acontece no Client Windows da estação do usuário, não no Host central;
- o artefato canônico de impressão é o mesmo PDF produzido pelo renderer da Etapa 2 para a revisão exata selecionada;
- não existe renderer separado de impressão e não se imprime HTML da UI nem DOCX;
- o Client usa uma WebView2 transitória/dedicada, sem navegar a webview principal para o PDF;
- a WebView2 recebe somente recurso PDF local controlado, sem Internet ou path arbitrário originado do conteúdo;
- baseline usa WebView2 `ShowPrintUI(System)` por adaptador Windows isolado sob Tauri `with_webview`;
- o diálogo padrão é o diálogo de impressão do Windows; impressão silenciosa e seletor próprio de impressoras ficam fora da primeira versão;
- StepFlow não enumera/persiste impressoras no Host e não gerencia drivers/spooler corporativo;
- `ShellExecute`/handler PDF externo, Word/COM, LibreOffice, browser/visualizador externo e spool PDF bruto não são baseline/fallback silencioso;
- recurso local de impressão é transitório; estratégia concreta, nomes, paths e limpeza ficam para a Etapa 10;
- `ShowPrintUI` não confirma impressão física: sucesso da integração significa fluxo entregue ao Windows, sem falso `Impresso com sucesso` ou auditoria `printed=true`;
- fechamento/cancelamento do diálogo não é erro funcional; falhas de geração, preparação local, compatibilidade WebView2 e abertura do diálogo são distintas;
- duplicidade concorrente acidental da mesma ação é impedida localmente sem criar fila/job persistente;
- gate técnico posterior valida Windows 10/11 x64, WebView2, PDF multipágina, Unicode/logo, impressoras locais/de rede, opções do diálogo e operação offline;
- versão mínima concreta do WebView2 fica para matriz corporativa/gate de implementação;
- layout físico final do Procedimento continua integralmente reservado para a Etapa 5.

A **Etapa 5 — Template físico de Procedimentos** é apenas a próxima. Ela ainda não está em análise. Etapas 6–12 continuam pendentes.

## Bloco 11 — Backup e restauração

**Ainda não iniciado.**

Fechar mecanismo técnico de backup consistente do SQLite + arquivos administrados.

UX normal já exige:

- operação Host-side;
- Restore autorizado + backup elegível;
- confirmação reforçada;
- safety backup antes da etapa destrutiva;
- disaster recovery sem Host funcional fora da UI normal.

Ainda fechar:

- pacote;
- atomicidade;
- verificações/checksums;
- retenção;
- compressão/criptografia quando aplicável;
- restart/reconexão/sessões;
- recuperação local de desastre.

## Bloco 12 — Fundação da Fase 2

Somente depois dos blocos anteriores:

- resolver parâmetros finais que bloqueiam implementação;
- fechar regra editorial de categoria arquivada;
- definir árvore oficial Client/Host/launcher/contratos/testes/assets;
- convenções/scripts;
- configuração de desenvolvimento;
- migrations oficiais iniciais;
- tarefas pequenas de fundação;
- plano oficial da Fase 2;
- sincronizar explicitamente o checkout local antes do primeiro trabalho de implementação com Codex.

## Pendências de autenticação/configuração

Antes da implementação correspondente:

- Argon2id final;
- senha mínima final;
- duração de sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup.

## Pendências do ambiente corporativo

- hostname/IP e paths reais;
- SMB/permissões/políticas;
- Windows/WebView2 reais;
- antivírus/EDR/firewall;
- HTTP/HTTPS;
- mecanismo real de start do Controller.

## Critérios de saída da Fase 1

- [x] Client/Windows definidos;
- [x] Host Pocket definido;
- [x] launcher/update definidos;
- [x] comunicação definida;
- [x] núcleo de autenticação/autorização definido;
- [ ] parâmetros finais de autenticação necessários à implementação;
- [x] modelo de dados original definido;
- [x] extensão operacional conceitual aprovada;
- [x] concorrência geral definida;
- [x] Telas 01–15 especificadas/aprovadas;
- [x] modelagem `Procedimento × Atendimento × Equipamento` aprovada;
- [x] lifecycle/execução/checklist decididos;
- [x] matriz operacional de permissões decidida;
- [x] códigos legíveis decididos;
- [x] gestão de categorias por preset decidida;
- [x] lifecycle/capacidade da ficha decididos;
- [x] arquitetura-base de geração documental decidida;
- [x] renderer PDF de Procedimentos decidido;
- [x] renderer DOCX de Procedimentos decidido;
- [x] impressão Windows de Procedimentos decidida;
- [ ] exportação/impressão + ficha tecnicamente definidas por completo;
- [ ] backup/restore técnico definido;
- [ ] regra editorial de categoria arquivada fechada;
- [ ] estrutura oficial definida;
- [x] pendências não bloqueantes registradas;
- [ ] plano da Fase 2 aprovado.

## Regra de execução

Não criar scaffold, runtime definitivo ou código de negócio durante a Fase 1.

Toda tarefa Codex futura que altere arquivos deve trazer base Git esperada, pré-flight de capacidade e obedecer `AGENTS.md`.