# Tela 05 — Leitor de Processo em Formato Livro

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco original:** Fase 1 — Bloco 8 (UI/UX)  
**Atualização operacional:** Bloco 9 + Bloco 10 / Etapa 6  
**Última consolidação:** 2026-08-28

## 1. Objetivo

Ser a principal superfície de consumo de Procedimentos do StepFlow, com leitura em formato de manual/livro e duas modalidades coerentes:

1. **consulta documental** — leitura de uma revisão sem estado operacional persistente;
2. **execução vinculada a Atendimento** — leitura da revisão exata vinculada, com checklist e observações de serviço persistentes daquele Atendimento.

A identidade visual principal é a mesma nos dois contextos.

## 2. Estrutura visual

O Shell/sidebar global permanece visível. Não existe segunda sidebar permanente.

Cabeçalho compacto:

```text
← Processos / ← Atendimento AT-...

PR-014 · Configuração de VLAN
Versão 2.0 · revisão r18 · Publicada
[ categorias ] · Infraestrutura

Etapa 3 de 7                         [ Sumário ] [ ações contextuais ]

●━━━━●━━━━◉────○────○────○────○
```

Quando em execução:

```text
Executando no atendimento AT-000142
PR-014 · Versão 2.0 · revisão r18
```

O contexto de Atendimento precisa ficar claro sem transformar o Reader em outra aplicação.

A UI segue o princípio transversal de **clareza com baixa densidade permanente**: informação essencial permanece visível; estados simples e ações recorrentes podem usar forma, cor, símbolo, posição e ícones reconhecíveis quando isso reduzir texto sem criar ambiguidade. Detalhes secundários podem aparecer sob demanda. Cor nunca é o único meio de comunicar um estado importante.

## 3. Páginas do manual

Antes da Etapa 1 existe uma página `Visão geral`, não numerada como etapa.

Pode apresentar:

- objetivo;
- pré-requisitos;
- observações;
- responsável documental;
- categorias;
- versão/revisão.

Cada `process_stage` é uma página do manual.

```text
Visão geral
→ Etapa 1
→ Etapa 2
→ ...
→ Etapa N
```

Ao mudar de página, o conteúdo começa no topo.

Se uma Etapa possuir conteúdo maior do que a área visível, o conteúdo não é truncado e a próxima Etapa não é fundida na mesma página lógica. Overflow/rolagem deve preservar uma Etapa por página lógica.

## 4. Navegação

A navegação do Reader oferece caminhos equivalentes para o mesmo conjunto de páginas:

- `Anterior`;
- `Próxima`;
- `Sumário` em painel temporário;
- indicador textual compacto `Etapa X de Y`;
- **stepper horizontal compacto e navegável** entre as Etapas.

O stepper é composto prioritariamente por círculos conectados por linha, usando preenchimento, contraste, forma/símbolo e cor para diferenciar:

- Etapas anteriores à atual → estado visual de percorridas/concluídas na navegação;
- Etapa atual → destaque inequívoco;
- Etapas seguintes → estado neutro/futuro.

Exemplo conceitual:

```text
●━━━━●━━━━◉────○────○────○────○
```

Regras do stepper:

- cada marcador representa uma Etapa;
- cada marcador é acionável por clique e teclado e abre diretamente aquela Etapa no topo;
- não repetir permanentemente nomes como `Etapa 1`, `Etapa 2`, etc. junto a todos os marcadores;
- o nome completo continua no título da página e no Sumário;
- tooltip/nome acessível pode apoiar identificação sem aumentar densidade visual permanente;
- o componente deve se adaptar horizontalmente à largura disponível da página;
- não transformar o stepper em uma faixa alta ou carregada de metadados.

`Visão geral` continua sendo a primeira página lógica do Sumário, mas não precisa ser representada como `Etapa 0` no stepper. A primeira Etapa pode voltar para `Visão geral` por `Anterior` ou pelo Sumário.

`Etapa X de Y`, stepper e páginas visitadas representam **posição/percurso de navegação**, nunca conclusão operacional do Atendimento. O estado visual de uma Etapa anterior no stepper não grava `completed=true`, não altera checklist e não confirma que o técnico executou aquela Etapa.

A última Etapa não oferece avanço inválido.

## 5. Blocos tipados

Tipos iniciais:

- `paragraph`;
- `numbered_steps`;
- `checklist`;
- `note`;
- `warning`;
- `command`;
- `code`.

Não renderizar HTML arbitrário como conteúdo oficial.

### Passos numerados

Suportam passos/subpassos dentro da hierarquia aprovada, com numeração derivada.

