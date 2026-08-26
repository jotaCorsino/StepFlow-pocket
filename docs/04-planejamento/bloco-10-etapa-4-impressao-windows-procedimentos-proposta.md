# Bloco 10 — Etapa 4 — Impressão Windows de Procedimentos — Proposta para análise

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-26  
**Base consolidada:** Bloco 10 / Etapas 1–3

## 1. Objetivo

Definir somente o contrato técnico da **impressão Windows de Procedimentos**, preservando:

- a revisão exata selecionada/autorizada;
- o `DocumentModel` e o renderer PDF já consolidados;
- a responsabilidade local do Client pela experiência de impressão;
- o runtime Pocket/autocontido;
- a UX consolidada na Tela 14.

Esta etapa não define margens, tipografia, cabeçalho/rodapé, A4 final, regras de quebra ou composição física do Procedimento. Esses pontos permanecem na **Etapa 5 — Template físico de Procedimentos**.

Também não define nomes concretos de temporários nem política final de arquivos temporários, reservados para a **Etapa 10**.

## 2. Contratos herdados

Permanecem vigentes:

- impressão é obrigatória para Procedimentos;
- impressão é leitura/derivação e não altera domínio;
- fonte = revisão exata selecionada/autorizada;
- Host captura snapshot consistente e materializa `DocumentModel` antes da renderização;
- Client não imprime DOM/HTML da interface;
- PDF e DOCX são artefatos derivados independentes do mesmo `DocumentModel`;
- PDF de Procedimentos usa o renderer Typst consolidado na Etapa 2;
- DOCX continua editável/refluível e não é baseline de impressão física;
- artefato retorna pela API autenticada;
- Host não escreve em path arbitrário do Client;
- Client é responsável por destino local/preview/impressão;
- runtime normal não depende de Office, LibreOffice, Adobe Reader, Chrome externo ou cloud.

## 3. Onde a impressão acontece

A impressão física acontece no **Client Windows da estação do usuário**, não no Host central.

Motivos:

- impressoras pertencem ao contexto local da estação/usuário;
- o diálogo deve refletir as impressoras e drivers instalados no Windows daquele usuário;
- múltiplos Clients podem usar impressoras diferentes simultaneamente;
- o Host não deve conhecer, selecionar ou armazenar impressoras das estações;
- imprimir pelo Host central acoplaria o sistema a impressoras/filas da máquina central e produziria comportamento incorreto em cenário multiusuário.

Fluxo conceitual:

```text
Leitor
→ Imprimir
→ Client solicita artefato da revisão esperada
→ Host autentica/autoriza
→ Host materializa DocumentModel
→ PdfRenderer consolidado da Etapa 2
→ PDF bytes
→ Client recebe
→ recurso local transitório de impressão
→ WebView2 dedicado carrega o PDF
→ diálogo nativo de impressão do Windows
→ usuário escolhe impressora/configuração ou cancela
```

## 4. Artefato canônico de impressão

O baseline de impressão de Procedimentos é o **mesmo PDF dedicado gerado pelo renderer consolidado na Etapa 2**.

Consequências:

- não existe um terceiro renderer "de impressão";
- não imprimir HTML da Tela 05;
- não imprimir o DOCX;
- não converter DOCX para impressão;
- não executar nova composição visual no Client;
- `Exportar PDF` e `Imprimir` compartilham o mesmo contrato de conteúdo/layout PDF para a mesma revisão e mesma versão do Host/template/assets.

A intenção da operação pode ser identificada separadamente para autorização/UX, mas os bytes documentais de impressão são os bytes de um PDF válido do renderer oficial.

## 5. Por que PDF e não DOCX

DOCX foi consolidado na Etapa 3 como formato editável/refluível e não promete paginação idêntica entre consumidores.

PDF foi consolidado na Etapa 2 como documento paginado com estabilidade visual/semântica sob a mesma versão do renderer.

Portanto:

```text
PDF  → baseline de impressão
DOCX → baseline de edição/exportação
```

Isso evita depender de Microsoft Word/Office e evita divergência de paginação entre exportação PDF e impressão.

## 6. Superfície transitória de impressão no Client

O Client usa uma **WebView2 dedicada e transitória para impressão**, separada da webview principal da aplicação.

