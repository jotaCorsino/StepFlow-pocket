# Bloco 10 — Etapa 7 — Template físico A4 da Ficha — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-28  
**Base consolidada:** Bloco 10 / Etapas 1–6  
**Base Git:** `main` em `bf45e4f266b9c2c1cedec2a62b869512025726e3`

## 1. Objetivo

Fechar somente o **template físico da única página A4 da Ficha de Atendimento**, sem reabrir o contrato de PDF/preview da Etapa 6.

A Ficha continua sendo uma **prestação de contas resumida ao cliente**. Ela não deve parecer relatório técnico completo, checklist ou formulário administrativo carregado.

Esta etapa define:

- geometria da página;
- hierarquia visual;
- ordem das seções;
- tipografia física;
- uso de linhas, espaçamento e contraste;
- apresentação de Equipamento, serviço realizado e observações;
- comportamento visual quando seções opcionais estiverem vazias.

Permanecem fora desta etapa:

- limites máximos de caracteres/linhas e priorização de overflow — Etapa 8;
- muitos MACs, muitas observações ou outros dados excepcionais — Etapa 9;
- nomes de arquivo e temporários — Etapa 10;
- QR/barcode — Etapa 11;
- matriz técnica final — Etapa 12.

## 2. Princípio visual

A folha deve comunicar primeiro **o que o cliente precisa entender**, não tudo que o StepFlow sabe.

Direção:

```text
identificar
→ resumir
→ registrar observações relevantes
```

Evitar:

- tabela pesada com muitas células;
- vários cards fechados por borda;
- excesso de badges/chips;
- labels repetidos;
- lista detalhada de Procedimentos/revisões;
- checklist/progresso/timeline;
- grandes áreas reservadas para campos vazios.

A informação deve usar espaço apenas quando existir.

## 3. Geometria física

Baseline proposto:

```text
papel:       A4
orientação:  retrato
páginas:     exatamente 1
margens:     15 mm em todos os lados
bleed:       nenhum
```

Consequências:

- área útil aproximada: `180 × 267 mm`;
- não existe segunda página;
- não existe redução dinâmica de fonte para forçar encaixe;
- não existe crop/truncamento silencioso;
- se o conteúdo não couber, aplica-se o gate `SHEET_OVERFLOW` consolidado.

A margem de 15 mm preserva área útil sem aproximar o conteúdo excessivamente das bordas físicas comuns de impressão.

## 4. Estrutura geral da folha

A composição é predominantemente **uma coluna vertical**, com microagrupamentos horizontais apenas para dados curtos do serviço/equipamento.

Ordem proposta:

```text
1. Identidade da empresa + Atendimento
2. Identificação curta do serviço
3. Equipamento/dispositivo, quando houver
4. Serviço realizado
5. Observações, quando houver
```

Não há capa, sumário, header repetitivo ou paginação porque a Ficha possui exatamente uma página.

Também não há footer obrigatório por padrão. Contato institucional pode permanecer no cabeçalho compacto.

## 5. Cabeçalho compacto

O topo deve ocupar pouco espaço.

Estrutura conceitual:

```text
[LOGO] Nome da Empresa                     AT-000142
       contato/site/e-mail                 28/08/2026
                                            Concluído?
────────────────────────────────────────────────────
```

Regras:

- logo somente quando configurado;
- preservar proporção do logo;
- nome da empresa é o elemento institucional principal;
- contato/site/e-mail aparecem em uma linha curta ou duas quando configurados;
- código do Atendimento fica facilmente localizável;
- data aplicável fica junto da identificação do Atendimento;
- não criar título grande `FICHA DE ATENDIMENTO` consumindo área útil se o contexto já estiver claro.

### Status

Status só recebe destaque quando necessário:

- `Em andamento` → indicar discretamente `Acompanhamento` ou `Em andamento`;
- `Cancelado` → texto explícito `CANCELADO`, com contraste/forma além de cor;
- `Concluído` → não precisa de faixa grande; data/status podem ficar na identificação compacta.

Não usar watermark grande atravessando a página.

## 6. Identificação do serviço

Logo abaixo do cabeçalho, uma faixa textual compacta reúne somente o que estiver preenchido e for útil.

Exemplo:

```text
Cliente: João Silva · OS: OS-4587 · Técnico: Maria Souza
```

Campos ausentes são simplesmente omitidos, sem imprimir `—`, `não informado` ou espaço vazio reservado.

Não usar tabela de formulário para esses três ou quatro campos.

## 7. Equipamento / dispositivo

Quando houver Equipamento, a seção deve funcionar como uma **ficha técnica resumida**, não cadastro completo.