### Comando/código

- fonte monoespaçada;
- preservar whitespace;
- não executar comando;
- botão de copiar apenas com ícone, mas com nome acessível;
- copiar conteúdo exato;
- feedback curto como `✓ Copiado`.

## 6. Consulta documental

Fluxo normal:

```text
Processos
→ selecionar Procedimento
→ Reader da revisão apropriada
```

Nesse contexto:

- checklist é definição documental;
- marcar/desmarcar não persiste execução;
- não existe `Observação do serviço` operacional;
- navegação não grava progresso;
- não existe estado operacional escondido;
- nenhum Atendimento é criado apenas por ler.

A primeira versão pode tratar checkbox documental como elemento visual/consulta, mas nunca como execução persistente fora de Atendimento.

## 7. Execução vinculada a Atendimento

Fluxo:

```text
Atendimento AT-...
→ Procedimento vinculado
→ Abrir revisão / Executar
→ Reader da revisão exata em contexto operacional
```

Nesse contexto:

- cabeçalho identifica o Atendimento;
- revisão fica presa ao `service_record_process`;
- itens de checklist usam estado persistente do Atendimento;
- cada Etapa pode receber uma `Observação do serviço` opcional e persistente;
- marcar/desmarcar checklist ou alterar observação depende de `Em andamento` + capacidade;
- voltar retorna à Tela 09;
- navegação entre páginas não marca item nem conclui Atendimento;
- publicação de revisão nova não muda a revisão em execução.

### Observação do serviço por Etapa

Durante a execução, a página da Etapa pode oferecer um campo simples próximo ao final do conteúdo:

```text
Observação do serviço
[ Ex.: unidade antiga apresentou setores defeituosos. ]
```

Contrato:

- é opcional;
- pertence ao **Atendimento + vínculo da revisão + Etapa**, nunca ao Procedimento oficial;
- não altera `process_revision`, `process_stage` ou o texto documental;
- é persistida pelo Host somente em contexto operacional;
- fica disponível para o resumo/ficha do serviço quando preenchida;
- campo vazio não cria ruído no material impresso;
- após `Concluído` ou `Cancelado`, fica somente leitura até eventual reabertura;
- publicação de nova revisão não move nem reatribui a observação para outra Etapa;
- o mecanismo de salvamento segue o contrato operacional explícito; não introduzir autosave por inferência.

A observação é uma nota de execução do técnico, não comentário social, chat, anotação colaborativa do Procedimento nem item de checklist.

## 8. Checklist em execução

O estado operacional não modifica o Procedimento.

Enquanto Atendimento está `Em andamento` e sessão autorizada:

- marcar item envia mutação ao Host;
- desmarcar item envia mutação ao Host;
- Client apresenta estado confirmado/reconciliado;
- concorrência é granular por item/equivalente;
- evento remoto nunca sobrescreve edição/estado local silenciosamente.

Em `Concluído` ou `Cancelado`, checklist fica somente leitura até eventual reabertura.

## 9. Progresso operacional

Somente no contexto de Atendimento, o Reader pode mostrar contagem derivada dos checklists:

```text
4 de 6 itens concluídos
```

Regras:

- não usar páginas visitadas;
- não usar `Etapa X de Y` ou o stepper como percentual operacional;
- revisão sem checklist não mostra `0%` artificial;
- 100% de checklist não conclui Atendimento automaticamente;
- conclusão continua ação explícita da Tela 09.

## 10. Revisão consultada

O Reader sempre deixa claro qual revisão está aberta.

### Operação normal

Funcionário abre normalmente revisão publicada autorizada.

### ADM/Gerência

Podem abrir revisão atual/histórica/não publicada quando já possuírem autorização correspondente.

### Revisão histórica

Marca persistente, por exemplo:

```text
Revisão histórica r17 · Versão 1.9
```

### Revisão mais nova disponível

Se surgir revisão nova:

- não trocar automaticamente;
- mostrar aviso discreto;
- permitir ação consciente de abrir a nova quando autorizada;
- em Atendimento, nunca substituir automaticamente a revisão vinculada.

## 11. Iniciar Atendimento a partir do Reader

Quando a sessão possuir capacidade `Criar Atendimento`, pode existir ação contextual:

```text
[ Iniciar atendimento ]
```

Fluxo consolidado:

```text
Reader da revisão consultada
→ Iniciar atendimento
→ Tela 09 em rascunho somente em memória
→ revisão consultada pré-selecionada
→ primeiro save aceito cria AT-......
```

