# Bloco 10 — Etapa 5 — Template físico de Procedimentos — Proposta corrigida para análise

**Status:** EM ANÁLISE / ESCOPO CORRIGIDO APÓS REVISÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-27  
**Base consolidada:** Bloco 10 / Etapas 1–4

## 1. Correção de escopo

A primeira versão desta proposta misturou indevidamente três superfícies distintas do produto. Essa mistura está corrigida neste documento.

O StepFlow possui contratos separados:

```text
A) Reader do app — uso diário
   → navegação lógica em formato de manual/livro
   → NÃO é uma folha A4
   → NÃO é um preview de PDF
   → NÃO herda margens, header/footer ou paginação física

B) Procedimento exportado
   → PDF / DOCX / impressão
   → documento físico derivado da revisão selecionada
   → pode ocupar várias páginas

C) Ficha compacta de Atendimento
   → resumo operacional do que foi feito
   → exportável/imprimível
   → máximo de uma página A4
```

O limite rígido de **uma A4** pertence à **Ficha compacta de Atendimento**, não ao Reader e não ao Procedimento completo.

Esta Etapa 5 trata somente do **template físico do Procedimento exportado**. Ela não redesenha o Reader.

## 2. Fonte de verdade do Reader já aprovada

A experiência diária já foi consolidada no Bloco 8 / Tela 05 e permanece vigente.

O modelo aprovado é conceitualmente:

```text
SHELL / SIDEBAR GLOBAL

← Processos

PR-014  Configuração de VLAN                              [⋯]
Redes  Infraestrutura      TI       Versão 2.0

Etapa 3 de 7                                      [ Sumário ▾ ]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌──────────────── PÁGINA LÓGICA ATUAL ──────────────────────┐
│                                                            │
│ 3. Configurar a VLAN no switch                             │
│ Breve introdução da etapa...                               │
│                                                            │
│ 1. Acesse o equipamento...                                 │
│ 2. Entre no modo de configuração...                        │
│                                                            │
│ ┌ Observação ────────────────────────────────────────────┐ │
│ │ ...                                                    │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
│ ┌ Comando ────────────────────────────────────────── [⧉]┐ │
│ │ configure terminal                                    │ │
│ └────────────────────────────────────────────────────────┘ │
│                                                            │
└────────────────────────────────────────────────────────────┘

                         [ ← Etapa anterior ] [ Próxima etapa → ]
```

Regras preservadas:

- Shell/sidebar global permanece visível;
- não existe segunda sidebar permanente;
- cabeçalho do Procedimento é compacto e fica fora do corpo da página lógica;
- o conteúdo técnico domina a área central;
- cada `process_stage` corresponde a uma página lógica do manual;
- duas Etapas não são fundidas apenas para reduzir cliques;
- ao trocar de página, o conteúdo inicia no topo;
- `Etapa X de Y` representa posição de navegação, nunca conclusão;
- `Anterior` e `Próxima` navegam entre páginas lógicas;
- `Sumário` é temporário, não permanente;
- comandos/código mantêm botão de copiar icon-only;
- checklist documental e checklist operacional continuam distintos;
- revisão aberta permanece estável.

Nada nesta Etapa 5 pode alterar essas decisões silenciosamente.

## 3. Visão geral dentro do Sumário

A `Visão geral` permanece uma **página lógica não numerada como Etapa**, mas integra a mesma navegação do Reader.

O Sumário começa por ela:

```text
Sumário

• Visão geral
• Etapa 1 — Preparação
• Etapa 2 — Configuração
• Etapa 3 — Validação
...
```

Fluxo lógico:

```text
Visão geral
→ Etapa 1
→ Etapa 2
→ ...
→ Etapa N
```

Isso não cria uma área fixa adicional na tela. `Visão geral` ocupa a área central somente quando é a página selecionada.

Ela pode apresentar os metadados já consolidados, quando existirem:

- Objetivo;
- Pré-requisitos;
- Observações gerais;
- Responsável documental;
- Categorias;
- Versão/revisão e contexto editorial essencial.

`Visão geral` continua sendo apresentação do domínio existente, não uma nova entidade persistente.

## 4. Página lógica do Reader não possui geometria A4

A página do Reader é uma **unidade de navegação da aplicação**, não uma página de impressão.

Portanto, no Reader não se definem por esta Etapa 5:

- 210 × 297 mm;
- margens em milímetros;
- tamanhos tipográficos em pontos de impressão;
- header/footer físico;
- `Página X de N` de documento;
- área imprimível de driver;
- quebras físicas de folha;
- fontes do PDF/DOCX;
- regras de impressão.

A página se adapta à janela desktop suportada, preservando a largura confortável de leitura e a hierarquia já aprovada.

Se o conteúdo de uma Etapa exceder a área visível disponível, ele **não é truncado e não autoriza fundir a próxima Etapa na mesma página lógica**. O comportamento de overflow/rolagem dentro da página deve preservar o modelo de uma Etapa por página lógica e será detalhado na implementação de UI sem introduzir geometria de impressão.

## 5. Blocos da página diária

Dentro da página lógica atual entram somente os conteúdos documentais da `Visão geral` ou da Etapa selecionada, conforme o caso.

Blocos consolidados:

- `paragraph`;
- `numbered_steps`;
- `checklist`;
- `note`;
- `warning`;
- `command`;
- `code`.

O Reader não deve receber por esta Etapa elementos próprios do documento físico, como:

- cabeçalho institucional repetido em cada página;
- rodapé de impressão;
- numeração física `Página X de N`;
- marcas de corte/margens;
- layout de folha;
- composição de capa;
- campos técnicos de geração.

## 6. Contrato funcional da exportação de Procedimento

O fluxo documental permanece separado:

```text
Reader da revisão selecionada
→ Exportar / Imprimir
→ PDF | DOCX | Imprimir
→ documento completo daquela revisão
```

O Procedimento exportado:

- usa a revisão exata selecionada/autorizada;
- é documento próprio, não screenshot do Reader;
- pode ocupar várias páginas físicas;
- contém o Procedimento completo, não somente a página lógica atualmente aberta;
- não inclui sidebar, botões, Sumário interativo, ícone de copiar, barra `Etapa X de Y` ou controles do Client;
- mantém identidade da empresa e contexto editorial necessário;
- preserva todos os blocos semânticos sem descarte silencioso.

A impressão da Etapa 4 continua usando o PDF oficial como artefato canônico.

## 7. A4 e Ficha compacta — distinção obrigatória

O contrato já aprovado para a Ficha compacta é diferente do Procedimento completo:

```text
Atendimento
→ Ficha / Imprimir
→ resumo operacional do estado confirmado
→ no máximo uma página A4
```

A Ficha compacta representa o resumo do Atendimento e do trabalho realizado, com ou sem Equipamento, dentro dos campos já aprovados.

O detalhamento técnico da ficha continua distribuído nas etapas próprias do Bloco 10:

- Etapa 6 — PDF + preview da Ficha compacta;
- Etapa 7 — Template físico A4 da Ficha;
- Etapa 8 — Limites textuais e densidade da Ficha;
- Etapa 9 — Múltiplos MACs / Procedimentos na Ficha.

A Etapa 5 não deve antecipar essas decisões.

## 8. Escopo correto da Etapa 5

Depois desta correção, a Etapa 5 deve decidir **somente para o Procedimento exportado em PDF/DOCX**:

1. formato físico do documento exportado, incluindo papel/orientação se necessário;
2. margens do documento físico;
3. tipografia documental do PDF e política compatível do DOCX;
4. hierarquia visual de título, metadados, Etapas e blocos;
5. identidade da empresa no documento exportado;
6. cabeçalho/rodapé físicos, se aprovados;
7. numeração física de páginas, se aprovada;
8. representação física de parágrafo, passos, checklist, nota, alerta, comando e código;
9. tratamento de imagens/logo;
10. política de quebras físicas entre Etapas/blocos;
11. eventual sumário **documental** do arquivo exportado, separado do Sumário interativo do Reader;
12. coerência visual entre PDF e DOCX sem prometer paginação idêntica.

