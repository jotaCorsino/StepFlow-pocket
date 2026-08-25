# Tela 09 — Atendimento / Execução + Equipamento

## 1. Identificação

- código/nome da tela: Tela 09 — Atendimento / Execução + Equipamento;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco original: Fase 1 — Bloco 8 (UI/UX);
- atualização operacional: Bloco 9;
- última consolidação: 2026-08-25.

## 2. Objetivo

Ser o workspace principal de um Atendimento real, reunindo numa única página vertical:

- lifecycle/status;
- dados básicos do Atendimento;
- responsável/técnico;
- cliente/solicitante e OS/referência opcionais;
- Equipamento opcional;
- Procedimentos/revisões utilizados;
- progresso/checklist operacional;
- resumo do trabalho;
- observações;
- histórico operacional compacto;
- `Ficha / Imprimir`.

A tela não transforma o StepFlow em CRM, estoque, RMM ou sistema completo de chamados.

## 3. Acesso e capacidades

Preset inicial:

- ADM: consulta/criação/gestão ampla;
- Gerência: consulta/criação/gestão ampla, sem administrar ADM;
- Funcionário: consulta/criação e edição/conclusão do Atendimento do qual é responsável.

Autorização real permanece no Host.

A tela pode estar:

- em rascunho de novo Atendimento;
- `Em andamento`;
- `Concluído`;
- `Cancelado`.

## 4. Entrada na tela

```text
Atendimentos
→ selecionar registro
→ Tela 09
```

ou:

```text
Atendimentos
→ Novo atendimento
→ Tela 09 em rascunho somente em memória
```

ou:

```text
Leitor de Processo
→ Iniciar atendimento
→ Tela 09 em rascunho
→ revisão consultada pré-selecionada quando elegível
```

Abrir a tela em modo novo não cria registro oficial. O primeiro save aceito pelo Host cria o Atendimento.

## 5. Lifecycle consolidado

```text
rascunho local
    ↓ primeiro save aceito
Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
   ↓ Reabrir
Em andamento
```

Estados visíveis iniciais:

- `Em andamento`;
- `Concluído`;
- `Cancelado`.

Não criar `Aguardando`, `Pausado`, `Resolvido`, SLA etc. sem novo requisito.

## 6. Layout consolidado

Exemplo em `Em andamento`:

```text
← Atendimentos

Atendimento AT-00142                    Em andamento
OS-4587 · João Silva · Responsável: Maria

[ Ficha / Imprimir ]                    [ Salvar ]
[ Concluir atendimento ]                [ Mais ▾ ]

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
EQUIPAMENTO                                              [ Editar ]

NOTE-15 · EQP-0031                                 Notebook

Nome               NOTE-15
Tipo               Notebook
Processador         Intel Core i5-1135G7
RAM                 16 GB
Armazenamento       SSD NVMe 512 GB
Sistema             Windows 11 Pro · 24H2
Serial              ABC123
Patrimônio          PAT-884
MAC                 A0:B1:C2:D3:E4:F5
Bateria             82%
Observações         Texto curto...

[ Trocar/Vincular equipamento ] [ Desvincular ]

────────────────────────────────────────────────────────────
PROCEDIMENTOS UTILIZADOS

PR-001  Manutenção preventiva
Versão 1.3 · revisão r18                  4 de 6 itens
[ Executar ] [ Abrir revisão ] [ Remover ]

PR-022  Substituição de SSD
Versão 2.0 · revisão r7                   2 de 2 itens
[ Executar ] [ Abrir revisão ] [ Remover ]

[ + Adicionar procedimento ]

────────────────────────────────────────────────────────────
PROGRESSO DO ATENDIMENTO
6 de 8 itens concluídos

────────────────────────────────────────────────────────────
HISTÓRICO
25/08 10:21 · Atendimento criado · Maria
25/08 10:28 · Equipamento EQP-0031 vinculado · Maria
```

Ações aparecem somente quando estado + capacidade permitirem.

## 7. Novo Atendimento

Antes do primeiro save:

- sem `AT-...` definitivo;
- rascunho somente em memória;
- se veio do Reader, revisão elegível pode vir pré-selecionada;
- Funcionário começa como responsável por padrão;
- sair com alterações pede confirmação;
- sair descarta rascunho; não existe draft persistente inicial.

No primeiro save aceito:

- Host cria `service_record_id`;
- gera `AT-000001`/sequência aplicável;
- define `started_at`;
- define `Em andamento`;
- registra criador/responsável;
- devolve estado confirmado.

## 8. Códigos

Atendimento:

```text
AT-000001
```

Equipamento:

```text
EQP-000001
```

Regras:

- Host-only;
- seis dígitos;
- gaps permitidos;
- não editáveis;
- não substituem IDs internos.

## 9. Dados do Atendimento

Campos principais:

- código legível — somente leitura;
- Status — derivado do lifecycle, não dropdown livre;
- OS/referência externa opcional;
- cliente/solicitante opcional;
- responsável/técnico;
- `started_at`/datas de lifecycle quando relevantes;
- resumo do trabalho;
- observações.

`Resumo do trabalho` é obrigatório para conclusão, não necessariamente para primeiro save.

## 10. Responsável

- Atendimento precisa de responsável para concluir;
- Funcionário cria inicialmente para si;
- Funcionário preset não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário inativo não é escolha normal para nova atribuição;
- histórico continua identificando usuários antigos;
- troca de responsável é auditável.

## 11. Edição por estado

### Em andamento

Conforme capacidade, pode alterar:

- OS/Ref.;
- cliente;
- responsável;
- resumo;
- observações;
- vínculo de Equipamento;
- cadastro do Equipamento em fluxo separado;
- Procedimentos/revisões;
- checklist operacional.

### Concluído

Somente leitura operacional.

Permitido conforme capacidade:

- consultar;
- abrir revisões usadas;
- consultar checklist final;
- reimprimir ficha;
- reabrir.

Qualquer correção operacional exige `Reabrir`.

### Cancelado

Somente leitura operacional.

- permanece consultável/pesquisável;
- não é excluído;
- pode ser reaberto por ADM/Gerência por preset;
- ficha, quando acessada, identifica `Cancelado`.

## 12. Salvar

Salvamento é explícito; não existe autosave inicial.

- Host valida capacidade, estado e `row_revision`/base;
- sucesso só aparece após confirmação de commit;
- conflito preserva alterações locais;
- evento remoto não substitui formulário;
- resultado incerto após desconexão é reconciliado, não repetido cegamente.

## 13. Concluir Atendimento

Disponível apenas em `Em andamento` e conforme capacidade.

Preset:

- ADM/Gerência: podem concluir qualquer Atendimento acessível;
- Funcionário: pode concluir Atendimento do qual é responsável.

Pré-condições:

- responsável definido;
- `Resumo do trabalho` válido;
- estado confirmado/sem conflito;
- alterações locais relevantes salvas.

Não são obrigatórios por si só:

- OS;
- cliente;
- Equipamento;
- Procedimento vinculado.

### Checklist incompleto

Se houver itens desmarcados:

```text
Ainda existem itens de checklist não concluídos.
Deseja concluir este atendimento mesmo assim?

[ Continuar execução ] [ Concluir mesmo assim ]
```

Não bloquear automaticamente; não existe semântica obrigatório/opcional nos checklists documentais iniciais.

### Ao concluir

Host:

- grava `Concluído`;
- define `completed_at`;
- preserva revisões usadas/checklist final;
- congela projeção relevante do Equipamento;
- registra evento;
- publica pós-commit.

## 14. Cancelar Atendimento

Ação contextual, por exemplo em `Mais`.

Preset:

- ADM/Gerência: sim;
- Funcionário: não.

Regras:

- apenas `Em andamento`;
- motivo curto obrigatório;
- confirmação explícita;
- preserva código/vínculos/histórico;
- não exclui;
- bloqueia edição após commit.

## 15. Reabrir

Preset:

- ADM/Gerência: sim;
- Funcionário: não.

Disponível em `Concluído` ou `Cancelado`.

