# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** CONCLUÍDO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Conclusão:** 2026-08-29

## 1. Objetivo

Fechar o contrato de geração documental, exportação, impressão e Ficha compacta do StepFlow, preservando a UX aprovada e o caráter Pocket.

Este arquivo é o **mapa técnico**. Detalhes ficam nas fontes específicas:

- `../02-telas/14-exportacao-impressao-ficha.md` — UX documental;
- `../03-arquitetura/arquitetura-vigente.md` — mapa arquitetural;
- `../03-arquitetura/launcher-distribuicao-client.md` — distribuição Pocket;
- `../03-arquitetura/compatibilidade-windows-client.md` — Windows/WebView2;
- `bloco-10-etapa-11-validacao-tecnica-final.md` — matriz técnica final;
- `../05-progresso/registro-de-decisoes.md` — decisões vigentes.

## 2. Etapas concluídas

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | CONSOLIDADO |
| 2 | PDF de Procedimentos | CONSOLIDADO |
| 3 | DOCX de Procedimentos | CONSOLIDADO |
| 4 | Impressão Windows de Procedimentos | CONSOLIDADO |
| 5 | Template físico de Procedimentos | CONSOLIDADO |
| 6 | PDF + preview da Ficha compacta | CONSOLIDADO |
| 7 | Template físico A4 da Ficha | CONSOLIDADO |
| 8 | Limites textuais e densidade da Ficha | CONSOLIDADO |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | CONSOLIDADO |
| 10 | Nomes de arquivo + artefatos temporários | CONSOLIDADO |
| 11 | Validação técnica final | CONSOLIDADO |

## 3. Arquitetura documental

```text
Client solicita fonte + revisão esperada
→ Host autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ admite renderização bounded
→ renderer trabalha fora da fila de mutações
→ Host devolve artefato
→ Client salva / preview / imprime
```

Regras:

- geração pertence ao Host;
- Client não envia documento montado;
- renderer não usa DOM/HTML e não reconsulta SQLite;
- sem `export_jobs` persistente inicialmente;
- artefato não vira histórico/backup por padrão;
- Host não grava em path arbitrário da workstation;
- runtime documental não depende de Word, LibreOffice, Adobe Reader, browser externo ou cloud.

## 4. Procedimentos

### PDF

- Typst embutido via crates oficiais sob adaptador;
- `World` restrito a template/assets/fontes autorizados;
- PDF 1.7 + Tagged PDF baseline;
- texto selecionável/pesquisável;
- Noto Sans/Noto Sans Mono incorporadas/subsetadas;
- PNG/JPEG/SVG controlados;
- multipágina automático;
- falha nunca retorna parcial como sucesso.

### DOCX

- OOXML/WordprocessingML/OPC Transitional;
- geração direta em Rust a partir do mesmo `DocumentModel`;
- `docx-rs` preferido sob adaptador;
- texto/listas/numeração editáveis;
- Arial/Consolas referenciadas;
- documento refluível, sem promessa de paginação idêntica ao PDF;
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
- sem impressão silenciosa/seletor próprio baseline;
- sucesso = entrega ao fluxo Windows, não confirmação física do papel.

### Template físico do Procedimento

- A4 retrato multipágina;
- margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- sem truncamento/redução silenciosa;
- PDF é referência física; DOCX é refluível.

## 5. Ficha compacta

A Ficha é **prestação de contas resumida ao cliente**, não relatório técnico completo.

Prioriza identificação do Atendimento/serviço, Equipamento quando houver, características relevantes, `Resumo do trabalho` e observações aplicáveis.

Por padrão não imprime checklist, progresso, passos, comandos/código, timeline/auditoria ou lista detalhada de Procedimentos vinculados.

### PDF + preview

```text
Atendimento confirmado
→ DocumentModel service_sheet
→ template Typst
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

| Campo | Soft limit recomendado |
|---|---:|
| `Resumo do trabalho` | 600 |
| Observação geral do Atendimento | 400 |
| Observação do Equipamento | 300 |
| Observação do serviço por Etapa | 280 por Etapa |

- orientação, não hard limit;
- aviso perto de aproximadamente 80%;
- não bloqueiam save/conclusão;
- Typst real decide encaixe;
- overflow é corrigido nos campos reais, sem editor paralelo, IA, resumo automático ou compactação automática.

### Dados excepcionais

- Procedimentos vinculados ficam fora da Ficha por padrão;
- MACs: 0 omite; 1–2 exibem valores; 3+ exibem quantidade cadastrada;
- não inventar `MAC principal`;
- observações legítimas não recebem cap/descarte automático;
- multiplicidade real pode produzir `SHEET_OVERFLOW`;
- strings longas quebram linha quando possível, sem truncamento/reticências/abreviação inventada.

## 6. Naming e artefatos temporários

Procedimento:

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

- sanitização afeta somente filename;
- impedir nome reservado/path injection;
- preservar Unicode válido;
- conflito não causa overwrite silencioso;
- save só é sucesso após gravação integral;
- helper opaco no mesmo destino + promoção segura são preferidos quando suportados;
- arquivo salvo pelo usuário nunca entra no cleanup normal.

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

## 7. Gate Pocket

```text
pasta pronta no servidor
→ usuário acessa share
→ executa Launcher
→ Client/resources são preparados localmente automaticamente
→ Client abre
```

Não aceitar como baseline instalador por estação, preparação manual, elevação administrativa, Internet obrigatória ou execução permanente do Client pelo SMB.

WebView2 Evergreen existente é preferível. Fixed Version não roda por UNC/SMB; fallback local só entra após PoC provar `%LOCALAPPDATA%` sem instalação/admin/manualidade.

## 8. Resultado da validação técnica

A Etapa 11 concluiu:

- nenhum bloqueador arquitetural conhecido para os contratos documentais;
- Typst/PDF/PagedDocument viáveis;
- DOCX direto em Rust viável com Word corporativo como gate real;
- impressão Windows viável via WebView2 nativo + `ShowPrintUI(System)`;
- família Tauri/Wry/WebView2 do adaptador deve ser pinada/testada;
- save local, naming, temporários e scavenging são viáveis com limites explícitos;
- SMB, Word, impressoras, Windows/WebView2 e EDR permanecem gates de ambiente real;
- memória/tamanho/concorrência/fila/timeout serão definidos por medição na fase executável.

## 9. Pendências externas ao fechamento arquitetural

Antes do deployment correspondente, validar no ambiente real:

- versões/edições de Windows 10/11;
- WebView2 Evergreen e PoC do fallback Pocket;
- execução do Launcher pelo compartilhamento;
- Word realmente instalado;
- impressoras/drivers;
- SMB real: permissões/rename/replace/queda de rede/quota/latência;
- EDR/antivírus;
- políticas de long path/execução.

Fora da empresa, registrar `NÃO APLICÁVEL NESTE AMBIENTE` em vez de inventar resultado.

## 10. Encerramento

O Bloco 10 está **concluído**. O histórico de PR/branch que levou a esse estado permanece no Git e não faz parte do contrato técnico vigente.