Regras:

- a webview principal não é navegada para o PDF;
- estado da UI/Reader não é substituído pelo documento de impressão;
- a superfície de impressão recebe somente o recurso local controlado pelo StepFlow;
- não navega para Internet;
- não recebe URL remota originada do Procedimento;
- termina junto com a operação de impressão;
- não vira janela funcional permanente nem item de navegação/sidebar.

A forma exata da janela auxiliar (visibilidade, chrome, ownership e detalhes de lifecycle) fica para implementação, desde que o diálogo de impressão permaneça associado à experiência do Client e não prejudique a janela principal.

## 7. Mecanismo Windows proposto

No Windows, Tauri usa **Microsoft Edge WebView2**, já parte da stack consolidada do Client.

A direção proposta é:

```text
Tauri WebviewWindow transitória
→ acesso platform-specific via `with_webview`
→ CoreWebView2 da WebView2
→ `ShowPrintUI(System)`
→ diálogo de impressão do Windows
```

O adaptador Windows de impressão fica isolado da UI/domínio.

Não depender da API genérica `Webview::print()` do Tauri como contrato arquitetural, porque seu suporte nativo é platform-specific. O StepFlow pode acessar o WebView2 subjacente pelo mecanismo oficial de `with_webview` do Tauri e usar a API de impressão do WebView2.

A chamada exata/interface COM e versões de crates ficam para o gate de implementação. Tauri/WebView2 devem ser fixados de forma compatível quando esse adaptador for implementado.

## 8. Diálogo baseline

O baseline proposto é **o diálogo de impressão do sistema Windows**, acionado por WebView2 `ShowPrintUI` com o tipo `System`.

Objetivo:

- usar experiência familiar do Windows;
- mostrar impressoras instaladas para o usuário;
- deixar opções dependentes do driver/Windows no próprio diálogo;
- não criar seletor de impressora próprio do StepFlow;
- não armazenar "impressora padrão do StepFlow";
- não imprimir silenciosamente por padrão.

A impressão silenciosa por `Print`/`PrintAsync` não é requisito inicial.

## 9. Recurso local usado pela WebView2

WebView2 suporta carregamento local de PDF.

O PDF recebido do Host deve ser apresentado à webview de impressão como **recurso local transitório controlado pelo Client**.

Esta etapa consolida apenas:

- bytes são os mesmos produzidos pelo renderer PDF oficial;
- nenhum path fornecido pelo usuário é necessário;
- nenhum recurso remoto é buscado;
- o recurso não vira histórico/backup/exportação persistente apenas por imprimir;
- a vida do recurso é limitada à operação.

A materialização concreta pode ser por recurso interno em memória/protocolo local ou arquivo temporário privado. Nomes, diretórios, estratégia de temporários e limpeza concreta ficam para a **Etapa 10**.

## 10. Permissão e revisão exata

Ao clicar `Imprimir`, o Client solicita a operação usando a identidade/revisão esperada.

O Host valida:

```text
capacidade de ler a revisão selecionada
+
capacidade de exportar/imprimir
```

Se a revisão esperada estiver indisponível/sem autorização, a impressão não começa.

Uma nova revisão publicada durante a operação não substitui silenciosamente a revisão já selecionada.

## 11. Impressoras e drivers

StepFlow não gerencia infraestrutura de impressora.

Ficam sob responsabilidade do Windows/ambiente corporativo:

- instalação de impressoras;
- drivers;
- descoberta de impressoras locais/de rede;
- spooler do Windows;
- preferências específicas do driver;
- disponibilidade física da impressora.

O Host central não enumera impressoras das estações.

O Client não persiste inventário próprio de impressoras na primeira versão.

## 12. Sem `ShellExecute`/visualizador PDF externo

Não usar como baseline:

```text
ShellExecute/ShellExecuteEx + verbo `print`
→ aplicativo associado a .pdf
→ impressão
```

Esse mecanismo depende do handler registrado para `.pdf`, podendo variar conforme a estação e iniciar software externo.

O StepFlow deve usar o WebView2 já requerido pelo próprio Client, evitando depender de Adobe Reader, Edge externo, outro visualizador PDF padrão ou associação de arquivo do Windows.

