# Roadmap — StepFlow Pocket

**Status:** FASE 1 CONCLUÍDA / TRANSIÇÃO PARA FASE 2 PENDENTE  
**Atualização:** 2026-09-01

O roadmap descreve fases e resultados. Gates operacionais correntes ficam no plano da fase e no README raiz.

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**CONCLUÍDA documental e tecnicamente em 2026-09-01.**

Resultados consolidados:

- Client Windows/Tauri;
- Host Pocket sob demanda;
- Launcher/distribuição Pocket;
- comunicação HTTP/JSON + WebSocket;
- autenticação/sessão/autorização;
- modelo de dados/migrations/histórico conceitual;
- concorrência/fila/conflitos/eventos;
- domínio Procedimento × Atendimento × Equipamento;
- Telas 01–15;
- execução operacional/checklist;
- geração documental/Ficha;
- Backup/Restore D11.1–D11.116;
- Bloco 12 D12.1–D12.108: estrutura/publicação, workspace/build, migrations/testes, parâmetros, plano da Fase 2 e validação final;
- contrato Pocket preservado como gate superior.

Nenhum scaffold/runtime oficial, migration SQL ou código de negócio foi criado na Fase 1.

## Fase 2 — Fundação técnica executável

**PENDENTE — depende de fechamento Git do Bloco 12 + remoto limpo + sync local seguro + autorização explícita do PO.**

Sequência aprovada:

```text
F2-T01 workspace/tooling + Host mínimo
→ F2-T02 Host runtime/readiness
→ F2-T03 SQLite + migrations runner
→ F2-T04 Controller lifecycle
→ F2-T05 Client Tauri + compatibilidade
→ F2-T06 Launcher Pocket
→ F2-T07 packaging Pocket
→ F2-T08 smoke integrado + gates Windows/Pocket
```

Resultados esperados:

- build reproduzível `--locked`;
- Host com config/logging/readiness;
- SQLite e migrations determinísticos;
- Controller lifecycle bounded;
- Client Tauri vanilla integrado ao Host;
- Launcher preparando Client local;
- packaging `StepFlow.exe + _internal/`;
- smoke integrado;
- gates Pocket corporativos registrados como PASS, `NÃO APLICÁVEL NESTE AMBIENTE` ou blocker real conforme contexto.

Gate: nenhum processo residual, nenhuma dependência dev em produção e nenhum requisito Pocket enfraquecido.

## Fase 3 — Autenticação, usuários e shell

**PENDENTE.** Login/logout/sessão, bootstrap ADM, usuários/permissões, perfil/avatar, shell/sidebar, configuração básica da empresa e autorização Host-side.

## Fase 4 — Núcleo documental de Procedimentos

**PENDENTE.** Lista/pesquisa, categorias, criação/edição/arquivamento, Etapas, histórico/revisões, permissões e conflitos.

## Fase 5 — Execução e registro operacional

**PENDENTE.** Reader, Atendimento, checklist, observação de serviço, lifecycle, Equipamento opcional, revisão exata, reprodução histórica, Ficha e estados transversais.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.** Múltiplos Clients, concorrência, eventos/reconexão, stress/tuning e validação LAN corporativa.

## Fase 7 — Exportação e identidade

**PENDENTE.** PDF/DOCX, impressão Windows, identidade, Ficha, naming/save/temporários e gates reais de Word/impressoras/SMB/EDR.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.** Pacote central, Launcher/Client local, Controller/Host sob demanda, implementação D11, disaster recovery, logs/auditoria e validação sem Internet em PCs corporativos. Não inclui serviço persistente.

## Fase 9 — Hardening e release interno

**PENDENTE.** Segurança, recuperação, Backup/Restore, concorrência/performance, distribuição/update, smoke E2E, revisão documental e validação final do Pocket no parque corporativo.

## Pendências transversais

- gate operacional de entrada na Fase 2;
- inventário Windows/Office;
- WebView2 real e fallback Pocket;
- SMB/impressoras/filesystem/ACL/EDR corporativos;
- adapter Win32 e crash injection de Backup/Restore.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudança de requisito atualiza a documentação vigente antes da implementação. Conclusão de uma fase não autoriza automaticamente a próxima tarefa.