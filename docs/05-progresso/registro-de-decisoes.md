# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-25

Este arquivo registra decisões vigentes e pendências atuais. Propostas não aprovadas não podem ser tratadas como contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade durante o fechamento documental restante da Fase 1;
- checkout local `C:\dev\StepFlow` será sincronizado explicitamente antes do primeiro trabalho de implementação com Codex;
- alterações locais preexistentes do PO devem ser preservadas;
- uma tarefa lógica por vez;
- uma branch ativa por trabalho;
- um PR por trabalho;
- revisão/aprovação → squash/merge → limpeza de branch quando possível → próximo trabalho;
- `AGENTS.md` é a regra operacional superior;
- todo avanço consolidado de fase/bloco/tela atualiza o README raiz no mesmo checkpoint;
- toda tarefa Codex futura exige pré-flight de capacidade separado do prompt;
- Fase 1 não autoriza runtime/scaffold/código de negócio oficial antes do gate correspondente.

## 2. Papéis

- PO: requisitos, prioridade, regra de negócio e aprovação final;
- Assistente: análise, arquitetura, coerência documental, gates e tarefas fechadas;
- Codex: implementação de tarefa pequena/aprovada, sem inventar produto.

## 3. Produto

StepFlow é aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada.

Uso amplo:

- manutenção;
- TI;
- Service Desk/Help Desk;
- infraestrutura/servidores;
- redes;
- procedimentos internos;
- guias técnicos.

Não transformar por inferência em:

- CRM;
- financeiro/faturamento;
- estoque;
- RMM;
- sistema completo de chamados/SLA.

## 4. Modelo de domínio

Consolidado:

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação/modelo oficial;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo físico opcional e reutilizável;
- Atendimento pode existir sem Equipamento;
- Atendimento pode usar zero, um ou vários Procedimentos;
- vínculo preserva revisão exata realmente utilizada;
- alteração futura do Procedimento não reescreve histórico do Atendimento.

## 5. Categorias

- configuráveis pela empresa;
- múltiplas por Procedimento;
- simples, sem árvore hierárquica inicial;
- pesquisáveis/filtráveis;
- arquivamento preserva histórico;
- nomes equivalentes normalizados devem ser evitados;
- gestão por preset: ADM e Gerência;
- Funcionário não gere categorias por preset;
- autorização real continua Host-side/capability-based.

Pendente:

- regra editorial de nova revisão de Procedimento ainda referenciando categoria arquivada.

## 6. Campos principais do Procedimento

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Categorias;
- Etapas;
- Histórico.

## 7. Reader / manual

- experiência principal em formato livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa = uma página;
- `Sumário` temporário;
- `Anterior`/`Próxima`;
- `Etapa X de Y` representa posição, não conclusão;
- blocos tipados: paragraph, numbered_steps, checklist, note, warning, command, code;
- sem HTML arbitrário;
- comando/código preserva whitespace e nunca é executado;
- copiar usa botão icon-only acessível + feedback curto;
- revisão aberta permanece estável quando nova revisão aparece;
- revisão histórica recebe identificação persistente.

### Reader standalone

- checklist é documental;
- marcação não persiste execução;
- navegação não grava progresso.

### Reader em Atendimento

- cabeçalho identifica `Executando no atendimento AT-...`;
- revisão fica presa ao vínculo;
- checklist é persistente;
- voltar retorna ao Atendimento;
- lifecycle do Atendimento controla editabilidade do checklist.

## 8. Editor e revisões de Procedimento

- Editor = `Informações` + `Etapas`;
- painel local `Estrutura`, sem segunda sidebar global;
- categorias existentes, sem criação inline;
- blocos tipados apenas;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- `base_revision`/controle otimista;
- `409` preserva alterações locais;
- sem merge automático;
- publicar é ação separada de salvar;
- revisão histórica nunca é alterada/destruída;
- `Criar nova revisão a partir desta` cria novo trabalho baseado no snapshot antigo;
- `revision_no` técnico separado de `display_version` editorial.

## 9. Atendimentos — lifecycle do Bloco 9

Estados iniciais:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho Client
→ primeiro save aceito
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- abrir tela nova não persiste registro;
- primeiro save cria ID, código e `started_at`;
- não existe draft persistente inicial;
- Concluído/Cancelado são read-only operacionalmente;
- correção posterior exige reabertura;
- lifecycle é auditável/versionado.

## 10. Códigos legíveis

Consolidado:

```text
Atendimento: AT-000001
Equipamento: EQP-000001
```

