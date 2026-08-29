# Bloco 9 — Atendimentos / Execução / Checklist

**Status:** CONCLUÍDO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Atualização:** 2026-08-29

## 1. Objetivo

Fechar o contrato operacional de Atendimento/Execução, checklist, observações de serviço, Equipamento, lifecycle, capacidades e reprodução histórica antes da implementação.

Este arquivo é o **mapa consolidado do Bloco 9**. Detalhes de UX e dados ficam nas fontes específicas:

- `../01-produto/categorizacao-atendimentos-equipamentos.md` — domínio;
- `../02-telas/05-leitor-processo.md` — Reader documental/operacional;
- `../02-telas/08-lista-pesquisa-atendimentos.md` — lista/busca;
- `../02-telas/09-atendimento-execucao-equipamento.md` — workspace operacional;
- `../03-arquitetura/modelo-dados-schema-fase-1.md` — persistência conceitual;
- `../03-arquitetura/concorrencia-fila-conflitos-eventos.md` — concorrência;
- `../03-arquitetura/autenticacao-sessao-autorizacao.md` — capacidades.

## 2. Domínio

```text
Procedimento oficial
        ↓ usado em
Atendimento / Execução
        ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação/modelo oficial;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode usar zero, um ou vários Procedimentos;
- cada vínculo preserva a revisão exata utilizada;
- alterações futuras não reescrevem silenciosamente histórico concluído.

## 3. Lifecycle do Atendimento

Estados:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho Client
→ primeiro save aceito pelo Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Regras:

- abrir tela não cria Atendimento;
- primeiro save cria ID/código e `started_at`;
- lifecycle não é dropdown livre;
- não criar SLA, prioridade, pausa, aprovação ou estados adicionais sem novo requisito;
- `Concluído` e `Cancelado` são read-only até reabertura.

## 4. Criação e códigos legíveis

Atendimento:

```text
AT-000001
```

Equipamento:

```text
EQP-000001
```

- gerados somente pelo Host;
- sequência numérica simples por implantação/banco ativo;
- seis dígitos com zero à esquerda;
- gaps permitidos;
- não editáveis;
- não substituem IDs internos estáveis;
- cancelamento mantém o código;
- sem ano, departamento, hostname ou dado pessoal no código inicial.

## 5. Conteúdo do Atendimento

Pode conter:

- OS/referência externa opcional;
- cliente/solicitante opcional;
- responsável/técnico;
- Equipamento opcional;
- `Resumo do trabalho`;
- observação geral;
- Procedimentos/revisões utilizados;
- checklist operacional;
- observações de serviço por Etapa;
- datas de lifecycle e histórico aplicável.

Responsável + `Resumo do trabalho` são obrigatórios para conclusão, não necessariamente para o primeiro save.

## 6. Responsável

- Funcionário cria por padrão para si;
- Funcionário opera/conclui Atendimento do qual é responsável;
- Funcionário padrão não reatribui;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, mas não é opção normal para nova atribuição;
- troca de responsável é auditável.

## 7. Procedimentos utilizados

Cada vínculo preserva:

- Procedimento;
- revisão exata;
- código snapshot;
- título snapshot;
- versão editorial snapshot.

Preset:

- Funcionário: revisão publicada que possa ler;
- ADM/Gerência: publicada por padrão; podem selecionar explicitamente revisão histórica/não publicada já autorizada;
- revisão não publicada/histórica nunca é escolhida silenciosamente;
- publicação futura não altera vínculo existente.

Adicionar/remover vínculo exige Atendimento editável. Remoção com checklist/observação existente exige confirmação e preservação do histórico necessário.

## 8. Reader operacional

```text
Atendimento
→ Procedimento vinculado
→ Executar
→ Reader da revisão exata em contexto daquele Atendimento
```

No Reader operacional:

- checklist persiste no Atendimento;
- `Observação do serviço` pode ser registrada por Etapa;
- navegação/stepper não gera progresso;
- nova publicação não muda revisão vinculada;
- lifecycle/capacidade controlam edição.

No Reader standalone:

- checklist é documental;
- nenhuma marcação operacional persiste;
- não existe `Observação do serviço` persistente.

## 9. Checklist operacional

- definição permanece no Procedimento imutável;
- estado marcado/desmarcado fica no Atendimento;
- `Em andamento` + capacidade permite marcar/desmarcar;
- `Concluído`/`Cancelado` ficam somente leitura;
- concorrência granular por item/equivalente;
- evento remoto não sobrescreve estado local silenciosamente.

Checklist incompleto, ao concluir:

```text
Ainda existem itens não concluídos.
→ continuar execução
ou
→ concluir mesmo assim
```

Não existe semântica obrigatório/opcional nos itens documentais iniciais; portanto checklist incompleto avisa, não bloqueia automaticamente.

## 10. Progresso

Derivado apenas do checklist persistente:

```text
PR-001        4 de 6
PR-022        2 de 2
Atendimento   6 de 8
```

- Etapas visitadas não contam;
- `Etapa X de Y` e stepper permanecem navegação;
- observações não contam como progresso;
- revisão sem checklist não mostra `0%` artificial;
- 100% não conclui automaticamente.

## 11. Observação do serviço por Etapa

- opcional;
- ligada a Atendimento + vínculo da revisão + Etapa;
- não altera Procedimento oficial;
- sem persistência no Reader standalone;
- `Em andamento` + capacidade permite editar;
- `Concluído`/`Cancelado` ficam somente leitura até reabertura;
- concorrência granular por Etapa/equivalente;
- evento remoto não sobrescreve texto local em edição;
- sem autosave por inferência;
- participa da reprodução histórica da Ficha.

## 12. Conclusão

Pré-condições:

- status `Em andamento`;
- capacidade;
- responsável definido;
- `Resumo do trabalho` válido;
- conflitos/alterações não confirmadas resolvidos.

Não são obrigatórios por si só:

- OS;
- cliente;
- Equipamento;
- Procedimento vinculado;
- observação por Etapa.

Ao concluir, Host:

- grava `Concluído` e `completed_at`;
- preserva revisões usadas;
- preserva checklist final;
- preserva observações de serviço aplicáveis;
- congela projeção relevante do Equipamento;
- registra evento/auditoria;
- publica mudança pós-commit.

## 13. Cancelamento

- somente `Em andamento`;
- exige capacidade própria;
- motivo curto obrigatório;
- não exclui Atendimento;
- preserva código e histórico;
- após commit fica somente leitura.

Preset: ADM/Gerência sim; Funcionário não.

## 14. Reabertura

- de `Concluído` ou `Cancelado` para `Em andamento`;
- explícita e auditável;
- não apaga lifecycle anterior;
- nova conclusão produz novo estado final aplicável;
- histórico anterior não é reescrito silenciosamente.

Preset: ADM/Gerência sim; Funcionário não.

## 15. Equipamento

Equipamento é opcional e reutilizável.

Para computação, tipos mínimos iniciais:

- Servidor;
- Desktop;
- Notebook.

Pode registrar conforme aplicabilidade:

- nome;
- processador;
- RAM;
- armazenamento;
- SO/versão;
- serial;
- patrimônio;
- múltiplos MACs;
- bateria;
- cliente/responsável relacionado;
- observações.

Bateria é contextual/opcional e percentual 0–100 quando informado. MAC/serial/patrimônio são atributos de busca, não identidade canônica.

Capacidades:

- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- Funcionário vincula/troca/desvincula quando responsável pelo Atendimento editável;
- não arquivar Equipamento vinculado a Atendimento `Em andamento`.

## 16. Histórico após conclusão

Alterar cadastro global do Equipamento depois da conclusão não altera silenciosamente histórico/Ficha final.

A conclusão congela a projeção relevante. Após reabertura e nova conclusão, novo estado final pode ser capturado sem apagar o anterior.

A forma física de persistência permanece decisão técnica de schema, sem criar tabela meramente de apresentação.

## 17. Lista e busca de Atendimentos

Tela 08:

- Status: Em andamento / Concluído / Cancelado;
- filtro por Responsável + Status + Período;
- `Período` usa `started_at`;
- coluna Data representa `started_at`;
- ordenação padrão: mais recente primeiro;
- busca/filtros preservados no retorno quando possível.

Busca operacional pode usar AT, OS/referência, cliente, Equipamento, serial, patrimônio e MAC.

Busca de Processos permanece separada.

## 18. Matriz operacional de capacidades

| Capacidade operacional | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar Atendimentos | sim | sim | sim |
| Criar Atendimento | sim | sim | sim |
| Editar Atendimento próprio em andamento | sim | sim | sim |
| Editar qualquer Atendimento em andamento | sim | sim | não |
| Concluir Atendimento próprio | sim | sim | sim |
| Concluir qualquer Atendimento | sim | sim | não |
| Cancelar Atendimento | sim | sim | não |
| Reabrir Atendimento | sim | sim | não |
| Vincular/trocar/desvincular Equipamento | sim | sim | sim, quando responsável |
| Criar/editar Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica | sim | sim | não |
| Marcar/desmarcar checklist | sim | sim | sim, quando responsável |
| Registrar/editar observação por Etapa | sim | sim | sim, quando responsável |
| Gerar/reimprimir Ficha acessível | sim | sim | sim |
| Gerir categorias | sim | sim | não |

Presets são defaults; autorização real continua Host-side e granular.

Pendentes fora deste bloco:

- Gerência × configuração da empresa;
- Gerência × Backup.

## 19. Ficha compacta — contrato atualmente vigente

A Ficha é prestação de contas resumida ao cliente.

Lifecycle:

- `Em andamento`: pode gerar estado confirmado atual;
- `Concluído`: pode reimprimir estado histórico aplicável;
- `Cancelado`: saída identifica claramente o estado;
- alterações não salvas/conflitos bloqueiam geração.

Conteúdo padrão prioriza identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações aplicáveis.

Não imprime por padrão checklist, progresso, passos, comandos, timeline ou lista detalhada de revisões.

O Bloco 10 consolidou integralmente o contrato documental:

- PDF canônico + preview do mesmo `PagedDocument`;
- exatamente uma A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- soft limits 600/400/300/280;
- sem truncamento, segunda página, resumo automático ou compactação dinâmica;
- Procedimentos vinculados não são listados por padrão;
- MACs: 0 omite; 1–2 valores; 3+ quantidade cadastrada;
- naming e temporários conforme contrato do Bloco 10.

Detalhes: `bloco-10-exportacao-impressao-ficha.md` e `../02-telas/14-exportacao-impressao-ficha.md`.

## 20. Concorrência operacional

### Atendimento

- mutações carregam revisão/base esperada;
- concluir/cancelar/reabrir são mutações versionadas;
- writer/fila define ordem, não validade;
- conflito preserva alterações locais e exige reconciliação.

### Checklist

- controle por item/equivalente;
- itens independentes não conflitam globalmente;
- mesma unidade concorrente recebe resultado determinístico/conflito apropriado.

### Observação de serviço

- controle por Etapa/equivalente;
- Etapas diferentes não conflitam globalmente;
- mesma observação concorrente exige reconciliação;
- evento remoto não sobrescreve texto local.

### Equipamento

Cadastro global possui revisão própria, separada do Atendimento.

## 21. Eventos

Eventos pós-commit podem sinalizar:

- Atendimento criado/alterado;
- status/responsável alterado;
- Equipamento vinculado/alterado;
- Procedimento vinculado/removido;
- checklist alterado;
- observação por Etapa alterada;
- conclusão/reabertura/cancelamento.

Client reconsulta estado relevante e não sobrescreve edição local silenciosamente.

## 22. Impacto no modelo de dados

Sem criar migration neste bloco documental, implementação futura deve refletir:

- status/lifecycle;
- cancelamento + motivo;
- eventos operacionais relevantes;
- checklist por vínculo de revisão;
- observação por Etapa;
- concorrência granular;
- reprodução histórica após reabertura;
- snapshot/projeção de Equipamento por conclusão;
- códigos legíveis Host-only;
- capacidades granulares.

Árvore física/migrations oficiais permanecem para o gate correspondente.

## 23. Pendências reais fora do Bloco 9

### Segurança/configuração

- parâmetros finais de Argon2id/senha/sessão/token;
- Gerência × configuração da empresa;
- Gerência × Backup.

### Categorias/editor

- regra editorial exata para nova revisão ainda referenciando categoria arquivada.

### Ambiente corporativo

- Windows/WebView2;
- Launcher/SMB/permissões;
- transporte aplicável;
- EDR/firewall;
- parâmetros reais de implantação.

### Implementação física

- forma final de schema/migrations;
- estrutura oficial somente no gate correspondente.

## 24. Fora do escopo

- SLA;
- fila complexa de chamados;
- prioridade/severidade obrigatória;
- aprovação gerencial de conclusão;
- atribuição por equipe/skill;
- chat social;
- apontamento complexo de horas;
- financeiro/faturamento;
- estoque/peças;
- RMM/inventário automático;
- workflow customizável;
- checklist com lógica condicional avançada;
- modo offline editável;
- autosave;
- implementação funcional nesta fase.

## 25. Encerramento

O Bloco 9 está concluído. Todas as decisões operacionais acima permanecem vigentes; detalhes posteriores da Ficha foram absorvidos pelo Bloco 10 e não constituem pendência deste documento.
