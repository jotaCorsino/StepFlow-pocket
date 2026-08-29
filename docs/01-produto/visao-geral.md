# Visão Geral do Produto — StepFlow Pocket

**Status:** CONSOLIDADO  
**Atualização:** 2026-08-29

## Propósito

O StepFlow é uma aplicação interna para centralizar Procedimentos técnicos, transformá-los em guias fáceis de consultar/executar e, quando necessário, registrar o trabalho real realizado.

O produto deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias técnicos e outros procedimentos internos sem se transformar em um portal burocrático.

## Usuários

- **ADM:** controle amplo, configurações, usuários, permissões e documentação;
- **Gerência:** gestão delegada conforme capacidades;
- **Funcionário/Técnico:** consulta e execução; por padrão não altera conteúdo oficial.

Autorização efetiva é sempre Host-side e granular.

## Experiência principal

```text
pasta StepFlow publicada no servidor Windows
→ usuário acessa o compartilhamento
→ executa StepFlowLauncher.exe
→ Client local é preparado/validado automaticamente
→ login
→ consulta / execução / registro
```

O usuário não instala o StepFlow nem prepara dependências manualmente no uso normal.

## Domínio

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

### Procedimento

Documentação/modelo oficial reutilizável e versionado.

Campos principais:

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

Categorias são configuráveis, pesquisáveis/filtráveis, podem ser múltiplas e não usam taxonomia hierárquica complexa inicialmente.

### Atendimento / Execução

Ocorrência concreta de trabalho.

- lifecycle inicial: `Em andamento / Concluído / Cancelado`;
- primeiro save aceito cria o registro e código `AT-000001`;
- responsável + `Resumo do trabalho` são obrigatórios para concluir;
- checklist incompleto gera confirmação, não bloqueio automático;
- revisão exata de cada Procedimento utilizado é preservada;
- checklist persistente existe somente em Atendimento;
- cada Etapa pode receber `Observação do serviço` opcional;
- progresso deriva somente do checklist;
- conclusão/reabertura preserva histórico suficiente para reprodução do estado aplicável.

### Equipamento

Ativo opcional e reutilizável.

- código legível `EQP-000001`;
- identidade interna própria;
- serial, patrimônio e MAC são atributos de busca, não identidade canônica;
- múltiplos MACs são permitidos;
- pode registrar, conforme aplicabilidade, tipo, processador, RAM, armazenamento, SO, bateria e observações;
- mudanças futuras no cadastro global não reescrevem silenciosamente a projeção histórica de Atendimento concluído.

Detalhes: `categorizacao-atendimentos-equipamentos.md`.

## Reader — Etapas como páginas de manual

- `Visão geral` precede Etapa 1;
- cada Etapa é uma página lógica própria;
- Anterior/Próxima, Sumário e stepper permitem navegação;
- stepper compacto representa posição/percurso, nunca conclusão operacional;
- comandos/código preservam whitespace e usam ação de copiar discreta/icon-only acessível;
- Reader standalone não persiste checklist nem observação operacional;
- Reader em Atendimento persiste checklist e `Observação do serviço` conforme lifecycle/autorização.

## Busca

`Processos` e `Atendimentos` possuem buscas separadas.

Procedimentos: código, título/termos, área e categoria.

Atendimentos/Equipamentos: código, OS/referência, cliente/solicitante, Equipamento, serial, patrimônio e MAC quando aplicável.

Não criar pesquisa global que misture os domínios sem requisito explícito.

## Exportação, impressão e Ficha compacta

Procedimentos suportam:

- PDF;
- DOCX;
- impressão Windows;
- identidade central da empresa.

A saída é documento próprio, nunca screenshot da UI.

Ficha compacta de Atendimento:

- prestação de contas resumida ao cliente;
- pode existir com ou sem Equipamento;
- PDF canônico + preview derivam do mesmo layout;
- deve ocupar exatamente uma A4 quando válida;
- `2+` páginas geram `SHEET_OVERFLOW`;
- não usa truncamento, segunda página ou redução automática para “caber”;
- checklist/progresso/timeline não são conteúdo padrão da Ficha;
- Procedimentos vinculados não são listados por padrão;
- MACs: 0 omite, 1–2 valores, 3+ quantidade cadastrada.

Detalhes: `../02-telas/14-exportacao-impressao-ficha.md` e documentos do Bloco 10.

## Backup / Restore

Backup/Restore simples é requisito do produto e pertence a Configurações.

UX consolidada:

- operação coordenada pelo Host;
- Client não escolhe SQLite/path;
- Restore normal exige backup elegível, confirmação reforçada e safety backup confirmado;
- disaster recovery sem Host funcional é fluxo técnico/local.

Mecanismo técnico final será fechado no Bloco 11.

## Multiusuário

- Clients nunca acessam SQLite diretamente;
- escritas são coordenadas pelo Host;
- revisão otimista impede sobrescrita silenciosa;
- eventos sinalizam mudanças e Clients reconsultam;
- fila de escrita não substitui controle de revisão;
- checklist/observações operacionais usam granularidade apropriada.

## Requisitos não funcionais

### Pocket

- pasta pronta no servidor;
- zero instalador tradicional por estação;
- preparação local automática;
- zero toolchain de desenvolvimento em produção;
- zero elevação administrativa no uso normal;
- nenhuma Internet obrigatória no uso normal;
- nenhum processo StepFlow após encerramento completo do ciclo central;
- sem serviço persistente como baseline.

### Compatibilidade

Baseline: Windows 10/11 x64 + WebView2. Validação do parque corporativo permanece gate de ambiente real.

### Manutenibilidade

- frontend modular HTML/CSS/JavaScript + ES Modules;
- baixo acoplamento;
- organização por responsabilidade/domínio;
- evitar monólitos e superengenharia.

### Segurança proporcional

- Argon2id;
- sessão opaca;
- autorização Host-side;
- auditoria relevante;
- nenhum segredo/dado real no Git.

## Fora do escopo inicial

- acesso público pela Internet;
- SaaS/multiempresa;
- CRM completo/faturamento;
- estoque de peças;
- RMM/inventário automatizado;
- help desk completo com SLA;
- workflow burocrático complexo;
- chat corporativo;
- edição colaborativa caractere a caractere;
- infraestrutura distribuída de grande porte.

## Pendências de produto ainda reais

- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de nova revisão ainda referenciando categoria arquivada.

Parâmetros técnicos e gates de ambiente pertencem às fontes arquiteturais/planejamento.

## Critério de sucesso

Um técnico deve localizar o Procedimento correto, executar o trabalho com baixo atrito e, quando houver necessidade de registro, manter um Atendimento útil e produzir uma prestação de contas compacta sem depender de controles paralelos dispersos.
