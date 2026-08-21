# Tela 08 — Lista e Pesquisa de Atendimentos

## 1. Identificação

- código/nome da tela: Tela 08 — Lista e Pesquisa de Atendimentos;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- origem do insumo visual: requisitos e decisões consolidadas do StepFlow;
- data da última consolidação: 2026-08-21.

## 2. Objetivo da tela

Permitir localizar e abrir rapidamente registros reais de serviço/execução, mantendo a separação conceitual entre:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — ocorrências concretas de serviço;
- `Equipamentos` — ativos físicos opcionais associados a atendimentos quando aplicável.

A tela favorece busca operacional por qualquer informação disponível do atendimento, sem exigir que o usuário saiba antecipadamente qual campo contém o dado procurado.

## 3. Ator(es) e permissões

A tela pode ser acessada por usuários autenticados com capacidade de consultar Atendimentos.

Perfis conceituais existentes:

- ADM;
- Gerência;
- Funcionário/Técnico.

A matriz exata de quem pode visualizar todos os atendimentos, apenas os próprios, criar, editar, concluir ou reabrir permanece para o Bloco 9.

O Client pode ocultar ações sem capacidade, mas a autorização real é sempre validada pelo Host.

## 4. Como o usuário chega à tela

Fluxo principal:

```text
Shell/sidebar
→ Atendimentos
→ Lista/Pesquisa de Atendimentos
```

O retorno futuro da Tela 09 deve preservar busca, filtros, ordenação vigente e posição da lista quando possível.

## 5. Layout e hierarquia visual

Estrutura aprovada:

```text
Atendimentos                                      [ Novo atendimento* ]

[ Buscar por atendimento, OS, cliente, equipamento,
  serial, patrimônio ou MAC...                                 ]

[ Responsável ▾ ] [ Período ▾ ]                 [ Limpar filtros ]

Atendimento   OS/Ref.   Cliente        Equipamento         Responsável   Data
AT-00142      OS-4587   João Silva     NOTE-15 · EQP-31    Maria         21/08
AT-00141      —         Financeiro     —                   Carlos        21/08
AT-00140      CH-991    Empresa Alfa   PC-ADM · EQP-28     Maria         20/08
```

`Novo atendimento` aparece somente quando a capacidade correspondente existir. Sua regra final permanece pendente do Bloco 9.

A tela usa a sidebar global aprovada e não cria navegação paralela.

## 6. Elementos fixos

- título `Atendimentos`;
- campo principal de busca;
- filtros `Responsável` e `Período`;
- resultado em lista/tabela compacta;
- ação `Limpar filtros` quando houver filtro ativo;
- feedback de loading/erro/vazio no espaço da lista.

## 7. Componentes e blocos

### 7.1 Busca principal

Campo único para procurar informações operacionais disponíveis.

Deve considerar, quando existentes:

- código do atendimento;
- ordem de serviço/referência externa;
- cliente/solicitante/responsável relacionado;
- código interno/legível do equipamento;
- nome do equipamento;
- número de série;
- patrimônio/asset tag;
- endereço MAC normalizado.

Exemplos conceituais:

```text
AT-00142
OS-4587
João Silva
NOTE-15
EQP-0031
SN123456
PAT-884
A0:B1:C2:D3:E4:F5
```

O usuário não precisa selecionar previamente o tipo de identificador.

### 7.2 Filtros iniciais

Filtros aprovados para a primeira versão desta tela:

- `Responsável`;
- `Período`.

Não incluir filtro definitivo de `Status` enquanto o lifecycle de Atendimento não for consolidado no Bloco 9.

### 7.3 Tabela/lista

Colunas aprovadas:

- Atendimento;
- OS/Referência;
- Cliente/Solicitante;
- Equipamento;
- Responsável;
- Data.

A coluna `Equipamento` mostra somente resumo curto, como nome/código. CPU, RAM, armazenamento, bateria e demais detalhes pertencem à futura Tela 09.

A coluna `Data` representa a referência temporal operacional disponível. A lista usa ordenação de mais recentes primeiro, mas qual timestamp exato governa essa ordenação será definido junto ao lifecycle no Bloco 9.

## 8. Interações do usuário

- digitar termo de busca;
- aplicar/remover filtros;
- limpar filtros;
- selecionar uma linha para abrir o atendimento;
- usar teclado para navegar pelos resultados;
- acionar `Novo atendimento` somente se autorizado.

Busca e filtros são combináveis.

## 9. Navegação e destinos

Ação principal da linha:

```text
Lista de Atendimentos
→ selecionar atendimento
→ Tela 09 — Atendimento/Execução + Equipamento
```

Ao retornar da Tela 09, preservar quando possível:

- termo de busca;
- filtros;
- ordenação vigente;
- posição de rolagem/página.

Não abrir `Processos` a partir da linha como ação principal.

## 10. Estados da interface

### Inicial

Lista recente compatível com a autorização do usuário.

### Loading

Exibir carregamento no conteúdo da lista, mantendo busca/filtros estáveis.

### Vazio sem busca/filtro

`Nenhum atendimento disponível.`

Se o usuário puder criar, pode haver ação contextual `Novo atendimento`.

### Vazio com busca/filtro

`Nenhum atendimento encontrado com os critérios informados.`

Oferecer `Limpar filtros` quando aplicável.

### Erro de consulta

`Não foi possível carregar os atendimentos.`

Permitir tentar novamente.

### Sem permissão

Não mostrar dados que o Host não autorizou. Se a própria área não for autorizada, o Shell deve ocultar `Atendimentos`; acesso direto deve resultar em estado de permissão negada.

### Host indisponível

Indicar indisponibilidade do Host sem oferecer edição de IP, porta ou caminhos técnicos.

### Item removido/indisponível

Se um item desaparecer entre listagem e abertura, informar que o atendimento não está mais disponível e atualizar a lista.

### Conflito

Esta tela é predominantemente de leitura. Conflitos de edição pertencem à futura Tela 09 e ao Bloco 9.

## 11. Validações

- busca vazia significa consulta normal, não erro;
- filtros inválidos ou combinações impossíveis devem ser rejeitados claramente;
- período não aceita intervalo invertido;
- normalização de MAC pertence ao contrato/Host e não deve exigir formato visual único do usuário quando puder ser interpretado com segurança.

## 12. Mensagens e feedbacks

Mensagens curtas e operacionais. Não expor stack trace, SQL, IDs internos desnecessários, caminhos locais ou detalhes de infraestrutura.

## 13. Dados exibidos

Por linha, conforme disponibilidade/autorização:

- código legível do atendimento;
- OS/referência externa;
- cliente/solicitante;
- resumo do equipamento;
- técnico/responsável;
- data operacional apropriada.

A ficha técnica completa não aparece na lista.

## 14. Dados enviados/alterados

A tela de lista não altera o Atendimento por si só.

Envia somente critérios de consulta, como:

- termo de busca;
- responsável;
- período;
- paginação/limite;
- ordenação quando suportada.

Criação/edição pertence à futura Tela 09 e ao Bloco 9.

## 15. Regras de negócio

- `Atendimentos` representa trabalho real, não documentação;
- equipamento é opcional;
- cliente/OS também podem ser opcionais;
- um atendimento pode usar múltiplos procedimentos;
- vínculo com procedimento preserva revisão efetivamente utilizada;
- MAC, serial e patrimônio são atributos de busca, não identidade canônica exclusiva;
- não criar lifecycle/status por inferência nesta tela;
- não transformar a lista em CRM, help desk completo, estoque ou RMM.

## 16. Regras de autorização

- Host filtra/autoriza os registros retornados;
- Client não assume que esconder botão é controle de segurança;
- capacidades exatas de visualizar todos/próprios atendimentos permanecem para o Bloco 9;
- `Novo atendimento` só aparece quando permitido.

## 17. Impacto em persistência

Nenhuma nova persistência é decidida por esta tela.

A especificação exige apenas que os dados operacionais aprovados possam ser consultados eficientemente. Índices, normalização e implementação de busca pertencem à arquitetura/implementação posterior.

## 18. Contratos Client ↔ Host necessários

Conceitualmente, listagem/pesquisa deve receber:

- termo de busca opcional;
- filtros autorizados;
- paginação;
- ordenação;
- contexto de sessão.

E retornar:

- registros resumidos autorizados;
- total/continuação quando necessário;
- metadados mínimos para renderização.

Rotas e payloads finais não são definidos no Bloco 8.

## 19. Eventos em tempo real relevantes

Evento pós-commit de criação/alteração/arquivamento de Atendimento relevante à consulta pode provocar reconsulta da lista.

A atualização deve preservar contexto e evitar deslocar silenciosamente o usuário de forma brusca. Eventos de Equipamento podem exigir reconsulta quando alterarem o resumo exibido.

## 20. Comportamento de concorrência

A lista não executa escrita concorrente.

Se um Atendimento mudar enquanto está listado:

- evento pode sinalizar/reconsultar;
- ao abrir, o Host fornece o estado atual autorizado;
- se deixou de existir/ficou indisponível, informar e remover da lista.

Controle otimista de alterações do Atendimento pertence à futura Tela 09/Bloco 9.

## 21. Acessibilidade e teclado

- busca com label acessível;
- filtros navegáveis por Tab;
- foco visível;
- linhas/ações acionáveis por teclado;
- headers de tabela semanticamente identificados;
- estado vazio/erro não depende apenas de cor;
- filtros ativos possuem indicação textual.

## 22. Comportamento em tamanhos de janela suportados

Desktop Windows é o alvo principal.

Em janelas menores suportadas:

- preservar busca e identificação principal;
- reduzir/ocultar progressivamente colunas secundárias antes de prejudicar legibilidade;
- não transformar automaticamente em experiência mobile;
- scroll horizontal apenas como último recurso e sem esconder a ação principal.

Dimensões mínimas exatas permanecem no contrato visual transversal do Bloco 8.

## 23. Preservação visual / decisões de UI aprovadas

- visual corporativo, limpo e discreto;
- sidebar esquerda persistente;
- sem topbar global redundante;
- lista/tabela compacta coerente com `Processos`;
- feedback textual curto;
- sem gráficos/KPIs;
- sem cards grandes como padrão para cada atendimento.

## 24. Divergências com documentação anterior

Nenhuma divergência de produto.

Esta tela concretiza a separação já aprovada entre busca documental de `Processos` e busca operacional de `Atendimentos`.

## 25. Decisões consolidadas nesta análise

Aprovadas pelo PO em 2026-08-21:

1. lista/tabela compacta como visualização padrão;
2. campo único de busca operacional;
3. filtros iniciais `Responsável` + `Período`;
4. equipamento aparece apenas como resumo na linha;
5. selecionar a linha abre a futura Tela 09;
6. retorno preserva busca/filtros/ordenação/posição quando possível;
7. coluna/filtro `Status` não entra antes da decisão de lifecycle do Bloco 9;
8. ordenação padrão por registros mais recentes, mantendo o timestamp exato pendente do Bloco 9.

Continuam vigentes as decisões superiores de existência da área `Atendimentos`, separação `Procedimento × Atendimento × Equipamento`, equipamento opcional, busca por identificadores operacionais e MAC não canônico.

## 26. Pendências

Não são pendências de aprovação da Tela 08. Permanecem para blocos posteriores:

- lifecycle/status do Atendimento;
- timestamp exato que define a ordenação por mais recentes;
- matriz de permissões de próprios/todos/criação/edição/conclusão/reabertura;
- comportamento de edição e concorrência do detalhe;
- checklist/progresso operacional;
- impressão/PDF da ficha compacta.

## 27. Fora do escopo

- lifecycle/status do Atendimento;
- concluir/reabrir;
- checklist/progresso operacional;
- edição do Atendimento;
- cadastro detalhado de equipamento;
- regras finais de quem vê próprios/todos;
- impressão da ficha compacta;
- PDF da ficha;
- QR/barcode;
- SLA, fila de chamados, financeiro, estoque ou CRM.

## 28. Critérios de aceite

- [x] PO aprovou tabela compacta;
- [x] PO aprovou busca operacional única;
- [x] PO aprovou filtros iniciais Responsável + Período;
- [x] PO aprovou preservação de busca/filtros no retorno;
- [x] PO aprovou manter Status fora até decisão do Bloco 9;
- [x] a tela não mistura busca de Processos e Atendimentos;
- [x] a tela não exige equipamento em todo atendimento;
- [x] nenhum lifecycle operacional foi inventado;
- [x] autorização permanece Host-side;
- [x] estados loading/vazio/erro/Host indisponível estão definidos;
- [x] nenhuma implementação funcional foi criada nesta análise.

## 29. Casos de teste/smoke sugeridos

Quando houver implementação:

- abrir Atendimentos sem filtros;
- buscar por código de atendimento;
- buscar por OS/referência;
- buscar por cliente;
- buscar por nome/código de equipamento;
- buscar por serial;
- buscar por patrimônio;
- buscar MAC em formatos aceitos;
- aplicar Responsável;
- aplicar Período;
- combinar busca + filtros;
- limpar filtros;
- abrir resultado e retornar preservando estado;
- consulta vazia;
- item removido entre lista e abertura;
- Host indisponível;
- usuário sem capacidade de consulta;
- atualização em tempo real preservando contexto visual.
