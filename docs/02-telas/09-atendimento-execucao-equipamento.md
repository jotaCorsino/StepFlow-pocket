# Tela 09 — Atendimento / Execução + Equipamento

## 1. Identificação

- código/nome da tela: Tela 09 — Atendimento / Execução + Equipamento;
- status: **EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO**;
- origem do insumo visual: requisitos e decisões consolidadas do StepFlow;
- data da última consolidação: 2026-08-21.

## 2. Objetivo da tela

Ser o workspace principal de um atendimento real, reunindo em uma única superfície:

- dados básicos do atendimento;
- cliente/solicitante e OS/referência quando existirem;
- equipamento opcional;
- procedimentos/revisões utilizados;
- resumo do trabalho realizado;
- observações;
- ponto de entrada para a futura ficha compacta/impressão.

A tela não deve transformar o StepFlow em sistema completo de chamados, CRM, estoque ou RMM.

## 3. Ator(es) e permissões

A tela pode ser acessada por usuários autenticados com capacidade de consultar Atendimentos.

Perfis conceituais existentes:

- ADM;
- Gerência;
- Funcionário/Técnico.

A matriz exata de quem cria, edita, conclui, reabre, vincula/troca equipamento, altera cadastro do equipamento ou gera ficha permanece para o Bloco 9.

O Client apresenta/oculta ações conforme capacidades recebidas, mas a autorização real permanece no Host.

## 4. Como o usuário chega à tela

Fluxos conceituais:

```text
Atendimentos
→ selecionar registro
→ Tela 09
```

ou futuramente:

```text
Leitor de Processo
→ Iniciar atendimento
→ Tela 09 com a revisão consultada pré-selecionada
```

ou:

```text
Atendimentos
→ Novo atendimento
→ Tela 09 em modo de criação
```

A regra exata de criação e os estados permitidos permanecem para o Bloco 9.

## 5. Layout e hierarquia visual

Proposta de estrutura em uma única página vertical, sem criar segunda sidebar nem esconder informações principais atrás de muitas abas:

```text
← Atendimentos

Atendimento AT-00142                              [ Ficha / Imprimir* ]
OS-4587 · João Silva · Responsável: Maria               [ Salvar* ]

────────────────────────────────────────────────────────────
ATENDIMENTO

OS / Referência        [ OS-4587                    ]
Cliente / Solicitante  [ João Silva                 ]
Responsável            [ Maria ▾                    ]

Resumo do trabalho
[ Manutenção preventiva, limpeza e troca de SSD...         ]

Observações
[ ......................................................... ]

────────────────────────────────────────────────────────────
EQUIPAMENTO                                              [ Editar* ]

NOTE-15 · EQP-0031                                 Notebook

Nome               NOTE-15
Processador         Intel Core i5-1135G7
RAM                 16 GB
Armazenamento       SSD NVMe 512 GB
Sistema             Windows 11 Pro · 24H2
Serial              ABC123
Patrimônio          PAT-884
MAC                 A0:B1:C2:D3:E4:F5
Bateria             82%
Observações         ...

[ Trocar/Vincular equipamento* ] [ Desvincular* ]

────────────────────────────────────────────────────────────
PROCEDIMENTOS UTILIZADOS

PR-001  Manutenção preventiva
Versão 1.3 · revisão r18                       [ Abrir revisão ]

PR-022  Substituição de SSD
Versão 2.0 · revisão r7                        [ Abrir revisão ]

[ + Adicionar procedimento* ]
```

`*` indica ação dependente de capacidade/lifecycle ainda pendente dos Blocos 9/10.

## 6. Elementos fixos

- retorno para `Atendimentos`;
- identificação legível do atendimento;
- resumo operacional curto no cabeçalho;
- seção `Atendimento`;
- seção `Equipamento`, quando houver, ou estado `Sem equipamento`;
- seção `Procedimentos utilizados`;
- feedback de salvamento/erro/conflito;
- ação contextual futura `Ficha / Imprimir` sem definir neste bloco sua tecnologia ou layout final.

## 7. Componentes e blocos

### 7.1 Dados do atendimento

Campos conceituais já aprovados:

- código legível do atendimento — gerado pelo sistema;
- OS/referência externa opcional;
- cliente/solicitante/responsável relacionado, quando aplicável;
- técnico/responsável pelo atendimento;
- datas aplicáveis conforme lifecycle futuro;
- resumo do trabalho realizado;
- observações.

