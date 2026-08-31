# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-31

O roadmap descreve **fases e resultados**, não PRs ou branches específicas. Gates operacionais correntes ficam no plano da fase e no `README.md`.

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO.**

Consolidado até aqui:

- Client Windows/Tauri;
- Host Pocket sob demanda;
- Launcher/distribuição Pocket;
- comunicação HTTP/JSON + WebSocket;
- autenticação/sessão/autorização no núcleo;
- modelo de dados/migrations/histórico conceitual;
- concorrência/fila/conflitos/eventos;
- domínio `Procedimento × Atendimento/Execução × Equipamento`;
- categorias configuráveis/múltiplas;
- UI/UX das Telas 01–15;
- lifecycle operacional de Atendimentos;
- checklist persistente e observação de serviço por Etapa;
- códigos `AT-000001` / `EQP-000001`;
- geração documental, exportação, impressão, Ficha compacta, naming e temporários;
- validação técnica final do Bloco 10;
- Backup/Restore consolidado até D11.82: envelope, consistência, catálogo/retenção, Restore/safety/compatibilidade e recovery de restart/sessões;
- contrato Pocket preservado como gate superior.

### Bloco 11 — Backup / Restore técnico

**EM ANÁLISE — Análise 6.**

Já aprovados:

- estado recuperável e pacote `.stepflow-backup`;
- snapshot SQLite via Online Backup API;
- consistência conjunta entre banco e arquivos administrados;
- staging/verificação/promoção no-replace;
- catálogo reconstruível e retenção sem scheduler/por quantidade;
- coordenação administrativa de Backup/Restore/migration;
- Restore com revalidação integral, migrations forward, safety backup e troca lógica de `data/`;
- journal de Restore, reconciliação antes de readiness, rollback conhecido/`uncertain`;
- fresh Host e invalidação de sessões após fase destrutiva.

Em revisão:

- disaster recovery local/controlado;
- capacidades finais de Backup/Restore;
- auditoria administrativa que atravesse Restore.

Depois:

- validação técnica final do Bloco 11.

Fontes: `bloco-11-backup-restauracao.md` e análises específicas 3–6.

### Bloco 12 — Estrutura oficial + Fase 2

**PENDENTE.**

Fechará:

- parâmetros técnicos finais ainda abertos;
- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- sincronização segura do checkout local;
- plano executável da Fase 2.

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

Login/logout/sessão, bootstrap ADM, usuários/permissões, perfil/avatar, shell/sidebar, configuração básica da empresa e autorização Host-side.

## Fase 4 — Núcleo documental de Procedimentos

**PENDENTE.**

Lista/pesquisa, categorização, criação/edição/arquivamento, Etapas/blocos, histórico/revisões, permissões e conflitos de revisão.

## Fase 5 — Execução e registro operacional

**PENDENTE.**

Reader, Atendimento, checklist persistente, observação de serviço, lifecycle, Equipamento opcional, busca/lista, resumo, revisão exata, reprodução histórica, Ficha e estados transversais.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

Múltiplos Clients, conflitos/fila, concorrência granular, eventos/reconexão, Host indisponível, stress/tuning e validação LAN corporativa.

## Fase 7 — Exportação e identidade

**PENDENTE.**

Implementará os contratos do Bloco 10: PDF/DOCX, impressão Windows, identidade, Ficha, naming/save/temporários e gates reais de Word/impressoras/SMB/EDR.

DOCX específico da Ficha não é requisito inicial.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote central por pasta;
- Controller/Host sob demanda;
- Launcher no share + Client local versionado;
- zero instalação/manualidade por estação;
- Backup/Restore;
- disaster recovery local;
- logs/auditoria operacional;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui serviço StepFlow persistente.

## Fase 9 — Hardening e release interno

**PENDENTE.**

Segurança/autorização, recuperação de falha/banco, Backup/Restore, concorrência/performance, logs, distribuição/update, smoke tests end-to-end, revisão documental e validação final do contrato Pocket no parque corporativo.

## Pendências transversais

- parâmetros finais Argon2/senha/sessão/token;
- Gerência × configuração da empresa;
- Gerência × Backup — em revisão no Bloco 11;
- regra editorial de categoria arquivada;
- valor/default final de retenção de backups;
- inventário Windows/Office;
- WebView2 real e fallback Pocket;
- SMB/impressoras/ACL/EDR corporativos.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudança de requisito atualiza a documentação vigente antes da implementação. Proposta só vira contrato após aprovação explícita do PO.