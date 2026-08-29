# Bloco 10 — Etapa 11 — Validação técnica final

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Data:** 2026-08-29  
**Base original da validação:** `main` em `d74459b2b342a9fda2ccc4e0645c02613edc4fc8`

## 1. Objetivo

Validar tecnicamente os contratos das Etapas 1–10, identificando mecanismos implementáveis, limites e evidências necessárias sem antecipar código de produção.

A validação não reabre produto por inferência. Requisito só volta ao PO diante de bloqueio técnico concreto.

## 2. Resultado executivo

**Nenhum bloqueador arquitetural foi identificado para os contratos documentais do Bloco 10.**

Permanecem três classes conscientes de limite:

1. **implementação** — adaptadores, pinagem de versões, fila bounded, cleanup conservador etc.;
2. **medição** — memória, tamanho, concorrência, fila e timeout definidos por benchmark/fixtures;
3. **ambiente real** — Word, impressoras, SMB, WebView2/Windows corporativo, políticas e EDR.

Nenhum número de performance foi inventado nesta fase.

## 3. Gate Pocket obrigatório

```text
pasta pronta publicada no servidor Windows
→ estação acessa o compartilhamento
→ usuário executa StepFlowLauncher.exe
→ Launcher prepara/valida recursos locais automaticamente
→ Client abre de %LOCALAPPDATA%
```

Uma solução não atende ao StepFlow Pocket se exigir, por estação:

- instalador tradicional obrigatório;
- configuração manual de dependência;
- privilégio administrativo no fluxo normal;
- toolchain de desenvolvimento;
- Internet obrigatória para uso normal;
- execução permanente do Client diretamente pelo SMB.

A preparação automática em `%LOCALAPPDATA%` faz parte do comportamento Pocket.

## 4. Matriz consolidada

| ID | Contrato | Estado | Consequência técnica |
|---|---|---|---|
| D01 | Typst embutido no Host | **VALIDADO** | crates oficiais sob adaptador; sem `typst.exe` |
| D02 | `World` Typst restrito | **VALIDADO** | somente template/assets/fontes autorizados; sem resolução remota arbitrária |
| D03 | PDF 1.7 + Tagged PDF baseline | **VALIDADO** | manter baseline testado; sem promessa formal PDF/A/PDF/UA |
| D04 | PDF/preview via mesmo `PagedDocument` | **VALIDADO** | page count é autoridade física da Ficha |
| D05 | `SHEET_OVERFLOW` | **VALIDADO** | 2+ páginas falham; sem truncamento/segunda folha/redução automática |
| D06 | Soft limits 600/400/300/280 | **VALIDADO COM LIMITE** | orientação editorial; ajuste somente por fixtures/evidência |
| D07 | DOCX OOXML direto em Rust | **VALIDADO COM LIMITE** | `docx-rs` preferido sob adaptador; Word real ainda deve ser testado |
| D08 | Arial/Consolas sem embedding | **VALIDADO COM LIMITE** | substituição pode alterar reflow; DOCX não promete paginação do PDF |
| D09 | Transporte binário Host → Client | **VALIDADO** | body binário quando apropriado; evitar base64 em JSON sem necessidade |
| D10 | Renderização fora da fila SQLite | **VALIDADO** | CPU/blocking separado da fila de mutações |
| D11 | Concorrência documental bounded | **VALIDADO COM LIMITE** | admissão + semáforo/executor; números por benchmark |
| D12 | `ShowPrintUI(System)` | **VALIDADO** | diálogo oficial do Windows; sem silent print/seletor próprio baseline |
| D13 | Tauri → WebView2 nativo | **VALIDADO COM LIMITE** | `with_webview` + adaptador Windows; pinagem/teste Tauri/Wry/WebView2 |
| D14 | Lifetime do PDF durante impressão | **PENDENTE DE AMBIENTE REAL** | validar load/print/cancel/close/cleanup em Windows real |
| D15 | Naming Windows | **VALIDADO** | sanitização e extensão controlada |
| D16 | Long paths | **VALIDADO COM LIMITE** | depende também de manifesto/política Windows |
| D17 | Save local NTFS | **VALIDADO COM LIMITE** | helper no mesmo destino + write/flush/close + promoção; sem promessa ACID absoluta |
| D18 | Save em SMB | **PENDENTE DE AMBIENTE REAL** | testar permissões, rename/replace, queda de rede, quota e pós-falha |
| D19 | Temp dir por usuário | **VALIDADO** | API OS/Tauri; sem path hardcoded |
| D20 | Isolamento por Client | **VALIDADO COM LIMITE** | subdiretório/ID próprios + sinal de atividade equivalente |
| D21 | Reparse-safe scavenging | **VALIDADO** | nunca seguir reparse para fora da raiz gerenciada; incerteza = skip |
| D22 | Locks WebView2/AV | **VALIDADO COM LIMITE** | cleanup best-effort; sem kill/unlock forçado/alteração de ACL |
| D23 | Word corporativo | **PENDENTE DE AMBIENTE REAL** | testar versões realmente existentes |
| D24 | Impressoras/driver corporativo | **PENDENTE DE AMBIENTE REAL** | impressora física + Microsoft Print to PDF + cancelamento/falhas |
| D25 | EDR/antivírus corporativo | **PENDENTE DE AMBIENTE REAL** | validar execução, handles e cleanup |
| D26 | WebView2 Evergreen existente | **VALIDADO COM LIMITE** | preferível quando presente/compatível; presença real deve ser detectada |
| D27 | WebView2 Fixed Version por UNC/SMB | **NÃO UTILIZAR** | Fixed Version não roda de localização de rede/UNC |
| D28 | Fallback WebView2 autocontido local | **VALIDADO COM LIMITE / PoC OBRIGATÓRIA** | só adotar se funcionar em `%LOCALAPPDATA%` sem instalação/admin/manualidade |
| D29 | Windows 10 + Fixed Runtime moderno | **VALIDADO COM LIMITE / PoC OBRIGATÓRIA** | requisitos ACL/AppContainer precisam ser automatizáveis no perfil do usuário |
| D30 | Contrato Pocket zero instalação/manualidade | **VALIDADO COMO GATE** | solução que exija setup/admin manual retorna à arquitetura |

