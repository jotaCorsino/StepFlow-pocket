# Bloco 10 — Etapa 11 — Validação técnica final

**Status:** VALIDAÇÃO TÉCNICA EXECUTADA / AGUARDANDO APROVAÇÃO FINAL DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Data da validação:** 2026-08-29  
**Base:** `main` em `d74459b2b342a9fda2ccc4e0645c02613edc4fc8`

## 1. Objetivo

Fechar tecnicamente o Bloco 10 verificando a viabilidade das decisões consolidadas nas Etapas 1–10, sem reabrir produto por inferência e sem antecipar implementação da Fase 2.

A validação usa documentação oficial atual e evidência de APIs disponíveis. Pontos que só podem ser comprovados na LAN/estações corporativas ficam explicitamente separados como dependência de ambiente real.

## 2. Estados usados

- **VALIDADO** — a arquitetura/mecanismo possui evidência técnica suficiente para seguir à implementação;
- **VALIDADO COM LIMITE** — o mecanismo é viável, mas existe condição técnica que precisa ser respeitada;
- **PENDENTE DE AMBIENTE REAL** — depende de Windows, Word, impressora, SMB, antivírus/EDR ou política corporativa reais;
- **BLOQUEADOR** — decisão consolidada não é tecnicamente sustentável e precisa retornar ao PO.

## 3. Resultado executivo

A matriz não encontrou bloqueador técnico que exija reabrir as decisões de produto das Etapas 1–10.

Resultado global:

| Área | Resultado |
|---|---|
| Typst / PDF Host-side | **VALIDADO** |
| Ficha PDF + preview / detecção de páginas | **VALIDADO** |
| DOCX OOXML direto | **VALIDADO COM LIMITE** |
| Transporte binário de artefatos | **VALIDADO** |
| Impressão Windows via WebView2 | **VALIDADO COM LIMITE** |
| Ficha A4 / `SHEET_OVERFLOW` | **VALIDADO COM LIMITE** |
| Naming Windows | **VALIDADO** |
| Save local / promoção do arquivo | **VALIDADO COM LIMITE** |
| Save em SMB corporativo | **PENDENTE DE AMBIENTE REAL** |
| Temporários / isolamento / scavenging | **VALIDADO COM LIMITE** |
| Limites de memória e concorrência | **VALIDADO COM LIMITE** |
| Word corporativo | **PENDENTE DE AMBIENTE REAL** |
| Impressoras reais | **PENDENTE DE AMBIENTE REAL** |
| Antivírus/EDR corporativo | **PENDENTE DE AMBIENTE REAL** |
| Bloqueadores | **NENHUM IDENTIFICADO** |

---

# 4. Pipeline Typst / PDF

## 4.1 Typst embutido no Host

**Resultado:** **VALIDADO**

A família atual de crates Typst oferece API de compilação em processo. `typst::compile` aceita um `World` e produz, entre outras saídas, um `PagedDocument`. Isso sustenta o contrato de geração no Host sem `typst.exe`, browser ou conversor externo.

O trait `World` expõe explicitamente as fontes de conteúdo, arquivos e fontes tipográficas (`main`, `source`, `file`, `font`). Portanto, o adaptador StepFlow pode implementar um mundo restrito ao template interno e aos assets/fontes permitidos, sem oferecer resolução arbitrária de filesystem ou rede.

**Contrato de implementação:**

```text
snapshot consistente
→ DocumentModel
→ adaptador Typst interno
→ World restrito
→ typst::compile<PagedDocument>
→ exportador PDF/SVG
```

Nenhum renderer consulta SQLite durante a renderização.

## 4.2 PDF 1.7 + Tagged PDF

**Resultado:** **VALIDADO**

A documentação atual do Typst informa PDF 1.7 como saída padrão e Tagged PDF habilitado por padrão. Isso sustenta exatamente a baseline aprovada do StepFlow.

