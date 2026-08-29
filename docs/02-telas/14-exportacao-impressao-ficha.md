# Tela 14 — Exportação / Impressão + Ficha Compacta — UX

## 1. Identificação

- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco original: Fase 1 — Bloco 8 (UI/UX);
- atualização técnica/funcional: Bloco 10 / Etapas 1–11;
- última consolidação: 2026-08-29.

## 2. Objetivo

Definir duas famílias distintas de saída sem transformar telas em documentos:

1. **Procedimento** — exportar/imprimir uma revisão específica como documento técnico completo;
2. **Ficha compacta de Atendimento** — prestação de contas resumida ao cliente, com exatamente uma folha A4 quando válida.

As duas superfícies compartilham a arquitetura documental do Host, mas possuem templates e objetivos diferentes.

## 3. Princípios

- documento nunca é screenshot da interface;
- geração pertence ao Host e usa estado/revisão autorizados;
- identidade da empresa vem da configuração central;
- exportar/imprimir não altera dados funcionais;
- revisão/estado selecionado não é substituído silenciosamente;
- campos vazios/não aplicáveis são omitidos;
- Client mantém a experiência local de salvar/preview/imprimir;
- autorização real permanece Host-side;
- baixa densidade textual orienta preview e documento;
- arquivo persistente escolhido pelo usuário e temporário interno possuem lifecycles distintos;
- validações de Word/SMB/impressoras/EDR não alteram UX aprovada sem bloqueador técnico.

## 4. Fluxos

Procedimento:

```text
Leitor de Processo
→ Exportar / Imprimir
→ PDF | DOCX | Imprimir
→ documento completo da revisão selecionada
```

Ficha:

```text
Atendimento
→ Ficha / Imprimir
→ preview da prestação de contas
→ Salvar PDF | Imprimir
```

Não criar item global `Exportações` na sidebar.

---

# Procedimentos

## 5. Fonte e permissão

A matriz vigente permite exportar/imprimir para ADM, Gerência e Funcionário quando a sessão também pode ler a revisão selecionada.

A revisão aberta é a fonte exata:

```text
r17 aberta
→ solicitar exportação de r17
→ r18 surge/publica
→ documento continua r17
```

Se a revisão deixar de estar autorizada antes da geração ser aceita, o Host rejeita a operação.

## 6. Entrada

```text
Leitor
→ menu contextual
→ Exportar / Imprimir
```

Exemplo:

```text
PR-014 · Configuração de VLAN
Versão 2.0 · revisão r18 · Publicada

[ Exportar PDF ] [ Exportar DOCX ] [ Imprimir ]
```

Revisão histórica/draft autorizado deve ser identificada inequivocamente.

## 7. Conteúdo

O Procedimento exportado contém, quando aplicável:

- identidade da empresa;
- código/título;
- Área/Departamento;
- responsável documental;
- categorias;
- versão/revisão/estado editorial necessário;
- objetivo;
- pré-requisitos;
- observações gerais;
- Etapas em ordem;
- parágrafos;
- passos/subpassos;
- checklist documental;
- notas/alertas;
- comandos/código.

Não entram sidebar, stepper, botões, toasts, ícones de copiar ou chrome da UI.

## 8. Saídas e layout

- PDF via Typst embutido no Host;
- DOCX OOXML real via pipeline Rust próprio;
- impressão usa o mesmo PDF oficial no Client Windows;
- A4 retrato multipágina;
- margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- PDF usa Noto Sans/Noto Sans Mono incorporadas;
- DOCX referencia Arial/Consolas;
- PDF é referência física; DOCX é refluível.

Nome persistente sugerido:

```text
{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}
```

Sem `display_version`, omitir somente esse segmento. O usuário pode editar o nome no diálogo nativo.

## 9. Estados

Suportar:

- preparando;
- pronto para salvar/imprimir;
- concluído;
- cancelado pelo usuário;
- falha de geração;
- Host indisponível;
- sem permissão;
- revisão indisponível/obsoleta;
- `SERVER_BUSY`/capacidade saturada.

Não inventar percentual sem progresso real.

---

# Ficha compacta de Atendimento

## 10. Finalidade

A Ficha é uma **prestação de contas resumida para o cliente**.

Deve responder rapidamente:

```text
qual serviço foi realizado?
em qual dispositivo?
quais características principais?
qual foi o panorama geral do trabalho?
houve alguma observação relevante?
```

Não é relatório técnico detalhado, checklist impresso, reprodução do Procedimento nem timeline operacional.

## 11. Entrada e lifecycle

```text
Tela 09 — Atendimento
→ Ficha / Imprimir
```

Preset:

- ADM: sim;
- Gerência: sim;
- Funcionário: sim para Atendimento acessível.

Lifecycle:

- `Em andamento`: pode gerar estado confirmado atual;
- `Concluído`: reimprime estado histórico aplicável;
- `Cancelado`: pode gerar/reimprimir identificando claramente `CANCELADO`.

Alterações locais não salvas ou conflito pendente bloqueiam a geração.

## 12. Fonte confirmada

```text
Ficha / Imprimir
→ existem alterações não salvas/conflito?
   → sim: salvar/reconciliar primeiro
   → não: Host captura estado/revisão esperada
→ gerar
```

Prévia aberta permanece ligada à `source_version`. Evento remoto não troca a folha silenciosamente.

## 13. Conteúdo essencial

### Serviço

Quando aplicável:

- código do Atendimento;
- OS/referência externa;
- cliente/solicitante;
- técnico/responsável;
- data/status necessário;
- `Resumo do trabalho`;
- observações gerais relevantes.

### Equipamento — quando houver

Priorizar identificação/nome/tipo, CPU, RAM, armazenamento, SO quando útil, serial/patrimônio quando úteis, bateria quando aplicável e observações relevantes.

MACs:

```text
0 → omitir
1 → exibir valor
2 → exibir ambos
3+ → "MACs: N identificadores cadastrados"
```

Não inventar `MAC principal`.

### Observações de serviço por Etapa

Observações preenchidas podem compor a Ficha em ordem de execução. Nome curto da Etapa aparece apenas quando necessário para contexto.

Observações legítimas não recebem cap/descarte automático apenas para caber.

## 14. O que não aparece por padrão

- checklist completo;
- percentual/progresso;
- passos/subpassos;
- comandos/código;
- timeline/auditoria;
- IDs internos;
- detalhes de concorrência;
- lista detalhada de revisões;
- lista de Procedimentos vinculados;
- metadados editoriais sem utilidade ao cliente.

Os vínculos/revisões permanecem internamente para histórico e consistência.

## 15. Atendimento sem Equipamento

- seção de Equipamento é omitida;
- não reservar área vazia;
- identificação do serviço, resumo e observações continuam válidos;
- não imprimir `Nenhum equipamento vinculado`.

## 16. Regra de uma A4

Ficha válida possui **exatamente uma página A4**.

```text
1 página → válida
2+ páginas → SHEET_OVERFLOW
```

Não criar segunda folha, cortar, truncar, reduzir fonte dinamicamente ou omitir conteúdo legítimo silenciosamente.

Mensagem funcional base:

`A ficha ficou extensa demais para uma página A4. Revise os textos indicados e gere novamente.`

### Soft limits

| Campo | Faixa recomendada |
|---|---:|
| `Resumo do trabalho` | 600 caracteres |
| Observação geral do Atendimento | 400 |
| Observação do Equipamento | 300 |
| Observação do serviço por Etapa | 280 por Etapa |

- orientativos;
- não bloqueiam save/conclusão;
- contador/aviso somente perto de aproximadamente 80%;
- o layout Typst real decide encaixe;
- hard limits técnicos são independentes da A4.

### Overflow

```text
SHEET_OVERFLOW
→ nenhum PDF final válido
→ nenhum preview final válido
→ nenhuma impressão
→ Atendimento permanece íntegro
```

Host devolve diagnóstico semântico dos campos que mais pressionam a folha. Client leva o técnico aos **campos reais**, sem editor paralelo exclusivo da Ficha.

Não resolver overflow com:

- IA/resumo automático;
- truncamento/reticências;
- deduplicação semântica;
- modo compacto;
- redução automática de fonte/margem/espaçamento;
- omissão silenciosa de conteúdo legítimo.

## 17. Template físico

Geometria:

```text
papel: A4
orientação: retrato
páginas: exatamente 1
margens: 15 mm
bleed: nenhum
```

Ordem:

```text
1. identidade da empresa + Atendimento
2. identificação curta do serviço
3. Equipamento, quando houver
4. SERVIÇO REALIZADO
5. OBSERVAÇÕES, quando houver
```

Regras:

- composição predominantemente vertical/uma coluna;
- cabeçalho institucional compacto;
- logo opcional;
- sem título gigante e sem footer obrigatório;
- `CANCELADO` textual/inequívoco;
- cliente/OS/técnico em linha curta, omitindo vazios;
- Equipamento em ficha técnica resumida sem grade pesada;
- `SERVIÇO REALIZADO` usa o Resumo como narrativa principal;
- `OBSERVAÇÕES` é uma única seção;
- seções vazias colapsam;
- sem caixas para escrita manual.

Tipografia PDF: Noto Sans.

```text
identificação principal: 14 pt
seção: 10,5 pt
corpo/resumo: 10 pt
ficha técnica: 9 pt
metadados: 8,5 pt
```

