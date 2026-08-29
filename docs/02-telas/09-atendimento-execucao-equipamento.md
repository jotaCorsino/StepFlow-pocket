# Tela 09 — Atendimento / Execução / Equipamento

## 1. Identificação

- status: **CONSOLIDADO / APROVADO PELO PO**;
- domínio: Atendimento/Execução + Equipamento opcional;
- atualização: 2026-08-29.

## 2. Objetivo

Ser o workspace principal de um Atendimento real, sem misturar edição do Procedimento oficial com dados operacionais.

A tela reúne numa página vertical:

- lifecycle/status;
- dados básicos do Atendimento;
- responsável/técnico;
- cliente/solicitante e OS/referência opcionais;
- Equipamento opcional;
- Procedimentos/revisões utilizados;
- progresso/checklist operacional;
- resumo do trabalho;
- observações gerais;
- acesso às observações de serviço por Etapa no Reader;
- histórico operacional compacto;
- `Ficha / Imprimir`.

Não transformar a superfície em CRM, estoque, RMM ou sistema completo de chamados.

## 3. Acesso e capacidades

Preset inicial:

- ADM: consulta/criação/gestão ampla;
- Gerência: consulta/criação/gestão ampla conforme capacidades;
- Funcionário: consulta/criação e edição/conclusão do Atendimento do qual é responsável.

Autorização real permanece Host-side.

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
Reader de Procedimento
→ Iniciar atendimento
→ Tela 09 em rascunho
→ revisão consultada pré-selecionada quando elegível
```

Abrir a tela em modo novo não cria registro oficial.

## 5. Rascunho e primeiro save

Antes do primeiro save:

- sem `AT-...` definitivo;
- rascunho somente em memória;
- se veio do Reader, revisão elegível pode vir pré-selecionada;
- Funcionário começa como responsável por padrão;
- sair com alterações pede confirmação;
- sair descarta o rascunho; não existe draft persistente inicial.

No primeiro save aceito pelo Host:

- cria `service_record_id`;
- gera código `AT-000001`/sequência aplicável;
- define `started_at`;
- define `Em andamento`;
- registra criador/responsável;
- devolve estado confirmado.

Código do Atendimento:

- Host-only;
- seis dígitos;
- sequência simples por implantação/banco ativo;
- gaps permitidos;
- não editável;
- não substitui o ID interno.

## 6. Estrutura visual

Workspace vertical único, com baixa densidade textual.

Ordem conceitual:

```text
← Atendimentos

Atendimento AT-00142                    Em andamento
OS-4587 · Cliente · Responsável

[ Ficha / Imprimir ] [ Salvar ] [ Concluir atendimento ] [ Mais ▾ ]

ATENDIMENTO
→ identificação / responsável / referência
→ Resumo do trabalho / observação geral

EQUIPAMENTO
→ ficha técnica resumida, quando houver
→ ações de vínculo/edição

PROCEDIMENTOS UTILIZADOS
→ revisão exata + progresso + ações

PROGRESSO
→ contagem derivada de checklist

HISTÓRICO
→ eventos de alto valor
```

As observações específicas de execução ficam junto da Etapa no Reader operacional, evitando uma lista textual permanente adicional na Tela 09.

Ações aparecem somente quando estado + capacidade permitirem.

## 7. Campos do Atendimento

Quando aplicável:

- código `AT-...` somente leitura;
- Status derivado do lifecycle, não dropdown livre;
- OS/referência externa opcional;
- cliente/solicitante opcional;
- responsável/técnico;
- `started_at`/datas de lifecycle quando úteis;
- `Resumo do trabalho`;
- observação geral;
- Equipamento opcional;
- Procedimentos/revisões utilizados.

Responsável e `Resumo do trabalho` são obrigatórios para conclusão, não necessariamente para o primeiro save.

Observação de serviço por Etapa é dado operacional separado da observação geral e da observação do Equipamento.

## 8. Soft limits de texto

Orientativos para favorecer a Ficha compacta:

- `Resumo do trabalho`: 600 caracteres;
- observação geral do Atendimento: 400;
- observação do Equipamento: 300;
- observação do serviço por Etapa: 280.

- não bloqueiam save/conclusão;
- não truncam dados;
- aviso aparece apenas próximo da faixa recomendada;
- layout real da Ficha continua autoridade de encaixe.

## 9. Lifecycle

```text
Em andamento
Concluído
Cancelado
```

```text
Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
   → Reabrir
   → Em andamento