O fato de Typst também oferecer perfis PDF/A e PDF/UA não altera o escopo: o StepFlow não assume conformidade formal PDF/A/PDF/UA nesta fase.

## 4.3 Toolchain Rust

**Resultado:** **VALIDADO COM LIMITE**

Na data desta validação, `typst-pdf 0.15.1` declara `rust-version = "1.92"`.

Consequência:

- a Fase 2 deve fixar uma família de crates Typst testada em conjunto;
- a toolchain de build deve satisfazer o maior MSRV das dependências escolhidas;
- se a família 0.15.1 for adotada, Rust 1.92 é o piso dessa dependência;
- isso é requisito de **build**, não toolchain a ser instalada na máquina central de produção.

Não congelar a versão 0.15.1 como requisito permanente somente porque foi a versão corrente na validação.

## 4.4 Fontes e assets do PDF

**Resultado:** **VALIDADO**

O `World` controlado permite fornecer somente Noto Sans, Noto Sans Mono e assets internos autorizados. O renderer não precisa consultar fontes remotas ou pacotes externos em runtime.

Na implementação, fontes distribuídas pelo projeto precisam ter licença compatível e ser empacotadas conforme a política já aprovada; esta validação não redistribui arquivos de fonte.

---

# 5. `PagedDocument`, preview e `SHEET_OVERFLOW`

## 5.1 Fonte única de layout

**Resultado:** **VALIDADO**

O engine de layout do Typst materializa um `PagedDocument` concluído com páginas/frames. A mesma saída paginada pode alimentar o PDF e a exportação de preview SVG.

Isso sustenta a decisão:

```text
DocumentModel da Ficha
→ PagedDocument
→ contar páginas

1 página
→ PDF + preview válidos

2+ páginas
→ SHEET_OVERFLOW
```

Não é necessário manter um segundo layout HTML para decidir se a Ficha cabe.

## 5.2 Preview SVG

**Resultado:** **VALIDADO**

A família `typst-svg` exporta páginas/documentos paginados para SVG. Para a Ficha válida, a implementação deve exportar apenas a única página aceita, mantendo PDF e preview vinculados ao mesmo `source_version` e à mesma compilação lógica.

## 5.3 Soft limits 600 / 400 / 300 / 280

**Resultado:** **VALIDADO COM LIMITE**

Os valores continuam adequados como **orientação editorial**, não como prova de encaixe físico.

A autoridade permanece:

```text
layout real Typst
→ 1 página: válido
→ 2+ páginas: SHEET_OVERFLOW
```

Os soft limits não serão convertidos em hard limits nem usados para truncar dados. Ajuste futuro só é autorizado por evidência de fixtures representativas, sem alterar a regra de uma A4.

## 5.4 Fixtures mínimas da implementação

A implementação deverá possuir casos determinísticos cobrindo pelo menos:

- Ficha sem Equipamento;
- Ficha com campos opcionais vazios;
- 0, 1, 2 e 3+ MACs;
- Unicode/acentos;
- strings estruturadas longas;
- muitas observações por Etapa;
- conteúdo que caiba exatamente em uma página;
- conteúdo que gere duas páginas e obrigatoriamente resulte em `SHEET_OVERFLOW`;
- ausência de truncamento e redução automática de fonte.

Essas fixtures pertencem à implementação/testes da Fase correspondente; não são código produzido nesta Etapa 11.

---

# 6. DOCX OOXML

## 6.1 Geração direta em Rust

**Resultado:** **VALIDADO COM LIMITE**

A versão corrente de `docx-rs` expõe construção de documento, parágrafos, tabelas, numeração, estilos, imagens e empacotamento DOCX. O caminho técnico de OOXML direto em Rust é viável e não exige Word/COM, LibreOffice ou conversão do PDF.

A decisão de manter `docx-rs` sob adaptador interno permanece adequada. O adaptador deve impedir que detalhes da biblioteca vazem para o domínio e permite substituição futura sem alterar `DocumentModel`.

## 6.2 Baseline Transitional

