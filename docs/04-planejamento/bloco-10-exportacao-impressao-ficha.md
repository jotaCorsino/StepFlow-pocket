# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** ETAPAS 1–11 CONSOLIDADAS / APROVADAS PELO PO — FECHAMENTO OPERACIONAL PENDENTE DO GATE GIT  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 11 consolidada:** 2026-08-29

## 1. Objetivo

Fechar o contrato de geração documental, exportação, impressão e Ficha compacta do StepFlow, preservando a UX aprovada e o caráter Pocket.

Este arquivo é o **mapa técnico** do Bloco 10. Detalhes ficam nas fontes específicas:

- `../02-telas/14-exportacao-impressao-ficha.md` — UX documental;
- `../03-arquitetura/arquitetura-vigente.md` — arquitetura vigente;
- `../03-arquitetura/launcher-distribuicao-client.md` — distribuição Pocket;
- `../03-arquitetura/compatibilidade-windows-client.md` — Windows/WebView2;
- `bloco-10-etapa-11-validacao-tecnica-final.md` — matriz técnica final;
- `../05-progresso/registro-de-decisoes.md` — decisões/gates.

Não pertence a este bloco implementar código de produção, fechar Backup/Restore técnico ou abrir o Bloco 11 antes do gate remoto.

## 2. Etapas

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 3 | DOCX de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 4 | Impressão Windows de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 5 | Template físico de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 6 | PDF + preview da Ficha compacta | **CONSOLIDADO / APROVADO PELO PO** |
| 7 | Template físico A4 da Ficha | **CONSOLIDADO / APROVADO PELO PO** |
| 8 | Limites textuais e densidade da Ficha | **CONSOLIDADO / APROVADO PELO PO** |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | **CONSOLIDADO / APROVADO PELO PO** |
| 10 | Nomes de arquivo + artefatos temporários | **CONSOLIDADO / APROVADO PELO PO** |
| 11 | Validação técnica final do Bloco 10 | **CONSOLIDADO / APROVADO PELO PO** |

## 3. Arquitetura documental

Fluxo consolidado:

```text
Client
→ solicita identidade da fonte + revisão esperada
Host
→ autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ admite renderização em capacidade bounded
→ renderer trabalha fora da fila de mutações
→ devolve artefato autenticado
Client
→ salva / preview / imprime conforme o contrato
```

Regras:

- geração pertence ao Host;
- Client não envia documento montado;
- renderer não usa DOM/HTML da UI e não reconsulta SQLite;
- sem `export_jobs` persistente inicialmente;
- artefato não vira histórico/backup por padrão;
- Host não grava em path arbitrário da workstation;
- runtime documental não depende de Word, LibreOffice, Adobe Reader, browser externo ou cloud.

## 4. Procedimentos

### PDF

- Typst embutido no Host via crates oficiais e adaptador interno;
- `World` restrito a template/assets/fontes autorizados;
- PDF 1.7 + Tagged PDF como baseline;
- texto real selecionável/pesquisável;
- fontes Noto Sans/Noto Sans Mono incorporadas/subsetadas;
- PNG/JPEG/SVG controlados;
- multipágina automático sem truncamento;
- falha nunca retorna parcial como sucesso.

### DOCX

- OOXML/WordprocessingML/OPC Transitional;
- geração direta em Rust a partir do mesmo `DocumentModel`;
- `docx-rs` preferido sob adaptador interno;
- texto/listas/numeração editáveis;
- Arial/Consolas referenciadas, sem embedding inicial;
- documento refluível; não promete paginação do PDF;
- sem Word/COM ou conversor externo.

### Impressão

```text
PDF oficial
→ Client Windows
→ recurso local transitório quando necessário
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo Windows
```

- impressão física ocorre no Client;
- usa o mesmo PDF oficial;
- sem impressão silenciosa/seletor próprio como baseline;
- sucesso = entrega ao fluxo Windows, não confirmação física do papel.

### Template físico

- A4 retrato multipágina;
- margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- PDF é referência física; DOCX é refluível;
- regra de uma A4 não se aplica ao Procedimento completo.

## 5. Ficha compacta

A Ficha é prestação de contas resumida ao cliente, não relatório técnico completo.

Prioriza:

- identificação do Atendimento/serviço;
- dispositivo/equipamento quando houver;
- características relevantes;
- `Resumo do trabalho`;
- observações gerais e observações de serviço por Etapa quando preenchidas.

Não imprime por padrão checklist, progresso, passos, comandos/código, timeline/auditoria, IDs internos ou lista detalhada de Procedimentos vinculados.

### PDF + preview

```text
Atendimento confirmado
→ DocumentModel service_sheet
→ template Typst da Ficha
→ PagedDocument

1 página
→ PDF canônico + preview SVG

2+ páginas
→ SHEET_OVERFLOW
```

Salvar e Imprimir reutilizam os mesmos bytes PDF da prévia correspondente à `source_version`.

### Template físico