Essas decisões não podem modificar o Reader.

## 9. Parâmetros prematuros retirados

Os seguintes itens da primeira versão da proposta **não estão aprovados e deixam de ser tratados como direção consolidável**:

- A4 como formato físico automaticamente inferido para o Procedimento completo;
- margens `20/20/18/18 mm`;
- Noto Sans / Noto Sans Mono como escolha já encaminhada;
- Arial / Consolas como escolha já encaminhada para DOCX;
- título 18 pt / Etapa 14 pt / corpo 10 pt / código 8,5 pt;
- paleta hexadecimal proposta;
- layout fechado da primeira página do PDF/DOCX;
- header institucional já definido;
- `Página X de N` já definido;
- sumário documental já obrigatório;
- regra de fluxo contínuo entre Etapas físicas;
- qualquer equivalência visual entre a página do Reader e uma folha exportada.

Esses pontos voltam a ser **opções em análise** e só podem ser reintroduzidos após validação visual/funcional com o PO.

## 10. Separação que deve aparecer nas fontes canônicas após aprovação

Quando a correção desta Etapa 5 for aprovada, as fontes canônicas afetadas deverão deixar inequívoco:

```text
Reader / uso diário
→ páginas lógicas do manual
→ Visão geral + uma página lógica por Etapa
→ Sumário temporário
→ Anterior / Próxima
→ sem geometria A4

Procedimento exportado
→ PDF / DOCX / impressão
→ documento físico multipágina
→ template físico próprio

Ficha compacta
→ resumo do Atendimento
→ máximo 1 A4
```

A atualização futura deve preservar o modelo visual originalmente aprovado da Tela 05 e apenas acrescentar a clarificação necessária para impedir nova mistura entre navegação e exportação.

## 11. Base histórica recuperada

A correção foi confrontada com o checkpoint de UI que consolidou as Telas 05–07.

Nesse checkpoint, o Reader já estava aprovado com:

- aparência de manual/livro sem skeuomorfismo pesado;
- uma Etapa por página lógica;
- `Visão geral` antes da Etapa 1;
- Sumário temporário listando `Visão geral + etapas`;
- cabeçalho compacto;
- `Etapa X de Y`;
- barra discreta de posição;
- Anterior/Próxima;
- observação, alerta, comando e código na área técnica;
- conteúdo técnico dominando a tela;
- categorias/metadados secundários;
- sem segunda sidebar permanente.

Este checkpoint é a referência visual/funcional do Reader e tem precedência sobre a primeira versão incorreta desta proposta da Etapa 5.

## 12. O que permanece aberto para análise física

A Etapa 5 continua aberta, mas agora com escopo correto.

Antes de consolidá-la, ainda devem ser analisados **somente para PDF/DOCX de Procedimentos**:

- papel/orientação do documento físico;
- margens;
- tipografia;
- primeira página do arquivo exportado;
- cabeçalho/rodapé;
- paginação física;
- estilo físico dos blocos;
- quebra de Etapas/blocos;
- sumário documental, se houver;
- coerência PDF × DOCX.

Não inferir essas escolhas a partir da UI do Reader nem do limite A4 da ficha.

## 13. Critério de fechamento desta correção

Esta proposta corrigida permanece **EM ANÁLISE**.

Até aprovação explícita do PO:

- Etapas 1–4 continuam consolidadas;
- Etapa 5 não é promovida às fontes canônicas;
- PR permanece draft;
- Etapa 6 continua pendente/não aberta;
- nenhuma mudança de UI consolidada é feita;
- nenhum código funcional, dependency, fonte binária, scaffold ou migration é autorizado.
