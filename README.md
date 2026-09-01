# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com registro operacional de Atendimentos e Equipamentos quando aplicável.

## Painel de acompanhamento

**Atualização:** 2026-09-01  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Checkpoint atual:** Bloco 12 — validação técnica/documental final em revisão  
**Próximo passo:** revisar P12.99–P12.108  
**Implementação funcional oficial:** ainda não iniciada

### Fase 1

| Bloco | Tema | Estado |
|---|---|---|
| 0 | Bootstrap do ambiente | ✅ Concluído |
| 1 | Client Windows / Tauri | ✅ Concluído |
| 2 | Host Pocket | ✅ Concluído |
| 3 | Launcher / distribuição | ✅ Concluído |
| 4 | Comunicação Client ↔ Host | ✅ Concluído |
| 5 | Autenticação / autorização | ✅ Consolidado, incluindo D12.56–D12.62 |
| 6 | Dados / schema / migrations | ✅ Núcleo + D12.35–D12.55 consolidados |
| 7 | Concorrência / fila / eventos | ✅ Concluído |
| 8 | UI/UX | ✅ Concluído |
| 9 | Atendimentos / execução / checklist | ✅ Concluído |
| 10 | Exportação / impressão / Ficha compacta | ✅ Concluído |
| 11 | Backup / restauração | ✅ Concluído |
| 12 | Estrutura oficial + plano da Fase 2 | 🟡 Validação final em revisão |

## Produto

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Procedimentos são documentação oficial versionada; Atendimentos registram trabalho real; Equipamento é opcional/reutilizável. Reader usa experiência de manual/livro, checklist persistente existe somente em Atendimento e a Ficha é prestação de contas resumida ao cliente.

## Contrato Pocket

```text
pasta publicada no servidor
→ usuário acessa compartilhamento
→ executa StepFlow.exe na raiz
→ Launcher prepara/valida Client em %LOCALAPPDATA%
→ Client abre localmente
→ Launcher encerra
```

`StepFlow.exe` é o único ponto de entrada normal. A árvore técnica publicada fica sob `_internal/`. Uso normal exige zero instalador por estação, zero preparação manual/admin/toolchain/Internet obrigatória e Client local em vez de execução permanente por SMB.

## Bloco 12 — decisões vigentes

**D12.1–D12.98 aprovadas.**

- source tree modular `apps/`/`crates/`;
- publicação `StepFlow.exe + _internal/`;
- Rust 1.98.0 + Edition 2024 + resolver 3;
- toolchain/lockfile versionados;
- Client vanilla sem Node/bundler;
- migrations Host-side imutáveis/embutidas com checksum;
- testes em SQLite temporário real e fixtures sintéticas;
- parâmetros finais de autenticação, Empresa/Categorias e Backup/Restore fechados;
- Fase 2 planejada em F2-T01…F2-T08, uma branch/PR por tarefa;
- nenhuma dessas decisões autoriza scaffold antes do gate final da Fase 1.

A validação final está em revisão como **P12.99–P12.108**, cobrindo configuração inválida, ownership de parâmetros, `deployment.json`, sync local seguro e gates corporativos.

## Fontes de verdade

- `AGENTS.md` — governança;
- `docs/README.md` — índice;
- `docs/05-progresso/registro-de-decisoes.md` — decisões vigentes;
- `docs/04-planejamento/plano-oficial-fase-1.md` — estado/gates;
- `docs/04-planejamento/roadmap.md` — fases;
- documentos específicos de Produto/Telas/Arquitetura — contratos detalhados.

## Pendências principais da Fase 1

- decidir P12.99–P12.108;
- consolidação final e gate Git do PR #27;
- sincronização segura do checkout local;
- autorização explícita da F2-T01;
- gates corporativos permanecem para as fases técnicas correspondentes.

## Regra deste painel

O README mostra somente estado executivo. Decisões detalhadas pertencem às fontes específicas.