Não introduzir campos burocráticos sem requisito aprovado.

### 7.2 Equipamento opcional

A tela deve funcionar corretamente nos dois cenários:

```text
Atendimento sem equipamento
Atendimento com equipamento
```

Quando não houver equipamento associado, mostrar estado simples:

`Nenhum equipamento vinculado a este atendimento.`

Quando houver capacidade correspondente, pode existir ação `Vincular equipamento`.

### 7.3 Busca/vínculo de equipamento

Proposta: `Vincular equipamento` abre painel/modal contextual com:

- busca por código StepFlow do equipamento;
- nome;
- serial;
- patrimônio;
- MAC;
- cliente/solicitante relacionado quando aplicável;
- ação `Cadastrar novo equipamento` quando autorizada.

A busca reutiliza a identidade própria do Equipamento e não cria duplicação silenciosa só porque o usuário informou um MAC ou serial já conhecido.

### 7.4 Ficha técnica do equipamento

Para computadores/notebooks, exibir conforme disponibilidade:

- código StepFlow;
- nome;
- tipo;
- processador;
- memória RAM;
- armazenamento;
- sistema operacional;
- versão do sistema;
- número de série;
- patrimônio/asset tag;
- um ou mais MACs;
- saúde da bateria quando aplicável;
- cliente/solicitante/responsável relacionado;
- observações.

Campos vazios ou não aplicáveis não devem ocupar a tela com ruído.

RAM, armazenamento e processador são inicialmente resumos textuais. O StepFlow não tenta inventariar automaticamente o hardware.

### 7.5 Edição do equipamento

Como Equipamento é entidade reutilizável, a proposta é não misturar silenciosamente suas alterações com os campos do Atendimento.

`Editar equipamento` abre modo/painel próprio dentro desta superfície, com salvamento explícito e conflito próprio.

A regra final de quem pode editar, quando pode alterar após conclusão e se determinadas alterações exigem tratamento histórico adicional pertence ao Bloco 9/10.

### 7.6 MACs

Um equipamento pode possuir múltiplos identificadores de rede.

Na UI:

- permitir listar múltiplos MACs;
- label opcional, como `Wi-Fi`, `Ethernet`, `Dock`;
- aceitar formatos comuns que o Host consiga normalizar com segurança;
- não usar MAC como identidade canônica do equipamento.

### 7.7 Saúde da bateria

Quando aplicável a notebook/dispositivo compatível:

- campo opcional;
- valor percentual de 0 a 100 quando informado;
- oculto/não obrigatório para equipamentos sem bateria relevante.

### 7.8 Procedimentos utilizados

A seção lista um ou mais procedimentos vinculados ao atendimento.

Cada item deve mostrar pelo menos:

- código snapshot;
- título snapshot;
- versão editorial utilizada quando disponível;
- revisão técnica utilizada;
- ação `Abrir revisão`.

A revisão vinculada não muda automaticamente quando o procedimento oficial receber nova versão.

### 7.9 Adicionar procedimento

Proposta: ação `Adicionar procedimento` abre busca de Processos e vincula uma revisão específica.

Se a Tela 09 foi aberta por `Iniciar atendimento` no Leitor, a revisão que estava sendo consultada pode vir pré-selecionada.

Qual revisão pode ser escolhida por cada perfil — publicada, atual, histórica — será fechado com permissões/regras operacionais no Bloco 9.

### 7.10 Resumo do trabalho

Campo de texto objetivo para descrever o que foi efetivamente realizado.

Ele não substitui os vínculos com procedimentos; complementa o registro com contexto específico do atendimento.

### 7.11 Observações

Campo livre para informações relevantes não cobertas pelos demais campos.

Não deve servir como depósito para dados estruturados que já possuem campo próprio.

### 7.12 Ficha / Imprimir

A Tela 09 deve possuir ponto de entrada contextual para a ficha compacta do atendimento/equipamento.

Neste Bloco 8 fica definido apenas o ponto de entrada de UX.

O Bloco 10 definirá:

- tamanho físico;
- conteúdo final;
- pré-visualização;
- impressão direta;
- PDF específico ou não;
- QR/barcode ou não;
- paginação e template.

## 8. Interações do usuário

Conforme capacidade/lifecycle futuro:

- consultar atendimento;
- editar campos do atendimento;
- salvar alterações explicitamente;
- vincular equipamento existente;
- cadastrar equipamento novo;
- editar equipamento vinculado;
- trocar/desvincular equipamento;
- adicionar/remover procedimentos;
- abrir revisão utilizada no Leitor;
- retornar à lista preservando contexto;
- acessar `Ficha / Imprimir`.

Nenhuma dessas ações recebe permissão definitiva neste Bloco 8.

## 9. Navegação e destinos

### Voltar

```text
Tela 09
→ Atendimentos
```

Preservar, quando possível:

- busca;
- filtros;
- ordenação;
- posição de rolagem/página.

### Abrir revisão utilizada

```text
Tela 09
→ Abrir revisão
→ Leitor em modo de revisão específica
```

O Leitor deve deixar claro quando a revisão é histórica.

### Ficha / Imprimir

```text
Tela 09
→ Ficha / Imprimir
→ futura UX da Tela 14
```

## 10. Estados da interface

### Carregando

Exibir estrutura estável com loading no conteúdo principal.

### Novo atendimento

Apresentar campos necessários à criação conforme regras do Bloco 9. O código legível é gerado pelo sistema, não digitado livremente pelo usuário.

### Atendimento existente

Exibir estado confirmado pelo Host e ações conforme capacidade.

### Sem equipamento

Mostrar estado simples e não ocupar espaço com ficha técnica vazia.

### Equipamento associado

Mostrar resumo/ficha técnica conforme dados disponíveis.

### Sem procedimentos vinculados

Mensagem curta:

`Nenhum procedimento vinculado a este atendimento.`

Ação `Adicionar procedimento` aparece apenas quando permitida.

### Erro de carregamento

`Não foi possível carregar este atendimento.`

Permitir tentar novamente quando apropriado.

### Host indisponível

Indicar indisponibilidade sem campos de IP/porta/path.

### Sem permissão

Não exibir dados ou ações não autorizados pelo Host.

### Registro indisponível/arquivado

Informar claramente quando o atendimento ou equipamento não puder mais receber determinadas ações. Regras definitivas de lifecycle ficam no Bloco 9.

### Alterações não salvas

Ao tentar sair com mudanças locais pendentes, solicitar confirmação antes de descartá-las.

### Conflito de Atendimento

Em `409 Conflict`:

- não sobrescrever automaticamente o estado mais recente;
- preservar alterações locais visíveis;
- informar que o atendimento foi alterado por outro usuário;
- oferecer reconsulta/reconciliação conforme contrato futuro.

### Conflito de Equipamento

Conflito no cadastro do equipamento deve ser tratado separadamente do Atendimento, preservando as alterações locais daquele formulário.

## 11. Validações

### Atendimento

- OS/referência pode ser vazia;
- cliente/solicitante pode ser vazio quando não aplicável;
- equipamento é opcional;
- pelo menos um responsável/técnico pode ser exigido conforme regra operacional futura;
- resumo e observações respeitam limites técnicos definidos na implementação;
- código interno/legível não é editado livremente.

### Equipamento

- nome pode ser exigido para novo equipamento, decisão final de Bloco 9/implementação;
- bateria, quando percentual, deve ficar entre 0 e 100;
- MAC deve ser normalizável quando informado;
- múltiplos MACs idênticos normalizados no mesmo equipamento não devem ser criados acidentalmente;
- serial/patrimônio/MAC não viram chave canônica exclusiva por inferência;
- campos não aplicáveis permanecem opcionais.

## 12. Mensagens e feedbacks

Exemplos curtos:

- `Atendimento salvo.`
- `Equipamento atualizado.`
- `Procedimento adicionado.`
- `Não foi possível salvar as alterações.`
- `Este atendimento foi alterado por outro usuário.`
- `Este equipamento foi alterado por outro usuário.`

Nunca expor stack trace, SQL, caminhos internos ou detalhes sensíveis.

## 13. Dados exibidos

### Atendimento

- código legível;
- OS/referência;
- cliente/solicitante;
- responsável;
- datas aplicáveis;
- resumo;
- observações.

### Equipamento

- código legível;
- nome/tipo;
- processador;
- RAM;
- armazenamento;
- SO/versão;
- serial;
- patrimônio;
- MACs;
- bateria;
- observações e vínculo de cliente/responsável quando disponível.

