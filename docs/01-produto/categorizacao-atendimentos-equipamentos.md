# Categorização, Atendimentos e Equipamentos — StepFlow

**Status:** CONSOLIDADO, INCLUINDO REGRAS OPERACIONAIS DO BLOCO 9  
**Atualização:** 2026-08-25

## 1. Objetivo

O StepFlow separa documentação reutilizável de trabalho real executado, preservando simplicidade operacional e uso amplo em manutenção, TI, Service Desk, Help Desk, infraestrutura, redes e procedimentos internos.

## 2. Modelo de domínio

Existem três conceitos distintos:

1. **Procedimento** — documentação/modelo oficial reutilizável;
2. **Atendimento/Execução** — ocorrência concreta de trabalho;
3. **Equipamento** — ativo físico opcional relacionado ao Atendimento quando aplicável.

```text
Procedimento oficial
        ↓ usado em
Atendimento/Execução
        ↓ relacionado opcionalmente a
Equipamento
```

Alterações futuras do Procedimento ou do cadastro global do Equipamento não devem reescrever silenciosamente o histórico já consolidado de um Atendimento concluído.

## 3. Categorias de Procedimentos

- configuráveis pela empresa, não hardcoded;
- um Procedimento pode ter múltiplas categorias simples;
- pesquisáveis/filtráveis;
- sem árvore hierárquica complexa inicialmente;
- podem ser arquivadas preservando histórico;
- categorias arquivadas deixam de ser opção normal para novas associações;
- nomes equivalentes após normalização não devem ser duplicados;
- gestão por preset: ADM e Gerência; Funcionário não;
- autorização real continua granular e Host-side.

Exemplos como `Manutenção`, `TI`, `Service Desk`, `Help Desk`, `Infraestrutura`, `Redes` e `Guias` são apenas exemplos.

Permanece pendente para fechamento antes da implementação editorial: regra exata de uma nova revisão de Procedimento que ainda carregue categoria arquivada.

## 4. Equipamento

Equipamento é opcional e reutilizável entre Atendimentos.

Para equipamentos de computação, o tipo deve suportar pelo menos:

- `Servidor`;
- `Desktop`;
- `Notebook`.

Esses valores não formam enum global rígida para todos os tipos futuros de equipamento.

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

- bateria é opcional/contextual e, quando informada como percentual, fica entre 0 e 100;
- campos vazios/não aplicáveis não viram burocracia visual;
- observações possuem limite explícito; valor numérico final será fechado no Bloco 10 junto da ficha A4;
- CPU/RAM/armazenamento são inicialmente resumos textuais, sem inventário automático.

## 5. Identidade e busca do Equipamento

Identidade canônica:

- `equipment_id` interno estável;
- código legível operacional gerado pelo Host.

Código inicial consolidado:

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

Múltiplos MACs podem existir para o mesmo Equipamento.

## 6. Lifecycle do Atendimento

Estados iniciais consolidados:

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

Concluído/Cancelado
      ↓ Reabrir
