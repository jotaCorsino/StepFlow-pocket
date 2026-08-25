# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com foco em simplicidade operacional e implantação de baixo impacto.

## Painel de acompanhamento do projeto

**Atualização:** 2026-08-25  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Bloco atual:** Bloco 8 — UI/UX  
**Implementação funcional oficial:** ainda não iniciada

Este painel é a visão rápida de andamento. Ele **não substitui** as fontes de decisão: `AGENTS.md`, `docs/05-progresso/registro-de-decisoes.md`, documentos específicos e `docs/04-planejamento/plano-oficial-fase-1.md` continuam sendo autoritativos.

### Andamento da Fase 1

| Bloco | Tema | Estado |
|---|---|---|
| 0 | Bootstrap do ambiente | ✅ Concluído |
| 1 | Client Windows / Tauri | ✅ Concluído |
| 2 | Host Pocket | ✅ Concluído |
| 3 | Launcher / distribuição | ✅ Concluído |
| 4 | Comunicação Client ↔ Host | ✅ Concluído |
| 5 | Autenticação / autorização | ✅ Núcleo concluído; parâmetros finais pendentes |
| 6 | Dados / schema / migrations | ✅ Núcleo + extensão operacional conceitual aprovados |
| 7 | Concorrência / fila / eventos | ✅ Núcleo concluído |
| 8 | UI/UX | 🟡 Em andamento |
| 9 | Atendimentos / execução / checklist | ⏳ Pendente |
| 10 | Exportação / impressão / ficha compacta | ⏳ Pendente |
| 11 | Backup / restauração | ⏳ Pendente |
| 12 | Estrutura oficial + plano da Fase 2 | ⏳ Pendente |

### Bloco 8 — telas e decisões

| Ordem | Superfície | Estado |
|---|---|---|
| 1 | Login | ✅ Consolidado |
| 2 | Shell / sidebar | ✅ Consolidado |
| 3 | Início / Dashboard | ✅ Consolidado |
| 4 | Lista / pesquisa de Processos | ✅ Consolidado |
| 5 | Leitor em formato livro | ✅ Consolidado |
| 6 | Editor de Processo + categorias | ✅ Consolidado |
| 7 | Histórico / revisões | ✅ Consolidado |
| 8 | Lista / pesquisa de Atendimentos | ✅ Consolidado |
| 9 | Atendimento / execução + equipamento | ✅ Consolidado |
| 10 | Usuários / permissões | 🟡 Próximo |
| 11 | Meu perfil | ⏳ Pendente |
| 12 | Configurações + categorias | ⏳ Pendente |
| 13 | Backup / restauração — UX | ⏳ Pendente |
| 14 | Exportação / impressão + ficha — UX | ⏳ Pendente |
| 15 | Estados transversais | ⏳ Pendente |

### Extensão de produto já aprovada

O StepFlow não fica restrito a manutenção de computadores. O produto deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias e outros procedimentos internos.

Estão aprovados conceitualmente:

- categorias configuráveis e múltiplas para procedimentos;
- separação `Procedimento × Atendimento/Execução × Equipamento`;
- `Atendimentos` como área operacional própria;
- equipamento opcional e reutilizável com identidade interna estável;
- MAC, serial, patrimônio, cliente e OS/referência como atributos de busca, não identidade canônica exclusiva;
- múltiplos procedimentos por atendimento;
- vínculo histórico com a revisão do procedimento realmente utilizada;
- ficha compacta imprimível de atendimento/equipamento;
- para computadores, tipos mínimos `Servidor`, `Desktop` e `Notebook`;
- saúde da bateria contextual para `Notebook`;
- observações curtas e limitadas do equipamento;
- ficha compacta com no máximo uma página A4 e cabeçalho com identidade da empresa.

Lifecycle, permissões operacionais e checklist/progresso pertencem ao Bloco 9. Template final, margens, pré-visualização e tecnologia de geração/impressão da ficha compacta pertencem ao Bloco 10.