Título curto:

```text
EQUIPAMENTO
```

Primeira linha de identificação:

```text
NOTE-15 · Notebook · EQP-0031
```

Características podem ocupar duas ou três linhas compactas:

```text
CPU  Intel Core i5-1135G7     RAM  16 GB     SSD  NVMe 512 GB
Sistema  Windows 11 Pro 24H2                 Bateria  82%
Serial  ABC123                 Patrimônio  PAT-884
```

Regras:

- não desenhar uma grade visível completa;
- usar alinhamento e espaçamento para formar grupos;
- labels curtos como `CPU`, `RAM`, `SSD`, `Sistema`, `Bateria` podem ser usados quando melhorarem a leitura;
- armazenamento não deve assumir sempre SSD: o valor do domínio continua descrevendo HD/SSD/tipo real;
- serial/patrimônio aparecem quando úteis e existentes;
- MAC continua sujeito à Etapa 9;
- observação do Equipamento não precisa ficar presa dentro desta grade; ela pode integrar a seção unificada de Observações abaixo.

### Sem Equipamento

A seção inteira desaparece e o restante do conteúdo sobe naturalmente.

Não imprimir `Nenhum equipamento vinculado` na prestação de contas.

## 8. Serviço realizado — conteúdo principal

`Resumo do trabalho` deve aparecer com rótulo client-facing simples:

```text
SERVIÇO REALIZADO
```

O conteúdo é texto corrido legível, não tabela.

Exemplo conceitual:

```text
Realizada limpeza preventiva, substituição do SSD e validação geral
do equipamento após reinstalação do sistema.
```

Regras:

- não repetir checklist ou passos;
- não listar revisão técnica;
- não transformar cada ação em card;
- usar parágrafo normal com bom espaçamento;
- a Etapa 8 definirá limites/priorização quando o resumo for excessivo.

## 9. Observações — maior área variável

Depois do resumo, o restante da área útil deve favorecer as observações realmente registradas durante o serviço.

Usar uma única seção client-facing:

```text
OBSERVAÇÕES
```

Ela pode combinar, quando existirem e forem relevantes:

- observação geral do Atendimento;
- observação do Equipamento;
- observações do serviço registradas nas Etapas.

Apresentação preferencial: lista simples, sem subcards.

```text
• Excesso de poeira próximo ao cooler.
• SSD — unidade anterior apresentou setores defeituosos.
• Validação final — bateria em 68%; recomendado acompanhamento.
```

### Contexto da Etapa

O título/nome curto da Etapa pode aparecer antes da observação **somente quando ajudar a compreender o contexto**.

Não imprimir permanentemente:

```text
Etapa 03 de 07
Procedimento PR-022 revisão r7
```

Preferir:

```text
SSD — unidade anterior apresentou setores defeituosos.
```

A forma exata de condensar títulos excepcionalmente longos permanece subordinada aos limites da Etapa 8.

### Sem observações

Se nenhuma observação aplicável existir, a seção inteira é omitida.

Não imprimir `Sem observações` apenas para preencher espaço.

## 10. Tipografia

A Ficha é somente PDF na v1, portanto reutiliza a família já controlada no Host:

```text
texto:       Noto Sans
```

`Noto Sans Mono` não é necessária no template normal da Ficha porque comandos/código não fazem parte da prestação de contas.

Escala proposta:

```text
nome/código principal:  14 pt
seção:                  10,5 pt semibold
corpo/resumo:            10 pt
ficha técnica:            9 pt
metadados institucionais: 8,5 pt
```

Regras:

- não reduzir dinamicamente abaixo do baseline para caber;
- peso/contraste substituem caixas sempre que possível;
- evitar ALL CAPS em parágrafos; usar somente em pequenos títulos de seção quando útil.

## 11. Linhas, cores e preenchimentos

O documento deve continuar legível em impressora monocromática.

Direção:

- texto principal escuro;
- divisórias horizontais finas apenas entre grandes grupos;
- cinza neutro para metadados secundários;
- sem grandes fundos preenchidos;
- sem depender de cor para `Cancelado`, `Em andamento` ou qualquer outro estado;
- identidade da empresa pode aparecer pelo logo, sem introduzir uma nova configuração obrigatória de paleta na v1.

Não congelar hex/RGB específicos nesta fase; o template pode trabalhar com tokens internos de contraste neutro.

## 12. Espaçamento

A folha deve parecer leve, mas não desperdiçar espaço.

Direção:

- espaçamento maior entre seções principais;
- espaçamento curto entre linhas relacionadas da ficha técnica;
- nenhuma altura fixa para seções textuais;
- seções vazias colapsam completamente;
- Observações crescem conforme o conteúdo disponível;
- não reservar caixas vazias para futura escrita manual.

## 13. Exemplo conceitual

```text
┌──────────────────────────────────────────────────────────┐
│ [LOGO] EMPRESA                              AT-000142    │
│        contato · site · e-mail              28/08/2026   │
│──────────────────────────────────────────────────────────│
│ João Silva · OS-4587 · Técnico: Maria Souza              │
│                                                          │
│ EQUIPAMENTO                                               │
│ NOTE-15 · Notebook · EQP-0031                            │
│ CPU i5-1135G7 · RAM 16 GB · SSD NVMe 512 GB             │
│ Windows 11 Pro 24H2 · Bateria 82%                        │
│ Serial ABC123 · Patrimônio PAT-884                       │
│                                                          │
│ SERVIÇO REALIZADO                                         │
│ Limpeza preventiva, substituição do SSD e validação      │
│ geral do equipamento.                                     │
│                                                          │
│ OBSERVAÇÕES                                               │
│ • Excesso de poeira próximo ao cooler.                    │
│ • SSD — unidade anterior apresentou setores defeituosos. │
│ • Validação final — bateria requer acompanhamento.        │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

Esse desenho é hierarquia funcional; não congela largura exata de cada label nem quantidade de caracteres.

## 14. Casos de lifecycle

### Em andamento

A mesma estrutura é usada, com identificação discreta de que se trata de acompanhamento.

Não criar template alternativo.

### Concluído

Usa o estado histórico aplicável. A folha continua simples; não precisa de selo grande de conclusão.

### Cancelado

Usa a mesma estrutura, mas `CANCELADO` precisa ser textual e inequívoco no topo.

Não usar apenas vermelho/cor como semântica.

## 15. O que o template não deve adicionar

Não introduzir na Etapa 7:

- assinatura do cliente/técnico;
- campo manual para data;
- termos jurídicos;
- garantia;
- valores/custos/financeiro;
- peças/estoque;
- SLA/prioridade;
- checklist;
- progresso;
- timeline;
- QR/barcode;
- lista detalhada de Procedimentos;
- página 2;
- rodapé promocional do StepFlow.

Qualquer um desses itens exige requisito próprio.

## 16. Overflow

O template nunca resolve excesso escondendo informação ou comprimindo tipografia dinamicamente.

```text
layout normal
→ 1 página
→ válido

layout normal
→ 2+ páginas
→ SHEET_OVERFLOW
```

A Etapa 8 decidirá:

- limites textuais;
- prioridade de campos;
- mensagens/diagnóstico do que precisa ser resumido;
- regras controladas para conteúdo excessivo.

A Etapa 9 tratará casos multiplicativos como muitos MACs/dados excepcionais.

## 17. Acessibilidade e conteúdo real

- texto permanece texto real no PDF;
- ordem visual segue ordem semântica;
- títulos de seção são identificáveis;
- contraste adequado também em escala de cinza;
- status não depende só de cor;
- não rasterizar a folha para criar o layout;
- logo continua asset controlado, mas ausência dele não prejudica a hierarquia.

## 18. Decisões propostas para aprovação

1. A4 retrato, exatamente uma página, margens de 15 mm;
2. composição predominantemente vertical/uma coluna;
3. cabeçalho institucional compacto, sem título gigante e sem footer obrigatório;
4. identificação do serviço em linha curta, omitindo campos vazios;
5. Equipamento em ficha técnica resumida sem tabela gradeada;
6. `SERVIÇO REALIZADO` como área narrativa principal;
7. uma única seção `OBSERVAÇÕES`, reunindo observações relevantes sem cards;
8. nome curto da Etapa somente quando necessário para contextualizar sua observação;
9. Noto Sans com escala 14 / 10,5 / 10 / 9 / 8,5 pt conforme hierarquia;
10. divisórias discretas e contraste neutro, legível em monocromático;
11. seções vazias colapsam; nenhuma caixa vazia é reservada;
12. lifecycle usa o mesmo template, com indicação textual proporcional quando necessária;
13. nenhum segundo layout, assinatura, financeiro, QR ou detalhe interno é acrescentado;
14. overflow continua falha explícita; Etapas 8–9 resolvem limites/dados excepcionais.

## 19. Gate

Enquanto esta proposta não for aprovada:

- somente este arquivo de proposta pode ser alterado nesta branch;
- nenhuma fonte canônica é promovida;
- Etapa 8 não é aberta.