- Host-only;
- seis dígitos;
- sequência simples por implantação/banco ativo;
- gaps permitidos;
- não editáveis;
- não substituem IDs internos estáveis;
- sem ano/departamento/hostname/dado pessoal inicialmente.

## 11. Responsável do Atendimento

- necessário para conclusão;
- Funcionário cria inicialmente para si;
- Funcionário padrão edita/conclui apenas Atendimento do qual é responsável;
- Funcionário não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, não em novas atribuições normais.

## 12. Conclusão

Para `Concluir atendimento`:

- estado `Em andamento`;
- capacidade apropriada;
- responsável definido;
- `Resumo do trabalho` obrigatório;
- alterações locais relevantes salvas;
- sem conflito/base obsoleta.

Não são obrigatórios por si só:

- OS;
- cliente;
- Equipamento;
- Procedimento vinculado.

Checklist incompleto:

- gera aviso/confirmação;
- não bloqueia automaticamente;
- não há semântica obrigatório/opcional nos itens iniciais.

Ao concluir:

- status `Concluído`;
- Host define `completed_at`;
- preserva revisões/checklist final;
- congela projeção relevante do Equipamento;
- grava evento de conclusão.

## 13. Cancelamento

- somente `Em andamento`;
- preset ADM/Gerência;
- Funcionário não por preset;
- exige motivo curto;
- não exclui registro;
- preserva código/histórico;
- bloqueia edição direta;
- pode ser reaberto quando autorizado.

## 14. Reabertura

- Concluído/Cancelado → Em andamento;
- preset ADM/Gerência;
- Funcionário não;
- explícita/auditável;
- preserva histórico anterior;
- nova conclusão gera novo estado final.

## 15. Procedimentos usados em Atendimento

- vínculo com revisão exata;
- snapshots de código/título/versão;
- nova publicação não altera vínculo existente;
- Funcionário seleciona revisão publicada que possa ler;
- ADM/Gerência podem selecionar explicitamente histórica/não publicada já autorizada;
- revisão histórica/não publicada nunca é escolhida silenciosamente;
- remoção só em Atendimento editável;
- remoção com checklist marcado exige confirmação.

## 16. Checklist operacional

Estado separado da revisão documental.

Cada item precisa preservar conceitualmente:

- identidade de execução;
- vínculo Atendimento × Procedimento/revisão;
- origem no item documental;
- texto snapshot quando necessário;
- marcado/desmarcado;
- data/usuário;
- revisão/controle concorrente próprio ou equivalente.

Regras:

- somente Atendimento `Em andamento` + capacidade permite marcar/desmarcar;
- Concluído/Cancelado = somente leitura;
- 100% não conclui automaticamente;
- Reader standalone não persiste estado.

## 17. Progresso operacional

Derivado exclusivamente de checklists:

```text
PR-001        4 de 6
PR-022        2 de 2
Atendimento   6 de 8
```

- etapas visitadas não contam;
- `Etapa X de Y` não é percentual;
- revisão sem checklist não mostra `0%` artificial.

## 18. Equipamento

Opcional/reutilizável.

Campos para computadores conforme aplicável:

- nome;
- tipo;
- CPU;
- RAM;
- armazenamento;
- SO/versão;
- serial;
- patrimônio;
- múltiplos MACs;
- bateria para Notebook;
- cliente/responsável relacionado;
- observações curtas.

Tipos mínimos:

- Servidor;
- Desktop;
- Notebook.

Não virar enum global rígida.

Bateria:

- opcional/contextual;
- percentual 0–100 quando informado.

MAC:

- múltiplos;
- label opcional;
- normalizado pelo Host;
- não é identidade canônica.

## 19. Capacidades de Equipamento

Preset:

- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- Funcionário vincula/troca/desvincula em Atendimento editável do qual é responsável.

Não arquivar Equipamento vinculado a Atendimento `Em andamento`.

## 20. Snapshot histórico do Equipamento

Alteração posterior do cadastro global não reescreve Atendimento concluído/ficha final.

Cada conclusão congela projeção relevante do Equipamento. Reabertura + nova conclusão pode gerar novo estado final; histórico anterior permanece.

## 21. Lista/Pesquisa de Atendimentos

- tabela compacta;
- busca por Atendimento/OS/cliente/Equipamento/serial/patrimônio/MAC;
- filtros Responsável + Status + Período;
- Status = Em andamento/Concluído/Cancelado;
- Data/Período usam `started_at`;
- ordenação padrão `started_at DESC`;
- linha abre Tela 09;
- retorno preserva busca/filtros/ordenação/página/scroll;
- busca de Atendimentos permanece separada de Processos.

## 22. Matriz operacional

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

Presets são defaults. Autorização real é granular e Host-side.