### Procedimentos

- código/título snapshot;
- versão snapshot;
- revisão efetivamente utilizada.

## 14. Dados enviados/alterados

Conceitualmente, a tela pode enviar mutações separadas para:

- criar/alterar Atendimento;
- vincular/desvincular Equipamento;
- criar/alterar Equipamento;
- adicionar/remover vínculo de Procedimento/Revisão.

Os limites transacionais e payloads finais não são definidos neste Bloco 8.

## 15. Regras de negócio

- Atendimento representa trabalho real;
- Equipamento é opcional e reutilizável;
- Atendimento pode utilizar múltiplos procedimentos;
- vínculo preserva revisão exata utilizada;
- alteração futura do procedimento não reescreve o atendimento histórico;
- MAC/serial/patrimônio são atributos pesquisáveis, não identidade principal;
- ficha técnica não é obrigatória para atendimentos sem equipamento;
- ficha compacta deriva do estado confirmado, não de screenshot;
- não criar lifecycle/status nesta tela por inferência;
- não criar checklist operacional persistente antes do Bloco 9;
- não transformar a tela em CRM, estoque, RMM ou help desk completo.

## 16. Regras de autorização

O Host decide:

- quem consulta;
- quem cria/edita Atendimento;
- quem vincula/troca equipamento;
- quem cria/edita/arquiva equipamento;
- quem adiciona/remove procedimento;
- quem acessa ficha/impressão.

A matriz final será consolidada no Bloco 9.

## 17. Impacto em persistência

Esta tela utiliza a extensão conceitual já aprovada:

- `service_records`;
- `equipment`;
- `equipment_network_identifiers`;
- `service_record_processes`.

Não cria migration nem schema novo neste Bloco 8.

## 18. Contratos Client ↔ Host necessários

Conceitualmente serão necessários contratos para:

- obter Atendimento por ID/código;
- criar/alterar Atendimento;
- pesquisar Equipamentos;
- obter Equipamento;
- criar/alterar Equipamento;
- vincular/desvincular Equipamento;
- listar Procedimentos/Revisões vinculados;
- adicionar/remover vínculo de Procedimento/Revisão;
- obter capacidades da sessão para cada ação.

Rotas, payloads, códigos de erro e atomicidade final serão definidos antes da implementação correspondente.

## 19. Eventos em tempo real relevantes

Eventos pós-commit conceituais podem indicar:

- Atendimento alterado;
- Equipamento alterado;
- procedimento/revisão vinculada ou removida;
- equipamento arquivado/indisponível.

Durante edição local:

- evento não substitui silenciosamente campos sendo editados;
- Client informa que existe alteração mais recente;
- próximo save pode encontrar conflito.

## 20. Comportamento de concorrência

Atendimento e Equipamento utilizam controle otimista equivalente ao restante do produto.

Regras de UX:

- nunca `last write wins` silencioso;
- preservar conteúdo local em conflito;
- informar qual recurso conflitou;
- reconsultar estado confirmado quando solicitado;
- não alterar automaticamente a revisão de procedimento já vinculada.

## 21. Acessibilidade e teclado

- labels visíveis para campos;
- ordem de Tab coerente;
- foco visível;
- botões icon-only com nome acessível;
- seções com headings semânticos;
- conflitos/erros não dependem apenas de cor;
- listas de MAC/procedimentos operáveis por teclado;
- modais/painéis contextuais com gerenciamento correto de foco.

## 22. Comportamento em tamanhos de janela suportados

Desktop Windows é o alvo principal.

Em janelas menores suportadas:

- campos em duas colunas podem passar para uma coluna;
- preservar código do atendimento e ações principais;
- ficha do equipamento quebra em linhas sem truncar identificadores importantes;
- procedimentos permanecem legíveis;
- não transformar automaticamente em layout mobile;
- evitar scroll horizontal sempre que possível.

## 23. Preservação visual / decisões de UI aprovadas

Manter:

- visual corporativo, limpo e discreto;
- sidebar global à esquerda;
- sem topbar global redundante;
- seções técnicas simples;
- densidade moderada;
- ações primárias limitadas;
- sem cards decorativos ou KPIs;
- identidade visual coerente com Lista de Atendimentos e Leitor.

## 24. Divergências com documentação anterior

Nenhuma divergência pretendida.