**Resultado:** **VALIDADO COM LIMITE**

O pacote gerado usa a família WordprocessingML/OPC esperada pelo Word. A validação documental da biblioteca é suficiente para manter OOXML Transitional como baseline arquitetural; a abertura concreta nas edições corporativas do Microsoft Word permanece teste obrigatório do ambiente real.

## 6.3 Arial / Consolas sem embedding

**Resultado:** **VALIDADO COM LIMITE**

O Word pode substituir uma fonte ausente para visualização e a própria Microsoft alerta que a substituição pode alterar design/layout.

Portanto:

- DOCX permanece refluível e não promete paginação idêntica ao PDF;
- Arial/Consolas podem continuar apenas referenciadas;
- ausência da fonte não deve corromper o documento;
- fidelidade visual nas máquinas corporativas precisa ser verificada com as fontes realmente disponíveis.

## 6.4 Word corporativo

**Resultado:** **PENDENTE DE AMBIENTE REAL**

Validar quando houver acesso ao ambiente:

- abrir DOCX gerado nas versões de Word realmente utilizadas;
- texto/listas/numeração editáveis;
- imagens;
- comandos/código preservando whitespace;
- comportamento com Arial/Consolas;
- salvar novamente sem corrupção relevante.

Não definir uma lista fictícia de versões de Office antes de conhecer o parque real.

---

# 7. Transporte dos artefatos Host → Client

## 7.1 Corpo binário

**Resultado:** **VALIDADO**

Axum suporta respostas de bytes (`Vec<u8>`, `Bytes` e corpos equivalentes). Portanto, PDF, DOCX e SVG não precisam ser convertidos para base64 dentro de JSON.

Contrato recomendado:

```text
controle/metadados/erros
→ JSON

artefato documental
→ body binário autenticado
→ Content-Type apropriado
→ source_version/metadados necessários em headers ou envelope de controle definido pela API
```

Isso preserva HTTP/JSON como protocolo de controle sem impor overhead de base64 aos artefatos.

## 7.2 Tamanho máximo

**Resultado:** **VALIDADO COM LIMITE**

A arquitetura aceita bytes em memória, mas nenhum número arbitrário é congelado nesta fase.

A Fase 2 deve medir:

- tamanho de Procedimentos representativos e extremos;
- PDF/DOCX simultaneamente em memória;
- consumo do Host e do Client;
- latência na LAN real.

Hard limits de payload devem resultar dessas medições e de critérios de proteção do Host, nunca da geometria A4.

---

# 8. Concorrência de renderização

**Resultado:** **VALIDADO COM LIMITE**

Renderização Typst/DOCX é trabalho CPU/blocking e não deve ocupar a fila lógica de mutações SQLite.

Tokio documenta `spawn_blocking` para trabalho bloqueante e recomenda semáforo ou outro mecanismo de limitação quando há muitas computações CPU-bound.

Baseline técnico:

```text
requisição autorizada
→ captura DocumentModel
→ fila/admissão bounded de documentos
→ semáforo de renderização
→ spawn_blocking / executor CPU apropriado
→ resultado
```

Quando saturado, o Host deve aplicar backpressure/`SERVER_BUSY` equivalente, sem criar fila persistente `export_jobs`.

Número de renderizações simultâneas, profundidade da fila e timeout ficam para medição na Fase 2. Não há justificativa para congelar valores sem benchmark.

---

# 9. Impressão Windows e WebView2

## 9.1 API de impressão

**Resultado:** **VALIDADO**

WebView2 oferece oficialmente `ShowPrintUI`, incluindo o diálogo de impressão do sistema. Isso sustenta a decisão de entregar o PDF oficial ao fluxo Windows sem seletor próprio e sem impressão silenciosa como baseline.

## 9.2 Fronteira Tauri → WebView2 nativo

**Resultado:** **VALIDADO COM LIMITE**

