# Bloco 10 — Etapa 5 — Template físico de Procedimentos — Proposta para análise

**Status:** PROPOSTA COMPLETA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-27  
**Base consolidada:** Bloco 10 / Etapas 1–4

## 1. Correção de escopo

A Etapa 5 trata somente do **template físico do Procedimento exportado em PDF/DOCX**.

Ela não redesenha o Reader diário do app e não deve herdar decisões da Ficha compacta de Atendimento.

O StepFlow possui três superfícies diferentes:

```text
Reader do app
→ uso diário
→ páginas lógicas do manual
→ sem geometria A4

Procedimento exportado
→ PDF / DOCX / impressão
→ documento completo da revisão selecionada
→ pode ocupar várias páginas físicas

Ficha compacta de Atendimento
→ resumo do que foi realizado
→ exportável / imprimível
→ máximo de uma página A4
```

Essa separação é obrigatória para as decisões seguintes.

## 2. Reader diário — contrato preservado

O modelo aprovado da Tela 05 continua sendo a referência funcional e visual.

Estrutura conceitual:

```text
← Processos

PR-014  Configuração de VLAN                        [⋯]
Redes · Infraestrutura · TI · Versão 2.0

Etapa 3 de 7                                [ Sumário ▾ ]

●━━━━●━━━━◉────○────○────○────○

3. Configurar a VLAN no switch

1. Acesse o equipamento...
2. Entre no modo de configuração...

┌ Observação ──────────────────────────────────────┐
│ ...                                              │
└──────────────────────────────────────────────────┘

┌ Comando ─────────────────────────────────── [⧉] ┐
│ configure terminal                              │
└──────────────────────────────────────────────────┘

                    [ Anterior ] [ Próxima ]
```

Regras preservadas:

- Shell/sidebar global permanece visível;
- não existe segunda sidebar permanente;
- cabeçalho do Procedimento é compacto;
- o conteúdo técnico domina a área central;
- `Visão geral` é a primeira página lógica da navegação;
- cada `process_stage` corresponde a uma página lógica própria;
- duas Etapas não são fundidas para reduzir cliques;
- ao trocar de página, o conteúdo inicia no topo;
- `Anterior`, `Próxima`, Sumário e stepper navegam pelo mesmo conjunto de páginas;
- comandos/código mantêm ação de copiar icon-only;
- revisão aberta permanece estável.

Nada na Etapa 5 pode alterar silenciosamente esse contrato.

## 3. Visão geral

`Visão geral` integra a mesma navegação do Reader, antes da Etapa 1:

```text
Sumário
├─ Visão geral
├─ Etapa 1 — Preparação
├─ Etapa 2 — Configuração
├─ Etapa 3 — Validação
└─ ...
```

Ela é uma página lógica não numerada como Etapa.

Pode apresentar, conforme disponibilidade no domínio:

- Objetivo;
- Pré-requisitos;
- Observações gerais;
- Área/Departamento;
- Responsável;
- Versão/revisão e contexto editorial essencial.

Não cria nova entidade persistente.

## 4. Stepper compacto e navegável

A barra superior do Reader não é apenas um indicador passivo de posição. Ela funciona como **stepper horizontal navegável entre as Etapas**.

Direção aprovada pelo PO:

```text
●━━━━●━━━━◉────○────○────○────○
```

O componente usa prioritariamente:

- círculos;
- linhas;
- preenchimento;
- cor;
- contraste/forma;
- interação de clique/teclado.

Ele não precisa repetir nomes de Etapas nem rótulos longos.

O texto auxiliar pode permanecer compacto, por exemplo `Etapa 3 de 7`.

Estados visuais:

- Etapas anteriores à atual → estado visual de percorridas/concluídas na navegação;
- Etapa atual → destaque inequívoco;
- Etapas seguintes → estado neutro/futuro.

Cada marcador é acionável e abre diretamente a Etapa correspondente no topo.

### 4.1 Semântica de "concluída" no stepper

O estado visual de Etapa anterior no stepper representa **posição/percurso de navegação**.

Ele não significa:

- conclusão operacional do Atendimento;
- checklist operacional concluído;
- persistência de `completed=true` no domínio;
- confirmação de que o técnico executou aquela Etapa.