## 5. Pipeline documental

```text
Client solicita fonte + revisão esperada
→ Host autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ admite renderização bounded
→ renderer PDF/DOCX trabalha fora da fila de mutações
→ Host devolve bytes
```

Typst:

```text
DocumentModel
→ adaptador interno
→ World restrito
→ compile → PagedDocument
→ PDF / SVG
```

Versões de crates serão pinadas em conjunto na fundação executável. O MSRV efetivo será o maior exigido pelas dependências escolhidas e afeta ambiente de build, não estações/produção.

## 6. Ficha A4

```text
1 página
→ PDF + preview válidos

2+ páginas
→ SHEET_OVERFLOW
```

Fixtures futuras devem cobrir:

- sem Equipamento;
- opcionais vazios;
- 0/1/2/3+ MACs;
- Unicode/acentos;
- strings estruturadas longas;
- muitas observações por Etapa;
- caso limite de uma página;
- caso determinístico de duas páginas;
- ausência de truncamento/redução automática.

Soft limits 600/400/300/280 continuam orientação, não hard limit de domínio.

## 7. DOCX

OOXML/WordprocessingML direto em Rust permanece viável sem Word/COM, LibreOffice ou conversão de PDF.

O adaptador deve isolar `docx-rs` do domínio. Validação corporativa futura deve confirmar:

- abertura nas versões reais do Word;
- texto/listas/numeração editáveis;
- imagens;
- whitespace de comandos/código;
- reflow com Arial/Consolas disponíveis ou substituídas;
- salvar novamente sem corrupção relevante.

## 8. Impressão Windows

```text
PDF oficial
→ Client Windows
→ recurso local transitório quando necessário
→ WebView2 dedicada/transitória
→ acesso nativo via adaptador
→ ShowPrintUI(System)
```

O método genérico de impressão do Tauri não substitui por inferência esse contrato. A família Tauri/Wry/WebView2 usada pelo adaptador deve ser pinada/testada.

Lifetime exato do PDF durante diálogo/spooler permanece ensaio Windows real.

## 9. WebView2 e Pocket

### Evergreen

Quando compatível e já presente, é preferível. Launcher/Client deve detectar disponibilidade real; não assumir presença em todo Windows corporativo.