Em andamento
```

Não criar workflow de chamados com SLA, prioridade, aprovação, pausa ou estados adicionais sem novo requisito.

## 7. Criação e identidade do Atendimento

Antes do primeiro save, `Novo atendimento`/`Iniciar atendimento` é rascunho somente em memória.

No primeiro save aceito pelo Host:

- cria ID interno estável;
- gera código legível;
- define `Em andamento`;
- define `started_at`;
- registra criador e responsável.

Código inicial:

```text
AT-000001
```

- seis dígitos;
- sequência simples por implantação/banco ativo;
- gaps permitidos;
- não editável;
- cancelamento não libera/reutiliza código.

## 8. Conteúdo do Atendimento

Pode conter:

- ID interno estável;
- código legível;
- OS/referência externa opcional;
- cliente/solicitante opcional;
- Equipamento opcional;
- responsável/técnico;
- `started_at` e datas de lifecycle aplicáveis;
- resumo do trabalho;
- observações;
- zero, um ou vários Procedimentos/revisões utilizados;
- checklist/progresso quando existirem itens de checklist nas revisões vinculadas.

## 9. Responsável

- Atendimento precisa de responsável antes de concluir;
- Funcionário cria por padrão para si mesmo;
- Funcionário padrão não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, mas não é escolha normal para nova atribuição;
- alteração de responsável é auditável.

## 10. Conclusão, cancelamento e reabertura

### Conclusão

Para concluir:

- Atendimento deve estar `Em andamento`;
- sessão precisa de capacidade;
- responsável deve estar definido;
- `Resumo do trabalho` é obrigatório;
- conflitos/alterações não confirmadas precisam ser resolvidos.

Não são obrigatórios por si só:

- OS;
- cliente;
- Equipamento;
- Procedimento vinculado.

Checklist incompleto gera confirmação, mas não bloqueia automaticamente porque os itens documentais não possuem semântica obrigatório/opcional na primeira versão.

Ao concluir:

- status vira `Concluído`;
- Host define `completed_at`;
- preserva revisões utilizadas e checklist final;
- congela projeção histórica relevante do Equipamento;
- registra evento de conclusão.

### Cancelamento

- somente em `Em andamento`;
- exige capacidade própria;
- exige motivo curto;
- não exclui o Atendimento;
- preserva código e histórico;
- bloqueia edição operacional direta.

### Reabertura

- retorna `Concluído` ou `Cancelado` para `Em andamento`;
- explícita e auditável;
- preset: ADM/Gerência sim; Funcionário não;
- não apaga lifecycle anterior;
- nova conclusão gera novo evento/estado final.

## 11. Procedimentos utilizados

Cada vínculo preserva a revisão exata realmente utilizada:

- ID do Procedimento;
- ID da revisão;
- código snapshot;
- título snapshot;
- versão editorial snapshot.

Seleção por preset:

- Funcionário: revisão publicada que possa ler;
- ADM/Gerência: publicada por padrão; podem selecionar explicitamente outra revisão histórica/não publicada que já possam ler;
- nenhuma revisão histórica/não publicada é escolhida silenciosamente;
- publicação futura não substitui vínculo existente.

Vínculo pode ser adicionado/removido apenas enquanto Atendimento estiver editável. Remoção com checklist já marcado exige confirmação explícita.

## 12. Checklist operacional

O checklist documental do Procedimento continua imutável. Estado operacional existe somente no contexto de um Atendimento.

```text
Processos → Leitor
→ checklist documental, sem persistência operacional

