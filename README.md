# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com simplicidade operacional e implantação de baixo impacto.

## Painel de acompanhamento

**Atualização:** 2026-08-28  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Bloco atual:** Bloco 10 — Exportação / impressão / ficha compacta — **Etapas 1–8 consolidadas; Etapa 9 próxima**  
**Implementação funcional oficial:** ainda não iniciada

Este painel é a visão rápida. Precedência e decisões completas permanecem em `AGENTS.md`, `docs/05-progresso/registro-de-decisoes.md` e documentos específicos.

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
| 10 | Exportação / impressão / ficha compacta | 🟡 Em andamento — Etapas 1–8 consolidadas |
| 11 | Backup / restauração | ⏳ Pendente |
| 12 | Estrutura oficial + plano da Fase 2 | ⏳ Pendente |

## Produto e UI

- `Procedimento`, `Atendimento/Execução` e `Equipamento` são domínios distintos;
- Reader usa experiência de manual/livro;
- `Visão geral` precede Etapa 1 e cada Etapa é página lógica própria;
- stepper compacto de círculos/linhas navega entre Etapas e não representa conclusão operacional;
- UI privilegia clareza com baixa densidade textual;
- cor nunca é o único meio para estado importante;
- Reader operacional persiste checklist e pode registrar `Observação do serviço` opcional por Etapa;
- Ficha compacta é prestação de contas resumida ao cliente, não relatório técnico completo.

## Bloco 9 — operação consolidada

- lifecycle `Em andamento / Concluído / Cancelado`, com reabertura explícita;
- primeiro save cria `AT-000001`; Equipamento usa `EQP-000001`;
- responsável + `Resumo do trabalho` são obrigatórios para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- Funcionário opera por padrão Atendimento do qual é responsável;
- revisão exata do Procedimento é preservada;
- checklist persiste somente em Atendimento;
- observação de serviço por Etapa é opcional, persistente e separada do Procedimento oficial;
- progresso deriva somente do checklist;
- Equipamento é opcional/reutilizável;
- conclusão preserva projeção histórica relevante de Equipamento/estado operacional;
- Ficha pode ser gerada/reimpressa conforme lifecycle/capacidade.

## Bloco 10 — etapas

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | ✅ Consolidado |
| 2 | PDF de Procedimentos | ✅ Consolidado |
| 3 | DOCX de Procedimentos | ✅ Consolidado |
| 4 | Impressão Windows de Procedimentos | ✅ Consolidado |
| 5 | Template físico de Procedimentos | ✅ Consolidado |
| 6 | PDF + preview da Ficha compacta | ✅ Consolidado |
| 7 | Template físico A4 da Ficha | ✅ Consolidado |
| 8 | Limites textuais e densidade da Ficha | ✅ Consolidado |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | 🟡 Próxima — ainda não aberta |
| 10 | Nomes de arquivo + artefatos temporários | ⏳ Pendente |
| 11 | QR / barcode | ⏳ Pendente |
| 12 | Validação técnica final do Bloco 10 | ⏳ Pendente |

### Etapas 1–5 — Procedimentos

- geração documental no Host a partir de snapshot consistente e `DocumentModel`;
- PDF via Typst embutido, sem processo conversor externo;
- DOCX OOXML Transitional via pipeline Rust próprio;
- impressão local no Client Windows usando o mesmo PDF oficial e WebView2 `ShowPrintUI(System)`;
- Procedimento físico A4 retrato multipágina, margens 18 mm;
- PDF usa Noto Sans/Noto Sans Mono incorporadas;
- DOCX usa Arial/Consolas referenciadas;
- nenhuma truncagem/redução silenciosa para caber.

### Etapa 6 — Ficha: PDF + preview

- Ficha é prestação de contas resumida ao cliente;
- conteúdo prioritário: identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações;
- PDF próprio/canônico via template Typst da Ficha;
- preview SVG e PDF derivam do mesmo `PagedDocument`;
- resultado válido exige exatamente uma página;
- `2+ páginas` = `SHEET_OVERFLOW`;
- Salvar/Imprimir reutilizam os mesmos bytes PDF;
- impressão reutiliza o fluxo Windows consolidado.

### Etapa 7 — template físico da Ficha

- A4 retrato, exatamente uma página, margens **15 mm**, sem bleed;
- composição predominantemente vertical/uma coluna;
- cabeçalho institucional compacto, sem título gigante e sem footer obrigatório;
- identificação curta de cliente/OS/técnico, omitindo vazios;
- Equipamento como ficha técnica resumida sem tabela gradeada;
- `SERVIÇO REALIZADO` como área narrativa principal;
- única seção `OBSERVAÇÕES` reunindo observações relevantes do Atendimento, Equipamento e Etapas;
- nome curto da Etapa apenas quando necessário para contexto;
- Noto Sans com baseline 14 / 10,5 / 10 / 9 / 8,5 pt;
- divisórias discretas, contraste neutro e legível em monocromático;
- seções vazias colapsam;
- sem assinatura, financeiro, checklist, progresso, timeline, QR, página 2 ou footer promocional.

### Etapa 8 — limites textuais e densidade

- soft limits recomendados: `Resumo do trabalho` 600, observação geral do Atendimento 400, observação do Equipamento 300 e observação por Etapa 280 caracteres;
- contador/aviso surge somente próximo de aproximadamente 80% da faixa recomendada;
- soft limit não bloqueia save nem conclusão e nunca trunca/altera o dado operacional;
- o layout Typst real é a autoridade final para encaixe em uma A4;
- `SHEET_OVERFLOW` bloqueia somente PDF/preview/impressão da Ficha;
- Host devolve diagnóstico semântico dos principais campos que pressionam a folha;
- correção ocorre nos campos reais do Atendimento, sem editor paralelo da Ficha;
- sem IA, resumo automático, reticências, deduplicação semântica, modo compacto ou redução automática de fonte/margem/espaçamento;
- observações seguem Atendimento → Equipamento → Etapas, preservando textos legítimos;
- hard limits técnicos de storage/API não derivam da geometria A4.

Fonte técnica: `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md`.

## Gate atual

A Etapa 9 não pode ser aberta antes de:

```text
squash merge da Etapa 8
→ remoção da branch remota da Etapa 8
→ remoto somente com main
→ zero PRs abertos
```

## Pendências principais

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- Etapas 9–12 do Bloco 10;
- mecanismo técnico do Bloco 11;
- validações reais do ambiente corporativo;
- estrutura oficial/Fase 2 no Bloco 12.

### Regra de atualização deste painel

Todo avanço consolidado de fase, bloco, tela ou etapa do bloco atual deve atualizar este README no mesmo checkpoint documental.