Esta tela materializa o domínio `Procedimento × Atendimento × Equipamento` já aprovado, sem antecipar lifecycle/checklist/permissões do Bloco 9 nem a tecnologia/layout final da ficha do Bloco 10.

## 25. Decisões consolidadas nesta análise

Nenhuma nova proposta abaixo é considerada consolidada antes de aprovação explícita do PO.

Já vêm consolidados de documentos superiores:

- Atendimento como ocorrência real;
- Equipamento opcional/reutilizável;
- identidade própria do equipamento;
- múltiplos MACs;
- múltiplos procedimentos por atendimento;
- vínculo à revisão exata utilizada;
- resumo + observações;
- ficha compacta imprimível como requisito;
- autorização Host-side.

## 26. Pendências para aprovação do PO nesta tela

1. usar uma única página vertical com seções `Atendimento`, `Equipamento` e `Procedimentos utilizados`, sem tabs obrigatórias;
2. reutilizar a mesma Tela 09 para novo atendimento e atendimento existente;
3. `Vincular equipamento` pesquisar existente antes de permitir cadastrar novo;
4. ficha técnica do equipamento mostrar somente campos preenchidos/aplicáveis;
5. edição do Equipamento ficar visualmente separada da edição do Atendimento, com salvamento/conflito próprio;
6. listar múltiplos MACs com label opcional;
7. procedimentos vinculados exibirem versão editorial + revisão técnica utilizada;
8. `Abrir revisão` levar ao Leitor na revisão específica;
9. se iniciado pelo Leitor, pré-selecionar a revisão consultada;
10. manter `Resumo do trabalho` e `Observações` como campos distintos;
11. expor ação contextual `Ficha / Imprimir`, delegando o fluxo final à Tela 14/Bloco 10;
12. não mostrar/definir `Status`, conclusão, reabertura ou checklist persistente antes do Bloco 9.

## 27. Fora do escopo

- lifecycle/status final do Atendimento;
- botão/semântica definitiva de concluir/reabrir;
- persistência de checklist/progresso;
- regra de edição após conclusão;
- matriz final de permissões;
- formato exato de `AT-...` e `EQP-...`;
- tamanho/layout físico da ficha;
- geração de PDF/print;
- QR/barcode;
- descoberta automática de hardware;
- estoque/peças;
- SLA/fila de chamados;
- CRM/financeiro.

## 28. Critérios de aceite

- [ ] PO aprova a estrutura vertical da Tela 09;
- [ ] PO aprova uso da mesma tela para novo/existente;
- [ ] PO aprova fluxo de vincular/pesquisar/cadastrar Equipamento;
- [ ] PO aprova ficha técnica com campos condicionais;
- [ ] PO aprova edição separada do Equipamento;
- [ ] PO aprova exibição de múltiplos MACs;
- [ ] PO aprova lista de Procedimentos com revisão exata;
- [ ] PO aprova `Resumo do trabalho` separado de `Observações`;
- [ ] PO aprova ponto de entrada `Ficha / Imprimir`;
- [ ] lifecycle/checklist não foram inventados;
- [ ] equipamento permanece opcional;
- [ ] alteração de procedimento não muda revisão vinculada;
- [ ] conflitos não sobrescrevem silenciosamente;
- [ ] nenhuma implementação funcional foi criada.

## 29. Casos de teste/smoke sugeridos

Quando houver implementação:

- abrir atendimento existente sem equipamento;
- abrir atendimento existente com equipamento;
- criar atendimento sem equipamento;
- iniciar atendimento a partir do Leitor;
- vincular equipamento existente por código;
- buscar equipamento por serial/patrimônio/MAC;
- cadastrar equipamento novo;
- editar CPU/RAM/armazenamento/SO;
- registrar múltiplos MACs;
- registrar bateria válida e rejeitar percentual inválido;
- adicionar dois procedimentos;
- abrir revisão vinculada no Leitor;
- procedimento receber versão nova sem alterar vínculo existente;
- salvar Atendimento;
- salvar Equipamento;
- conflito de Atendimento;
- conflito de Equipamento;
- sair com alterações não salvas;
- Host indisponível;
- usuário sem capacidade de edição;
- atendimento sem cliente/OS;
- equipamento com campos técnicos parciais;
- acessar ponto `Ficha / Imprimir` quando permitido.