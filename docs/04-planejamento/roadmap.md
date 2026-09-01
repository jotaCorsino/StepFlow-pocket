# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-09-01

O roadmap descreve **fases e resultados**, não PRs ou branches específicas. Gates operacionais correntes ficam no plano da fase e no `README.md`.

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO — Bloco 12 em análise.**

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
- Backup/Restore técnico D11.1–D11.116;
- estrutura/publicação D12.1–D12.18;
- workspace/build/dependências D12.19–D12.34;
- migrations/scripts/testes/fixtures D12.35–D12.55;
- contrato Pocket preservado como gate superior.

### Bloco 12 — Estrutura oficial + Fase 2

**EM ANÁLISE.**

Análises 1–3 aprovadas — D12.1–D12.55:

- workspace Rust modular e publicação Pocket com `StepFlow.exe` como entrada única;
- Client em ES modules e sem Node/bundler no baseline;
- toolchain/lockfile/dependências/configuração/packaging reproduzíveis;
- migrations Host-side imutáveis, embutidas e verificadas por checksum;
- `pre_migration` backup + lote transacional + validação de integridade/FKs;
- testes em SQLite temporário real e fixtures sintéticas;
- scripts finos de check/test/build/package;
- nenhuma autorização de scaffold antes do gate final do Bloco 12.

Análise 4 em revisão — P12.56–P12.79:

- segurança de senha/sessão;
- configuração da empresa e categoria arquivada;
- parâmetros numéricos de Backup/Restore;
- timeouts/reconexão/readiness;
- rotação de logs/admin audit.

Depois dela, resta fechar o plano executável da Fase 2 e o gate final da Fase 1.

## Fase 2 — Fundação técnica executável

**PENDENTE — depende do encerramento do Bloco 12/Fase 1.**

Resultados esperados:

- árvore oficial Client/Launcher/Controller/Host;
- builds reproduzíveis;
- configuração de desenvolvimento;
- comunicação mínima + health/readiness;
- SQLite + runner de migrations determinístico;
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
- `StepFlow.exe` na raiz como Launcher amigável;
- artefatos técnicos encapsulados sob `_internal/`;
- Controller/Host sob demanda;
- Client local versionado;
- zero instalação/manualidade por estação;
- implementação dos contratos D11.1–D11.116 de Backup/Restore;
- disaster recovery local;
- logs/auditoria operacional;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui serviço StepFlow persistente.

## Fase 9 — Hardening e release interno

**PENDENTE.**

Segurança/autorização, recuperação de falha/banco, Backup/Restore, concorrência/performance, logs, distribuição/update, smoke tests end-to-end, revisão documental e validação final do contrato Pocket no parque corporativo.

## Pendências transversais

- P12.56–P12.79 até decisão do PO;
- inventário Windows/Office;
- WebView2 real e fallback Pocket;
- SMB/impressoras/filesystem/ACL/EDR corporativos;
- adapter Win32 e crash injection de Backup/Restore.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudança de requisito atualiza a documentação vigente antes da implementação. Proposta só vira contrato após aprovação explícita do PO.
