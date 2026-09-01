# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com registro operacional de Atendimentos e Equipamentos quando aplicável.

## Painel de acompanhamento

**Atualização:** 2026-09-01  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Checkpoint atual:** Bloco 12 — Estrutura oficial + plano da Fase 2 em análise  
**Próximo passo:** revisar a Análise 4 — parâmetros finais (P12.56–P12.79)  
**Implementação funcional oficial:** ainda não iniciada

### Fase 1

| Bloco | Tema | Estado |
|---|---|---|
| 0 | Bootstrap do ambiente | ✅ Concluído |
| 1 | Client Windows / Tauri | ✅ Concluído |
| 2 | Host Pocket | ✅ Concluído |
| 3 | Launcher / distribuição | ✅ Concluído |
| 4 | Comunicação Client ↔ Host | ✅ Concluído |
| 5 | Autenticação / autorização | ✅ Núcleo concluído; parâmetros finais em revisão no Bloco 12 |
| 6 | Dados / schema / migrations | ✅ Núcleo conceitual + disciplina D12.35–D12.55 consolidada |
| 7 | Concorrência / fila / eventos | ✅ Núcleo concluído |
| 8 | UI/UX | ✅ Concluído |
| 9 | Atendimentos / execução / checklist | ✅ Concluído |
| 10 | Exportação / impressão / Ficha compacta | ✅ Concluído |
| 11 | Backup / restauração | ✅ Concluído |
| 12 | Estrutura oficial + plano da Fase 2 | 🟡 Em análise — Análises 1–3 aprovadas |

## Produto

O domínio separa:

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Princípios centrais:

- Procedimentos são documentação oficial versionada;
- Atendimentos registram ocorrências reais de trabalho;
- Equipamento é opcional e reutilizável;
- Reader usa experiência de manual/livro, com uma Etapa por página lógica;
- checklist persistente existe somente no contexto de Atendimento;
- `Observação do serviço` por Etapa é opcional e operacional;
- Ficha compacta é prestação de contas resumida ao cliente;
- UI privilegia clareza e baixa densidade textual.

## Contrato Pocket

```text
pasta publicada no servidor
→ usuário acessa o compartilhamento
→ executa StepFlow.exe na raiz
→ Launcher prepara/valida o Client em %LOCALAPPDATA%
→ Client abre localmente
→ Launcher encerra
```

`StepFlow.exe` é o Launcher com nome/ícone amigáveis e o único ponto de entrada normal. Artefatos técnicos publicados ficam encapsulados sob `_internal/`; `.lnk` não é requisito baseline.

É obrigatório no uso normal: zero instalador por estação, zero preparação manual, zero elevação administrativa, nenhuma toolchain em produção, nenhuma Internet obrigatória e Client operacional local em vez de execução permanente por SMB.

## Bloco 12 — decisões vigentes

Análises 1–3 aprovadas: **D12.1–D12.55**.

- source tree modular por `apps/` e `crates/`;
- `StepFlow.exe` como entrada única e `_internal/` como área técnica;
- Rust 1.98.0, Edition 2024, resolver 3, toolchain + `Cargo.lock` versionados;
- Client vanilla modular sem Node/npm/Vite/bundler/framework no baseline;
- dependências somente com uso real;
- migrations Host-side imutáveis, embutidas e verificadas por checksum;
- `pre_migration` backup antes de lote pendente em banco existente;
- lote transacional com `quick_check` + `foreign_key_check` antes de readiness;
- testes em SQLite temporário real e fixtures sintéticas;
- nenhuma dessas decisões autoriza scaffold antes do gate final da Fase 1.

A Análise 4 está em revisão como **P12.56–P12.79**.

## Fontes de verdade

- `AGENTS.md` — governança e regras de execução;
- `docs/README.md` — índice e ownership documental;
- `docs/05-progresso/registro-de-decisoes.md` — decisões vigentes e pendências;
- `docs/04-planejamento/plano-oficial-fase-1.md` — estado da Fase 1;
- `docs/04-planejamento/roadmap.md` — fases do projeto;
- documentos específicos de Produto, Telas e Arquitetura — contratos detalhados.

## Pendências principais da Fase 1

- decidir P12.56–P12.79 — autenticação/sessão, configuração da empresa/categoria e limites operacionais;
- plano detalhado da Fase 2 e sequência de tarefas Codex;
- validação final da Fase 1 + gate Git;
- sincronização segura do checkout local antes do primeiro scaffold;
- validações corporativas de Windows/WebView2/Launcher/SMB/Word/impressoras/filesystem/ACL/EDR no momento apropriado.

## Regra deste painel

O README mostra somente o estado executivo. Decisões detalhadas não devem ser duplicadas aqui; o painel aponta para a fonte específica vigente.
