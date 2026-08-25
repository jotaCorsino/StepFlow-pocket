# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-25

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
- modelo de dados/migrations/histórico;
- concorrência/fila/conflitos/eventos;
- extensão conceitual `Procedimento × Atendimento/Execução × Equipamento`;
- categorias configuráveis/múltiplas e identidade interna de equipamento.

Bloco atual: **UI/UX**.

No Bloco 8 já estão consolidadas as **Telas 01–12**, cobrindo:

- Login;
- Shell/sidebar com `Atendimentos`;
- Dashboard enxuto;
- Lista/Pesquisa de Processos com categorização;
- Leitor em formato livro;
- Editor de Processo + categorias;
- Histórico/Revisões;
- Lista/Pesquisa de Atendimentos;
- Atendimento/Execução + Equipamento;
- Usuários/Permissões;
- Meu perfil;
- Configurações + Categorias, incluindo identidade central da empresa.

Próxima superfície: **Tela 13 — Backup/Restauração — UX**.

Depois dela, o Bloco 8 ainda precisa fechar:

- Tela 14 — Exportação/Impressão + ficha compacta — UX;
- Tela 15 — estados transversais.

Próximos blocos da Fase 1:

1. concluir UI/UX;
2. fechar execução operacional/Atendimentos + checklist;
3. fechar exportação PDF/DOCX/impressão + ficha compacta;
4. fechar backup/restore;
5. fechar estrutura oficial e plano da Fase 2.

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
- categorização;
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
- Atendimentos;
- equipamento opcional;
- busca/lista operacional;
- resumo do trabalho;
- vínculo à revisão executada;
- ficha compacta conforme contrato dos Blocos 8/10;
- estados de UI.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

- múltiplos Clients;
- conflitos/fila;
- eventos/reconexão;
- Host indisponível;
- concorrência dos registros operacionais;
- validação LAN corporativa.

## Fase 7 — Exportação e identidade

**PENDENTE.**

- PDF;
- DOCX;
- impressão;
- template usando identidade da empresa administrada centralmente;
- ficha compacta de atendimento/equipamento;
- validação em leitores/impressoras esperados.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote central por pasta com Controller/Host sob demanda;
- launcher em rede + Client local versionado;
- backup/restore incluindo categorias, equipamentos, atendimentos e arquivos administrados;
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
→ quando necessário, registra Atendimento/equipamento
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