```

Não criar `Aguardando`, `Pausado`, `Resolvido`, SLA ou estados adicionais sem novo requisito.

## 10. Edição por estado

### Em andamento

Conforme capacidade, pode alterar:

- OS/referência;
- cliente;
- responsável;
- resumo;
- observação geral;
- vínculo de Equipamento;
- cadastro do Equipamento em fluxo próprio;
- Procedimentos/revisões;
- checklist operacional;
- observações de serviço por Etapa no Reader.

### Concluído

Somente leitura operacional.

Conforme capacidade, pode:

- consultar;
- abrir revisões usadas;
- consultar checklist final;
- consultar observações registradas;
- reimprimir Ficha;
- reabrir.

Correção operacional exige `Reabrir`.

### Cancelado

Somente leitura operacional.

- permanece consultável/pesquisável;
- não é excluído;
- pode ser reaberto conforme capacidade;
- Ficha, quando gerada, identifica claramente `Cancelado`.

## 11. Salvar

Salvamento é explícito; não existe autosave inicial.

- Host valida capacidade, estado e `row_revision`/base;
- sucesso só aparece após confirmação de commit;
- conflito preserva alterações locais;
- evento remoto não substitui formulário;
- resultado incerto após desconexão é reconciliado, não repetido cegamente.

Recursos granulares, como checklist e observação de serviço por Etapa, podem usar mutações próprias sem transformar cada alteração em save global de toda a Tela 09.

## 12. Responsabilidade e permissões

Funcionário:

- cria Atendimento inicialmente para si;
- opera/conclui o Atendimento do qual é responsável;
- não reatribui para outro usuário por preset;
- não cancela/reabre por preset.

ADM/Gerência:

- podem atribuir/reatribuir;
- podem operar Atendimentos acessíveis conforme capacidades.

Usuário desativado permanece identificável no histórico, mas não é opção normal para nova atribuição.

Mudança de responsável é auditável.

## 13. Concluir Atendimento

Disponível somente em `Em andamento` e com capacidade.

Pré-condições:

- responsável definido;
- `Resumo do trabalho` válido;
- estado confirmado/sem conflito;
- alterações locais relevantes salvas.

Não são obrigatórios por si só:

- OS;
- cliente;
- Equipamento;
- Procedimento vinculado;
- observação de serviço por Etapa.

### Checklist incompleto

Se houver itens desmarcados:

```text
Ainda existem itens de checklist não concluídos.
Deseja concluir este atendimento mesmo assim?

