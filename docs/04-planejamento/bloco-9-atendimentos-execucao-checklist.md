# Bloco 9 — Execução Operacional / Atendimentos + Checklist

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Consolidação:** 2026-08-28

## 1. Objetivo

Fechar as regras operacionais de `Atendimento/Execução`, `Equipamento`, checklist persistente e observações de serviço, sem transformar o StepFlow em sistema burocrático de chamados.

Este bloco consolida:

- lifecycle mínimo do Atendimento;
- criação, edição, conclusão, cancelamento e reabertura;
- responsável/técnico;
- Equipamento opcional e reutilizável;
- vínculo com revisões exatas de Procedimentos;
- checklist persistente em contexto de execução;
- observação de serviço opcional por Etapa;
- progresso operacional;
- histórico operacional proporcional;
- concorrência específica;
- matriz operacional de capacidades/presets;
- códigos legíveis de Atendimento e Equipamento;
- preset para gestão de categorias;
- capacidade e lifecycle de `Ficha / Imprimir`.

Não pertence a este bloco fechar engine documental, template físico final da ficha, backup/restore técnico nem scaffold/runtime oficial.

## 2. Contratos herdados

Permanecem consolidados:

- `Procedimento`, `Atendimento/Execução` e `Equipamento` são entidades distintas;
- Equipamento é opcional e reutilizável;
- Atendimento pode existir sem Equipamento;
- Atendimento pode usar zero, um ou vários Procedimentos;
- cada vínculo de Procedimento preserva a revisão efetivamente utilizada;
- revisões de Procedimento são imutáveis;
- Atendimento e Equipamento são dados oficiais do Host;
- Clients nunca acessam SQLite diretamente;
- concorrência relevante usa controle otimista;
- ficha usa somente estado confirmado/histórico aplicável do Host;
- ficha pode existir com ou sem Equipamento;
- ficha permanece limitada a uma página A4;
- Reader standalone continua documental;
- checklist e observações de serviço persistentes só existem em contexto operacional de Atendimento;
- nenhuma edição offline, fila local persistente ou autosave é introduzida.

## 3. Lifecycle mínimo

A primeira versão usa somente três estados visíveis:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
novo Atendimento
      ↓ primeiro save aceito
Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído ─→ Reabrir ─→ Em andamento
Cancelado ─→ Reabrir ─→ Em andamento
```

Não criar inicialmente `Novo`, `Aguardando`, `Pausado`, `Em aprovação`, `Resolvido`, `Fechado`, SLA ou equivalentes sem novo requisito explícito.

Toda transição é autorizada pelo Host, auditável e sujeita à revisão/base esperada.

## 4. Criação do Atendimento

### 4.1 Antes do primeiro save

`Novo atendimento` ou `Iniciar atendimento` pelo Leitor abre rascunho somente em memória no Client.

- nenhum registro é criado apenas por abrir a tela;
- código legível ainda não existe;
- se o fluxo veio do Leitor, a revisão consultada pode vir pré-selecionada;
- sair antes do primeiro save descarta o rascunho mediante proteção de alterações não salvas;
- não existe draft persistente inicial.

### 4.2 Primeiro save aceito

O Host:

- cria `service_record_id` estável;
- gera `service_code` legível;
- define `Em andamento`;
- define `started_at` no Host;
- registra criador;
- define responsável/técnico conforme capacidade;
- retorna estado confirmado ao Client;
- publica evento pós-commit quando aplicável.

## 5. Responsável / técnico

O Atendimento deve possuir responsável antes da conclusão.

- Funcionário cria Atendimento inicialmente atribuído a si mesmo;
- Funcionário no preset padrão não atribui Atendimento a outro usuário;
- ADM/Gerência podem atribuir ou reatribuir para usuário ativo adequado;
- usuário desativado continua identificável no histórico, mas não é opção normal para nova atribuição;
- mudança de responsável é auditável.

## 6. Edição por estado

### Em andamento

Pode receber, conforme capacidade:

- OS/referência;
- cliente/solicitante;
- responsável;
- resumo do trabalho;
- observações gerais;
- Equipamento vinculado;
- Procedimentos/revisões vinculados;
- checklist/progresso;
- observações de serviço por Etapa no Reader operacional.

### Concluído

É estado histórico bloqueado para edição operacional direta.

Permitido:

- consultar;
- abrir revisões utilizadas;
- consultar checklist final;
- consultar observações de serviço registradas;
- gerar/reimprimir ficha conforme capacidade;
- reabrir quando autorizado.

Qualquer mudança operacional exige `Reabrir` primeiro.

### Cancelado

É estado histórico/inativo.

- não recebe edição operacional direta;
- permanece pesquisável/consultável conforme autorização;
- não é excluído fisicamente pela operação normal;
- pode ser reaberto quando autorizado;
- ficha, quando gerada/reimpressa por sessão autorizada, identifica claramente `Cancelado`.

## 7. Conclusão

`Concluir atendimento` existe somente para Atendimento `Em andamento` e sessão autorizada.

Validações mínimas:

- responsável definido;
- `Resumo do trabalho` preenchido com conteúdo válido;
- nenhum conflito pendente;
- nenhuma alteração local não salva que faça a UI divergir do estado a concluir;
- base/revisão ainda atual.

Não são obrigatórios para conclusão:

- OS/referência externa;
- cliente/solicitante;
- Equipamento;
- Procedimento vinculado;
- checklist, quando a revisão utilizada não possuir itens;
- observação de serviço por Etapa.

### 7.1 Checklist incompleto

Checklist incompleto não bloqueia automaticamente a conclusão, pois a definição documental inicial não distingue item obrigatório de opcional.

Quando houver itens desmarcados:

```text
Ainda existem itens de checklist não concluídos.
Deseja concluir este atendimento mesmo assim?