## 23. Usuários/segurança

- Gerência não administra ADM;
- usuário não eleva a própria autoridade;
- pelo menos um ADM ativo;
- `is_primary_admin` não é toggle comum;
- senha Argon2id;
- sessão opaca server-side;
- token em memória;
- troca da própria senha mantém sessão corrente e revoga demais sessões da conta;
- sessão expirada exige nova autenticação.

Pendentes:

- custo final Argon2id;
- senha mínima;
- duração da sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup.

## 24. Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista;
- `409` para estado obsoleto;
- eventos pós-commit;
- sem soft/hard lock inicial;
- Atendimento e Equipamento têm revisões independentes;
- checklist usa controle granular por item/equivalente;
- fila ordena, não valida edição obsoleta;
- mutação de resultado incerto exige reconciliação, não retry cego;
- evento remoto não sobrescreve formulário local.

## 25. Estados transversais

- menor superfície: campo → seção → página → Shell;
- sem indicador permanente de conexão saudável;
- loading não mostra cache antigo como atual;
- `sem registros` distinto de `sem resultados`;
- Host indisponível separado de WebSocket degradado quando HTTP continua saudável;
- perda de permissão limpa conteúdo protegido;
- conflito preserva edição local;
- incompatibilidade Client↔Host bloqueia uso;
- sem offline queue/autosave/draft persistente.

## 26. Ficha compacta

- pertence ao Atendimento;
- com ou sem Equipamento;
- estado confirmado do Host;
- Em andamento: pode gerar para acompanhamento;
- Concluído: reimprime estado histórico aplicável;
- Cancelado: saída identifica claramente o estado;
- capacidade padrão para ADM/Gerência/Funcionário em Atendimento acessível;
- máximo uma página A4;
- não gerar segunda página normal;
- conteúdo excessivo bloqueia saída em vez de truncamento silencioso;
- cabeçalho usa identidade da empresa;
- impressão é requisito;
- DOCX específico não é requisito inicial.

Bloco 10 fecha engine/template/margens/limites/preview/PDF específico/QR se aprovado.

## 27. Backup/Restore

- dentro de Configurações;
- Host coordena;
- Client não escolhe SQLite/path;
- Restore exige capacidade + backup elegível;
- confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva normal;
- disaster recovery sem Host funcional fica no Bloco 11.

## 28. Pocket / implantação

- máquina central recebe pasta pronta;
- sem instalação normal/toolchain;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon;
- Controller inicia Host como filho;
- fechar Client individual não encerra Host;
- encerrado ciclo central, zero processo StepFlow residual;
- auto-shutdown por último Client/timeout não está consolidado;
- launcher em workstation não inicia remotamente Host central por si só.

## 29. Tecnologias consolidadas

Client:

- Tauri 2;
- HTML/CSS/JS modular;
- Windows 10/11 x64;
- WebView2.

Host:

- Rust;
- Tokio/Axum;
- `rusqlite` bundled;
- SQLite Host-only.

Comunicação:

- HTTP/JSON `/api/v1` inicialmente;
- WebSocket autenticado;
- handshake de compatibilidade;
- `deployment.json` sem segredos;
- sem edição offline.

## 30. Estado da Fase 1

- Bloco 0: concluído;
- Bloco 1: concluído;
- Bloco 2: concluído;
- Bloco 3: concluído;
- Bloco 4: concluído;
- Bloco 5: núcleo concluído / parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Bloco 8: concluído;
- **Bloco 9: concluído**;
- **Bloco 10: próximo, ainda não iniciado**;
- Bloco 11: pendente;
- Bloco 12: pendente.

## 31. Pendências vigentes

### Bloco 10

- engines PDF/DOCX/impressão;
- template físico final da ficha;
- margens/tipografia/densidade;
- limites numéricos dos textos da ficha;
- preview;
- impressão Windows;
- PDF específico da ficha;
- tratamento de muitos MACs/Procedimentos;
- QR/barcode se houver valor.

### Bloco 11

- mecanismo/pacote de Backup/Restore;
- atomicidade/checksums;
- retenção;
- restart/reconexão/sessões;
- disaster recovery local.

### Bloco 12 / antes da implementação correspondente

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão;
- árvore oficial/migrations/scripts/testes;
- plano Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

### Ambiente corporativo

- hostname/IP/path reais;
- SMB/permissões;
- WebView2/Windows reais;
- HTTP/HTTPS;
- EDR/firewall;
- mecanismo real de start central.

## 32. Regra de precedência

Em divergência:

1. `AGENTS.md`;
2. este registro de decisões, na versão mais recente;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.