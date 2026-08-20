# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-20

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates foram estabelecidos. O histórico detalhado permanece no Git e no diário.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO.**

Blocos concluídos:

- Client/Tauri e compatibilidade Windows;
- Host Pocket sob demanda;
- launcher e atualização do Client;
- comunicação HTTP/JSON + WebSocket;
- autenticação/sessão/autorização;
- modelo de dados/migrations/histórico;
- concorrência/fila/conflitos/eventos.

Próximos blocos:

1. UI/UX;
2. comportamento do checklist durante execução;
3. exportação PDF/DOCX/impressão;
4. backup/restore;
5. estrutura oficial do repositório e plano da Fase 2.

Detalhes em `plano-oficial-fase-1.md`.

## Fase 2 — Fundação técnica executável

**PENDENTE.**

Criar somente a fundação real:

- árvore oficial de Client/launcher/Controller/Host;
- builds reproduzíveis;
- configuração de desenvolvimento;
- comunicação mínima e health/readiness;
- SQLite + migrations iniciais;
- logging mínimo;
- testes de fundação.

Gate: Client abre, Host Pocket inicia sob demanda, comunicação mínima funciona, banco inicializa deterministicamente e build limpo passa.

## Fase 3 — Autenticação, usuários e shell

**PENDENTE.**

- login/logout/sessão;
- bootstrap ADM;
- usuários/permissões;
- perfil/avatar;
- shell/sidebar;
- configuração básica da empresa;
- autorização real no Host.

## Fase 4 — Núcleo documental de processos

**PENDENTE.**

- lista/pesquisa;
- criação/edição/arquivamento;
- etapas e blocos estruturados;
- histórico/revisões;
- permissões;
- conflitos de revisão.

## Fase 5 — Experiência de execução em formato livro

**PENDENTE.**

- páginas/etapas;
- navegação e progresso;
- passos/alertas/blocos copiáveis;
- checklist conforme decisão da Fase 1;
- estados de UI.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

- testes com múltiplos Clients;
- conflitos e fila;
- eventos/reconexão;
- comportamento de Host indisponível;
- validação na LAN corporativa quando disponível.

## Fase 7 — Exportação e identidade

**PENDENTE.**

- PDF;
- DOCX;
- impressão;
- template e identidade da empresa;
- validação em leitores esperados.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote da máquina central por pasta, com Controller/Host sob demanda;
- launcher publicado na rede e Client local versionado;
- backup/restore;
- logs de diagnóstico;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui instalar serviço StepFlow persistente na máquina central.

Cenário final conceitual:

```text
Controller central é iniciado quando o StepFlow será usado
→ técnico acessa o ponto de entrada interno
→ launcher prepara Client local
→ login/uso multiusuário
→ encerramento operacional fecha Host/Controller
→ nenhum processo StepFlow residual
```

## Fase 9 — Hardening e release interno

**PENDENTE.**

- segurança/autorização;
- recuperação de falha/banco;
- backup/restore;
- concorrência/performance;
- logs;
- distribuição/atualização;
- smoke tests end-to-end;
- revisão documental e limpeza de débitos prioritários.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudanças de requisito atualizam documentação antes da implementação afetada. A execução continua em tarefas pequenas e verificáveis.
