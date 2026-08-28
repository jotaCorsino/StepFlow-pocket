# Bloco 10 — Etapa 6 — PDF + preview da Ficha compacta — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-28  
**Base consolidada:** Bloco 10 / Etapas 1–5  
**Base Git:** `main` em `80f9b53c7bb3610c60ae73c6599c57af2eb6951c`

## 1. Objetivo

Fechar somente o contrato técnico da **Ficha compacta de Atendimento em PDF** e sua **pré-visualização no Client**, preservando:

- a arquitetura documental já consolidada;
- o limite rígido de uma única A4;
- o estado confirmado do Atendimento;
- a experiência visual de baixa densidade do Pocket;
- o mecanismo Windows de impressão já definido na Etapa 4;
- a separação entre preview, artefato PDF e UI da Tela 09.

Esta etapa não define o template físico final da Ficha, limites numéricos de textos, tratamento final de muitos MACs/Procedimentos, nomes de arquivo, temporários concretos ou QR/barcode.

## 2. Contratos herdados

Permanecem vigentes:

- `Ficha / Imprimir` parte do Atendimento/Tela 09;
- ficha usa somente estado confirmado pelo Host;
- alterações locais não salvas ou conflito pendente bloqueiam a geração;
- `Em andamento` pode gerar ficha de acompanhamento;
- `Concluído` pode reimprimir o estado histórico aplicável;
- `Cancelado` deve aparecer inequivocamente como cancelado;
- capacidade padrão de gerar/reimprimir ficha existe para ADM/Gerência/Funcionário em Atendimento acessível;
- Atendimento sem Equipamento também pode gerar ficha;
- campos vazios/não aplicáveis são omitidos;
- ficha lista revisões efetivamente utilizadas dos Procedimentos;
- ficha nunca ultrapassa uma página A4 como comportamento normal;
- conteúdo excessivo não cria segunda página nem sofre truncamento silencioso;
- DOCX específico da ficha não é requisito inicial;
- geração documental pertence ao Host;
- Client não monta documento a partir de HTML/DOM;
- artefatos gerados não viram histórico/backup automaticamente;
- renderização usa limite próprio bounded de concorrência, fora da fila de mutações.

## 3. Decisão proposta — a Ficha possui PDF próprio

A primeira versão terá **PDF específico da Ficha compacta**.

Esse PDF é um artefato documental próprio, diferente do PDF de Procedimento.

Fluxo conceitual:

```text
Atendimento confirmado
→ Host captura snapshot/revisão esperada
→ DocumentModel com document_kind = service_sheet
→ template Typst interno da Ficha
→ uma página diagramada
→ PDF canônico da Ficha
```

Consequências:

- ficha não é screenshot da Tela 09;
- ficha não reutiliza o template de Procedimento;
- ficha não depende de HTML/CSS da aplicação;
- `Salvar PDF`, preview e impressão compartilham o mesmo conteúdo/layout físico;
- nenhum segundo renderer documental é criado somente para impressão.

## 4. Fonte e consistência

A geração recebe identidade estável do Atendimento + revisão/versão esperada do estado confirmado.

### Em andamento

```text
Client mostra Atendimento na revisão 42
→ solicita Ficha para revisão esperada 42
→ Host já está na revisão 43
→ não gerar silenciosamente a 43
→ informar estado obsoleto
→ Client reconsulta antes de nova tentativa
```

### Concluído

Usa o estado histórico congelado aplicável, inclusive snapshots/projeções históricas já consolidadas.

### Cancelado

Usa o estado oficial aplicável e inclui identificação inequívoca do cancelamento.

A identidade corporativa segue a regra vigente: usa a configuração central válida no momento da geração; versionamento histórico da identidade da empresa não é requisito v1.

## 5. DocumentModel da Ficha

Não é criada uma segunda arquitetura documental.

A Ficha usa o `DocumentModel` consolidado na Etapa 1 com `document_kind` específico e conteúdo estruturado apropriado ao Atendimento.

Conceitualmente:

```text
DocumentModel
├── document_kind = service_sheet
├── source_identity = Atendimento
├── source_version
├── company_identity
├── metadata
│   ├── atendimento
│   ├── lifecycle/status
│   ├── responsável
│   ├── referência externa
│   └── data aplicável
├── sections
│   ├── equipamento? 
│   ├── procedimentos utilizados
│   ├── resumo do trabalho
│   └── observações aplicáveis
└── generation_metadata
```

A forma exata das structs fica para implementação; esta etapa consolida apenas a semântica.

## 6. Renderer PDF da Ficha