## 13. Sem spool direto de PDF

Não enviar PDF bruto diretamente ao spooler/impressora como contrato inicial.

Motivos:

- suporte nativo a PDF varia conforme driver/impressora;
- exigiria negociação de capacidade ou renderer/rasterização adicional;
- duplicaria responsabilidade já atendida pela pilha WebView2/Windows;
- ampliaria superfície técnica sem necessidade de produto.

Também não adicionar PDFium/MuPDF/engine de rasterização apenas para imprimir enquanto WebView2 atender ao contrato.

## 14. Resultado, cancelamento e verdade da UI

Abrir o diálogo nativo com `ShowPrintUI` **não fornece ao StepFlow uma confirmação confiável de "impresso" versus "usuário cancelou"**.

Portanto a primeira versão não pode inventar sucesso físico.

Contrato proposto:

- falha ao gerar PDF = falha de geração;
- falha ao preparar recurso local/WebView2 = falha de preparação de impressão;
- falha ao abrir o diálogo = falha de impressão/compatibilidade;
- diálogo aberto com sucesso = fluxo de impressão entregue ao Windows;
- cancelar antes de abrir o diálogo, quando detectável pelo StepFlow, é cancelamento voluntário;
- depois que o diálogo do Windows está aberto, o StepFlow não afirma `Impresso com sucesso` apenas porque o diálogo fechou;
- fechamento/cancelamento do diálogo não vira erro funcional;
- não existe auditoria persistente `printed=true` na primeira versão.

Se no futuro for necessário confirmar status de submissão a uma impressora, isso exigirá outro contrato usando APIs que exponham status (`Print`/`PrintAsync`) e possivelmente uma experiência de seleção própria; não inferir agora.

## 15. Estados UX técnicos da impressão

Mapeamento mínimo:

```text
preparando PDF
→ preparando impressão local
→ abrindo diálogo do Windows
→ fluxo entregue ao Windows
```

Erros distintos:

- Host indisponível;
- sem permissão;
- revisão indisponível/obsoleta;
- falha do renderer PDF;
- recurso local não pôde ser preparado;
- WebView2 incompatível/indisponível;
- diálogo de impressão não pôde ser aberto.

Não mostrar percentual sem métrica real.

Não mostrar confirmação física falsa de impressão concluída.

## 16. Concorrência e duplicidade local

A geração do PDF continua sujeita ao limite de renderização documental do Host.

A etapa não cria fila persistente nem job de impressão.

No Client, a implementação deve impedir múltiplas invocações acidentais concorrentes da mesma ação enquanto o fluxo correspondente está sendo preparado/aberto.

Esse controle é local e transitório; não vira lock global de Procedimento nem estado persistente.

## 17. Compatibilidade WebView2

A impressão passa a exigir que o WebView2 disponível na estação suporte o mecanismo consolidado.

O gate técnico deve validar, em Windows 10/11 x64 representativos:

- carregamento do PDF local na WebView2;
- PDF multipágina;
- Unicode/acentos;
- imagens/logo;
- abertura do `ShowPrintUI(System)`;
- impressoras locais e de rede instaladas no Windows;
- seleção de página/quantidade/orientação conforme o diálogo/driver;
- cancelamento sem erro funcional;
- ausência de dependência de Internet;
- retorno à janela principal sem perder estado;
- nenhuma exigência de visualizador PDF externo.

Se a interface de impressão necessária não estiver disponível no runtime WebView2 suportado, o Client deve reportar incompatibilidade explicitamente. Não usar fallback silencioso para `ShellExecute`/Office/visualizador externo.

A versão mínima concreta de WebView2 continua dependente da matriz corporativa e do gate de implementação.

## 18. Segurança

A webview transitória de impressão:

- só carrega o PDF controlado pelo StepFlow;
- não recebe HTML arbitrário do Procedimento;
- não busca Internet;
- não recebe token/senha em URL;
- não abre path fornecido pelo conteúdo;
- não expõe API de seleção de arquivo/pasta ao Procedimento;
- é descartada após o fluxo;
- não altera a revisão oficial nem estado operacional.

Links textuais presentes no PDF não autorizam navegação externa automática durante a impressão.