Tauri atual expõe `with_webview`, que fornece o handle específico da plataforma na main thread. A própria documentação alerta que crates nativas como `webview2-com` podem mudar em releases minor do Tauri e recomenda pinagem pelo menos na minor quando `with_webview` for usado.

Além disso, o método Rust de alto nível `WebviewWindow::print()` é documentado como suportado no `wry` apenas no macOS. Portanto, no Windows o StepFlow não deve depender dessa API genérica para o contrato aprovado.

Baseline Windows:

```text
WebviewWindow dedicada
→ with_webview(...)
→ WebView2 nativo
→ ICoreWebView2 / interface compatível
→ ShowPrintUI(System)
```

Consequências para a Fase 2:

- pinagem de uma família Tauri/Wry/WebView2 testada;
- pequeno adaptador Windows isolando a chamada nativa;
- nenhum acesso WebView2 espalhado pela UI/business code.

## 9.3 PDF local e lifecycle do temporário

**Resultado:** **PENDENTE DE AMBIENTE REAL**

WebView2 suporta conteúdo local, mas documentação de API isolada não prova o momento exato em que um PDF local pode ser removido com segurança durante todos os caminhos do diálogo/spooler.

Teste obrigatório em Windows real:

```text
criar PDF transitório
→ carregar WebView2 dedicada
→ abrir ShowPrintUI(System)
→ imprimir / cancelar
→ fechar consumidor
→ tentar cleanup
```

Critérios:

- não remover enquanto o consumidor ainda exige o arquivo;
- cancelamento não é erro de renderer;
- lock posterior vira cleanup best-effort;
- nenhuma dependência de Adobe Reader ou browser externo.

## 9.4 Impressoras

**Resultado:** **PENDENTE DE AMBIENTE REAL**

O mecanismo de diálogo do sistema está validado. Permanecem testes com:

- ao menos uma impressora física representativa;
- Microsoft Print to PDF;
- driver corporativo real;
- cancelamento;
- indisponibilidade de impressora.

Sucesso do StepFlow continua significando entrega ao fluxo Windows, não confirmação de papel fisicamente impresso.

---

# 10. Save dialog e naming Windows

## 10.1 Diálogo nativo

**Resultado:** **VALIDADO**

O plugin oficial de diálogo do Tauri oferece diálogo nativo de salvar no Windows. O usuário continua escolhendo pasta/nome e o StepFlow não precisa implementar um navegador de arquivos próprio.

## 10.2 Sanitização

**Resultado:** **VALIDADO**

As regras aprovadas estão alinhadas às restrições documentadas do Windows:

- bloquear caracteres inválidos e controles;
- impedir componentes de path injetados pelo domínio;
- impedir nomes reservados;
- remover ponto/espaço final inválido;
- preservar Unicode válido;
- extensão vem do tipo de artefato;
- fallback seguro se o segmento sanitizado ficar vazio.

O título variável pode ser limitado no **filename** para manter o nome manejável; isso não é truncamento do conteúdo documental.

## 10.3 Caminhos longos

**Resultado:** **VALIDADO COM LIMITE**

O Windows moderno possui suporte a caminhos longos para APIs compatíveis, mas a ativação também depende de manifesto `longPathAware` e configuração/política do sistema para APIs afetadas.

Consequências:

- o executável StepFlow deve declarar suporte quando aplicável;
- o produto não deve depender de paths arbitrariamente longos para funcionar;
- nomes sugeridos permanecem manejáveis;
- erro por path/política é reportado sem alterar o documento;
- política real da empresa será verificada no ambiente corporativo.

---

# 11. Escrita segura do arquivo persistente

## 11.1 Destino local NTFS

**Resultado:** **VALIDADO COM LIMITE**

APIs Windows fornecem primitivas para flush e substituição/movimentação de arquivo. Isso sustenta a estratégia aprovada de auxiliar opaco no mesmo diretório do destino e promoção ao final.

Baseline conceitual:

```text
bytes completos
→ helper opaco no diretório final
→ escrever completamente
→ flush/close
→ promover/substituir conforme fluxo já confirmado pelo usuário
→ somente então declarar sucesso
```

Garantias:

- StepFlow não reporta sucesso sobre arquivo parcial;
- falha remove best-effort somente o helper/parcial criado pela própria operação;
- arquivo preexistente do usuário nunca é apagado por cleanup genérico;
- confirmação de substituição continua responsabilidade do fluxo visível ao usuário.

Não declarar transação ACID de filesystem nem promessa absoluta contra qualquer falha de energia.

## 11.2 SMB

**Resultado:** **PENDENTE DE AMBIENTE REAL**

Algumas primitivas Windows funcionam sobre SMB, mas não é seguro prometer sem teste que todos os servidores, NAS, políticas, antivírus e interrupções de rede forneçam semântica idêntica ao NTFS local.

Validar no compartilhamento corporativo real:

- criar helper no mesmo diretório;
- write/flush/close;
- replace/move permitido;
- permissões de criação/rename/delete;
- conexão interrompida;
- destino existente;
- espaço/quota insuficiente;
- latência;
- estado observável após falha.

Regra permanente: **não prometer atomicidade idêntica entre NTFS e qualquer SMB sem evidência do ambiente alvo**.

---

# 12. Temporários e isolamento por instância

## 12.1 Diretório temporário

**Resultado:** **VALIDADO**

Tauri expõe resolução de diretório temporário do sistema. A raiz não precisa ser hardcoded e não precisa ficar em instalação, SMB central, banco, backup, Documents, Desktop ou Downloads.

Namespace lógico preservado:

```text
<temp do usuário>/StepFlow/artifacts/<client-instance-id>/
```

## 12.2 `client-instance-id` e `active.lock`

**Resultado:** **VALIDADO COM LIMITE**

A detecção de instância ativa, que havia ficado para esta Etapa 11, pode ser implementada de forma simples no Windows sem daemon:

```text
iniciar Client
→ criar diretório opaco da instância
→ criar active.lock
→ manter handle aberto durante toda a vida da instância
→ abrir com share_mode(0)
```

Rust/Windows permite controlar `dwShareMode`; remover flags de compartilhamento impede que outro processo realize operações conflitantes enquanto o handle estiver aberto.

Scavenger:

```text
para cada diretório de outra instância
→ verificar que pertence ao namespace StepFlow
→ rejeitar travessia por reparse point
→ tentar abrir active.lock de forma exclusiva
   → sharing violation / incerteza: SKIP
   → lock adquirível: candidato órfão
→ cleanup best-effort
```

Esse mecanismo é uma decisão **técnica de implementação**, não um novo conceito de produto.

## 12.3 Reparse points / symlinks

**Resultado:** **VALIDADO**

Windows expõe atributos/tags de reparse point e opções para abrir o próprio ponto sem segui-lo normalmente. O scavenger deve conferir cada componente relevante dentro da raiz administrada e nunca atravessar um reparse point para apagar conteúdo fora do namespace StepFlow.

Regra conservadora:

```text
incerteza sobre ownership/atividade/reparse
→ não apagar
```

## 12.4 Locks por WebView2/antivírus

**Resultado:** **VALIDADO COM LIMITE** para o comportamento seguro; **PENDENTE DE AMBIENTE REAL** para EDR corporativo específico.

A política aprovada permanece:

- falhou cleanup por lock → registrar diagnóstico mínimo;
- não matar processo;
- não forçar unlock;
- não mudar ACL;
- tentar novamente em ponto seguro;
- se continuar bloqueado, deixar para scavenging posterior.

EDR/antivírus real precisa ser testado na empresa.

---

# 13. Múltiplos Clients

**Resultado:** **VALIDADO COM LIMITE**

A arquitetura possui isolamento suficiente:

- `client-instance-id` opaco;
- subdiretório temporário exclusivo;
- `active.lock` por instância;
- nomes de artefatos opacos/únicos;
- scavenger conservador;
- Host com admissão bounded de geração.