[ Continuar execução ] [ Concluir mesmo assim ]
```

Não bloquear automaticamente; os checklists documentais iniciais não distinguem obrigatório/opcional.

### Após conclusão

Host:

- grava `Concluído`;
- define `completed_at`;
- preserva revisões usadas/checklist final;
- preserva estado final aplicável das observações de serviço;
- congela projeção relevante do Equipamento;
- registra evento/auditoria;
- publica mudança pós-commit.

O histórico deve permitir reimprimir a prestação de contas do estado final aplicável mesmo após reabertura e alterações posteriores.

## 14. Cancelar

Preset inicial:

- ADM/Gerência: sim;
- Funcionário: não.

Regras:

- somente `Em andamento`;
- motivo curto obrigatório;
- confirmação explícita;
- preserva código, vínculos e histórico;
- não exclui;
- após commit fica somente leitura.

## 15. Reabrir

Preset inicial:

- ADM/Gerência: sim;
- Funcionário: não.

- disponível em `Concluído` ou `Cancelado`;
- ação explícita e auditável;
- retorna para `Em andamento`;
- preserva histórico anterior;
- checklist, observações e vínculos continuam disponíveis;
- nova conclusão cria novo estado final aplicável;
- conclusão anterior não é reescrita silenciosamente.

## 16. Equipamento opcional

A tela funciona com ou sem Equipamento.

Sem Equipamento:

```text
Nenhum equipamento vinculado a este atendimento.
[ Vincular equipamento ]
```

Não renderizar ficha técnica vazia.

Busca de Equipamento pode usar:

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

## 17. Cadastro do Equipamento

Campos conforme aplicabilidade:

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
- observações.

Tipos mínimos iniciais:

- Servidor;
- Desktop;
- Notebook.

Não formar enum global rígida para tipos futuros.

### Bateria

- opcional/contextual;
- especialmente aplicável a Notebook;
- percentual válido 0–100 quando informado;
- não aparece como obrigação para Servidor/Desktop.

### MACs

- múltiplos;
- labels opcionais como Wi‑Fi/Ethernet/Dock;
- Host normaliza formatos;
- MAC não é identidade canônica.

## 18. Capacidades e lifecycle do Equipamento

- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- Funcionário vincula/troca/desvincula em Atendimento editável quando responsável;
- Equipamento possui save/conflito separado do Atendimento;
- não arquivar Equipamento vinculado a Atendimento `Em andamento`.

Equipamento arquivado não aparece para novo vínculo normal. Reativação depende de capacidade.

## 19. Histórico do Equipamento

Conclusão congela projeção relevante do Equipamento.

Alteração futura no cadastro global não reescreve Ficha/estado histórico de conclusão anterior. Nova conclusão após reabertura pode produzir nova projeção sem apagar a anterior.

## 20. Procedimentos utilizados

Cada vínculo mostra:

- código snapshot;
- título snapshot;
- versão editorial usada;
- revisão técnica exata;
- progresso de checklist quando houver;
- ações contextuais.

A revisão vinculada nunca muda automaticamente após nova publicação.

Esses detalhes permanecem importantes para operação/histórico interno, mas não precisam aparecer por padrão na Ficha entregue ao cliente.

## 21. Adicionar/remover Procedimento

Enquanto `Em andamento` e autorizado:

- abrir busca de Processos;
- selecionar revisão elegível;
- vincular explicitamente.

Preset:

- Funcionário: revisão publicada que possa ler;
- ADM/Gerência: publicada por padrão; podem selecionar explicitamente histórica/não publicada já autorizada.

Se Atendimento veio do Reader, a revisão consultada pode vir pré-selecionada conforme essas regras.

Remoção com checklist marcado ou observação de serviço registrada exige confirmação e preservação do histórico necessário.

## 22. Executar Procedimento

Ação `Executar` abre a Tela 05 na revisão exata e no contexto do Atendimento.

```text
Atendimento AT-00142
→ PR-001 r18
→ Executar
→ Reader: Executando no atendimento AT-00142
```

Nesse contexto:

- checklist persiste;
- cada Etapa pode receber `Observação do serviço` opcional;
- observação pertence ao Atendimento + revisão + Etapa;
- nada altera o Procedimento oficial.

Em Reader standalone, checklist e observação operacional não persistem.

## 23. Progresso

```text
PR-001        4 de 6
PR-022        2 de 2
Atendimento   6 de 8
```

- deriva apenas de checklist;
- Etapas visitadas não contam;
- observações não contam;
- revisão sem checklist não mostra `0%` artificial;
- 100% não conclui automaticamente.

## 24. Concorrência granular

Checklist:

- controle por item/equivalente;
- itens independentes não conflitam globalmente;
- alteração concorrente do mesmo item recebe resultado determinístico/conflito apropriado.

Observação de serviço:

- controle por Etapa/equivalente;
- Etapas independentes não conflitam globalmente;
- conflito no mesmo campo preserva texto local e exige reconciliação;
- evento remoto não sobrescreve texto em edição.

Atendimento e Equipamento possuem revisões próprias. Não usar revisão global para invalidar cada checkbox/observação por conveniência.

## 25. Histórico operacional compacto

Eventos de alto valor podem aparecer numa área simples:

- criado;
- responsável alterado;
- Equipamento vinculado/trocado/desvinculado;
- Procedimento adicionado/removido;
- concluído;
- cancelado + motivo;
- reaberto.

Não mostrar timeline enorme de cada checkbox, observação de Etapa ou campo por padrão.

## 26. Ficha / Imprimir

Preset:

- ADM: sim;
- Gerência: sim;
- Funcionário: sim para Atendimento acessível.

Lifecycle:

- `Em andamento`: gera estado confirmado atual;
- `Concluído`: reimprime estado histórico aplicável;
- `Cancelado`: identifica claramente o estado;
- alterações não salvas/conflitos bloqueiam geração.

A Ficha é prestação de contas resumida ao cliente.

Prioriza, quando aplicável:

- identificação do Atendimento/serviço;
- cliente/solicitante e técnico sem excesso de metadados;
- Equipamento e características relevantes;
- `Resumo do trabalho`;
- observação geral;
- observações do Equipamento/Etapas.

Por padrão não imprime:

- checklist completo;
- percentual/progresso;
- Etapas/passos do Procedimento;
- comandos;
- timeline operacional;
- IDs técnicos internos;
- lista detalhada de revisões/Procedimentos utilizados.

Contrato consolidado:

- usa estado confirmado do Host;
- pode existir com ou sem Equipamento;
- nunca é screenshot;
- PDF próprio + preview do mesmo `PagedDocument`;
- exatamente uma A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- sem truncamento, segunda página ou compactação automática;
- MACs: 0 omite; 1–2 valores; 3+ quantidade;
- `Salvar PDF`/`Imprimir` reutilizam o PDF da prévia aberta;
- usa identidade central da empresa.

Detalhes: `14-exportacao-impressao-ficha.md`.

## 27. Voltar para lista

Preservar quando possível:

- busca;
- filtros Responsável/Status/Período;
- ordenação;
- página;
- scroll.

## 28. Estados transversais

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

## 29. Validações principais

### Atendimento

- código não editável;
- Equipamento opcional;
- OS opcional;
- cliente opcional;
- responsável obrigatório para conclusão;
- resumo obrigatório para conclusão;
- observação de serviço opcional e vinculada à Etapa/revisão em execução;
- cancelamento exige motivo;
- lifecycle é transição explícita, não dropdown livre.

### Equipamento

- código não editável;
- múltiplos MACs normalizáveis;
- bateria 0–100 quando informada;
- bateria contextual;
- campos não aplicáveis opcionais.

## 30. Eventos em tempo real

Pós-commit podem sinalizar:

- Atendimento alterado;
- status alterado;
- responsável alterado;
- Equipamento alterado/vinculado;
- Procedimento vinculado/removido;
- checklist alterado;
- observação de serviço por Etapa alterada;
- conclusão/reabertura/cancelamento.

Client reconsulta estado relevante; nunca sobrescreve formulário/texto local silenciosamente.

## 31. Acessibilidade e janelas

- labels visíveis;
- headings semânticos;
- foco visível;
- icon-only com nome acessível;
- listas/checklists operáveis por teclado;
- campos de observação com label inequívoco;
- mensagens não dependem só de cor;
- desktop Windows como alvo;
- em janela menor, colunas empilham sem transformar em UI mobile/hamburger.

## 32. Decisões preservadas

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
- observação de serviço persiste por Etapa somente em Atendimento;
- progresso deriva só de checklist;
- snapshot de Equipamento e estado final aplicável protegem histórico;
- Ficha segue lifecycle e estado confirmado;
- concorrência continua otimista e granular;
- não há autosave/offline queue por inferência.

## 33. Fora do escopo

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
