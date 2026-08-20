# Registro de Decisões — StepFlow Pocket

## Objetivo

Registrar decisões relevantes do projeto com data, status e consequência, preservando distinção entre o que já foi aprovado e o que ainda é apenas direção/proposta.

---

## Decisões consolidadas

### 2026-08-20 — O Host será sob demanda; processo residente permanente é rejeitado

**Status:** CONSOLIDADA COMO REQUISITO ARQUITETURAL

O conceito Pocket exige mais do que copy-deploy. No servidor/máquina central, o StepFlow não deve permanecer consumindo recursos quando não estiver sendo utilizado.

Consequências obrigatórias:

- nenhum Windows Service persistente como solução padrão;
- nenhum serviço auto-start;
- nenhuma tarefa agendada, watchdog, tray agent ou daemon residente apenas para manter o StepFlow disponível;
- processos StepFlow iniciam sob demanda quando o produto entra em uso;
- quando o uso termina, todos os processos transitórios StepFlow devem encerrar de forma controlada;
- quando fechado/sem uso, o consumo de CPU/memória do StepFlow deve tender a zero;
- copiar/publicar a pasta pronta continua sendo o modelo de implantação desejado;
- nenhuma instalação de Rust, Node.js, Visual Studio, SQLite Server ou toolchain no servidor.

A PoC de Windows Service realizada no PC de desenvolvimento permanece apenas como evidência técnica descartável e **não é decisão de produção**.

A questão ainda aberta é o bootstrap/orquestração do Host central sob demanda para múltiplos Clients.

Referências: `docs/03-arquitetura/implantacao-pocket.md` e `docs/03-arquitetura/host-operacao-windows.md`.

### 2026-08-19 — O nome do aplicativo é StepFlow

**Status:** CONSOLIDADA

O produto Pocket passa a ser identificado como **StepFlow**.

### 2026-08-19 — GitHub é a fonte principal de verdade

**Status:** CONSOLIDADA

O repositório `jotaCorsino/StepFlow-pocket` centraliza documentação, decisões e futura implementação.

A pasta local prevista para execução pelo Codex é `C:\dev\StepFlow`.

### 2026-08-19 — O projeto seguirá o método PO + Assistente + Codex

**Status:** CONSOLIDADA

- PO define produto, prioridade e aprovação;
- Assistente analisa, arquiteta, documenta e transforma decisões em tarefas;
- Codex executa tecnicamente dentro de escopo explícito.

O método reutilizável está em `docs/00-governanca/metodo-padrao-trabalho-assistido.md`.

### 2026-08-19 — Toda tarefa Codex terá pré-flight de capacidade separado do prompt

**Status:** CONSOLIDADA

Antes de cada nova tarefa destinada ao Codex, o Assistente deve avaliar complexidade, risco e custo da execução e recomendar ao PO o modelo e o nível de raciocínio adequados.

Essa recomendação:

- é destinada somente ao PO/usuário;
- deve aparecer antes e separada do prompt técnico;
- não deve ser incorporada ao enunciado enviado ao Codex;
- deve buscar a menor capacidade que ainda ofereça margem adequada de segurança;
- deve ser refeita para cada tarefa, sem herdar automaticamente a capacidade da tarefa anterior.

A política genérica e reutilizável está em `docs/00-governanca/politica-capacidade-codex.md` e o formato padrão em `docs/templates/template-preflight-capacidade-codex.md`.

### 2026-08-19 — A implementação deve ser modular e evitar monólito HTML/JavaScript

**Status:** CONSOLIDADA

A interface poderá utilizar HTML, CSS e JavaScript, mas o código deve ser organizado em módulos, componentes e domínios. ES Modules são a direção padrão. Classes serão usadas apenas onde estado/comportamento justificarem.

### 2026-08-19 — O modelo de dados de processo será enxuto

**Status:** CONSOLIDADA

Campos principais aprovados:

- Código;
- Título;
- Área / Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas do processo;
- Histórico de alterações.

Campos burocráticos adicionais não devem ser incluídos sem necessidade aprovada.

### 2026-08-19 — Etapas serão apresentadas como páginas de um manual/livro

**Status:** CONSOLIDADA

Cada etapa deve possuir sua própria experiência de leitura, com passos, observações, checklist e blocos copiáveis quando necessário. A navegação deve reforçar a ideia de páginas/etapas sequenciais.

