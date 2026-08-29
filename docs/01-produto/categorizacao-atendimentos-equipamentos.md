# Categorização, Atendimentos e Equipamentos — StepFlow Pocket

**Status:** CONSOLIDADO  
**Atualização:** 2026-08-29

## 1. Objetivo

Separar documentação reutilizável de trabalho real executado, preservando simplicidade operacional e uso amplo em manutenção, TI, Service Desk, Help Desk, infraestrutura, redes e outros procedimentos internos.

## 2. Modelo de domínio

```text
Procedimento oficial
        ↓ usado em
Atendimento / Execução
        ↓ opcionalmente relacionado a
Equipamento
```

- **Procedimento** — documentação/modelo oficial reutilizável;
- **Atendimento/Execução** — ocorrência concreta de trabalho;
- **Equipamento** — ativo físico opcional e reutilizável.

Alterações futuras do Procedimento ou do cadastro global do Equipamento não devem reescrever silenciosamente o histórico consolidado de um Atendimento.

## 3. Categorias de Procedimentos

- configuráveis pela empresa, não hardcoded;
- um Procedimento pode possuir múltiplas categorias simples;
- pesquisáveis/filtráveis;
- sem árvore hierárquica complexa inicialmente;
- podem ser arquivadas preservando histórico;
- categoria arquivada deixa de ser opção normal para nova associação;
- nomes equivalentes após normalização não devem ser duplicados;
- gestão por preset: ADM e Gerência;
- autorização real continua granular e Host-side.

Pendente antes da implementação editorial: regra exata ao criar nova revisão de Procedimento que ainda carregue categoria arquivada.

## 4. Equipamento

Equipamento é opcional e reutilizável entre Atendimentos.

Para equipamentos de computação, tipos mínimos iniciais:

- Servidor;
- Desktop;
- Notebook.

Esses valores não formam enum global rígida para todos os tipos futuros.

Campos conforme aplicabilidade:

- nome;
- tipo;
- processador;
- RAM;
- armazenamento;
- sistema operacional e versão;
- serial;
- patrimônio/asset tag;
- um ou mais MACs;
- saúde da bateria para Notebook;
- cliente/solicitante/responsável relacionado;
- observações curtas.

Regras:

- bateria é opcional/contextual e, quando percentual, fica entre 0 e 100;
- campos vazios/não aplicáveis não viram burocracia visual;
- CPU/RAM/armazenamento são resumos textuais inicialmente;
- observação do Equipamento possui soft limit recomendado de 300 caracteres para favorecer a Ficha, sem virar hard limit de domínio por causa do layout.

## 5. Identidade e busca do Equipamento

Identidade canônica:

- `equipment_id` interno estável;
- código legível operacional gerado pelo Host.

Formato inicial:

```text
EQP-000001
```

- seis dígitos;
- sequência simples por implantação/banco ativo;
- gaps permitidos;
- não editável pelo usuário.

Atributos pesquisáveis, mas não identidade canônica exclusiva:

- código;
- nome;
- serial;
- patrimônio;
- MAC normalizado;
- cliente/solicitante relacionado.

Múltiplos MACs podem existir no mesmo Equipamento.

## 6. Lifecycle do Atendimento

Estados iniciais:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho somente no Client
      ↓ primeiro save aceito
Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
      ↓ Reabrir