Contraste neutro e legível em monocromático. Não reduzir dinamicamente abaixo do baseline.

## 18. PDF, preview, salvar e imprimir

```text
Atendimento confirmado
→ DocumentModel service_sheet
→ template Typst
→ PagedDocument de 1 página
   ├─→ PDF canônico
   └─→ SVG preview
```

Preview:

- modal/overlay grande;
- folha A4 centralizada;
- controles compactos;
- sem nova sidebar/toolbar extensa;
- foco acessível;
- PDF e preview pertencem à mesma `source_version`.

`Salvar PDF` e `Imprimir` reutilizam os mesmos bytes PDF da prévia aberta.

Impressão usa WebView2 dedicada/transitória + `ShowPrintUI(System)`; StepFlow informa entrega ao fluxo Windows, não confirmação física do papel.

## 19. Save e temporários

Ficha sugere:

```text
{service_code} - Ficha.pdf
```

Não incluir cliente, equipamento, serial/patrimônio/MAC, resumo, técnico, status ou timestamp no filename por padrão.

Conflito de nome não sobrescreve silenciosamente arquivo existente. Save só é sucesso após gravação integral.

Temporários:

```text
bytes canônicos
→ manter em memória quando suficiente
→ materializar localmente apenas se integração exigir filesystem
→ consumir
→ remover best-effort após liberação
```

- raiz temporária por usuário, namespace StepFlow e diretório opaco por Client;
- nomes opacos sem dados de negócio;
- não usar instalação, share central, SQLite, backup, Documents/Desktop/Downloads como raiz normal;
- não apagar enquanto WebView2/Windows ainda puder usar;
- lock não autoriza kill/unlock forçado/alteração de ACL;
- crash pode deixar órfãos para scavenging posterior conservador;
- reparse point não pode levar cleanup para fora da raiz gerenciada;
- sem serviço/daemon/tarefa agendada para limpeza.

## 20. Validação técnica final do Bloco 10

A Etapa 11 confirmou a viabilidade arquitetural deste contrato sem alterar a UX.

- Typst/PDF/PagedDocument: validados;
- DOCX direto: viável, com Word corporativo pendente;
- impressão Windows: viável por WebView2 nativo + `ShowPrintUI(System)`;
- Tauri/Wry/WebView2 do adapter precisam ser pinados/testados;
- lifetime do PDF durante impressão depende de teste Windows real;
- SMB, impressoras, Word e EDR são gates de ambiente real;
- limites de memória/concorrência serão definidos por benchmark;
- nenhum bloqueador arquitetural identificado.

O contrato Pocket de distribuição não muda a Tela 14: o usuário abre o sistema pelo Launcher no compartilhamento e o Client roda localmente sem instalação manual.

## 21. Estados e acessibilidade

Estados mínimos:

- preparando;
- pronta;
- desatualizada;
- cancelada pelo usuário;
- sem permissão;
- fonte indisponível;
- `SHEET_OVERFLOW`;
- falha de renderer/preview;
- Host indisponível;
- `SERVER_BUSY`;
- falha de gravação;
- falha de cleanup não bloqueante.

- não exibir percentual fictício;
- ações operáveis por teclado;
- icon-only somente quando inequívoco, com nome acessível;
- foco gerenciado no modal;
- status importantes não dependem só de cor;
- folha permanece A4 independentemente da janela.

## 22. Fora do escopo inicial

- assinatura digital;
- envio por e-mail;
- nuvem/compartilhamento externo;
- exportação em lote/ZIP;
- editor visual de templates;
- armazenamento permanente de toda exportação;
- DOCX da Ficha;
- implementação funcional nesta fase.

## 23. Decisões consolidadas pelo PO

- exportação/impressão contextual;
- revisão exata preservada;
- PDF/DOCX/impressão de Procedimento próprios;
- Ficha = prestação de contas resumida;
- Ficha com/sem Equipamento;
- observações por Etapa podem compor a Ficha;
- checklist/progresso/revisões detalhadas não poluem a saída;
- Ficha válida = exatamente uma A4;
- PDF + preview do mesmo `PagedDocument`;
- Salvar/Imprimir reutilizam o PDF da prévia;
- soft limits orientativos;
- `SHEET_OVERFLOW` preserva o Atendimento;
- overflow é corrigido nos dados reais;
- sem IA/truncamento/dedup/compactação automática;
- Procedimentos vinculados não são listados por padrão;
- MACs usam projeção 0 / 1–2 / 3+;
- observações legítimas não sofrem cap automático;
- naming Windows previsível/sanitizado;
- arquivo persistente separado de temporários;
- temporários isolados por Client e cleanup best-effort;
- DOCX da Ficha não é requisito inicial;
- validação técnica final não encontrou bloqueador arquitetural para a UX aprovada.
