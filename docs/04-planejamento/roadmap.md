# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-21

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates foram estabelecidos. O histórico detalhado permanece no Git e no diário.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO.**

Núcleo arquitetural já fechado:

- Client/Tauri e compatibilidade Windows;
- Host Pocket sob demanda;
- launcher e atualização do Client;
- comunicação HTTP/JSON + WebSocket;
- autenticação/sessão/autorização no núcleo;
- modelo de dados/migrations/histórico;
- concorrência/fila/conflitos/eventos.

Novo requisito de 2026-08-21 incorporado ao fechamento da Fase 1:

- categorização configurável de procedimentos;
- atendimento/execução formal quando necessário;
- equipamento opcional com ficha técnica;
- busca operacional;
- ficha compacta imprimível.

Próximos blocos:

1. UI/UX incluindo categorias, atendimentos e equipamentos;
2. execução operacional + checklist;
3. exportação PDF/DOCX/impressão + ficha compacta;
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

## Fase 4 — Núcleo documental de procedimentos

**PENDENTE.**

- lista/pesquisa;
- categorias configuráveis e filtros;
- criação/edição/arquivamento;
- etapas e blocos estruturados;
- histórico/revisões;
- permissões;
- conflitos de revisão.

## Fase 5 — Execução operacional e experiência em formato livro

**PENDENTE.**

- páginas/etapas do procedimento;
- navegação e progresso;
- passos/alertas/blocos copiáveis;
- atendimento/execução formal conforme decisão da Fase 1;
- equipamento opcional e ficha técnica;
- vínculo com revisão do procedimento utilizada;
- checklist/progresso conforme decisão do Bloco 9;
- busca/lista de atendimentos;
- estados de UI.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

- testes com múltiplos Clients;
- conflitos e fila;
- eventos/reconexão;
- comportamento de Host indisponível;
- concorrência de procedimentos/equipamentos/atendimentos;
- validação na LAN corporativa quando disponível.

## Fase 7 — Exportação e identidade

**PENDENTE.**

- PDF;
- DOCX;
- impressão;
- template e identidade da empresa;
- ficha/relatório compacto de atendimento/equipamento;
- validação em leitores/impressoras esperados.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote central por pasta com Controller/Host sob demanda;
- launcher publicado na rede e Client local versionado;
- backup/restore incluindo dados operacionais;
- logs de diagnóstico;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui instalar serviço StepFlow persistente na máquina central.

Cenário final conceitual:

```text
Controller central é iniciado quando o StepFlow será usado
→ técnico acessa ponto de entrada interno
→ launcher prepara Client local
→ login/uso multiusuário
→ consulta procedimento e, quando aplicável, registra atendimento/equipamento
→ impressão/exportação quando necessária
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