Em andamento
```

Não criar workflow de chamados com SLA, prioridade, pausa, aprovação ou estados adicionais sem novo requisito.

## 7. Criação e identidade do Atendimento

Antes do primeiro save, `Novo atendimento` é rascunho somente em memória.

No primeiro save aceito pelo Host:

- cria ID interno estável;
- gera código legível;
- define `Em andamento`;
- define `started_at`;
- registra criador e responsável.

Formato inicial:

```text
AT-000001
```

- seis dígitos;
- gaps permitidos;
- não editável;
- cancelamento não reutiliza código.

## 8. Conteúdo do Atendimento

Pode conter:

- código legível e ID interno;
- OS/referência externa opcional;
- cliente/solicitante opcional;
- Equipamento opcional;
- responsável/técnico;
- datas de lifecycle;
- `Resumo do trabalho`;
- observação geral;
- zero, um ou vários Procedimentos/revisões utilizados;
- checklist/progresso quando existirem itens de checklist;
- observações de serviço por Etapa.

Soft limits orientativos ligados à Ficha:

- `Resumo do trabalho`: 600 caracteres;
- observação geral do Atendimento: 400;
- observação do Equipamento: 300;
- observação de serviço por Etapa: 280.

Esses valores orientam densidade; não bloqueiam save/conclusão nem truncam dados.

## 9. Responsável

- Atendimento precisa de responsável antes de concluir;
- Funcionário cria por padrão para si;
- Funcionário padrão não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, mas não é opção normal para nova atribuição;
- alteração de responsável é auditável.

## 10. Conclusão, cancelamento e reabertura

### Conclusão

Para concluir:

- Atendimento deve estar `Em andamento`;
- sessão precisa de capacidade;
- responsável deve estar definido;
- `Resumo do trabalho` é obrigatório;
- conflitos/alterações não confirmadas precisam estar resolvidos.

Não são obrigatórios por si só: OS, cliente, Equipamento ou Procedimento vinculado.

Checklist incompleto gera confirmação, não bloqueio automático.

Ao concluir, o Host preserva estado suficiente para reprodução histórica, incluindo revisões usadas, checklist final, observações de serviço e projeção relevante do Equipamento.

### Cancelamento

- somente em `Em andamento`;
- exige capacidade e motivo curto;
- não exclui o Atendimento;
- preserva código e histórico;
- bloqueia edição direta após commit.

### Reabertura

- retorna `Concluído` ou `Cancelado` para `Em andamento`;
- explícita e auditável;
- preset: ADM/Gerência sim; Funcionário não;
- não apaga lifecycle anterior;
- nova conclusão gera novo estado final aplicável sem destruir o anterior.

## 11. Procedimentos utilizados

Cada vínculo preserva a revisão exata realmente utilizada:

- ID do Procedimento;
- ID da revisão;
- código snapshot;
- título snapshot;
- versão editorial snapshot.

Seleção por preset:

- Funcionário: revisão publicada que possa ler;
- ADM/Gerência: publicada por padrão; podem selecionar explicitamente outra revisão autorizada;
- revisão histórica/não publicada nunca é escolhida silenciosamente;
- publicação futura não substitui vínculo existente.

Vínculo só é alterado enquanto Atendimento estiver editável. Remoção com checklist/observação operacional existente exige confirmação proporcional e preservação do histórico necessário.

## 12. Checklist operacional

O checklist documental do Procedimento permanece imutável. Estado operacional existe somente em Atendimento.

```text
Processos → Reader
→ checklist documental, sem persistência operacional