### 2026-08-19 — O controle de cópia será somente por ícone

**Status:** CONSOLIDADA

Blocos destinados a comandos, caminhos ou instruções copiáveis utilizarão ícone discreto de cópia, sem botão textual grande.

### 2026-08-19 — O logo ficará na sidebar esquerda

**Status:** CONSOLIDADA

O logo da empresa será pequeno, clássico, alinhado à esquerda e posicionado no topo do menu lateral.

### 2026-08-19 — O StepFlow será iniciado pelo técnico a partir de um ponto de entrada na rede interna

**Status:** CONSOLIDADA COMO REQUISITO DE UX

O requisito consolidado é a experiência:

`ponto de entrada interno do StepFlow` → duplo clique → login → uso.

O endereço IP, hostname, nome do compartilhamento e subpasta reais ainda não estão definidos/confirmados.

Até que o ambiente corporativo seja conhecido, usar somente notação conceitual, por exemplo:

`\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\`

O exemplo `\\192.168.5.7\Arquivos\StepFlow\` usado anteriormente **não é uma configuração oficial** e não deve ser embutido em código ou tratado como requisito.

A implementação técnica pode usar launcher/cópia local desde que preserve a experiência simples de acesso aprovada.

### 2026-08-19 — Desenvolvimento atual e implantação corporativa são ambientes distintos

**Status:** CONSOLIDADA

O projeto está sendo desenvolvido em um computador pessoal fora da LAN da empresa. Testes de infraestrutura interna, como SMB, Host real, permissões de rede e caminhos corporativos, só terão valor de validação definitiva quando realizados em ambiente conectado à rede da empresa e usando endereços reais confirmados.

Resultados de acesso a caminhos internos enquanto o desenvolvimento ocorrer fora da LAN devem ser classificados como `NÃO APLICÁVEL NESTE AMBIENTE`, e não como bloqueio do produto.

Referência: `docs/00-governanca/contexto-ambientes.md`.

### 2026-08-19 — A implantação no servidor seguirá o princípio Pocket / copy-deploy

**Status:** CONSOLIDADA COMO REQUISITO ARQUITETURAL

O servidor Windows da empresa já executa outros serviços e não poderá ser remodelado livremente para receber o StepFlow.

A implantação deve buscar a menor interferência possível no ambiente. O cenário ideal é copiar/arrastar uma pasta pronta do StepFlow para uma pasta fixa do servidor e realizar apenas a configuração/inicialização mínima necessária.

Consequências obrigatórias para as escolhas técnicas:

- evitar instalação tradicional quando possível;
- evitar dependências globais no servidor;
- evitar alterações amplas em registro, PATH, políticas ou features do Windows;
- evitar reinicialização do servidor em implantação/atualização normal;
- não exigir Node.js, Rust, compiladores ou toolchain de desenvolvimento no servidor de produção;
- favorecer binários/artefatos self-contained ou equivalentes;
- manter configuração, dados e logs isolados do restante do sistema;
- permitir atualização e rollback com substituição controlada de artefatos/pastas;
- preservar os demais serviços existentes no servidor.

Referência: `docs/03-arquitetura/implantacao-pocket.md`.

### 2026-08-19 — Login interno simples continua obrigatório

**Status:** CONSOLIDADA

O sistema terá autenticação local para uso interno. Não há necessidade inicial de MFA, recuperação por email ou autenticação pública complexa.

Senhas não serão armazenadas em texto puro.

### 2026-08-19 — Perfis padrão serão ADM, Gerência e Funcionário

**Status:** CONSOLIDADA

- ADM: controle total;
- Gerência: criação/edição de documentações e poderes administrativos delegados;
- Funcionário: leitura/execução, sem edição da documentação oficial por padrão.

Gerência não deve poder criar/promover ADM ou alterar a autoridade do ADM principal sem futura decisão explícita.

### 2026-08-19 — Usuários poderão editar dados do próprio perfil

**Status:** CONSOLIDADA

Cada conta pode editar avatar, nome de exibição, cargo e senha dentro das regras de autorização. Relacionamentos internos/históricos devem usar identificador estável, não apenas o nome exibido.

### 2026-08-19 — Uso simultâneo por vários computadores é requisito obrigatório

**Status:** CONSOLIDADA

O projeto deve suportar múltiplas instâncias do Client utilizando a mesma base central sem corrupção ou sobrescrita silenciosa.

### 2026-08-19 — Clientes não acessarão diretamente um SQLite compartilhado pela rede

**Status:** CONSOLIDADA

O SQLite será acessado por uma camada Host localizada junto aos dados. Clients não abrirão diretamente o mesmo `.sqlite` por SMB.

### 2026-08-19 — Fila de escrita não substitui controle de revisão

**Status:** CONSOLIDADA

A arquitetura deve combinar ordenação/serialização das escritas necessárias com detecção de conflito baseada em revisão/versão, impedindo que uma edição antiga sobrescreva silenciosamente uma nova.

### 2026-08-19 — Atualizações relevantes devem chegar aos clientes sem refresh manual desnecessário

**Status:** CONSOLIDADA COMO REQUISITO FUNCIONAL

O mecanismo técnico ainda será definido, mas alterações relevantes devem ser propagadas para as instâncias conectadas.

### 2026-08-19 — Exportação para PDF e DOCX permanece no escopo do produto

**Status:** CONSOLIDADA COMO REQUISITO DE PRODUTO

O sistema deverá oferecer exportação para PDF e DOCX, além de impressão. A Fase 1 deverá validar a estratégia e as bibliotecas para implementar esse requisito de forma offline e manutenível. A existência da função não depende dessa validação; apenas sua solução técnica.

O modelo exportável será separado da tela de execução.

### 2026-08-19 — O estado marcado do checklist durante uma execução ainda não está definido

**Status:** CONSOLIDADA COMO PENDÊNCIA DE PRODUTO

O checklist faz parte da documentação de uma etapa. Porém, ainda será decidido se as marcações feitas pelo técnico durante o uso são apenas temporárias, locais por usuário/dispositivo, persistidas no Host ou registradas como uma entidade formal de execução.

Nenhuma dessas alternativas deve ser implementada como padrão antes da decisão correspondente.

### 2026-08-19 — O projeto começa por documentação e arquitetura

**Status:** CONSOLIDADA

Nenhuma implementação funcional ampla está autorizada antes do fechamento da fundação documental e do gate arquitetural correspondente.

### 2026-08-19 — A Fase 0 foi concluída e a Fase 1 está autorizada

**Status:** CONSOLIDADA

A revisão cruzada da fundação documental foi concluída e registrada em `docs/05-progresso/revisao-cruzada-fase-0.md`.

A próxima fase autorizada é **Fase 1 — Fechamento arquitetural e especificação**, conforme `docs/04-planejamento/plano-oficial-fase-1.md`.

A Fase 1 autoriza investigação, especificação, decisões técnicas e provas descartáveis quando necessárias, mas não autoriza antecipar funcionalidades de negócio das fases seguintes.

---

## Direções propostas que ainda exigem validação

### 2026-08-19 — Tauri para o StepFlow Client

**Status:** PROPOSTA / VALIDAÇÃO TÉCNICA PENDENTE

Tauri é a direção atual para empacotar a UI HTML/CSS/JavaScript como aplicativo Windows, mas compatibilidade, runtime e distribuição ainda serão validados formalmente.

### 2026-08-19 — Launcher de rede com execução local do Client

**Status:** PROPOSTA / PROTÓTIPO PENDENTE

Um ponto de entrada localizado no compartilhamento ou mecanismo equivalente da rede interna pode verificar versão, manter cópia local e iniciar o Client local, preservando a experiência de duplo clique para o técnico.

O caminho físico real ainda será definido no ambiente corporativo.

### 2026-08-19 — Canal de eventos em tempo real

**Status:** PROPOSTA / TECNOLOGIA PENDENTE

WebSocket ou solução equivalente será avaliada para atualização dos clientes.

### 2026-08-20 — Bootstrap/orquestração do Host central sob demanda

**Status:** PROPOSTA / MECANISMO PENDENTE

O Host é parte consolidada da arquitetura lógica, mas seu disparo na máquina central precisa ocorrer sem serviço persistente. A solução deve permitir múltiplos Clients enquanto o Host estiver ativo e desligá-lo com segurança quando o StepFlow deixar de ser utilizado.