### Fixed Version

- não pode rodar diretamente de UNC/SMB;
- precisa ser mantido/atualizado pelo distribuidor;
- em Windows 10, versões modernas para app Win32 unpackaged possuem requisitos adicionais de ACL/AppContainer.

Fallback, se necessário:

```text
share
→ Launcher
→ copiar Client + runtime aprovado para área local versionada
→ aplicar automaticamente preparação permitida
→ iniciar Client local
```

### PoC obrigatória

Antes de afirmar suporte Pocket para estação sem Evergreen utilizável, provar em PoC descartável:

- Windows 10 x64 alvo;
- Windows 11 x64;
- usuário padrão, sem elevação;
- share acessível;
- Client + Fixed Runtime em `%LOCALAPPDATA%`;
- configuração sem instalador;
- ACL necessária automática, se aplicável;
- funcionamento sem Internet;
- atualização sem registro/PATH global;
- rollback/remoção sem afetar outros aplicativos.

Se estação que deva ser suportada exigir instalação, admin ou preparação manual, o fallback fica bloqueado até redesign.

## 10. Filesystem e temporários

Save persistente:

```text
bytes completos
→ helper opaco no diretório escolhido
→ write
→ flush/close
→ promoção/replace após fluxo aplicável
→ sucesso
```

Temporários:

```text
<temp do usuário>/StepFlow/artifacts/<client-instance-id>/
```

- nomes opacos;
- instância não limpa outra potencialmente ativa;
- mecanismo equivalente a sinal de atividade pode ser usado;
- reparse point nunca é seguido para fora da raiz gerenciada;
- lock de WebView2/EDR gera cleanup best-effort;
- falha de cleanup não retroage sucesso funcional;
- sem daemon, serviço ou Task Scheduler para limpeza.

## 11. Performance e backpressure

Na fundação executável medir:

- tamanho representativo/extremo de PDF/DOCX;
- consumo simultâneo Host/Client;
- renderizações simultâneas;
- profundidade de admissão/fila;
- timeout operacional;
- latência LAN;
- comportamento saturado (`SERVER_BUSY` ou equivalente).

Esses limites são técnicos e independentes da regra de uma A4.

## 12. Gates de ambiente real

### Windows/WebView2

- versões/edições reais de Windows 10/11;
- Evergreen disponível/compatível;
- PoC de fallback local;
- execução do Launcher pelo compartilhamento;
- cenário sem Internet.

### Impressão

- impressora física representativa;
- Microsoft Print to PDF;
- driver corporativo;
- cancelamento/indisponibilidade;
- lifecycle/cleanup do PDF transitório.

### Word

- versões realmente instaladas;
- Arial/Consolas e substituição;
- abrir/editar/salvar DOCX gerado.

### SMB

- permissões reais;
- helper/rename/replace;
- interrupção de rede;
- quota/espaço;
- latência;
- estado pós-falha.

### Segurança corporativa

- antivírus/EDR;
- política de execução de binário vindo do share;
- long paths/políticas relevantes.

Fora da empresa, registrar `NÃO APLICÁVEL NESTE AMBIENTE` no ensaio local e manter gate corporativo aberto.

## 13. Encerramento

A Etapa 11 e o Bloco 10 estão documental e operacionalmente encerrados no repositório.

A evidência histórica do PR/branch permanece no Git; não é requisito técnico ativo.

## 14. Fontes primárias verificadas em 2026-08-29

- Typst PDF: `https://typst.app/docs/reference/pdf/`
- Typst crates/docs: `https://docs.rs/typst/latest/typst/`
- `typst-svg`: `https://docs.rs/typst-svg/latest/typst_svg/`
- `docx-rs`: `https://docs.rs/docx-rs/latest/docx_rs/`
- Tauri WebView/`with_webview`: `https://docs.rs/tauri/latest/tauri/webview/struct.WebviewWindow.html`
- Tauri Windows/WebView2 distribution: `https://v2.tauri.app/distribute/windows-installer/`
- Microsoft WebView2 distribution/Fixed Version: documentação oficial Microsoft Learn
- Microsoft WebView2 `ShowPrintUI`: documentação oficial Microsoft Learn
- Microsoft Windows naming/long paths/filesystem: documentação oficial Microsoft Learn