- ação explícita;
- auditável;
- volta para `Em andamento`;
- preserva histórico anterior;
- checklist/vínculos continuam disponíveis;
- nova conclusão grava novo estado final.

## 16. Equipamento opcional

A tela funciona com ou sem Equipamento.

Sem Equipamento:

```text
Nenhum equipamento vinculado a este atendimento.
[ Vincular equipamento ]
```

Não renderizar ficha técnica vazia.

## 17. Vincular Equipamento

Busca por:

- código `EQP-...`;
- nome;
- serial;
- patrimônio;
- MAC;
- cliente/solicitante relacionado quando aplicável.

Fluxo:

```text
Vincular equipamento
→ pesquisar cadastro existente
→ selecionar
ou
→ Cadastrar novo equipamento, se autorizado
```

Evitar duplicação silenciosa baseada apenas em MAC/serial/patrimônio.

## 18. Cadastro do Equipamento

Campos para computadores, conforme aplicável:

- nome;
- tipo;
- processador;
- RAM;
- armazenamento;
- SO/versão;
- serial;
- patrimônio;
- um ou mais MACs com label opcional;
- saúde da bateria;
- cliente/responsável relacionado;
- observações curtas.

Tipos mínimos:

- Servidor;
- Desktop;
- Notebook.

Não transformar isso em enum global rígida.

### Bateria

- contextual para Notebook;
- opcional;
- percentual 0–100 quando informado;
- não aparece como obrigação para Servidor/Desktop.

### MACs

- múltiplos;
- labels opcionais como Wi-Fi/Ethernet/Dock;
- Host normaliza formatos;
- MAC não é identidade canônica.

### Observações

- texto curto;
- limite explícito;
- valor numérico final no Bloco 10 por causa da ficha A4.

## 19. Capacidades de Equipamento

Preset:

- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- Funcionário pode vincular/trocar/desvincular em Atendimento editável do qual é responsável.

Equipamento possui salvamento/conflito separado do Atendimento.

Não arquivar Equipamento vinculado a Atendimento `Em andamento`.

## 20. Histórico do Equipamento no Atendimento concluído

Alteração futura no cadastro global não reescreve o estado histórico/ficha final já concluídos.

Ao concluir, Host congela a projeção relevante do Equipamento.

Após reabertura e nova conclusão, um novo estado final pode ser capturado; o histórico anterior não desaparece.

## 21. Procedimentos utilizados

Cada item mostra:

- código snapshot;
- título snapshot;
- versão editorial usada;
- revisão técnica exata;
- progresso de checklist quando houver;
- ações contextuais.

A revisão nunca é atualizada automaticamente após nova publicação.

## 22. Adicionar Procedimento

Enquanto `Em andamento` e autorizado:

- abrir busca de Processos;
- selecionar revisão elegível;
- vincular explicitamente.

Por preset:

- Funcionário: revisão publicada que possa ler;
- ADM/Gerência: publicada por padrão; podem selecionar explicitamente histórica/não publicada autorizada.

Se Atendimento veio do Reader, a revisão consultada pode estar pré-selecionada conforme essas regras.

## 23. Remover Procedimento

Somente `Em andamento` + capacidade.

Se houver checklist marcado:

- mostrar confirmação;
- remover da composição ativa somente após ação consciente;
- preservar auditoria da remoção.

## 24. Executar Procedimento / Checklist

Ação `Executar` abre Tela 05 na revisão exata e contexto do Atendimento.

```text
Atendimento AT-00142
→ PR-001 r18
→ Executar
→ Reader: "Executando no atendimento AT-00142"
```

Checklist nesse contexto é persistente.

Em Reader standalone, continua documental.

## 25. Progresso

Por Procedimento e agregado do Atendimento:

```text
PR-001        4 de 6
PR-022        2 de 2
Atendimento   6 de 8
```

- deriva apenas de checklist;
- etapas visitadas não contam;
- revisão sem checklist não mostra 0%;
- 100% não conclui automaticamente.

## 26. Concorrência do checklist