A Ficha reutiliza a infraestrutura **Typst embutida no Host** consolidada para PDF, mas com template interno próprio.

Não criar:

- biblioteca PDF diferente apenas para Ficha;
- processo conversor externo;
- browser/headless para gerar PDF;
- HTML → PDF;
- canvas/screenshot → PDF.

Baseline técnico herdado:

- PDF 1.7;
- Tagged PDF habilitado como baseline estrutural;
- texto real selecionável/pesquisável/copiável;
- fontes necessárias incorporadas/subsetadas;
- assets controlados;
- nenhum recurso remoto em runtime;
- template confiável/versionado;
- domínio entra apenas como dados estruturados;
- falha não produz PDF parcial tratado como sucesso.

Tagged PDF continua sem promessa formal PDF/UA ou PDF/A.

A família tipográfica e demais detalhes físicos da Ficha permanecem para a Etapa 7.

## 7. Gate estrutural de uma única página

A regra de uma A4 é validada **depois do layout**, não por estimativa do Client.

Fluxo:

```text
DocumentModel da Ficha
→ Typst compila/layout
→ PagedDocument
→ validar quantidade de páginas

1 página
→ válido

0 páginas
→ erro de geração

2+ páginas
→ SHEET_OVERFLOW
→ nenhum PDF/preview é confirmado
```

Regras:

- não retornar apenas a primeira página;
- não cortar conteúdo excedente;
- não reduzir fonte dinamicamente para forçar encaixe;
- não criar segunda folha silenciosamente;
- não omitir seção conhecida para fazer caber.

A Etapa 7 define o template A4 final. A Etapa 8 define limites/priorização textual e diagnósticos úteis de excesso.

## 8. Preview — princípio central

A prévia deve representar **o mesmo layout que originará o PDF**, sem reconstruir a Ficha em HTML.

Direção proposta:

```text
mesmo DocumentModel
→ mesmo template Typst
→ mesmo PagedDocument de 1 página
   ├─→ typst-pdf → PDF canônico
   └─→ typst-svg → SVG de preview
```

O preview SVG é derivado da **mesma página já diagramada** que alimenta o PDF.

Isso evita:

- um segundo template de preview;
- divergência HTML × PDF;
- screenshot da Tela 09;
- dependência do toolbar do visualizador PDF;
- necessidade de abrir software externo apenas para conferir a Ficha.

## 9. Por que SVG para o preview

Typst possui renderer oficial de página para SVG. O SVG mantém a geometria vetorial da página diagramada e é apropriado para exibição no Client.

Para a Ficha, que obrigatoriamente possui uma única página, isso permite:

- página A4 escalada responsivamente;
- preview nítido em diferentes tamanhos de janela;
- ausência de toolbar/browser chrome;
- controles de salvar/imprimir pertencendo ao próprio Pocket;
- manutenção da Tela 09 e do Shell intactos;
- nenhum arquivo PDF temporário necessário apenas para visualizar.

O SVG de preview não é o artefato oficial exportável. O **PDF continua sendo o documento canônico**.

## 10. Segurança do preview

O SVG é produzido pelo Host a partir de template e dados controlados.

No Client, a representação deve ser tratada como **imagem/documento visual**, não como HTML arbitrário a ser injetado na aplicação.

Direção:

- usar recurso/Blob controlado como imagem;
- não executar script do SVG;
- não habilitar navegação externa automática;
- não receber SVG fornecido diretamente pelo usuário;
- nenhum URL/path remoto originado do conteúdo;
- links textuais permanecem texto, salvo futura semântica explícita.

A forma concreta de transporte/Blob URL permanece detalhe de implementação.

## 11. Superfície de preview no Client

A Ficha não ganha item permanente na sidebar nem uma nova área pesada da aplicação.

Direção visual:

```text
Tela 09 — Atendimento
→ Ficha / Imprimir
→ gera artefato
→ abre preview sobre o contexto atual
```

Superfície proposta:

- modal/overlay grande dentro da própria experiência do Client;
- página A4 centralizada e escalada para a área disponível;
- fundo neutro apenas para separar a folha do restante da UI;
- sem painel lateral permanente;
- sem metadados repetidos ao redor da página;
- sem toolbar textual extensa.

Cabeçalho do preview pode ser mínimo:

```text
Ficha AT-000142                      [ salvar ] [ imprimir ] [ × ]
```

Os ícones podem ser icon-only quando inequívocos, sempre com nome acessível/tooltip.

## 12. Ações do preview

### Salvar PDF

`Salvar PDF`/ícone equivalente salva **os mesmos bytes PDF já produzidos para aquela prévia**.