Atendimento → revisão vinculada → Executar
→ checklist persistente daquele Atendimento
```

Cada item operacional preserva conceitualmente:

- identidade própria;
- vínculo Atendimento × revisão;
- referência ao item de origem;
- texto snapshot quando necessário;
- marcado/desmarcado;
- data/usuário da marcação quando aplicável;
- revisão/controle concorrente próprio ou equivalente.

Em `Em andamento`, usuário autorizado pode marcar/desmarcar. Em `Concluído`/`Cancelado`, fica somente leitura até reabertura.

## 13. Progresso

Progresso deriva apenas dos itens persistentes de checklist:

```text
PR-001        4 de 6 itens
PR-022        2 de 2 itens
Atendimento   6 de 8 itens
```

- etapas visitadas não contam;
- `Etapa X de Y` continua sendo navegação;
- revisão sem checklist não mostra `0%` artificial;
- 100% não conclui Atendimento automaticamente.

## 14. Equipamento e histórico

Preset de capacidades:

- criar/editar Equipamento: ADM, Gerência e Funcionário;
- arquivar/reativar Equipamento: ADM e Gerência;
- Funcionário pode vincular/trocar/desvincular em Atendimento editável quando for responsável.

Equipamento arquivado não aparece para novos vínculos normais.

Não arquivar Equipamento enquanto estiver vinculado a Atendimento `Em andamento`; antes, o Atendimento deve ser concluído/cancelado ou o vínculo removido.

Ao concluir um Atendimento, a projeção relevante do Equipamento fica congelada para reprodução histórica. Mudanças posteriores no cadastro global não reescrevem a ficha final daquele estado concluído.

## 15. Ficha compacta

A ficha pertence ao Atendimento e pode existir com ou sem Equipamento.

Regras operacionais consolidadas:

- `Em andamento`: pode ser gerada para acompanhamento, usando somente estado confirmado do Host;
- `Concluído`: pode ser reimpressa a partir do estado histórico congelado aplicável;
- `Cancelado`: quando gerada/reimpressa, identifica inequivocamente o estado;
- alterações não salvas/conflitos bloqueiam geração;
- preset de gerar/reimprimir: ADM, Gerência e Funcionário para Atendimentos acessíveis.

Requisitos físicos já consolidados:

- documento próprio, não screenshot;
- máximo uma página A4;
- pode ser menor que A4;
- não gera segunda página como comportamento normal;
- sem redução tipográfica excessiva;
- conteúdo que não caiba de forma legível bloqueia a saída;
- não truncar silenciosamente informação importante;
- campos vazios/não aplicáveis são omitidos;
- cabeçalho usa logo/nome/contato/site/e-mail configurados.

A impressão é requisito. DOCX específico da ficha não é requisito inicial. PDF específico permanece para decisão do Bloco 10.

## 16. Lista/Pesquisa de Atendimentos

Com o lifecycle consolidado:

- lista mostra `Em andamento`, `Concluído` ou `Cancelado`;
- existe filtro por Status;
- `Data` representa `started_at`;
- filtro `Período` usa `started_at`;
- ordenação padrão: mais recente por `started_at`;
- busca operacional permanece separada da busca de Processos.

Busca pode usar:

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
| Gerar/reimprimir ficha acessível | sim | sim | sim |
| Gerir categorias | sim | sim | não |

Presets não substituem capacidades granulares nem autorização Host-side.

Gerência × configuração da empresa e Gerência × Backup permanecem pendentes.

## 18. Concorrência e eventos

- Atendimento usa revisão otimista por recurso;
- concluir/cancelar/reabrir também são mutações versionadas;
- Equipamento possui revisão própria;
- checklist usa controle granular por item/equivalente para evitar conflito global desnecessário;
- evento pós-commit sinaliza mudança e Client reconsulta;
- evento nunca sobrescreve edição local;
- timeout/desconexão após mutação exige reconciliação, não retry cego.

## 19. Histórico operacional

Preservar eventos de alto valor:

- criação;
- mudança de responsável;
- vínculo/troca/desvínculo de Equipamento;
- Procedimento adicionado/removido;
- conclusão;
- cancelamento + motivo;
- reabertura;
- alterações administrativas relevantes.

Não criar timeline burocrática de cada campo/checkbox por padrão.

## 20. Pendências remanescentes

### Bloco 10

- template físico final A4;
- margens, tipografia e densidade;
- limites numéricos de textos destinados à ficha;
- regras finais de resumo/truncamento controlado;
- muitos MACs/procedimentos;
- preview;
- impressão Windows;
- PDF específico da ficha;
- QR/barcode se houver valor.

### Editorial/categorias

- comportamento exato ao criar nova revisão de Procedimento ainda referenciando categoria arquivada.

### Autenticação/configuração

- parâmetros finais de Argon2/senha/sessão;
- Gerência × configuração da empresa;
- Gerência × Backup.

## 21. Fora do escopo inicial

Não transformar o StepFlow automaticamente em:

- CRM;
- financeiro/faturamento;
- estoque;
- RMM/inventário automatizado;
- descoberta automática de hardware;
- sistema completo de chamados/SLA;
- workflow customizável;
- taxonomia hierárquica complexa;
- DOCX específico da ficha.

Esses itens exigem requisito futuro explícito.