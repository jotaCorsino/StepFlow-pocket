# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-21

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates foram estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO.**

Núcleo arquitetural já fechado:

- Client/Tauri e Windows;
- Host Pocket sob demanda;
- launcher/update;
- HTTP/JSON + WebSocket;
- autenticação/sessão/autorização no núcleo;
- modelo de dados/migrations/histórico original;
- concorrência/fila/conflitos/eventos.

Novo requisito confirmado em 2026-08-21:

- categorização de procedimentos;
- registro das informações reais de serviço/equipamento quando aplicável;
- busca operacional;
- resumo do trabalho/procedimentos realizados;
- ficha compacta imprimível.

A separação específica `Procedimento × Atendimento/Execução × Equipamento` é recomendação em proposta e será decidida ainda na Fase 1.

Próximos blocos:

1. UI/UX;
2. modelagem/execução operacional + checklist;
3. exportação PDF/DOCX/impressão + ficha compacta;
4. backup/restore;
5. estrutura oficial e plano da Fase 2.

Detalhes em `plano-oficial-fase-1.md`.

## Fase 2 — Fundação técnica executável

**PENDENTE.**

- árvore oficial Client/launcher/Controller/Host;
- builds reproduzíveis;
- configuração de desenvolvimento;
- comunicação mínima + health/readiness;
- SQLite + migrations iniciais;
- logging mínimo;
- testes de fundação.

Gate: Client abre, Host inicia sob demanda, comunicação mínima funciona, banco inicializa deterministicamente e build limpo passa.

## Fase 3 — Autenticação, usuários e shell

**PENDENTE.**

- login/logout/sessão;
- bootstrap ADM;
- usuários/permissões;
- perfil/avatar;
- shell/sidebar;
- configuração básica da empresa;
- autorização Host-side.

## Fase 4 — Núcleo documental de procedimentos

**PENDENTE.**

- lista/pesquisa;
- categorização conforme modelo aprovado;
- criação/edição/arquivamento;
- etapas/blocos;
- histórico/revisões;
- permissões;
- conflitos de revisão.

## Fase 5 — Experiência de execução e registro operacional

**PENDENTE.**

- páginas/etapas em formato livro;
- navegação/progresso;
- passos/alertas/blocos copiáveis;
- checklist conforme decisão da Fase 1;
- registro das informações reais do serviço/equipamento conforme modelagem aprovada;
- busca/lista operacional quando aplicável;
- resumo do trabalho;
- estados de UI.

Se a separação Atendimento/Equipamento for aprovada, esta fase implementará esses domínios; caso a Fase 1 aprove modelagem diferente, o roadmap deverá seguir a decisão final.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

- múltiplos Clients;
- conflitos/fila;
- eventos/reconexão;
- Host indisponível;
- concorrência dos novos registros aprovados;
- validação LAN corporativa.

## Fase 7 — Exportação e identidade

**PENDENTE.**

- PDF;
- DOCX;
- impressão;
- template/identidade da empresa;
- ficha compacta de serviço/equipamento;
- validação em leitores/impressoras esperados.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote central por pasta com Controller/Host sob demanda;
- launcher em rede + Client local versionado;
- backup/restore incluindo novos dados aprovados;
- logs;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui serviço StepFlow persistente.

Cenário final conceitual:

```text
Controller iniciado quando StepFlow será usado
→ launcher prepara Client local
→ login/uso multiusuário
→ consulta procedimento
→ quando necessário, registra informações reais do serviço/equipamento
→ impressão/exportação quando necessária
→ encerramento fecha Host/Controller
→ zero processo StepFlow residual
```

## Fase 9 — Hardening e release interno

**PENDENTE.**

- segurança/autorização;
- recuperação de falha/banco;
- backup/restore;
- concorrência/performance;
- logs;
- distribuição/update;
- smoke tests end-to-end;
- revisão documental.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudanças de requisito atualizam documentação antes da implementação. Propostas só viram implementação após aprovação explícita.
