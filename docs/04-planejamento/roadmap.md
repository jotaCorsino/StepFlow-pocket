# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-09-01

O roadmap descreve **fases e resultados**, não PRs ou branches específicas. Gates operacionais correntes ficam no plano da fase e no `README.md`.

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO — Bloco 12 em análise.**

Consolidado:

- Client Windows/Tauri;
- Host Pocket sob demanda;
- Launcher/distribuição Pocket;
- comunicação HTTP/JSON + WebSocket;
- autenticação/sessão/autorização, incluindo parâmetros D12.56–D12.62;
- modelo de dados/migrations/histórico conceitual;
- concorrência/fila/conflitos/eventos;
- domínio `Procedimento × Atendimento/Execução × Equipamento`;
- categorias configuráveis/múltiplas;
- UI/UX Telas 01–15;
- lifecycle operacional de Atendimentos;
- checklist persistente e observação por Etapa;
- códigos `AT-000001` / `EQP-000001`;
- geração documental, exportação, impressão, Ficha, naming e temporários;
- Backup/Restore D11.1–D11.116;
- estrutura/publicação D12.1–D12.18;
- workspace/build/dependências D12.19–D12.34;
- migrations/scripts/testes/fixtures D12.35–D12.55;
- parâmetros finais D12.56–D12.79;
- contrato Pocket preservado como gate superior.

### Bloco 12 — Estrutura oficial + Fase 2

**EM ANÁLISE.**

Análises 1–4 aprovadas — D12.1–D12.79:

- workspace Rust modular e publicação com `StepFlow.exe` na raiz + `_internal/` técnico;
- Client em ES modules e sem Node/bundler no baseline;
- toolchain/lockfile/dependências/configuração/packaging reproduzíveis;
- migrations Host-side imutáveis/embutidas e verificadas;
- testes em SQLite temporário real e fixtures sintéticas;
- parâmetros finais de senha/sessão, empresa/categoria, Backup/Restore, reconexão e logs fechados;
- nenhuma autorização de scaffold antes do gate final do Bloco 12.

Análise 5 em revisão — P12.80–P12.98:

```text
Gate Fase 1 + sync local seguro
→ F2-T01 workspace/tooling + Host mínimo
→ F2-T02 Host runtime/readiness
→ F2-T03 SQLite + migrations runner
→ F2-T04 Controller lifecycle
→ F2-T05 Client Tauri + compatibilidade
→ F2-T06 Launcher Pocket
→ F2-T07 packaging Pocket
→ F2-T08 smoke integrado + gates Windows/Pocket
```

Após a decisão da Análise 5, resta a validação final da Fase 1, gate Git, sincronização segura do checkout local e autorização do primeiro scaffold.

## Fase 2 — Fundação técnica executável

**PENDENTE — depende do encerramento do Bloco 12/Fase 1.**

Resultados esperados:

- workspace/build/tooling oficiais;
- Host mínimo com configuração/logging/health/readiness;
- SQLite + runner de migrations determinístico;
- Controller com lifecycle bounded e sem serviço/watchdog;
- Client Tauri vanilla local e compatibilidade mínima com Host;
- Launcher que prepara/executa Client em `%LOCALAPPDATA%` sem admin/instalador/Internet;
- packaging `StepFlow.exe + _internal/`;
- smoke integrado e gates Pocket/Windows da fundação;
- PoC de fallback WebView2 somente se necessário.

Gate: Client abre pelo fluxo Pocket, Host inicia sob demanda, banco inicializa deterministicamente, Controller encerra sem processo residual, packaging é reproduzível e build/testes limpos passam.

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
- evolução da distribuição Pocket além da fundação da Fase 2;
- implementação completa D11.1–D11.116 de Backup/Restore;
- disaster recovery local;
- logs/auditoria operacional;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui serviço StepFlow persistente.

## Fase 9 — Hardening e release interno

**PENDENTE.**

Segurança/autorização, recuperação de falha/banco, Backup/Restore, concorrência/performance, logs, distribuição/update, smoke tests end-to-end, revisão documental e validação final do contrato Pocket no parque corporativo.

## Pendências transversais

- P12.80–P12.98 até decisão do PO;
- inventário Windows/Office;
- WebView2 real e fallback Pocket;
- SMB/impressoras/filesystem/ACL/EDR corporativos;
- adapter Win32 e crash injection de Backup/Restore.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudança de requisito atualiza a documentação vigente antes da implementação. Proposta só vira contrato após aprovação explícita do PO.
