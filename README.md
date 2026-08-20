# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar processos técnicos de forma guiada, com foco em simplicidade operacional e implantação de baixo impacto.

## Estado do projeto

A **Fase 1 — Fechamento arquitetural e especificação** está em andamento.

Os Blocos 0 a 7 estão fechados em nível arquitetural. O próximo bloco é o **Bloco 8 — especificação de UI/UX**. Ainda não existe implementação funcional oficial do produto.

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
- servidor recebe uma pasta pronta do StepFlow, sem Rust, Node, Visual Studio ou banco externo instalados para runtime;
- quando o StepFlow está fechado, não deve restar processo StepFlow consumindo recursos no servidor.

## Documentação principal

Comece por:

1. `AGENTS.md` — regras obrigatórias para agentes/Codex;
2. `docs/README.md` — índice da documentação vigente;
3. `docs/01-produto/visao-geral.md` — produto e requisitos;
4. `docs/03-arquitetura/arquitetura-vigente.md` — visão técnica consolidada;
5. `docs/05-progresso/registro-de-decisoes.md` — decisões e pendências atuais;
6. `docs/04-planejamento/plano-oficial-fase-1.md` — estado e próximos blocos.

## Ambiente de desenvolvimento

- repositório oficial: `jotaCorsino/StepFlow-pocket`;
- branch principal: `main`;
- checkout local previsto: `C:\dev\StepFlow`;
- desenvolvimento atual fora da LAN corporativa.

Endereços IP, hostnames e caminhos SMB reais da empresa ainda não estão consolidados. Exemplos históricos não podem ser tratados como configuração oficial.

## Regra de implementação

A Fase 1 permite documentação, decisões arquiteturais e provas descartáveis quando realmente necessárias. Código definitivo de negócio só deve começar após os gates correspondentes e o plano da Fase 2.
