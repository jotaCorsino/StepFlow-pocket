# Bloco 10 — Etapa 5 — Template físico de Procedimentos — Proposta para análise

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
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

## 9. Escopo físico correto da Etapa 5

Depois das correções acima, esta Etapa deve decidir apenas para o **Procedimento exportado em PDF/DOCX**:

1. formato físico e orientação;
2. margens;
3. tipografia documental;
4. hierarquia de título, metadados, Etapas e blocos;
5. identidade da empresa;
6. cabeçalho/rodapé, se necessários;
7. paginação física, se necessária;
8. representação de parágrafo, passos, checklist, nota, alerta, comando e código;
9. imagens/logo;
10. quebra física entre Etapas e blocos;
11. eventual sumário documental;
12. coerência visual PDF × DOCX sem exigir paginação idêntica.

## 10. Direção física proposta para análise

A mesma filosofia de baixa densidade deve orientar o documento físico.

Direção inicial:

- documento técnico limpo e compacto;
- sem capa exclusiva apenas para título/logo;
- identidade e metadados essenciais apresentados de forma curta;
- conteúdo começa cedo e ocupa a maior parte da página;
- cada Etapa possui separação visual clara, sem card pesado para cada bloco;
- notas, alertas, checklists e código usam forma/símbolo/hierarquia antes de excesso de labels;
- comandos e código permanecem texto real, sem screenshot;
- imagens preservam proporção;
- não reduzir fonte dinamicamente para forçar conteúdo a caber;
- não truncar conteúdo;
- não forçar uma folha física nova para cada Etapa sem necessidade;
- DOCX preserva semântica e hierarquia, mas continua refluível;
- PDF permanece a referência física de impressão.

## 11. Decisões físicas ainda abertas

Os seguintes pontos **não estão consolidados** nesta proposta:

- tamanho de papel do Procedimento completo;
- orientação;
- margens exatas;
- família de fontes;
- tamanhos tipográficos;
- cores exatas;
- composição fechada da primeira página;
- header/footer;
- `Página X de N`;
- sumário documental;
- política exata de page break entre Etapas.

Esses itens precisam ser fechados no contexto do documento exportado, sem derivá-los da UI do Reader nem do limite de uma A4 da Ficha compacta.

## 12. Critério de fechamento

Esta proposta permanece **EM ANÁLISE**.

Até aprovação explícita do PO:

- Etapas 1–4 continuam consolidadas;
- Etapa 5 não é promovida às fontes canônicas;
- PR permanece draft;
- Etapa 6 continua pendente/não aberta;
- nenhuma alteração funcional de UI é consolidada;
- nenhum código, dependency, fonte binária, scaffold ou migration é autorizado.
