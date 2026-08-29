# Tela 09 — Atendimento / Execução / Equipamento

## 1. Identificação

- status: **CONSOLIDADO / APROVADO PELO PO**;
- domínio: Atendimento/Execução + Equipamento opcional;
- atualização: 2026-08-29.

## 2. Objetivo

Ser o workspace operacional do serviço real, sem misturar edição do Procedimento oficial com dados do Atendimento.

A tela permite:

- criar e editar Atendimento;
- vincular Equipamento opcional;
- vincular uma ou mais revisões de Procedimento;
- abrir o Reader em contexto operacional;
- acompanhar checklist/progresso;
- registrar resumo e observações;
- concluir/cancelar/reabrir conforme capacidade;
- gerar/reimprimir Ficha compacta.

## 3. Princípios de UX

- workspace vertical único;
- baixa densidade textual;
- ações críticas contextuais, sem painel burocrático;
- estado/lifecycle sempre compreensível;
- Equipamento é opcional;
- checklist e observações operacionais não alteram Procedimento oficial;
- autorização real é Host-side;
- nenhum autosave/offline queue por inferência.

## 4. Rascunho e primeiro save

`Novo atendimento` começa somente em memória.

```text
Novo atendimento
→ preencher dados
→ primeiro save aceito pelo Host
→ cria registro oficial
→ código AT-000001
→ status Em andamento
```

Sair antes do primeiro save não cria Atendimento.

O código é Host-only, seis dígitos, gaps permitidos e não editável.

## 5. Estrutura da tela

Ordem conceitual:

```text
cabeçalho do Atendimento
→ identificação / responsável / referência
→ Equipamento opcional
→ Procedimentos utilizados
→ Resumo do trabalho / observações
→ ações de lifecycle
→ Ficha / histórico compacto quando aplicável
```

Evitar múltiplos painéis competindo pela atenção.

## 6. Campos do Atendimento

Quando aplicável:

- código `AT-...` somente leitura;
- OS/referência externa opcional;
- cliente/solicitante opcional;
- responsável/técnico;
- status;
- data/início aplicável;
- `Resumo do trabalho`;
- observação geral;
- Equipamento opcional;
- Procedimentos/revisões utilizados.

Responsável e `Resumo do trabalho` são obrigatórios para conclusão, não necessariamente para o primeiro save.

## 7. Soft limits de texto

Orientativos para favorecer a Ficha compacta:

- `Resumo do trabalho`: 600 caracteres;
- observação geral do Atendimento: 400;
- observação do Equipamento: 300;
- observação do serviço por Etapa: 280.

- não bloqueiam save/conclusão;
- não truncam dados;
- aviso aparece apenas próximo da faixa recomendada;
- layout real da Ficha continua autoridade de encaixe.

## 8. Lifecycle

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

Lifecycle não é dropdown livre.

## 9. Concluir Atendimento

Pré-condições:

- estado `Em andamento`;
- capacidade de conclusão;
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

Se houver itens desmarcados, exibir confirmação clara:

```text
Ainda existem itens de checklist não concluídos.
Deseja concluir este atendimento mesmo assim?
```

Checklist incompleto avisa, não bloqueia automaticamente.

### Após conclusão

Host:

- grava `Concluído`;
- define `completed_at`;
- preserva revisões usadas/checklist final;
- preserva observações de serviço aplicáveis;
- congela projeção relevante do Equipamento;
- registra evento/auditoria;
- publica mudança pós-commit.

## 10. Cancelar

Preset inicial:

- ADM/Gerência: sim;
- Funcionário: não.

Regras:

- somente `Em andamento`;
- motivo curto obrigatório;
- confirmação explícita;
- preserva código, vínculos e histórico;
- não exclui;
- após commit fica read-only.

## 11. Reabrir

Preset inicial:

- ADM/Gerência: sim;
- Funcionário: não.

- disponível em `Concluído` ou `Cancelado`;
- ação explícita e auditável;
- retorna para `Em andamento`;
- preserva histórico anterior;
- nova conclusão cria novo estado final aplicável;
- conclusão anterior não é reescrita silenciosamente.

## 12. Responsabilidade e permissões

Funcionário:

- cria Atendimento inicialmente para si;
- opera/conclui o Atendimento do qual é responsável;
- não reatribui para outro usuário por preset;
- não cancela/reabre por preset.

ADM/Gerência:

- podem atribuir/reatribuir;
- podem operar Atendimentos acessíveis conforme capacidades.

Autorização real permanece granular e Host-side.

## 13. Equipamento opcional

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

## 14. Cadastro do Equipamento

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

Bateria é opcional/contextual; percentual válido 0–100 quando informado.

MAC não é identidade canônica. Evitar duplicação silenciosa baseada apenas em identificadores pesquisáveis.

## 15. Capacidades de Equipamento

- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- Funcionário vincula/troca/desvincula em Atendimento editável quando responsável;
- Equipamento possui save/conflito separado do Atendimento;
- não arquivar Equipamento vinculado a Atendimento `Em andamento`.

## 16. Histórico do Equipamento

Conclusão congela projeção relevante do Equipamento.

Alteração futura no cadastro global não reescreve Ficha/estado histórico de conclusão anterior. Nova conclusão após reabertura pode produzir nova projeção sem apagar a anterior.

## 17. Procedimentos utilizados

Cada vínculo mostra de forma operacional:

- código snapshot;
- título snapshot;
- versão editorial;
- revisão técnica exata;
- progresso de checklist quando houver;
- ações contextuais.

A revisão vinculada nunca muda automaticamente após nova publicação.

## 18. Adicionar/remover Procedimento

Enquanto `Em andamento` e autorizado:

- buscar em Processos;
- selecionar revisão elegível;
- vincular explicitamente.

Preset:

- Funcionário: revisão publicada que possa ler;
- ADM/Gerência: publicada por padrão; podem selecionar explicitamente histórica/não publicada já autorizada.

Remoção com checklist marcado ou observação de serviço registrada exige confirmação e preservação do histórico necessário.

## 19. Executar Procedimento

Ação `Executar` abre a Tela 05 na revisão exata e no contexto do Atendimento.

```text
Atendimento AT-00142
→ PR-001 r18
→ Executar
→ Reader: Executando no atendimento AT-00142
```

Nesse contexto:

- checklist persiste;
- cada Etapa pode receber `Observação do serviço`;
- observação pertence ao Atendimento + revisão + Etapa;
- nada altera o Procedimento oficial.

## 20. Progresso

Exemplo:

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

## 21. Concorrência granular

Checklist:

- controle por item/equivalente;
- itens independentes não conflitam globalmente.

Observação de serviço:

- controle por Etapa/equivalente;
- conflito no mesmo campo preserva texto local e exige reconciliação;
- evento remoto não sobrescreve texto em edição.

Atendimento e Equipamento possuem revisões próprias. Não usar uma revisão global para invalidar cada checkbox/observação por conveniência.

## 22. Histórico operacional compacto

Eventos de alto valor podem aparecer numa área simples:

- criado;
- responsável alterado;
- Equipamento vinculado/trocado/desvinculado;
- Procedimento adicionado/removido;
- concluído;
- cancelado + motivo;
- reaberto.

Não mostrar timeline enorme de cada checkbox/campo por padrão.

## 23. Ficha / Imprimir

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

Prioriza:

- identificação do serviço;
- cliente/solicitante e técnico quando úteis;
- Equipamento e características relevantes;
- `Resumo do trabalho`;
- observação geral;
- observações do Equipamento/Etapas quando aplicáveis.

Por padrão não imprime checklist, progresso, passos/comandos, timeline, IDs internos ou lista detalhada de Procedimentos.

Contrato consolidado:

- PDF próprio + preview do mesmo `PagedDocument`;
- exatamente uma A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- sem truncamento, segunda página ou compactação automática;
- MACs: 0 omite; 1–2 valores; 3+ quantidade;
- `Salvar PDF`/`Imprimir` reutilizam o PDF da prévia aberta.

Detalhes: `14-exportacao-impressao-ficha.md`.

## 24. Estados transversais

Segue Tela 15:

- loading;
- Host indisponível;
- WebSocket degradado;
- sessão expirada;
- sem permissão;
- recurso indisponível;
- conflito;
- `SERVER_BUSY`;
- resultado incerto;
- alterações não salvas.

## 25. Acessibilidade e janelas

- labels e headings semânticos;
- foco visível;
- icon-only com nome acessível;
- listas/checklists operáveis por teclado;
- mensagens não dependem só de cor;
- desktop Windows como alvo;
- em janela menor, colunas podem empilhar sem criar UI mobile/hamburger inicial.

## 26. Fora do escopo

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
