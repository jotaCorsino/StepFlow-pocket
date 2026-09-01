# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com registro operacional de Atendimentos e Equipamentos quando aplicável.

## Painel de acompanhamento

**Atualização:** 2026-09-01  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Checkpoint atual:** Bloco 12 — Estrutura oficial + plano da Fase 2 em análise  
**Próximo passo:** revisar a Análise 3 — migrations/scripts/testes/fixtures (P12.35–P12.55)  
**Implementação funcional oficial:** ainda não iniciada

### Fase 1

| Bloco | Tema | Estado |
|---|---|---|
| 0 | Bootstrap do ambiente | ✅ Concluído |
| 1 | Client Windows / Tauri | ✅ Concluído |
| 2 | Host Pocket | ✅ Concluído |
| 3 | Launcher / distribuição | ✅ Concluído |
| 4 | Comunicação Client ↔ Host | ✅ Concluído |
| 5 | Autenticação / autorização | ✅ Núcleo concluído; parâmetros finais pendentes |
| 6 | Dados / schema / migrations | ✅ Núcleo + extensão operacional conceitual consolidados |
| 7 | Concorrência / fila / eventos | ✅ Núcleo concluído |
| 8 | UI/UX | ✅ Concluído |
| 9 | Atendimentos / execução / checklist | ✅ Concluído |
| 10 | Exportação / impressão / Ficha compacta | ✅ Concluído |
| 11 | Backup / restauração | ✅ Concluído |
| 12 | Estrutura oficial + plano da Fase 2 | 🟡 Em análise — Análises 1–2 aprovadas |

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

O StepFlow deve ser publicado como **pasta pronta em um servidor Windows** e usado pelas estações autorizadas sem instalação individual do aplicativo.

Contrato vigente:

```text
pasta publicada no servidor
→ usuário acessa o compartilhamento
→ executa StepFlow.exe na raiz
→ Launcher prepara/valida o Client em %LOCALAPPDATA%
→ Client abre localmente
→ Launcher encerra
```

`StepFlow.exe` é o Launcher com nome/ícone amigáveis e o único ponto de entrada normal. Artefatos técnicos da publicação ficam encapsulados sob `_internal/`; `.lnk` não é requisito baseline.

É obrigatório no uso normal:

- zero instalador tradicional por estação;
- zero preparação manual de dependências;
- zero elevação administrativa;
- nenhuma toolchain de desenvolvimento em produção;
- nenhuma Internet obrigatória;
- Client operacional local, não executado permanentemente pelo SMB.

O Controller/Host continua sob demanda na máquina central. WebView2 não pode enfraquecer o contrato Pocket.

## Bloco 12 — decisões vigentes

Análises 1–2 aprovadas: **D12.1–D12.34**.

- source tree modular por `apps/` e `crates/`;
- Client web modular em ES modules;
- Launcher/Controller/Host com responsabilidades separadas;
- `StepFlow.exe` como entrada única e `_internal/` como área técnica publicada;
- Rust 1.98.0 + Edition 2024 + resolver 3;
- `rust-toolchain.toml` e `Cargo.lock` versionados;
- baseline Client sem Node/npm/Vite/bundler/framework;
- dependências somente quando houver uso real;
- produção montada por packaging, não diretamente de `target/release`;
- nenhuma dessas decisões autoriza scaffold antes do gate final da Fase 1.

A Análise 3 está em revisão como **P12.35–P12.55**.

## Fontes de verdade

- `AGENTS.md` — governança e regras de execução;
- `docs/README.md` — índice e ownership documental;
- `docs/05-progresso/registro-de-decisoes.md` — decisões vigentes e pendências;
- `docs/04-planejamento/plano-oficial-fase-1.md` — estado da Fase 1;
- `docs/04-planejamento/roadmap.md` — fases do projeto;
- documentos específicos de Produto, Telas e Arquitetura — contratos detalhados.

## Pendências principais da Fase 1

- Análise 3 do Bloco 12 — migrations/scripts/testes/fixtures;
- parâmetros finais de autenticação/sessão e Backup/Restore;
- Gerência × configuração da empresa;
- regra editorial de categoria arquivada;
- plano detalhado da Fase 2 e gate do primeiro scaffold;
- validações corporativas de Windows/WebView2/Launcher/SMB/Word/impressoras/filesystem/ACL/EDR no momento apropriado.

## Regra deste painel

O README mostra somente o estado executivo. Decisões detalhadas não devem ser duplicadas aqui; o painel aponta para a fonte específica vigente.
