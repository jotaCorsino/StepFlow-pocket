# Registro de Decisões — StepFlow Pocket

## Objetivo

Registrar decisões relevantes do projeto com data, status e consequência, preservando distinção entre o que já foi aprovado e o que ainda é apenas direção/proposta.

---

## Decisões consolidadas

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

### 2026-08-19 — O StepFlow será iniciado pelo técnico a partir do compartilhamento de rede

**Status:** CONSOLIDADA COMO REQUISITO DE UX

Cenário de uso desejado:

`\\192.168.5.7\Arquivos\StepFlow\` → ponto de entrada do StepFlow → duplo clique → login → uso.

A implementação técnica pode usar launcher/cópia local desde que preserve essa experiência.

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

O ponto de entrada localizado no compartilhamento pode verificar versão, manter cópia local e iniciar o Client local, preservando a experiência de duplo clique para o técnico.

### 2026-08-19 — Canal de eventos em tempo real

**Status:** PROPOSTA / TECNOLOGIA PENDENTE

WebSocket ou solução equivalente será avaliada para atualização dos clientes.

### 2026-08-19 — StepFlow Host como serviço/processo leve central

**Status:** DIREÇÃO ARQUITETURAL / FORMATO FINAL PENDENTE

O Host é parte consolidada da arquitetura lógica. Tecnologia, empacotamento e modo de inicialização no Windows ainda precisam ser definidos.