[ Continuar execução ] [ Concluir mesmo assim ]
```

A decisão é explícita; não há conclusão automática.

### 7.2 Estado final

Ao concluir, o Host:

- revalida autorização e revisão;
- grava `Concluído`;
- define `completed_at`;
- preserva revisões de Procedimentos utilizadas;
- preserva o estado final dos checklists;
- preserva o estado final aplicável das observações de serviço por Etapa;
- congela a projeção histórica relevante do Equipamento para que alterações futuras do cadastro global não reescrevam o Atendimento/ficha final;
- registra evento de conclusão;
- publica evento pós-commit.

Cada nova conclusão após reabertura produz novo evento e novo estado final aplicável. Conclusões anteriores não desaparecem nem podem ter sua ficha histórica reescrita silenciosamente.

## 8. Cancelamento

`Cancelar atendimento` é usado para registro criado por engano, trabalho abortado ou ocorrência que não deve ser tratada como concluída.

- somente em `Em andamento`;
- exige capacidade própria;
- exige motivo curto;
- preserva código, vínculos e histórico;
- registra ator e data/hora;
- bloqueia edição operacional direta;
- pode ser reaberto por sessão autorizada;
- não há exclusão física normal de Atendimento.

## 9. Reabertura

`Reabrir atendimento` retorna `Concluído` ou `Cancelado` para `Em andamento`.

- ação explícita e auditável;
- ADM/Gerência recebem por preset;
- Funcionário não recebe por preset;
- preserva vínculos, checklist, observações de serviço e conteúdo existentes;
- histórico mantém conclusões/cancelamentos anteriores;
- volta a aceitar edição conforme capacidades normais;
- nova conclusão grava novo estado final;
- alterações posteriores não reescrevem silenciosamente o estado final anterior.

Não existe edição escondida de Atendimento histórico sem reabertura.

## 10. Procedimentos utilizados

### 10.1 Vinculação

Enquanto `Em andamento`, usuário autorizado pode adicionar/remover Procedimentos.

Cada vínculo preserva:

- `process_id`;
- `process_revision_id` exato;
- código snapshot;
- título snapshot;
- versão editorial snapshot.

### 10.2 Revisões selecionáveis

- Funcionário: normalmente apenas revisão **publicada** que possa ler;
- ADM/Gerência: publicada por padrão e, explicitamente, outra revisão histórica/atual não publicada que já possam ler;
- revisão não publicada/histórica nunca é escolhida silenciosamente;
- revisão vinculada não muda quando outra revisão é publicada depois.

### 10.3 Remoção

Permitida somente em `Em andamento` e conforme capacidade.

Se houver checklist marcado ou observação de serviço registrada para aquele vínculo, a UI exige confirmação explícita. O vínculo deixa a composição ativa somente após ação consciente; histórico/auditoria necessários são preservados.

## 11. Reader em contexto de execução

O Reader possui dois contextos.

### Consulta documental

```text
Processos → Leitor
```

- checklist é definição documental;
- não persiste estado operacional;
- não existe observação de serviço persistente;
- `Etapa X de Y`/stepper representam navegação, não progresso.

### Execução vinculada ao Atendimento

```text
Atendimento AT-...
→ Procedimento vinculado
→ Abrir revisão / Executar
→ Leitor da revisão exata no contexto do Atendimento
```

Nesse contexto:

- cabeçalho identifica `Executando no atendimento AT-...`;
- a revisão continua presa ao vínculo;
- itens de checklist usam estado persistente daquele Atendimento;
- cada Etapa pode receber `Observação do serviço` opcional;
- navegação entre etapas não altera conclusão/progresso;
- voltar retorna ao Atendimento preservando contexto.

## 12. Checklist operacional

### 12.1 Origem

Ao vincular revisão de Procedimento, o Host identifica os itens dos blocos `checklist` e associa/cria o estado operacional daquele vínculo no Atendimento.

O estado operacional nunca modifica a revisão documental.

### 12.2 Estado mínimo por item

Conceitualmente:

```text
execution_checklist_item_id
service_record_process_id
origem: revisão/bloco/item
text_snapshot quando necessário
is_checked
checked_at NULL
checked_by_user_id NULL
row_revision/equivalente
```

O schema físico final pode variar desde que preserve essa semântica e concorrência segura.

### 12.3 Marcar/desmarcar

Enquanto `Em andamento` e autorizado:

- pode marcar/desmarcar;
- Host confirma a alteração;
- Client reconcilia estado confirmado;
- conflito nunca sobrescreve silenciosamente estado mais novo.

Em `Concluído`/`Cancelado`, checklist fica somente leitura até eventual reabertura.

### 12.4 Progresso

Deriva exclusivamente de checklist persistente.

```text
Procedimento PR-001   4 de 6 itens
Procedimento PR-022   2 de 2 itens
Atendimento           6 de 8 itens
```

- etapas visitadas não contam como progresso;
- observações de serviço não contam como progresso;
- `Etapa X de Y` não é percentual de execução;
- revisão sem checklist não mostra `0%` artificial;
- percentual visual, se usado, deriva de marcados/total;
- 100% não conclui Atendimento automaticamente.

## 13. Observação do serviço por Etapa

Durante a execução, o técnico pode registrar uma nota simples relacionada à Etapa atual.

Contrato:

```text
Atendimento
+ service_record_process
+ Etapa da revisão exata
→ Observação do serviço opcional
```

- pertence à execução, não ao Procedimento oficial;
- não cria nova revisão do Procedimento;
- texto vazio é omitido da Ficha;
- pode ser alterado apenas enquanto o Atendimento estiver editável e autorizado;
- fica somente leitura em `Concluído`/`Cancelado` até reabertura;
- precisa de controle concorrente por Etapa/equivalente;
- evento remoto não sobrescreve texto local em edição silenciosamente;
- não é chat/comentário social;
- não há autosave implícito como requisito.

Observações preenchidas podem compor a Ficha como evidência resumida do serviço realizado.

## 14. Histórico operacional

Preservar ao menos eventos de alto valor:

- Atendimento criado;
- responsável alterado;
- Equipamento vinculado/trocado/desvinculado;
- Procedimento adicionado/removido;
- Atendimento concluído;
- Atendimento reaberto;
- Atendimento cancelado e motivo;
- alterações administrativas relevantes.

Não é necessário transformar cada campo, checkbox ou observação de Etapa em uma timeline visível. Auditoria técnica pode registrar detalhes adicionais de forma proporcional.

## 15. Equipamento

### 15.1 Cadastro e edição

Preset inicial:

- ADM/Gerência/Funcionário podem criar e atualizar Equipamento;
- arquivar/reativar Equipamento: ADM/Gerência;
- capacidades continuam granulares/personalizáveis dentro dos limites de delegação.

### 15.2 Arquivamento

- Equipamento arquivado não aparece para novo vínculo normal;
- histórico permanece;
- não arquivar enquanto estiver vinculado a Atendimento `Em andamento`;
- antes disso, é necessário concluir/cancelar o Atendimento ou desvincular;
- reativação depende de capacidade.

### 15.3 Histórico após conclusão

Alterar cadastro global depois da conclusão não altera silenciosamente o histórico/ficha final do Atendimento concluído.

A conclusão congela a projeção relevante do Equipamento. A forma física de persistência será fechada antes da implementação correspondente, sem criar tabela meramente de apresentação.

## 16. Códigos legíveis

Formato inicial consolidado:

```text
Atendimento: AT-000001
Equipamento: EQP-000001
```

- gerados somente pelo Host;
- sequência numérica simples por implantação/banco ativo;
- seis dígitos com zero à esquerda;
- gaps permitidos;
- não editáveis pelo usuário;
- não substituem IDs internos estáveis;
- cancelamento mantém o código;
- busca aceita código como referência operacional principal;
- sem ano, departamento, hostname ou dados pessoais no código inicial.

## 17. Lista de Atendimentos

A Tela 08 usa o lifecycle consolidado:

- coluna `Status`: `Em andamento`, `Concluído`, `Cancelado`;
- filtro por Status;
- `Período` usa `started_at`;
- coluna `Data` representa `started_at`;
- ordenação padrão: `started_at` mais recente primeiro;
- busca/filtros continuam combináveis e preservados no retorno.

## 18. Matriz operacional de capacidades

A autorização é Host-side; presets são defaults.

| Capacidade operacional | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar Atendimentos | sim | sim | sim |
| Criar Atendimento | sim | sim | sim |
| Editar Atendimento em andamento do qual é responsável | sim | sim | sim |
| Editar qualquer Atendimento em andamento | sim | sim | não |
| Concluir Atendimento do qual é responsável | sim | sim | sim |
| Concluir qualquer Atendimento | sim | sim | não |
| Cancelar Atendimento | sim | sim | não |
| Reabrir Atendimento | sim | sim | não |
| Vincular/trocar/desvincular Equipamento em Atendimento editável | sim | sim | sim, quando responsável |
| Criar/editar cadastro de Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento em Atendimento editável | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica para execução | sim | sim | não |
| Marcar/desmarcar checklist em Atendimento editável | sim | sim | sim, quando responsável |
| Registrar/editar observação de serviço por Etapa | sim | sim | sim, quando responsável |
| Gerar/reimprimir ficha de Atendimento acessível | sim | sim | sim |
| Gerir categorias de Procedimentos | sim | sim | não |

Funcionário significa o preset padrão. Capacidades personalizadas continuam possíveis dentro dos limites de delegação já consolidados.

Permanecem PENDENTES e não são alterados por este bloco:

- Gerência × configuração da empresa;
- Gerência × Backup.

## 19. Ficha / Imprimir — lifecycle e finalidade

- exige capacidade e acesso ao Atendimento;
- em `Em andamento`, pode ser gerada para acompanhamento a partir do estado confirmado do Host;
- em `Concluído`, pode ser reimpressa usando o estado histórico congelado aplicável;
- em `Cancelado`, quando gerada/reimpressa, identifica inequivocamente `Cancelado`;
- alterações não salvas/conflitos bloqueiam a geração;
- Funcionário recebe capacidade por preset para Atendimentos que pode consultar.

A Ficha é uma **prestação de contas resumida ao cliente**. Prioriza identificação do serviço/dispositivo, características relevantes do Equipamento, resumo do trabalho e observações relevantes — incluindo observações registradas nas Etapas.

Checklist, percentual/progresso, passos, comandos, timeline e lista detalhada de revisões permanecem no histórico interno e não aparecem por padrão na folha.

Bloco 10 / Etapa 6 consolidou PDF próprio da Ficha e preview derivado do mesmo layout; Etapas 7–9 ainda fecham template físico, limites/densidade e casos de múltiplos dados na única A4.

## 20. Concorrência operacional

### Atendimento

- mutações carregam base/revisão esperada;
- concluir/cancelar/reabrir são mutações versionadas;
- writer/fila define ordem, não validade;
- conflito preserva alterações locais e exige reconciliação.

### Checklist

- controle atômico/versionado por item ou equivalente;
- usuários podem marcar itens diferentes sem conflito global desnecessário;
- alteração concorrente do mesmo item tem resultado determinístico/conflito apropriado.

### Observação de serviço

- controle versionado por Etapa/equivalente;
- Etapas diferentes não conflitam globalmente;
- alteração concorrente da mesma observação exige reconciliação proporcional;
- não sobrescrever texto local em edição por evento remoto.

### Equipamento

Cadastro global tem revisão própria, separada da revisão do Atendimento.

## 21. Eventos em tempo real

Eventos pós-commit podem sinalizar:

- Atendimento criado/alterado;
- status alterado;
- responsável alterado;
- Equipamento vinculado/alterado;
- Procedimento vinculado/removido;
- checklist alterado;
- observação de serviço por Etapa alterada;
- conclusão/reabertura/cancelamento.

Client reconsulta o estado relevante e não sobrescreve edição local silenciosamente.

## 22. Impacto conceitual no modelo de dados

Sem criar migration neste bloco documental, a implementação futura precisa refletir:

- status consolidado;
- motivo/evento de cancelamento;
- eventos operacionais de lifecycle;
- estado de checklist por `service_record_process`;
- observação de serviço vinculada ao `service_record_process` + Etapa;
- concorrência granular dessas observações;
- preservação histórica do estado final aplicável das observações após reabertura;
- snapshot/projeção histórica de Equipamento por conclusão;
- geração segura de `AT-000001` / `EQP-000001`;
- capacidades operacionais granulares.

Árvore física/migrations oficiais permanecem para o gate correspondente antes do código.

## 23. Pendências fora do Bloco 9

### Bloco 10

Após Etapa 6, permanecem:

- template A4 final da Ficha;
- margens/tipografia/densidade;
- limites numéricos finais de resumo/observações;
- regras físicas de priorização/tratamento de excesso;
- tratamento de muitos MACs/Procedimentos/dados excepcionais;
- nomes de arquivo + temporários;
- QR/barcode se houver benefício;
- validação técnica final.

### Autenticação/configuração

- Argon2id exato;
- senha mínima final;
- duração de sessão;
- Gerência × configuração da empresa;
- Gerência × Backup.

### Categorias/editor

- regra editorial exata ao criar nova revisão de Procedimento que ainda referencie categoria arquivada.

### Ambiente corporativo

- hostname/porta;
- Windows/WebView2;
- SMB/permissões;
- HTTP/HTTPS;
- EDR/firewall;
- start real do Controller.

## 24. Fora do escopo

- SLA;
- fila complexa de chamados;
- prioridade/severidade obrigatória;
- aprovação gerencial de conclusão;
- atribuição por equipe/skill;
- chat social;
- apontamento de horas complexo;
- financeiro/faturamento;
- estoque/peças;
- RMM/inventário automático;
- workflow customizável;
- checklist com lógica condicional avançada;
- modo offline editável;
- autosave;
- implementação funcional.

## 25. Decisões consolidadas

1. lifecycle `Em andamento → Concluído/Cancelado`, com reabertura explícita;
2. primeiro save cria Atendimento e gera código; abrir tela não persiste registro;
3. `Resumo do trabalho` é obrigatório para conclusão;
4. checklist incompleto gera aviso, mas não bloqueia automaticamente;
5. cancelamento exige motivo curto e não exclui registro;
6. conclusão/cancelamento bloqueiam edição direta; mudança exige reabertura;
7. ADM/Gerência podem reabrir por preset; Funcionário não;
8. Funcionário edita/conclui por preset apenas Atendimento do qual é responsável;
9. Funcionário pode criar/editar Equipamento; arquivamento fica ADM/Gerência;
10. Gerência gere categorias por preset; configuração da empresa/Backup continuam pendentes;
11. Funcionário seleciona revisão publicada; ADM/Gerência podem selecionar explicitamente outras revisões legíveis/autorizadas;
12. Reader em contexto de Atendimento persiste checklist; Reader standalone continua documental;
13. cada Etapa em execução pode persistir `Observação do serviço` opcional, sem alterar o Procedimento;
14. progresso deriva somente de checklist marcado/total;
15. checklist não conclui Atendimento automaticamente;
16. Equipamento global alterado depois da conclusão não reescreve histórico/ficha final;
17. observações de serviço também precisam preservar estado histórico aplicável após conclusão/reabertura;
18. códigos `AT-000001` e `EQP-000001`;
19. Status entra na Tela 08; Data/Período usam `started_at`, com mais recente primeiro;
20. ficha pode ser gerada em `Em andamento`, reimpressa em `Concluído` e identificada como cancelada quando aplicável;
21. ficha é prestação de contas resumida ao cliente;
22. checklist e observações usam concorrência granular por recurso/equivalente;
23. nenhum workflow completo de chamados/SLA é criado.

## 26. Critérios de aceite — concluídos

- [x] lifecycle aprovado;
- [x] criação/edição/conclusão/cancelamento/reabertura aprovadas;
- [x] regra de responsável aprovada;
- [x] revisão selecionável por perfil aprovada;
- [x] checklist persistente/progresso aprovados;
- [x] observação de serviço por Etapa aprovada;
- [x] histórico operacional aprovado;
- [x] regra de Equipamento após conclusão aprovada;
- [x] códigos legíveis aprovados;
- [x] matriz operacional aprovada;
- [x] gestão de categorias por preset aprovada;
- [x] lifecycle/capacidade/finalidade da ficha aprovados;
- [x] concorrência operacional aprovada;
- [x] nenhuma implementação funcional criada.