- não regenerar silenciosamente;
- destino é escolhido localmente pelo usuário;
- Host não recebe path arbitrário da workstation;
- cancelamento do diálogo é voluntário, não erro;
- naming final permanece Etapa 10.

### Imprimir

`Imprimir` usa **os mesmos bytes PDF da prévia**.

O Client reaproveita o mecanismo Windows já consolidado:

```text
PDF canônico da Ficha
→ recurso local transitório controlado
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo de impressão do Windows
```

Não existe um renderer ou layout separado de impressão da Ficha.

Os detalhes concretos do recurso temporário continuam na Etapa 10.

### Fechar

Fechar a prévia descarta o estado transitório local daquela visualização. Não altera o Atendimento.

## 13. Uma geração, dois derivados transitórios

A operação conceitual produz um par coerente:

```text
ServiceSheetGenerationResult
├── source_identity
├── source_version
├── pdf_bytes
└── preview_svg
```

A forma exata do transporte HTTP não é congelada nesta etapa.

Requisitos:

- PDF e SVG derivam do mesmo `PagedDocument`;
- o par pertence à mesma revisão esperada;
- não existe job persistente;
- não existe URL pública permanente;
- artefatos não são gravados automaticamente no Host;
- Client pode manter o resultado somente enquanto a prévia estiver aberta/necessária.

A geração do par conta como uma única operação documental para fins de limite de concorrência.

## 14. Preview estável e mudanças em tempo real

Uma prévia aberta não muda silenciosamente quando chega evento WebSocket.

Ela permanece ligada a:

```text
Atendimento X
source_version Y
```

Se o Client confirmar que o Atendimento mudou depois da geração:

- a prévia atual permanece visualmente estável;
- deve ser marcada discretamente como desatualizada;
- salvar/imprimir como estado atual não pode acontecer silenciosamente;
- o usuário deve regenerar a prévia antes de produzir nova saída operacional atual.

Direção de UX curta:

```text
Atendimento atualizado. Gere novamente a ficha.
[ Atualizar ]
```

Não regenerar automaticamente em background e não trocar a folha enquanto o usuário está conferindo.

## 15. Estados de geração

Estados mínimos:

```text
preparando ficha
→ preparando prévia
→ pronta
```

Falhas distintas:

- alterações locais não salvas;
- conflito pendente;
- sem autenticação/permissão;
- Atendimento indisponível;
- revisão esperada obsoleta;
- `SHEET_OVERFLOW`;
- falha do renderer/template/asset;
- falha na geração do preview;
- limite de recursos/backpressure;
- conexão interrompida.

Não exibir percentual sem progresso real.

Nenhum PDF/preview parcial é apresentado como sucesso.

## 16. Overflow e mensagem ao usuário

Até a Etapa 8 fechar diagnósticos específicos, a mensagem funcional consolidada continua válida:

`A ficha possui conteúdo demais para uma página A4. Revise os campos indicados antes de imprimir.`

Nesta Etapa 6 fica consolidado apenas que a detecção real é feita pelo Host após layout e que 2+ páginas resultam em falha explícita.

A identificação exata dos campos responsáveis pelo excesso permanece Etapa 8.

## 17. Acessibilidade

- ações do preview operam por teclado;
- ícones têm nomes acessíveis;
- foco fica contido/gerenciado enquanto modal estiver aberto;
- a prévia visual possui identificação acessível como `Pré-visualização da ficha AT-...`;
- cor não é o único meio para indicar status/cancelamento/desatualização;
- PDF final mantém o baseline Tagged PDF;
- o SVG é representação visual da página, não substituto semântico do Atendimento na UI.

Não se cria promessa formal PDF/UA nesta etapa.

## 18. Dimensões e responsividade do preview

O documento continua fisicamente A4 independentemente da janela.

Na tela:

- a folha é escalada proporcionalmente;
- não deformar a razão A4;
- não alterar o layout físico porque a janela é menor;
- controles permanecem acessíveis;
- a página pode usar rolagem da superfície de preview se necessário;
- zoom fino pode ser adicionado na implementação se testes demonstrarem necessidade, sem virar requisito funcional desta etapa.

## 19. Concorrência e recursos

Aplicam-se os limites documentais já consolidados:

- geração fora da fila de mutações;
- bounded concurrency/backpressure;
- não criar fila persistente;
- não renderizar novamente ao clicar `Salvar` ou `Imprimir` enquanto a prévia válida já contém o PDF correspondente;
- limites numéricos de bytes/memória/tempo ficam para implementação/Etapa 12.

## 20. Alternativas avaliadas

### Screenshot/HTML da Tela 09

