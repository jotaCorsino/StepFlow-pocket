# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-29

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO.**

Já consolidados:

- Client/Tauri e Windows;
- Host Pocket sob demanda;
- launcher/distribuição Pocket;
- HTTP/JSON + WebSocket;
- autenticação/sessão/autorização no núcleo;
- modelo de dados/migrations/histórico conceitual;
- concorrência/fila/conflitos/eventos;
- `Procedimento × Atendimento/Execução × Equipamento`;
- categorias configuráveis/múltiplas;
- UI/UX completa das Telas 01–15;
- lifecycle operacional de Atendimentos;
- checklist persistente em contexto de execução;
- observação de serviço por Etapa;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`;
- Bloco 10 / Etapas 1–11 de geração documental, exportação, impressão, Ficha, naming, temporários e validação técnica final.

### Contrato Pocket

O produto deve ser distribuído como pasta pronta no servidor Windows.

```text
share do servidor
→ StepFlowLauncher.exe
→ preparação local automática
→ Client em %LOCALAPPDATA%
→ uso normal sem instalador/manualidade/admin
```

O modelo não aceita instalação obrigatória por estação, toolchain, Internet obrigatória ou execução permanente do Client pelo SMB.

WebView2 Evergreen existente é preferível. Fixed Version não roda por UNC/SMB; fallback local só entra após PoC comprovar preparação automática sem instalação/elevação/manualidade.

### Bloco 8 — UI/UX

**CONCLUÍDO.**

Telas 01–15 consolidadas, incluindo estados transversais.

### Bloco 9 — Execução operacional / Atendimentos + checklist

**CONCLUÍDO.**

Consolidado:

- `Em andamento`, `Concluído`, `Cancelado`;
- primeiro save cria Atendimento;
- responsabilidade por técnico;
- conclusão/reabertura/cancelamento;
- checklist persistente apenas em Atendimento;
- progresso por checklist;
- vínculo com revisão exata;
- observação de serviço por Etapa;
- snapshot histórico de Equipamento/estado relevante;
- matriz operacional;
- códigos legíveis;
- lifecycle da Ficha.

### Bloco 10 — Exportação / impressão + Ficha compacta

**ETAPAS 1–11 CONSOLIDADAS / APROVADAS PELO PO.**

Consolidado:

- geração documental Host-side por snapshot consistente + `DocumentModel`;
- PDF de Procedimentos via Typst embutido;
- DOCX OOXML direto em Rust;
- impressão Windows pelo mesmo PDF oficial via WebView2 + `ShowPrintUI(System)`;
- Procedimento físico A4 retrato multipágina;
- Ficha PDF + preview SVG pelo mesmo `PagedDocument`;
- Ficha física exatamente uma A4;
- `SHEET_OVERFLOW` sem truncamento/segunda página/redução automática;
- soft limits orientativos;
- regras para MACs/dados multiplicativos;
- naming persistente e lifecycle de temporários;
- validação final sem bloqueador arquitetural identificado;
- gates de ambiente real para Word, impressoras, SMB, WebView2/Windows e EDR;
- limites de performance definidos por benchmark na fase executável;
- preservação explícita do contrato Pocket.

O Bloco 10 só está operacionalmente encerrado após squash merge da Etapa 11, remoção da branch e remoto com somente `main`/zero PRs.

**Próximo bloco após o gate: Bloco 11 — Backup/Restore técnico.**

### Bloco 11 — Backup / Restore técnico

**PENDENTE.**

Fechará:

- formato/pacote administrado;
- atomicidade/checksums;
- retenção;
- restart/reconexão/sessões;
- safety backup;
- disaster recovery local quando Host não inicia.

### Bloco 12 — Estrutura oficial + Fase 2

**PENDENTE.**

Fechará:

- parâmetros técnicos finais;
- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- sincronização segura do checkout local;
- plano de implementação da Fase 2.

## Fase 2 — Fundação técnica executável

**PENDENTE.**

- árvore oficial Client/Launcher/Controller/Host;
- builds reproduzíveis;
- configuração de desenvolvimento;
- comunicação mínima + health/readiness;
- SQLite + migrations iniciais;
- logging mínimo;
- testes de fundação;
- PoCs/gates técnicos exigidos pela Fase 1, incluindo fallback WebView2 Pocket quando necessário.

Gate: Client abre sem instalação manual, Host inicia sob demanda, comunicação mínima funciona, banco inicializa deterministicamente e build limpo passa.

## Fase 3 — Autenticação, usuários e shell

**PENDENTE.**

- login/logout/sessão;
- bootstrap ADM;
- usuários/permissões;
- perfil/avatar;
- shell/sidebar;
- configuração básica da empresa;
- autorização Host-side.

## Fase 4 — Núcleo documental de Procedimentos

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

- Reader em páginas/Etapas;
- passos/alertas/blocos copiáveis;
- Atendimento;
- checklist persistente;
- observação de serviço por Etapa;
- progresso por checklist;
- lifecycle de três estados;
- Equipamento opcional;
- busca/lista operacional;
- resumo do trabalho;
- revisão exata utilizada;
- snapshot histórico;
- Ficha compacta conforme Blocos 8–10;
- estados transversais.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

- múltiplos Clients;
- conflitos/fila;
- concorrência granular;
- eventos/reconexão;
- Host indisponível;
- stress/tuning;
- validação LAN corporativa.

## Fase 7 — Exportação e identidade

**PENDENTE.**

Implementará os contratos do Bloco 10:

- geração Host-side;
- PDF de Procedimentos;
- DOCX de Procedimentos;
- impressão Windows;
- identidade central da empresa;
- Ficha compacta PDF/preview/impressão;
- reprodução histórica;
- naming/save/temporários;
- fixtures de overflow;
- gates reais de Word/impressoras/SMB/EDR.

DOCX específico da Ficha não é requisito inicial.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote central por pasta;
- Controller/Host sob demanda;
- Launcher no share + Client local versionado;
- zero instalação/manualidade por estação;
- backup/restore de banco + arquivos administrados;
- safety backup antes do Restore normal;
- disaster recovery local;
- logs;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui serviço StepFlow persistente.

Cenário final conceitual:

```text
Controller iniciado quando StepFlow será usado
→ usuário executa Launcher no share
→ Launcher prepara Client local
→ login/uso multiusuário
→ consulta Procedimento
→ registra/executa Atendimento
→ checklist/progresso por revisão vinculada
→ conclusão/cancelamento/reabertura
→ impressão/exportação
→ encerramento central fecha Host/Controller
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
- revisão documental;
- validação final do contrato Pocket no parque corporativo.

## Pendências transversais

- parâmetros finais Argon2/senha/sessão;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- inventário Windows/Office;
- WebView2 real e fallback Pocket;
- SMB/impressoras/EDR corporativos.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudança de requisito atualiza documentação antes da implementação. Proposta só vira contrato após aprovação explícita.