A revisão pré-selecionada precisa ser elegível conforme o preset:

- Funcionário: revisão publicada;
- ADM/Gerência: revisão autorizada selecionada explicitamente.

Se o usuário sair antes do primeiro save, nenhum Atendimento oficial é criado.

## 12. Ações contextuais documentais

Conforme capacidade:

- `Editar`;
- `Histórico`;
- `Exportar / Imprimir`;
- `Iniciar atendimento`.

Ações não aplicáveis não ocupam a UI por padrão.

Ações recorrentes podem privilegiar ícones reconhecíveis quando o significado permanecer claro e houver nome acessível/tooltip adequado.

Exportação/impressão usa exatamente a revisão aberta, conforme Tela 14/Bloco 10.

## 13. Estados

### Loading

Shell permanece estável; conteúdo usa skeleton/estado local.

### Sem permissão

Host rejeita; conteúdo protegido não fica exposto.

### Host indisponível

Segue Tela 15; não exibir campos de IP/porta/path.

### Revisão arquivada/histórica

Mostrar identificação contextual sem fingir que é a revisão publicada atual.

### Revisão nova disponível

Aviso discreto, sem troca automática.

### Atendimento concluído/cancelado

Quando Reader está no contexto desse Atendimento:

- checklist e observações do serviço ficam somente leitura;
- contexto/lifecycle visível;
- revisão continua estável.

### Conflito operacional

Checklist e `Observação do serviço` usam reconciliação proporcional ao recurso afetado. Não transformar um checkbox ou uma observação de Etapa em conflito global de toda a Tela 09 por conveniência.

## 14. Eventos em tempo real

Eventos são sinais de mudança.

### Reader documental

- nova revisão → aviso/reconsulta de metadados;
- revisão aberta não muda silenciosamente.

### Reader operacional

- checklist alterado → reconsultar estado relevante;
- observação de serviço alterada → reconsultar apenas a observação/Etapa relevante;
- status do Atendimento alterado → atualizar capacidade de edição;
- reabertura/conclusão/cancelamento → refletir lifecycle;
- evento não substitui conteúdo local de forma silenciosa.

## 15. Concorrência

- Reader documental é leitura e não cria lock;
- checklist operacional usa controle granular por item/equivalente;
- observação de serviço usa controle granular por Etapa/equivalente;
- usuários alterando recursos operacionais independentes não devem conflitar globalmente;
- writer/fila do Host ordena mutações, mas não autoriza estado obsoleto;
- resultado incerto após desconexão é reconciliado antes de retry não idempotente.

## 16. Acessibilidade

- navegação por teclado;
- headings semânticos;
- foco visível;
- botões icon-only com nome acessível;
- marcadores do stepper com nome/estado acessível e acionamento por teclado;
- campo de observação com label acessível inequívoco;
- feedback de cópia anunciável;
- estados de stepper e checklist não dependem só de cor;
- avisos de revisão/lifecycle acessíveis.

## 17. Janelas suportadas

Alvo desktop Windows.

Em janelas menores suportadas:

- conteúdo pode reduzir colunas/empilhar metadados;
- stepper redistribui o espaço horizontal sem exigir rótulos permanentes por Etapa;
- controles permanecem acessíveis;
- sem transformação mobile/hamburger inicial;
- evitar scroll horizontal no conteúdo técnico sempre que possível.

## 18. Decisões preservadas

- manual/livro é a experiência principal;
- `Visão geral` precede Etapa 1;
- uma Etapa = uma página lógica;
- Sumário é temporário;
- stepper compacto de círculos/linhas é navegável entre Etapas;
- estados anterior/atual/seguinte do stepper são visuais de navegação, não conclusão operacional;
- baixa densidade textual e uso responsável de cor/forma/símbolo orientam a apresentação;
- blocos são tipados;
- copiar é icon-only com feedback breve;
- revisão aberta permanece estável;
- Reader standalone não persiste checklist nem observação de serviço;
- Reader em Atendimento persiste checklist e observação de serviço por Etapa;
- observação operacional não altera o Procedimento;
- progresso operacional deriva somente de checklist;
- checklist não conclui Atendimento automaticamente;
- nenhum lock/merge automático/offline editing/autosave é introduzido.

## 19. Fora do escopo

- edição do Procedimento dentro do Reader;
- colaboração simultânea tipo editor compartilhado;
- presença/soft lock;
- conclusão automática por navegação;
- persistência de checklist/observação operacional fora de Atendimento;
- fila offline/autosave;
- UI mobile dedicada;
- implementação funcional nesta fase.