## 19. Alternativas consideradas

### Imprimir HTML da interface

Rejeitado. Duplicaria regras de layout, quebraria a separação `DocumentModel`/renderer e poderia divergir do PDF oficial.

### Imprimir DOCX pelo Word

Rejeitado. DOCX é refluível e exigiria Microsoft Word/Office ou outro processador externo.

### `ShellExecuteEx("print")` no PDF

Rejeitado como baseline. Depende do verbo/handler registrado para `.pdf` e inicia aplicativo associado externo.

### Spool direto do PDF / GDI próprio

Rejeitado inicialmente. Nem todo driver aceita PDF bruto e a alternativa exigiria renderer/rasterização adicional e lógica de impressão de baixo nível.

### WebView2 + PDF oficial + `ShowPrintUI(System)`

Direção proposta porque reutiliza dois componentes já exigidos pelo produto:

- renderer PDF oficial do Host;
- WebView2 do Client Tauri.

Mantém impressão local, usa o diálogo Windows e não adiciona visualizador/conversor externo.

## 20. Decisões propostas ao PO — Etapa 4

A Etapa 4 propõe consolidar somente:

1. impressão física de Procedimentos acontece no **Client Windows**, não no Host;
2. o artefato canônico de impressão é o **PDF produzido pelo renderer da Etapa 2** para a revisão exata;
3. não existe renderer separado de impressão nem impressão de HTML/DOCX;
4. o Client usa uma WebView2 transitória/dedicada, sem navegar a webview principal para o PDF;
5. a WebView2 recebe recurso PDF local controlado, sem Internet/path de usuário;
6. mecanismo baseline usa WebView2 `ShowPrintUI(System)` por adaptador Windows isolado sob Tauri `with_webview`;
7. diálogo padrão é o **diálogo de impressão do Windows**; sem impressão silenciosa inicial e sem seletor próprio de impressoras;
8. StepFlow não enumera/persiste impressoras no Host e não gerencia drivers/spooler corporativo;
9. `ShellExecute`/handler PDF externo, Word/COM, LibreOffice, browser externo e spool PDF bruto não são baseline;
10. recurso local de impressão é transitório; mecanismo/nome/path concreto fica para Etapa 10;
11. `ShowPrintUI` não autoriza declarar impressão física concluída; fluxo entregue ao Windows é sucesso da integração, não confirmação de papel impresso;
12. cancelar/fechar o diálogo não é erro funcional e não gera auditoria `printed=true`;
13. falhas de geração, preparação local, compatibilidade WebView2 e abertura do diálogo são distintas;
14. a implementação evita duplicidade concorrente local sem criar fila/job persistente;
15. gate técnico valida Windows 10/11 x64, WebView2, PDF multipágina, impressoras reais/de rede e operação offline;
16. versão mínima concreta de WebView2 e detalhes do recurso temporário ficam para validação/Etapas 10 e 12;
17. layout físico do Procedimento permanece integralmente na Etapa 5.

## 21. Referências técnicas consultadas

- Microsoft Learn — Printing from WebView2 apps: `https://learn.microsoft.com/en-us/microsoft-edge/webview2/how-to/print`
- Microsoft Learn — Using local content in WebView2 apps: `https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/working-with-local-content`
- Microsoft Learn — `CoreWebView2.ShowPrintUI` / `CoreWebView2.PrintAsync` / `CoreWebView2PrintStatus`
- Microsoft Learn — Shell printing / `ShellExecuteEx` print verb
- Tauri 2 — `Webview::with_webview` e acesso platform-specific ao WebView2
- Tauri 2 — compatibilidade/runtime WebView2 no Windows

## 22. Critério de fechamento

Esta proposta só deve ser promovida para `bloco-10-exportacao-impressao-ficha.md`, `AGENTS.md`, README, arquitetura, registro, índice e plano oficial após **aprovação explícita do PO**.

Até lá:

- Etapas 1–3 permanecem consolidadas;
- Etapa 4 permanece em análise apenas nesta branch/PR;
- Etapa 5 continua pendente e não deve ser aberta;
- fontes canônicas não mudam de estado;
- nenhum código funcional, dependency, scaffold ou migration é autorizado por este documento.