Rejeitado. Criaria um segundo contrato visual e misturaria UI com documento.

### HTML dedicado → imprimir/salvar PDF pelo browser

Rejeitado como baseline. Duplicaria o template e introduziria divergência entre HTML e PDF oficial.

### WebView2 PDF viewer como preview principal

É tecnicamente capaz de abrir PDF local e permanece útil para o fluxo de impressão. Não é a direção principal do preview porque:

- introduz toolbar/chrome próprio do viewer;
- reduz controle visual do Pocket;
- acopla a experiência de preview a detalhes do viewer do runtime;
- exige recurso PDF local/materialização apenas para visualizar.

### PDF + SVG derivados do mesmo PagedDocument

Direção proposta porque preserva uma única composição documental, mantém PDF como artefato oficial e permite preview vetorial leve dentro da própria UI.

## 21. Referências técnicas verificadas

Verificação realizada em 2026-08-28:

- Typst PDF: `https://typst.app/docs/reference/pdf/`;
- Typst SVG: `https://typst.app/docs/reference/svg/`;
- `typst-pdf`: `https://docs.rs/typst-pdf/`;
- `typst-svg`: `https://docs.rs/typst-svg/`;
- WebView2 — local content/PDF: `https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/working-with-local-content`;
- WebView2 — APIs/PDF toolbar: `https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/overview-features-apis`;
- Tauri 2 WebviewWindow/Webview APIs: `https://v2.tauri.app/reference/javascript/api/namespacewebviewwindow/`.

As versões atuais observadas em documentação não são congeladas como contrato da Fase 1. Versões exatas continuam para Cargo.lock/gate técnico.

## 22. Fora do escopo da Etapa 6

- margens finais da Ficha;
- tipografia física final da Ficha;
- hierarquia/composição detalhada das seções da única A4;
- limites numéricos de resumo/observações;
- regras finais de priorização/truncamento controlado;
- tratamento físico de muitos MACs;
- tratamento físico de muitos Procedimentos utilizados;
- nomes finais de arquivo;
- diretórios/temporários e política concreta de limpeza;
- QR/barcode;
- DOCX da Ficha;
- envio por e-mail/cloud;
- histórico persistente de exportações;
- implementação funcional.

Esses pontos permanecem nas Etapas 7–12 correspondentes.

## 23. Decisões propostas para aprovação do PO

1. a Ficha compacta terá **PDF próprio e canônico**;
2. o PDF será gerado pelo Host a partir do estado confirmado/revisão esperada do Atendimento;
3. a Ficha usa o mesmo `DocumentModel` arquitetural, com `document_kind = service_sheet` ou equivalente;
4. o PDF da Ficha reutiliza a infraestrutura Typst embutida, com template interno específico da Ficha;
5. mantém PDF 1.7 + Tagged PDF + texto real + fontes incorporadas como baseline técnico;
6. layout é validado pelo Host e precisa resultar em **exatamente uma página**;
7. 2+ páginas geram erro explícito `SHEET_OVERFLOW`/equivalente, sem segunda folha, truncamento ou redução automática de fonte;
8. preview não é HTML reconstruído nem screenshot;
9. o preview é SVG produzido da mesma página do `PagedDocument` que origina o PDF;
10. preview aparece em modal/overlay grande dentro do Client, sem nova sidebar nem toolbar de navegador;
11. controles permanentes do preview são mínimos: salvar PDF, imprimir e fechar, podendo usar ícones acessíveis;
12. salvar e imprimir reutilizam **os mesmos bytes PDF** apresentados por aquela geração, sem regenerar silenciosamente;
13. impressão reutiliza o mecanismo local WebView2 + `ShowPrintUI(System)` já consolidado;
14. PDF + SVG formam um resultado transitório coerente da mesma `source_version`, sem job/persistência no Host;
15. evento remoto não troca a prévia aberta silenciosamente;
16. se a fonte mudar, a prévia fica marcada como desatualizada e exige regeneração antes de nova saída atual;
17. preview SVG é tratado como recurso visual controlado, sem execução de script/navegação externa;
18. template físico da Ficha permanece Etapa 7; limites textuais permanecem Etapa 8; muitos MACs/Procedimentos Etapa 9; nomes/temporários Etapa 10; QR Etapa 11; matriz/limites técnicos Etapa 12.

## 24. Gate de fechamento

A Etapa 6 só será consolidada após aprovação explícita do PO.

Até lá:

- este arquivo é proposta;
- fontes canônicas não mudam para `Etapa 6 consolidada`;
- Etapa 7 não é aberta;
- nenhum código/dependency/migration/scaffold é criado.