- A4 retrato;
- exatamente uma página;
- margens 15 mm;
- composição predominantemente vertical;
- cabeçalho institucional compacto;
- Equipamento em ficha técnica resumida sem grade pesada;
- `SERVIÇO REALIZADO` como área narrativa principal;
- uma seção `OBSERVAÇÕES` para observações aplicáveis;
- seções vazias colapsam;
- Noto Sans com baseline 14 / 10,5 / 10 / 9 / 8,5 pt;
- sem redução dinâmica de fonte, segunda página ou truncamento.

### Limites textuais

Soft limits:

| Campo | Faixa recomendada |
|---|---:|
| `Resumo do trabalho` | 600 caracteres |
| Observação geral do Atendimento | 400 |
| Observação do Equipamento | 300 |
| Observação do serviço por Etapa | 280 por Etapa |

- orientação, não hard limit;
- contador/aviso perto de aproximadamente 80%;
- não bloqueiam save/conclusão;
- Typst real decide o encaixe;
- overflow orienta revisão dos campos reais, sem editor paralelo/IA/resumo automático/compactação automática.

### Dados excepcionais

- Procedimentos vinculados permanecem fora da Ficha por padrão;
- MACs: 0 omite; 1–2 exibem valores; 3+ exibem apenas a quantidade cadastrada;
- não inventar `MAC principal`;
- observações legítimas não recebem cap/descarte automático;
- multiplicidade real pode produzir `SHEET_OVERFLOW`;
- strings longas quebram linha quando possível, sem reticências/abreviação inventada.

## 6. Naming e artefatos temporários

Nome sugerido de Procedimento:

```text
{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}
```

Sem versão editorial:

```text
{codigo} - {titulo} - r{revision_no}.{ext}
```

Ficha:

```text
{service_code} - Ficha.pdf
```

Regras:

- sanitização afeta somente filename;
- impedir caracteres/nome reservado/path injection;
- preservar Unicode válido;
- conflito não gera overwrite silencioso;
- save só é sucesso após gravação integral;
- helper opaco no mesmo destino + promoção segura são preferidos quando suportados;
- arquivo salvo pelo usuário nunca entra no cleanup normal do StepFlow.

Temporários:

```text
<temp do usuário>/StepFlow/artifacts/<client-instance-id>/
```

- materializar somente quando integração local exigir filesystem;
- nomes opacos sem dados de negócio;
- isolamento por instância;
- cleanup/scavenging best-effort;
- lock não autoriza kill/unlock forçado/alteração de ACL;
- reparse point nunca é seguido para fora da raiz gerenciada;
- sem serviço/daemon/Task Scheduler/watchdog.

## 7. Validação técnica final — Etapa 11

A matriz final em `bloco-10-etapa-11-validacao-tecnica-final.md` concluiu:

- **nenhum bloqueador arquitetural** para os contratos documentais das Etapas 1–10;
- Typst/PDF/PagedDocument viáveis;
- DOCX Rust direto viável com teste corporativo do Word pendente;
- impressão Windows viável via WebView2 nativo + `ShowPrintUI(System)`;
- Tauri/Wry/WebView2 do adaptador precisam ser pinados/testados;
- save local, naming, temporários e scavenging são viáveis com limites explícitos;
- SMB, Word, impressoras e EDR permanecem gates de ambiente real;
- memória, tamanho, concorrência, fila e timeout serão definidos por medição na fase executável.

## 8. Gate Pocket

O StepFlow continua sendo Pocket no sentido operacional:

```text
pasta pronta no servidor
→ usuário acessa share
→ executa Launcher
→ Client/resources preparados localmente de forma automática
→ Client abre
```

Não aceitar como baseline:

- instalador obrigatório por estação;
- preparação manual de dependência;
- elevação administrativa no uso normal;
- Internet obrigatória;
- execução permanente do Client pelo SMB.

WebView2:

- Evergreen compatível já existente é preferível;
- Fixed Version não pode ser executado por UNC/SMB;
- fallback Fixed/local só pode ser adotado após PoC provar preparação em `%LOCALAPPDATA%` sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo;
- se isso não for possível numa estação que deva ser suportada, o fallback volta à arquitetura como bloqueador.

## 9. Pendências de ambiente real

Antes do deployment correspondente, validar:

- versões/edições reais de Windows 10/11;
- WebView2 Evergreen e PoC do fallback Pocket;
- execução do Launcher pelo compartilhamento corporativo;
- Word realmente instalado;
- impressora física + Microsoft Print to PDF + driver corporativo;
- lifecycle/cleanup do PDF durante impressão;
- SMB real: permissões/rename/replace/queda de rede/quota/latência;
- EDR/antivírus;
- políticas de long path/execução.

Quando fora da empresa, registrar `NÃO APLICÁVEL NESTE AMBIENTE` em vez de inventar resultado.

## 10. Encerramento

O Bloco 10 está **documentalmente consolidado** nas Etapas 1–11.

O fechamento operacional ainda exige:

```text
PR da Etapa 11
→ validação final do diff
→ ready
→ squash merge em main
→ apagar branch remota
→ verificar somente main + zero PRs abertos
```

Somente após esse gate o Bloco 11 — Backup / restauração pode ser formalmente aberto.
