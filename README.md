# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com simplicidade operacional e implantação de baixo impacto.

## Painel de acompanhamento

**Atualização:** 2026-08-28  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Bloco atual:** Bloco 10 — Exportação / impressão / ficha compacta — Etapas 1–6 consolidadas; Etapa 7 próxima  
**Implementação funcional oficial:** ainda não iniciada

Este painel é apenas a visão rápida. Precedência e decisões completas permanecem em `AGENTS.md`, `docs/05-progresso/registro-de-decisoes.md` e documentos específicos.

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
| 10 | Exportação / impressão / ficha compacta | 🟡 Em andamento — Etapas 1–6 consolidadas |
| 11 | Backup / restauração | ⏳ Pendente |
| 12 | Estrutura oficial + plano da Fase 2 | ⏳ Pendente |

## Direção de produto e UI

- Procedimento, Atendimento/Execução e Equipamento são domínios distintos;
- Reader usa experiência de manual/livro;
- `Visão geral` precede Etapa 1;
- cada Etapa é uma página lógica própria;
- stepper compacto de círculos/linhas navega entre Etapas;
- stepper representa navegação, não conclusão operacional;
- UI privilegia clareza com baixa densidade textual;
- cor, forma, símbolo, posição e ícones podem reduzir texto repetitivo quando o significado continuar claro;
- cor nunca é o único meio para estado importante.

## Bloco 9 — operação consolidada

Lifecycle:

```text
rascunho local
→ primeiro save aceito
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Principais decisões:

- códigos `AT-000001` e `EQP-000001`, gerados pelo Host;
- responsável + `Resumo do trabalho` obrigatórios para conclusão;
- checklist incompleto avisa, não bloqueia automaticamente;
- Reader standalone não persiste execução;
- Reader operacional persiste checklist;
- cada Etapa em execução pode receber `Observação do serviço` opcional e persistente;
- observação pertence ao Atendimento/revisão/Etapa e não altera o Procedimento;
- progresso deriva somente de checklist;
- Equipamento é opcional/reutilizável;
- conclusão preserva projeção histórica do Equipamento e estado final aplicável das observações;
- gerar/reimprimir Ficha é permitido por preset para ADM/Gerência/Funcionário em Atendimento acessível;
- concorrência de checklist/observações é granular por recurso.

## Bloco 10 — etapas

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | ✅ Consolidado |
| 2 | PDF de Procedimentos | ✅ Consolidado |
| 3 | DOCX de Procedimentos | ✅ Consolidado |
| 4 | Impressão Windows de Procedimentos | ✅ Consolidado |
| 5 | Template físico de Procedimentos | ✅ Consolidado |
| 6 | PDF + preview da Ficha compacta | ✅ Consolidado |
| 7 | Template físico A4 da Ficha | 🟡 Próxima — ainda não aberta |
| 8 | Limites textuais e densidade da Ficha | ⏳ Pendente |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | ⏳ Pendente |
| 10 | Nomes de arquivo + artefatos temporários | ⏳ Pendente |
| 11 | QR / barcode | ⏳ Pendente |
| 12 | Validação técnica final do Bloco 10 | ⏳ Pendente |

### Etapas 1–5 — baseline

- geração documental Host-side por `DocumentModel` semântico;
- renderização fora da fila de mutações, com limite próprio;
- PDF de Procedimentos via Typst embutido;
- DOCX real via pipeline Rust direto;
- impressão Windows usa o mesmo PDF oficial em WebView2 transitória + `ShowPrintUI(System)`;
- Procedimento físico usa A4 retrato multipágina;
- Reader não possui geometria A4;
- PDF usa Noto Sans/Noto Sans Mono; DOCX referencia Arial/Consolas.

### Etapa 6 — Ficha compacta

A Ficha é uma **prestação de contas resumida ao cliente**.

Conteúdo prioritário:

- identificação do serviço;
- identificação/características do computador/dispositivo;
- processador, RAM, HD/SSD, SO quando útil, bateria quando aplicável;
- observações do Equipamento;
- `Resumo do trabalho`;
- observações gerais;
- observações de serviço registradas nas Etapas.

Por padrão não imprime checklist, percentual/progresso, passos, comandos, timeline, IDs internos ou lista detalhada de revisões.

Geração:

```text
Atendimento confirmado + source_version
→ DocumentModel service_sheet
→ template Typst da Ficha
→ PagedDocument de exatamente 1 página
   ├─→ PDF canônico
   └─→ SVG de preview
```

- 2+ páginas = `SHEET_OVERFLOW`;
- sem segunda folha, corte ou redução silenciosa de fonte;
- preview é modal/overlay simples com folha A4 centralizada;
- `Salvar PDF` e `Imprimir` usam os mesmos bytes PDF da prévia;
- impressão reutiliza WebView2 + diálogo Windows;
- prévia permanece presa à `source_version` e não muda silenciosamente;
- reimpressão de concluído usa estado histórico aplicável.

## Próximo gate

A Etapa 7 só pode começar após:

```text
PR da Etapa 6
→ validação
→ squash merge
→ apagar branch
→ verificar somente main + zero PRs abertos
```

## Fontes principais

- `AGENTS.md`
- `docs/README.md`
- `docs/05-progresso/registro-de-decisoes.md`
- `docs/03-arquitetura/arquitetura-vigente.md`
- `docs/04-planejamento/plano-oficial-fase-1.md`
- `docs/04-planejamento/bloco-9-atendimentos-execucao-checklist.md`
- `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md`

## Regra de atualização deste painel

Todo avanço consolidado de **fase, bloco, tela ou etapa do bloco atual** atualiza este README no mesmo checkpoint documental. Um avanço não está documentalmente encerrado se este painel ficar atrasado.