- controle granular por item/equivalente;
- usuários marcando itens diferentes não conflitam globalmente;
- mesmo item concorrente recebe resultado determinístico/conflito apropriado;
- não usar revisão global do Atendimento para invalidar todo checkbox por conveniência.

## 27. Histórico operacional compacto

Eventos de alto valor podem aparecer numa área/painel compacto:

- criado;
- responsável alterado;
- Equipamento vinculado/trocado/desvinculado;
- Procedimento adicionado/removido;
- concluído;
- cancelado + motivo;
- reaberto.

Não mostrar timeline enorme de cada checkbox/campo por padrão.

## 28. Ficha / Imprimir

Preset de capacidade:

- ADM: sim;
- Gerência: sim;
- Funcionário: sim para Atendimento acessível.

Lifecycle:

- `Em andamento`: pode gerar ficha de acompanhamento;
- `Concluído`: pode reimprimir usando estado histórico aplicável;
- `Cancelado`: saída precisa identificar claramente o estado;
- alterações não salvas/conflitos bloqueiam geração.

A ficha:

- usa estado confirmado do Host;
- pode existir com ou sem Equipamento;
- nunca é screenshot;
- máximo uma página A4;
- usa identidade central da empresa.

Template, preview, PDF específico, margens, limites textuais e impressão Windows ficam no Bloco 10.

## 29. Voltar para lista

Preservar quando possível:

- busca;
- filtros Responsável/Status/Período;
- ordenação;
- página;
- scroll.

## 30. Estados transversais

Segue Tela 15:

- loading local;
- Host indisponível;
- WebSocket degradado;
- sessão expirada;
- sem permissão;
- recurso indisponível;
- conflito;
- `SERVER_BUSY`;
- resultado incerto;
- alterações não salvas.

## 31. Validações principais

### Atendimento

- código não é editável;
- equipamento é opcional;
- OS é opcional;
- cliente é opcional;
- responsável obrigatório para conclusão;
- resumo obrigatório para conclusão;
- cancelamento exige motivo;
- lifecycle é transição explícita, não dropdown livre.

### Equipamento

- código não editável;
- múltiplos MACs normalizáveis;
- bateria 0–100 quando informada;
- bateria contextual;
- observações limitadas;
- campos não aplicáveis opcionais.

## 32. Eventos em tempo real

Pós-commit podem sinalizar:

- Atendimento alterado;
- status alterado;
- responsável alterado;
- Equipamento alterado/vinculado;
- Procedimento vinculado/removido;
- checklist alterado;
- conclusão/reabertura/cancelamento.

Client reconsulta estado relevante; nunca sobrescreve formulário local silenciosamente.

## 33. Acessibilidade e janelas

- labels visíveis;
- headings semânticos;
- foco visível;
- icon-only com nome acessível;
- listas/checklists operáveis por teclado;
- mensagens não dependem só de cor;
- desktop Windows como alvo;
- em janela menor, colunas empilham sem transformar em UI mobile/hamburger.

## 34. Decisões consolidadas

- workspace vertical único;
- lifecycle de três estados;
- primeiro save cria registro;
- Funcionário opera por responsabilidade;
- resumo obrigatório para conclusão;
- checklist incompleto avisa, não bloqueia;
- concluído/cancelado são read-only até reabertura;
- cancelamento exige motivo;
- ADM/Gerência reabrem por preset;
- Equipamento opcional/reutilizável;
- Funcionário cria/edita Equipamento;
- ADM/Gerência arquivam/reativam;
- revisão exata do Procedimento é preservada;
- Funcionário usa revisão publicada por padrão;
- checklist persiste só em contexto de Atendimento;
- progresso deriva só de checklist;
- snapshot de Equipamento protege histórico concluído;
- ficha segue lifecycle e estado confirmado;
- concorrência continua otimista e granular.

## 35. Fora do escopo

- CRM;
- estoque;
- RMM;
- financeiro;
- SLA/prioridade;
- aprovação gerencial;
- workflow customizável;
- apontamento complexo de horas;
- chat social;
- autosave/offline queue;
- implementação funcional nesta fase.