Atendimento → revisão vinculada → Executar
→ checklist persistente daquele Atendimento
```

Regras:

- marcação/desmarcação somente em estado editável e com capacidade;
- concorrência granular por item/equivalente;
- `Concluído`/`Cancelado` ficam somente leitura até reabertura;
- progresso deriva de itens marcados / total;
- Etapas visitadas não contam;
- revisão sem checklist não mostra `0%` artificial;
- 100% não conclui Atendimento automaticamente.

## 13. Observação do serviço por Etapa

- opcional;
- pertence ao Atendimento + vínculo da revisão + Etapa;
- não altera Procedimento oficial;
- concorrência granular por Etapa/equivalente;
- evento remoto não sobrescreve texto local silenciosamente;
- editável somente enquanto Atendimento estiver editável/autorizado;
- somente leitura em `Concluído`/`Cancelado` até reabertura;
- participa da reprodução histórica da Ficha;
- sem autosave por inferência.

## 14. Equipamento e histórico

Preset:

- criar/editar Equipamento: ADM, Gerência e Funcionário;
- arquivar/reativar: ADM e Gerência;
- Funcionário pode vincular/trocar/desvincular em Atendimento editável quando responsável.

Equipamento arquivado não aparece para novos vínculos normais.

Não arquivar Equipamento enquanto estiver vinculado a Atendimento `Em andamento`.

Conclusão congela a projeção relevante do Equipamento para reprodução histórica. Alteração posterior do cadastro global não reescreve a Ficha daquele estado final.

## 15. Ficha compacta

A Ficha pertence ao Atendimento e pode existir com ou sem Equipamento.

Lifecycle:

- `Em andamento`: gera estado confirmado atual;
- `Concluído`: reimprime estado histórico aplicável;
- `Cancelado`: saída identifica claramente o estado;
- alterações locais não salvas/conflitos bloqueiam geração.

Contrato físico/documental:

- prestação de contas resumida ao cliente;
- PDF canônico + preview do mesmo `PagedDocument`;
- exatamente uma página A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- nenhuma truncagem, segunda página, resumo automático ou redução dinâmica para caber;
- campos vazios/não aplicáveis são omitidos;
- Procedimentos vinculados não são listados por padrão;
- checklist/progresso/timeline não são impressos por padrão;
- MACs: 0 omite; 1–2 exibem valores; 3+ exibem somente a quantidade cadastrada;
- observações legítimas não recebem descarte automático.

Detalhes: `../02-telas/14-exportacao-impressao-ficha.md`.

## 16. Lista e busca de Atendimentos

- Status visível: Em andamento / Concluído / Cancelado;
- filtro por Responsável, Status e Período;
- período usa `started_at`;
- ordenação padrão: mais recente primeiro;
- busca operacional separada de Processos.

Pode pesquisar por:

- código do Atendimento;
- OS/referência;
- cliente;
- Equipamento/código/nome;
- serial;
- patrimônio;
- MAC.

## 17. Matriz operacional de presets

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar Atendimentos | sim | sim | sim |
| Criar Atendimento | sim | sim | sim |
| Editar Atendimento próprio em andamento | sim | sim | sim |
| Editar qualquer Atendimento em andamento | sim | sim | não |
| Concluir Atendimento próprio | sim | sim | sim |
| Concluir qualquer Atendimento | sim | sim | não |
| Cancelar | sim | sim | não |
| Reabrir | sim | sim | não |
| Vincular/trocar/desvincular Equipamento editável | sim | sim | sim, quando responsável |
| Criar/editar Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica | sim | sim | não |
| Marcar/desmarcar checklist | sim | sim | sim, quando responsável |
| Gerar/reimprimir Ficha acessível | sim | sim | sim |
| Gerir categorias | sim | sim | não |

Presets não substituem capacidades granulares nem autorização Host-side.

Gerência × configuração da empresa e Gerência × Backup permanecem pendentes.

## 18. Concorrência e histórico

- Atendimento usa revisão otimista;
- Equipamento possui revisão própria;
- checklist usa granularidade por item;
- observação de serviço usa granularidade por Etapa;
- eventos pós-commit sinalizam mudança e Client reconsulta;
- timeout/desconexão após mutação exige reconciliação, não retry cego.

Preservar eventos de alto valor: criação, mudança de responsável, vínculo de Equipamento/Procedimento, conclusão, cancelamento + motivo, reabertura e alterações administrativas relevantes.

Não criar timeline burocrática de cada campo/checkbox por padrão.

## 19. Pendências remanescentes

- regra editorial de nova revisão ainda associada a categoria arquivada;
- parâmetros finais de autenticação/sessão;
- Gerência × configuração da empresa;
- Gerência × Backup;
- formas físicas finais de schema/migrations, somente no gate correspondente.

## 20. Fora do escopo inicial

- CRM;
- financeiro/faturamento;
- estoque;
- RMM/inventário automatizado;
- descoberta automática de hardware;
- sistema completo de chamados/SLA;
- workflow customizável;
- taxonomia hierárquica complexa;
- DOCX específico da Ficha.