Stepper do Reader e progresso operacional são conceitos distintos.

## 5. Princípio transversal de baixa densidade visual

O Pocket deve buscar **clareza com a menor densidade de informação permanente necessária**.

Direção geral para grande parte do app:

```text
mostrar sempre
→ somente o necessário para entender e agir agora

mostrar visualmente
→ posição, progresso, estado e ações recorrentes

mostrar sob demanda
→ contexto secundário e detalhes

usar texto
→ quando forma, cor, símbolo ou posição não forem suficientes
```

Aplicação prática:

- preferir cor + forma + símbolo para estados simples;
- preferir ícones reconhecíveis para ações recorrentes como copiar, editar, excluir, voltar, pesquisar e expandir;
- usar tooltip/popover quando o significado precisar de apoio textual;
- evitar repetir em texto algo que a posição ou o componente já comunica claramente;
- manter metadados secundários em segundo plano ou sob demanda;
- evitar chips, badges, labels e caixas quando não acrescentarem leitura útil;
- preservar espaço para o conteúdo de trabalho principal.

### 5.1 Limites do princípio

Baixa densidade não significa esconder informação necessária.

Quando retirar texto criar ambiguidade, o texto permanece.

Cor não deve ser o único meio de comunicar um estado importante. Forma, símbolo, contraste, posição ou texto acessível devem complementar a leitura.

Esse princípio é transversal e deverá orientar revisões futuras das telas já documentadas, sem reabrir contratos funcionais consolidados sem necessidade.

## 6. Reader não possui geometria A4

A página lógica do Reader é uma unidade de navegação da aplicação.

Portanto, não se aplicam ao Reader:

- dimensões físicas em milímetros;
- margens de impressão;
- header/footer físico;
- `Página X de N` de documento;
- área imprimível de driver;
- fontes específicas do PDF/DOCX;
- quebras físicas de folha.

Se uma Etapa tiver conteúdo maior do que a área visível, o conteúdo não é truncado e a próxima Etapa não é fundida na mesma página lógica.

O comportamento concreto de overflow/rolagem pertence à implementação de UI e deve preservar o modelo de uma Etapa por página lógica.

## 7. Procedimento exportado — contrato funcional

O fluxo documental permanece separado do Reader:

```text
revisão selecionada
→ Exportar / Imprimir
→ PDF | DOCX | Imprimir
→ documento completo daquela revisão
```

O Procedimento exportado:

- usa a revisão exata selecionada/autorizada;
- é documento próprio, não screenshot do Reader;
- contém o Procedimento completo;
- pode ocupar várias páginas físicas;
- preserva todos os blocos semânticos;
- não inclui sidebar, botões, stepper, Sumário interativo, ícone de copiar ou outros controles do Client.

A impressão Windows continua usando o PDF oficial consolidado na Etapa 4.

## 8. Ficha compacta — contrato separado

O limite rígido de uma A4 pertence à **Ficha compacta de Atendimento**:

```text
Atendimento
→ Ficha / Imprimir
→ resumo operacional do que foi realizado
→ máximo 1 página A4
```

O detalhamento permanece nas etapas próprias:

- Etapa 6 — PDF + preview da Ficha compacta;
- Etapa 7 — Template físico A4 da Ficha;
- Etapa 8 — Limites textuais e densidade da Ficha;
- Etapa 9 — Múltiplos MACs / Procedimentos na Ficha.

A Etapa 5 não antecipa essas decisões.

## 9. Baseline físico proposto para o Procedimento completo

### 9.1 Papel e orientação

Proposta:

```text
papel: A4
orientação: retrato
quantidade de páginas: conforme necessário
```

A4 aqui é **formato do documento físico exportado**, não geometria do Reader.

O limite de `1 A4` continua exclusivo da Ficha compacta.

PDF e DOCX usam o mesmo tamanho físico base para manter impressão previsível.

### 9.2 Margens

Proposta simples:

```text
18 mm em todos os lados
```

Um único valor reduz regras especiais, mantém área útil ampla e ainda reserva espaço seguro para impressão corporativa comum.

Header/footer, quando existirem, ocupam essa própria zona de margem e não criam uma segunda moldura visual.

### 9.3 Primeira página

Não há capa exclusiva.

A página começa diretamente com informação útil:

```text
[logo] Empresa

PR-014
Configuração de VLAN
versão/revisão · estado editorial

Área/Departamento · Responsável

VISÃO GERAL
Objetivo...
Pré-requisitos...
Observações...

01 · Preparação
...
```

Direção:

- identidade compacta;
- código + título com hierarquia principal;
- somente metadados necessários para identificar o documento;
- conteúdo começa na mesma página;
- campos vazios não reservam espaço;
- sem tabela pesada apenas para metadados.

### 9.4 Sumário documental

Proposta v1: **não mostrar sumário físico por padrão**.

Razões:

- reduz densidade e páginas consumidas;
- evita duplicação dos títulos de Etapas;
- o documento já possui hierarquia sequencial clara;
- evita depender de paginação estável do DOCX para manter referências numéricas coerentes.

Um sumário documental pode ser reintroduzido futuramente se procedimentos muito extensos demonstrarem necessidade real.

### 9.5 Etapas

O título físico evita repetir texto desnecessário:

```text
01 · Preparação
02 · Configuração
03 · Validação
```

Não é necessário escrever `ETAPA` antes de todos os títulos quando a hierarquia visual já deixa a estrutura clara.

Regras:

- uma Etapa não força automaticamente nova folha;
- o título permanece junto do primeiro bloco sempre que possível;
- se não houver espaço útil suficiente para título + início de conteúdo, ambos seguem para a próxima página;
- etapas curtas podem compartilhar a mesma folha física;
- a separação permanece clara por espaço, hierarquia e linha/forma discreta, não por cards pesados.

### 9.6 Cabeçalho e rodapé

Proposta: **sem cabeçalho repetitivo** nas páginas internas.

Isso libera espaço e reduz poluição.

Rodapé compacto:

```text
PR-014 · r18                              3 / 8
```

ou equivalente.

Regras:

- código/revisão identificam folhas separadas do conjunto;
- paginação usa formato curto;
- informações essenciais também aparecem no corpo da primeira página, nunca apenas no rodapé;
- sem timestamp, hostname, usuário, caminho local ou dados técnicos internos visíveis.

### 9.7 Blocos semânticos

A representação física segue o princípio de baixa densidade:

- parágrafo → texto normal, sem card;
- passos → numeração e indentação;
- checklist → `□` + texto;
- nota → forma/símbolo discreto + conteúdo;
- alerta → símbolo + contraste mais forte; texto `Atenção` quando necessário para não depender apenas de cor;
- comando → bloco monoespaçado compacto;
- código → bloco monoespaçado compacto;
- imagens → proporção preservada, sem crop automático.

Não envolver cada passo em card/borda.

Comandos/código permanecem texto real, selecionável e pesquisável.

### 9.8 Quebras físicas

Regras propostas:

- paginação automática é o padrão;
- evitar widow/orphan em texto normal;
- título de Etapa não fica isolado no fim da página;
- subtítulo/label permanece com o primeiro conteúdo relacionado;
- pequenos blocos de nota/alerta/comando/checklist ficam inteiros quando razoável;
- blocos excepcionalmente longos podem quebrar entre páginas em vez de truncar ou reduzir fonte;
- nenhuma Etapa é silenciosamente omitida para fazer o documento caber.

### 9.9 PDF × DOCX

Os dois formatos compartilham:

- A4 retrato;
- margens-base;
- hierarquia;
- ordem do conteúdo;
- identidade;
- representação semântica dos blocos.

Não compartilham uma promessa de quebra de página idêntica.

O PDF continua sendo a referência física de impressão; o DOCX continua sendo editável e refluível.

## 10. Tipografia proposta

### 10.1 PDF

Famílias:

```text
texto:          Noto Sans
comando/código: Noto Sans Mono
```

Regras:

- fontes empacotadas com o Host;
- carregadas apenas pelo `World` controlado do renderer Typst;
- incorporadas/subsetadas no PDF;
- não dependem de fontes instaladas no Windows do Host;
- arquivos de fonte e licença entram no pacote/runtime conforme requisitos de distribuição da OFL 1.1.

Essa escolha preserva o caráter autocontido da Etapa 2 e evita dependência de fontes proprietárias no renderer PDF.

### 10.2 DOCX

Famílias declaradas:

```text
texto:          Arial
comando/código: Consolas
```

Regras:

- não incorporar fontes no DOCX v1;
- não empacotar/redistribuir arquivos Arial/Consolas com StepFlow;
- o DOCX apenas referencia essas famílias;
- a matriz de compatibilidade deve validar Word/Office nos Windows corporativos suportados;
- se a família não estiver disponível no consumidor, o Word pode aplicar substituição conforme seu ambiente; isso não altera o conteúdo semântico.

Arial e Consolas são escolhas deliberadas por compatibilidade ampla no ambiente Windows/Office, não por equivalência métrica com Noto.

### 10.3 Escala tipográfica

Baseline compacto:

| Uso | Tamanho |
|---|---:|
| Título do Procedimento | 18 pt |
| Código / linha editorial | 9 pt |
| Título de Etapa | 14 pt |
| Subtítulo estrutural | 11 pt |
| Corpo | 10.5 pt |
| Metadados secundários | 9 pt |
| Nota / alerta | 10 pt |
| Comando / código | 9 pt |
| Rodapé | 8 pt |

Pesos:

- regular para corpo;
- semibold/bold apenas em títulos, números estruturais e labels realmente necessários;
- evitar negrito em excesso.

### 10.4 Espaçamento

Direção:

```text
corpo:          entrelinha ~1.25
código/comando: entrelinha ~1.15
```

Regras:

- sem recuo de primeira linha;
- texto normal alinhado à esquerda;
- espaçamento entre blocos substitui bordas/cards desnecessários;
- títulos usam espaço suficiente para criar hierarquia sem grandes áreas vazias;
- não reduzir tamanho de fonte dinamicamente para fazer conteúdo caber;
- não usar fonte condensada como workaround de densidade.

### 10.5 Coerência visual

PDF e DOCX não precisam usar a mesma família para serem percebidos como o mesmo documento.

A coerência vem de:

- escala tipográfica equivalente;
- mesma ordem e hierarquia;
- espaçamento semelhante;
- mesmos símbolos e estrutura dos blocos;
- mesma identidade institucional;
- mesma lógica de numeração de Etapas.

## 11. Validação técnica

O baseline é compatível com os renderers já escolhidos:

- Typst suporta tamanho de papel, margens, header/footer, numeração e paginação automática;
- Typst possui tratamento de widow/orphan e page breaks;
- `docx-rs` atual expõe page size, page margins, headers/footers e controles como `keep_next`, `keep_lines`, `page_break_before` e `widow_control`.

Portanto, nenhuma decisão visual acima exige um terceiro renderer nem dependência externa de Office/browser/conversor.

## 12. Decisões propostas para consolidação

Se aprovadas pelo PO, a Etapa 5 consolida:

1. Reader diário e documento físico são superfícies independentes;
2. princípio transversal de baixa densidade visual no Pocket;
3. stepper compacto do Reader com círculos/linhas/estados e navegação direta;
4. `Visão geral` como primeira página lógica do Reader;
5. Procedimento físico A4 retrato e multipágina;
6. margens de 18 mm;
7. sem capa exclusiva;
8. sem sumário físico por padrão na v1;
9. Etapas físicas em fluxo normal, sem nova folha obrigatória;
10. título físico de Etapa enxuto (`01 · Título`);
11. sem header repetitivo;
12. rodapé curto com identidade técnica e paginação;
13. blocos semânticos visualmente leves;
14. paginação automática com proteção contra títulos/linhas órfãs;
15. PDF com Noto Sans / Noto Sans Mono empacotadas e incorporadas;
16. DOCX referenciando Arial / Consolas sem embedding v1;
17. escala tipográfica compacta definida nesta proposta;
18. PDF como referência física de impressão e DOCX refluível sem promessa de paginação idêntica.

## 13. Critério de fechamento

Esta proposta está **completa para análise do PO**, mas ainda não está consolidada.

Até aprovação explícita:

- Etapas 1–4 continuam consolidadas;
- Etapa 5 não é promovida às fontes canônicas;
- PR permanece draft;
- Etapa 6 continua pendente/não aberta;
- nenhuma alteração funcional de UI é consolidada;
- nenhum código, dependency, fonte binária, scaffold ou migration é autorizado.