Teste de stress com múltiplas estações/instâncias simultâneas permanece para a fase executável e, idealmente, para a LAN corporativa.

Nenhuma instância pode limpar indiscriminadamente a pasta de outra instância potencialmente ativa.

---

# 14. Windows 10/11 e WebView2 Runtime

**Resultado:** **VALIDADO COM LIMITE**

O baseline Windows 10/11 x64 permanece tecnicamente compatível com o ecossistema WebView2 atual. Entretanto, a presença/versão real do Runtime nas máquinas corporativas deve ser validada no deployment.

A implementação deve:

- detectar erro de runtime ausente/incompatível de forma clara;
- não baixar dependência da Internet silenciosamente em produção;
- seguir a estratégia de distribuição Pocket aprovada;
- não transformar ausência do Runtime numa justificativa para browser externo.

A versão exata mínima do WebView2 Runtime será fixada junto da família Tauri/Wry testada na Fase 2, não por número arbitrário nesta etapa documental.

---

# 15. Matriz final consolidada

| ID | Contrato | Estado | Consequência |
|---|---|---|---|
| D01 | Typst embutido no Host | **VALIDADO** | usar crates oficiais sob adaptador interno |
| D02 | `World` restrito | **VALIDADO** | somente template/assets/fontes autorizados |
| D03 | PDF 1.7 | **VALIDADO** | manter default/setting explícito testado |
| D04 | Tagged PDF baseline | **VALIDADO** | não desligar tags por padrão |
| D05 | Typst atual exige Rust 1.92 | **VALIDADO COM LIMITE** | toolchain de build deve satisfazer MSRV da família fixada |
| D06 | `PagedDocument` como fonte física | **VALIDADO** | PDF/preview/page-count derivam da mesma compilação lógica |
| D07 | Ficha 1 página / `SHEET_OVERFLOW` | **VALIDADO** | page count é autoridade física |
| D08 | soft limits 600/400/300/280 | **VALIDADO COM LIMITE** | orientação; ajustar somente por fixtures/evidência |
| D09 | DOCX Rust direto | **VALIDADO COM LIMITE** | `docx-rs` sob adapter; testar Word real |
| D10 | Arial/Consolas sem embedding | **VALIDADO COM LIMITE** | substituição de fonte pode alterar reflow |
| D11 | Word corporativo | **PENDENTE DE AMBIENTE REAL** | testar versões realmente instaladas |
| D12 | artefato binário HTTP | **VALIDADO** | não usar base64 JSON sem necessidade |
| D13 | geração fora da fila SQLite | **VALIDADO** | render CPU/blocking separado |
| D14 | bounded rendering | **VALIDADO COM LIMITE** | semáforo/admissão; números via benchmark |
| D15 | `ShowPrintUI(System)` | **VALIDADO** | diálogo Windows oficial |
| D16 | Tauri → WebView2 via `with_webview` | **VALIDADO COM LIMITE** | pin Tauri minor + adapter Windows |
| D17 | API Tauri Rust `print()` no Windows | **NÃO UTILIZAR** | não atende o contrato Windows atual |
| D18 | PDF local + lifetime de print | **PENDENTE DE AMBIENTE REAL** | teste Windows end-to-end obrigatório |
| D19 | impressoras reais | **PENDENTE DE AMBIENTE REAL** | driver/diálogo/cancelamento |
| D20 | save dialog nativo | **VALIDADO** | plugin oficial Tauri |
| D21 | sanitização Windows | **VALIDADO** | regras já consolidadas permanecem |
| D22 | long paths | **VALIDADO COM LIMITE** | manifesto + política do Windows; nome manejável |
| D23 | helper no diretório final | **VALIDADO COM LIMITE** | write/flush/close/promote; sem promessa ACID |
| D24 | SMB corporativo | **PENDENTE DE AMBIENTE REAL** | validar servidor/permissões/falhas reais |
| D25 | temp dir via Tauri/OS | **VALIDADO** | sem path hardcoded |
| D26 | isolamento por instância | **VALIDADO** | diretório e IDs próprios |
| D27 | `active.lock` | **VALIDADO COM LIMITE** | handle exclusivo durante lifecycle do Client |
| D28 | reparse-safe scavenging | **VALIDADO** | incerteza sempre resulta em skip |
| D29 | locks WebView2/AV | **VALIDADO COM LIMITE** | cleanup best-effort, nunca force unlock |
| D30 | EDR corporativo | **PENDENTE DE AMBIENTE REAL** | testar comportamento/locks reais |
| D31 | múltiplos Clients | **VALIDADO COM LIMITE** | stress/tuning posterior |
| D32 | Windows 10/11 + WebView2 | **VALIDADO COM LIMITE** | runtime real e versão fixada na Fase 2 |