### Próximo passo

**Tela 10 — Usuários / permissões.**

A Tela 10 ainda **não está em análise nem aprovada** neste checkpoint. Ela só será aberta após a Tela 09 estar integrada em `main`, a branch encerrada e o checkout local sincronizado.

Antes de implementação funcional, a Fase 1 ainda precisa concluir os Blocos 8–12 e seus gates.

### Regra de atualização deste painel

Todo avanço consolidado de **fase, bloco ou tela** deve atualizar este README **no mesmo checkpoint documental**. Um avanço não é considerado documentalmente encerrado se o painel de acompanhamento permanecer atrasado.

## Papéis no desenvolvimento

| Papel | Responsabilidade |
|---|---|
| **PO** | define requisitos, prioridades, regras de negócio e aprova UX/visual |
| **Assistente** | analisa, arquiteta, mantém documentação coerente, controla fases/gates e prepara tarefas fechadas |
| **Codex** | executa tecnicamente uma tarefa pequena e aprovada, sem inventar produto ou ampliar escopo |

Fluxo esperado:

```text
PO aprova requisito/decisão
        ↓
Assistente consolida especificação
        ↓
pré-flight de capacidade Codex
        ↓
uma tarefa fechada e verificável
        ↓
Codex executa dentro do escopo
        ↓
validação + evidências
        ↓
aceite / correção
```

Durante a Fase 1 o Codex não deve criar scaffold/runtime oficial nem implementação funcional do produto, salvo PoC explicitamente autorizada e descartável.

## Arquitetura vigente

```text
ponto de entrada interno
        ↓
launcher transitório
        ↓
Client Tauri local em %LOCALAPPDATA%
        ↓ HTTP/JSON + WebSocket
Host Pocket Rust sob demanda
        ↓
SQLite local + arquivos persistentes
```

Princípios obrigatórios:

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + SQLite bundled;
- múltiplos Clients simultâneos;
- Clients nunca abrem SQLite diretamente;
- revisão otimista + writer coordenado no Host;
- launcher e Host não permanecem residentes sem necessidade;
- máquina central recebe pasta pronta do StepFlow, sem toolchain de desenvolvimento exigida em runtime;
- quando o ciclo central StepFlow termina, não deve restar processo StepFlow consumindo recursos.

## Documentação principal

Comece por:

1. `AGENTS.md` — regras obrigatórias para agentes/Codex;
2. `docs/README.md` — índice da documentação vigente;
3. `docs/05-progresso/registro-de-decisoes.md` — decisões e pendências atuais;
4. `docs/04-planejamento/plano-oficial-fase-1.md` — plano, gates e detalhes do andamento;
5. `docs/01-produto/visao-geral.md` — produto e requisitos;
6. `docs/03-arquitetura/arquitetura-vigente.md` — visão técnica consolidada;
7. `docs/02-telas/README.md` — andamento detalhado do Bloco 8.

## Disciplina de Git

Para manter o repositório limpo:

```text
1 trabalho lógico
→ 1 branch ativa
→ 1 PR
→ revisão/aprovação
→ squash/merge em main
→ apagar branch encerrada
→ sincronizar checkout local
→ iniciar o próximo trabalho
```

Branches auxiliares descartáveis não devem permanecer no remoto. Alterações locais preexistentes do PO não podem ser resetadas, stashed, descartadas ou incorporadas por agentes sem autorização explícita.

## Ambiente de desenvolvimento

- repositório oficial: `jotaCorsino/StepFlow-pocket`;
- branch principal: `main`;
- checkout local previsto: `C:\dev\StepFlow`;
- desenvolvimento atual fora da LAN corporativa.

Endereços IP, hostnames e caminhos SMB reais da empresa ainda não estão consolidados. Exemplos históricos não podem ser tratados como configuração oficial.