**BLOQUEADORES identificados: 0.**

---

# 16. Decisões técnicas refinadas pela validação

Estas conclusões detalham a implementação sem alterar o produto aprovado:

1. **Artefatos documentais trafegam preferencialmente como body binário**, não base64 dentro de JSON.
2. **Renderização CPU/blocking usa admissão bounded + semáforo**, separada da fila lógica de mutações.
3. **Impressão Windows usa acesso nativo ao WebView2 por `with_webview`**, porque o método Rust genérico de print do Tauri/Wry não é o baseline Windows.
4. **A família Tauri/Wry/WebView2 usada nesse adapter deve ser pinada e testada**, no mínimo no nível minor recomendado pelo próprio Tauri para `with_webview`.
5. **A toolchain Rust de build deve satisfazer o MSRV da família Typst fixada**; na referência atual 0.15.1, `typst-pdf` exige Rust 1.92.
6. **Scavenging usa sinal explícito de atividade por `active.lock`** e política conservadora de skip.
7. **Reparse point nunca é seguido por cleanup para fora da raiz gerenciada.**
8. **Long path não é garantia unilateral do aplicativo**; depende também da configuração/política Windows.
9. **Save local usa helper no mesmo destino + write/flush/close + promoção**, sem prometer transação absoluta de filesystem.
10. **Nenhuma garantia de atomicidade genérica é feita para SMB** até teste do compartilhamento real.
11. **Valores de memória, concorrência, fila e timeout não são congelados nesta Fase 1**; serão medidos na fundação executável.

---

# 17. Pendências de ambiente real

Estas pendências não reabrem arquitetura e não impedem o fechamento documental do Bloco 10, desde que permaneçam gates explícitos antes do rollout correspondente:

### Windows / WebView2

- carregar PDF local real;
- `ShowPrintUI(System)`;
- cancelar/imprimir;
- fechar WebView2;
- verificar momento de cleanup do temporário.

### Impressão

- impressora física representativa;
- Microsoft Print to PDF;
- driver real corporativo.

### Word

- versões realmente instaladas;
- Arial/Consolas presentes ou substituídas;
- abrir/editar/salvar DOCX gerado.

### SMB

- permissões reais;
- helper/rename/replace;
- interrupção de rede;
- quota/espaço;
- latência;
- comportamento pós-falha.

### Segurança corporativa

- antivírus/EDR segurando handles;
- firewall/políticas relevantes;
- long paths habilitados ou não.

Quando o teste não puder ser feito fora da empresa, registrar `NÃO APLICÁVEL NESTE AMBIENTE` no ensaio local e manter o gate corporativo aberto.

---

# 18. Critérios técnicos para a implementação futura

A Fase 2/implementação correspondente não deve iniciar o renderer documental sem definir no código/testes:

- versões pinadas e compatíveis de Typst/Tauri/Wry/WebView2/docx-rs;
- MSRV/toolchain do build;
- adaptadores `pdf`, `docx`, `windows_print` e filesystem isolados de domínio/UI;
- `World` Typst restrito;
- testes de page count / `SHEET_OVERFLOW`;
- fixtures Unicode e conteúdo extenso;
- política bounded de renderização;
- transporte binário e limites medidos;
- sanitização de filename;
- escrita completa + promoção segura;
- isolamento de temporários + `active.lock`;
- proteção de reparse points;
- cleanup best-effort;
- tratamento explícito de `SERVER_BUSY`, renderer failure, filesystem failure e runtime/print failure.

Nenhum desses itens autoriza scaffold ou código nesta Etapa 11.

---

# 19. Critério proposto de encerramento do Bloco 10

Tecnicamente, o Bloco 10 pode ser encerrado após aprovação final do PO desta matriz porque:

1. os contratos das Etapas 1–10 possuem mecanismo técnico identificável;
2. não foi encontrado bloqueador arquitetural;
3. limitações reais estão explícitas e não são mascaradas como sucesso;
4. pontos dependentes da empresa estão separados como validação de ambiente;
5. nenhum requisito de produto novo foi introduzido;
6. nenhum código de produção foi criado;
7. parâmetros que exigem benchmark foram conscientemente deixados para medição, em vez de inventados.

Após aprovação do PO, esta Etapa 11 deve ser promovida ao estado consolidado nos mapas canônicos e o Bloco 10 pode ser marcado como concluído documentalmente, sujeito aos gates de ambiente real registrados para a implementação/deployment correspondentes.

---

# 20. Fontes externas verificadas em 2026-08-29

Fontes oficiais/primárias usadas para confirmar mecanismos; versões listadas são referência da data da validação e não fixação automática de dependência:

- Typst — PDF: https://typst.app/docs/reference/pdf/
- `typst::compile`: https://docs.rs/typst/latest/typst/fn.compile.html
- Typst `World`: https://docs.rs/typst/latest/typst/trait.World.html
- `typst_layout` / `PagedDocument`: https://docs.rs/typst-layout/latest/typst_layout/
- `typst-svg`: https://docs.rs/typst-svg/latest/typst_svg/
- `typst-pdf` Cargo metadata: https://docs.rs/crate/typst-pdf/latest/source/Cargo.toml
- `docx-rs`: https://docs.rs/docx-rs/latest/docx_rs/
- Tauri `WebviewWindow` / `with_webview`: https://docs.rs/tauri/latest/tauri/webview/struct.WebviewWindow.html
- Tauri dialog plugin: https://v2.tauri.app/plugin/dialog/
- Tauri path APIs: https://docs.rs/tauri/latest/tauri/path/struct.PathResolver.html
- Microsoft WebView2 printing: https://learn.microsoft.com/en-us/microsoft-edge/webview2/how-to/print
- Microsoft WebView2 local content: https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/working-with-local-content
- Windows filename/path rules: https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file
- Windows long paths: https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation
- `ReplaceFileW`: https://learn.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-replacefilew
- `FlushFileBuffers`: https://learn.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-flushfilebuffers
- Windows reparse points: https://learn.microsoft.com/en-us/windows/win32/fileio/reparse-point-operations
- Rust Windows `OpenOptionsExt::share_mode`: https://doc.rust-lang.org/std/os/windows/fs/trait.OpenOptionsExt.html
- Tokio `spawn_blocking`: https://docs.rs/tokio/latest/tokio/task/fn.spawn_blocking.html
- Axum `IntoResponse`: https://docs.rs/axum/latest/axum/response/trait.IntoResponse.html
- Microsoft Office — missing fonts: https://support.microsoft.com/en-us/office/fonts/use-the-modern-font-picker-in-office

## 21. Não objetivos preservados

Esta validação não autoriza:

- scaffold/runtime oficial;
- migrations;
- implementação funcional do renderer;
- implementação do fluxo de impressão;
- serviço/daemon/watchdog;
- template extra;
- formato de exportação adicional;
- alteração de UX sem bloqueador técnico;
- sincronização destrutiva do checkout local;
- valores arbitrários de performance tratados como contrato.
