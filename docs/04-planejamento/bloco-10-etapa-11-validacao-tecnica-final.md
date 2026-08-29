# Bloco 10 — Etapa 11 — Validação técnica final

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Data:** 2026-08-29  
**Base de abertura:** `main` em `d74459b2b342a9fda2ccc4e0645c02613edc4fc8`

## 1. Objetivo

Fechar tecnicamente o Bloco 10 verificando se os contratos aprovados nas Etapas 1–10 possuem mecanismos implementáveis, limites explícitos e critérios de validação suficientes, sem antecipar código de produção.

A Etapa 11 não reabre produto por inferência. Um requisito só retorna ao PO quando houver bloqueio técnico concreto.

## 2. Resultado executivo

**Nenhum bloqueador arquitetural foi identificado para os contratos documentais do Bloco 10.**

Há três classes de limite que permanecem conscientes:

1. **limite de implementação** — precisa de adaptador, pinagem de versão, fila bounded, cleanup conservador etc.;
2. **limite de medição** — memória, tamanho, concorrência, fila e timeout serão definidos por benchmark/fixtures na fase executável;
3. **limite de ambiente real** — Word, impressoras, SMB, WebView2 corporativo, políticas Windows e EDR só podem ser fechados nas estações/LAN reais.

Nenhum número de performance foi inventado nesta fase.

## 3. Gate Pocket obrigatório

O critério Pocket é superior a qualquer conveniência de biblioteca.

Para o StepFlow, a experiência suportada é:

```text
pasta pronta publicada no servidor Windows
→ estação acessa o compartilhamento
→ usuário executa StepFlowLauncher.exe
→ Launcher prepara/valida recursos locais automaticamente
→ Client abre de %LOCALAPPDATA%
→ Launcher encerra
```

Uma solução **não atende ao StepFlow Pocket** se exigir, por estação:

- instalador tradicional obrigatório;
- configuração manual de dependência;
- privilégio administrativo no fluxo normal;
- toolchain de desenvolvimento;
- Internet obrigatória para uso normal;
- execução permanente do Client diretamente pelo SMB.

A preparação automática em `%LOCALAPPDATA%` é parte do comportamento Pocket; não é tratada como instalação tradicional.

## 4. Matriz consolidada

| ID | Contrato | Estado | Consequência técnica |
|---|---|---|---|
| D01 | Typst embutido no Host | **VALIDADO** | usar crates oficiais sob adaptador interno; sem `typst.exe` |
| D02 | `World` Typst restrito | **VALIDADO** | somente template/assets/fontes autorizados; sem resolução remota arbitrária |
| D03 | PDF 1.7 + Tagged PDF baseline | **VALIDADO** | manter baseline testado; sem promessa formal PDF/A/PDF/UA |
| D04 | PDF/preview via mesmo `PagedDocument` | **VALIDADO** | page count é autoridade física da Ficha |
| D05 | `SHEET_OVERFLOW` | **VALIDADO** | 2+ páginas falham; sem truncamento/segunda folha/redução automática |
| D06 | Soft limits 600/400/300/280 | **VALIDADO COM LIMITE** | orientação editorial; ajuste somente por fixtures/evidência |
| D07 | DOCX OOXML direto em Rust | **VALIDADO COM LIMITE** | `docx-rs` preferido sob adaptador; Word real ainda deve ser testado |
| D08 | Arial/Consolas sem embedding | **VALIDADO COM LIMITE** | substituição pode alterar reflow; DOCX não promete paginação do PDF |
| D09 | Transporte binário Host → Client | **VALIDADO** | artefatos como body binário; não base64 em JSON sem necessidade |
| D10 | Renderização fora da fila SQLite | **VALIDADO** | CPU/blocking separado da fila de mutações |
| D11 | Concorrência documental bounded | **VALIDADO COM LIMITE** | admissão + semáforo/executor; números por benchmark |
| D12 | `ShowPrintUI(System)` | **VALIDADO** | diálogo oficial do Windows; sem seletor próprio/silent print baseline |
| D13 | Tauri → WebView2 nativo | **VALIDADO COM LIMITE** | `with_webview` + adaptador Windows; pinagem/teste da família Tauri/Wry/WebView2 |
| D14 | Lifetime do PDF durante impressão | **PENDENTE DE AMBIENTE REAL** | validar load/print/cancel/close/cleanup em Windows real |
| D15 | Naming Windows | **VALIDADO** | sanitização e extensão controlada permanecem |
| D16 | Long paths | **VALIDADO COM LIMITE** | depende também de manifesto/política Windows; nomes sugeridos continuam manejáveis |
| D17 | Save local NTFS | **VALIDADO COM LIMITE** | helper no mesmo destino + write/flush/close + promoção; sem promessa ACID absoluta |
| D18 | Save em SMB | **PENDENTE DE AMBIENTE REAL** | testar permissões, rename/replace, queda de rede, quota e pós-falha |
| D19 | Temp dir por usuário | **VALIDADO** | API OS/Tauri; sem path hardcoded |
| D20 | Isolamento por Client | **VALIDADO COM LIMITE** | subdiretório/ID próprios + `active.lock`; cleanup conservador |
| D21 | Reparse-safe scavenging | **VALIDADO** | nunca seguir reparse para fora da raiz gerenciada; incerteza = skip |
| D22 | Locks WebView2/AV | **VALIDADO COM LIMITE** | cleanup best-effort; sem kill/unlock forçado/alteração de ACL |
| D23 | Word corporativo | **PENDENTE DE AMBIENTE REAL** | testar versões realmente existentes |
| D24 | Impressoras/driver corporativo | **PENDENTE DE AMBIENTE REAL** | impressora física + Microsoft Print to PDF + cancelamento/falhas |
| D25 | EDR/antivírus corporativo | **PENDENTE DE AMBIENTE REAL** | validar execução, handles e cleanup |
| D26 | WebView2 Evergreen existente | **VALIDADO COM LIMITE** | preferível quando presente/compatível; presença real deve ser detectada |
| D27 | WebView2 Fixed Version por UNC/SMB | **NÃO UTILIZAR** | Microsoft documenta que Fixed Version não roda de localização de rede/UNC |
| D28 | Fallback WebView2 autocontido local | **VALIDADO COM LIMITE / PoC OBRIGATÓRIA** | só adotar se funcionar em `%LOCALAPPDATA%` sem instalação, elevação ou ação manual |
| D29 | Windows 10 + Fixed Runtime moderno | **VALIDADO COM LIMITE / PoC OBRIGATÓRIA** | requisitos ACL/AppContainer precisam ser automatizáveis no perfil do usuário |
| D30 | Contrato Pocket zero instalação/manualidade | **VALIDADO COMO GATE** | qualquer solução que exija setup/admin manual volta à arquitetura |

## 5. Pipeline documental

Baseline aprovado:

```text
Client solicita fonte + revisão esperada
→ Host autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel
→ encerra leitura/transação SQLite
→ admite renderização em capacidade bounded
→ renderer PDF/DOCX trabalha fora da fila de mutações
→ Host devolve bytes
```

Para Typst:

```text
DocumentModel
→ adaptador interno
→ World restrito
→ compile → PagedDocument
→ PDF / SVG
```

Não há consulta SQLite durante renderização nem recurso remoto arbitrário.

### Toolchain

Versões de crates serão pinadas em conjunto na fundação executável. O MSRV efetivo será o maior exigido pelas dependências escolhidas.

Na referência verificada em 2026-08-29, a família atual de `typst-pdf` impõe MSRV superior a toolchains antigas; isso afeta apenas o **ambiente de build**, nunca a máquina central de produção ou as estações.

## 6. Ficha A4

A autoridade final continua sendo o layout real do Typst:

```text
1 página
→ PDF + preview válidos

2+ páginas
→ SHEET_OVERFLOW
```

Fixtures futuras devem cobrir pelo menos:

- sem Equipamento;
- campos opcionais vazios;
- 0/1/2/3+ MACs;
- Unicode/acentos;
- strings estruturadas longas;
- muitas observações por Etapa;
- caso limite de uma página;
- caso determinístico de duas páginas;
- ausência de truncamento/redução automática.

Os valores 600 / 400 / 300 / 280 continuam soft limits, não hard limits de domínio.

## 7. DOCX

OOXML/WordprocessingML direto em Rust permanece viável sem Word/COM, LibreOffice ou conversão de PDF.

O adaptador deve isolar `docx-rs` do domínio. A validação corporativa futura deve confirmar:

- abertura nas versões reais do Word;
- texto/listas/numeração editáveis;
- imagens;
- whitespace de comandos/código;
- reflow com Arial/Consolas disponíveis ou substituídas;
- salvar novamente sem corrupção relevante.

## 8. Impressão Windows

Fluxo validado arquiteturalmente:

```text
PDF oficial
→ Client Windows
→ recurso local transitório quando necessário
→ WebView2 dedicada/transitória
→ acesso nativo via adapter
→ ShowPrintUI(System)
```

O método genérico de impressão do Tauri não deve substituir por inferência esse contrato. O adapter Windows deve ficar isolado e a família Tauri/Wry/WebView2 usada por ele deve ser pinada/testada.

O lifetime exato do PDF durante diálogo/spooler continua ensaio Windows real.

## 9. WebView2 e Pocket

### 9.1 Evergreen

Quando um Evergreen Runtime compatível já existe na estação, ele é a opção preferível porque recebe servicing de segurança do ecossistema Microsoft e reduz o pacote do StepFlow.

O Launcher/Client deve detectar disponibilidade real; não assumir cegamente presença em toda edição corporativa de Windows.

### 9.2 Fixed Version

Fixed Version pode ser distribuído junto da aplicação, porém:

- não pode rodar diretamente de localização de rede/UNC;
- precisa ser mantido/atualizado pelo distribuidor da aplicação;
- em Windows 10, versões modernas para app Win32 unpackaged possuem requisitos adicionais de ACL/AppContainer.

Portanto, um eventual fallback é necessariamente local:

```text
share
→ Launcher
→ copiar Client + runtime aprovado para área local versionada
→ aplicar automaticamente preparação permitida
→ iniciar Client local
```

### 9.3 PoC obrigatória do fallback

Antes de afirmar suporte Pocket para uma estação sem Evergreen utilizável, a Fase 2 deve provar em PoC descartável:

- Windows 10 x64 alvo;
- Windows 11 x64;
- usuário padrão, sem elevação;
- share acessível;
- Client + Fixed Runtime copiados para `%LOCALAPPDATA%`;
- configuração do runtime local sem instalador;
- requisitos ACL realizados automaticamente, se necessários;
- funcionamento sem Internet;
- atualização de versão sem modificar registro/PATH global;
- remoção/rollback da versão local sem afetar outros aplicativos.

Se uma estação que deva ser suportada exigir instalação, admin ou preparação manual, o fallback é **BLOQUEADOR** até redesign. Não se reduz o requisito Pocket para acomodar a dependência.

## 10. Filesystem e temporários

Save persistente:

```text
bytes completos
→ helper opaco no diretório escolhido
→ write
→ flush/close
→ promoção/replace após fluxo de confirmação
→ sucesso
```

Temporários:

```text
<temp do usuário>/StepFlow/artifacts/<client-instance-id>/
```

Regras:

- nomes opacos;
- uma instância não limpa outra potencialmente ativa;
- `active.lock`/mecanismo equivalente mantém sinal explícito de atividade;
- reparse point nunca é seguido para fora da raiz gerenciada;
- lock de WebView2/EDR gera cleanup best-effort;
- falha de cleanup não retroage sucesso funcional já obtido;
- sem daemon, serviço ou Task Scheduler para limpeza.

## 11. Performance e backpressure

Nenhum valor é congelado sem medição.

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

Não bloqueiam o fechamento arquitetural do Bloco 10, mas devem ser executados antes do deployment correspondente:

### Windows/WebView2

- edições/versões reais de Windows 10/11;
- Evergreen disponível/compatível;
- PoC de fallback local sem instalação/elevação;
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

Quando não for possível testar fora da empresa, registrar `NÃO APLICÁVEL NESTE AMBIENTE` no ensaio local e manter o gate corporativo aberto.

## 13. Critério de encerramento do Bloco 10

Com esta matriz aprovada, o Bloco 10 está documentalmente consolidado porque:

1. todas as Etapas 1–10 possuem mecanismo técnico identificável;
2. não há bloqueador arquitetural conhecido;
3. limites estão explícitos;
4. medições não foram substituídas por números arbitrários;
5. dependências do ambiente real estão separadas;
6. o contrato Pocket foi preservado como gate superior;
7. nenhum código de produção, migration ou scaffold foi criado nesta etapa.

O encerramento **operacional** ainda exige o gate Git normal do PR da Etapa 11: squash merge, remoção da branch e remoto somente com `main` e zero PRs abertos.

## 14. Fontes primárias verificadas em 2026-08-29

- Typst PDF: https://typst.app/docs/reference/pdf/
- Typst compile/World/PagedDocument: https://docs.rs/typst/latest/typst/ ; https://docs.rs/typst-layout/latest/typst_layout/
- `typst-svg`: https://docs.rs/typst-svg/latest/typst_svg/
- `docx-rs`: https://docs.rs/docx-rs/latest/docx_rs/
- Tauri WebView/`with_webview`: https://docs.rs/tauri/latest/tauri/webview/struct.WebviewWindow.html
- Tauri WebView2 distribution: https://v2.tauri.app/distribute/windows-installer/
- Microsoft WebView2 distribution/Fixed Version: https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/distribution
- Microsoft WebView2 Evergreen vs Fixed: https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/evergreen-vs-fixed-version
- Microsoft WebView2 printing: https://learn.microsoft.com/en-us/microsoft-edge/webview2/how-to/print
- Windows filename/path rules: https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file
- Windows long paths: https://learn.microsoft.com/en-us/windows/win32/fileio/maximum-file-path-limitation
- Windows filesystem primitives: `ReplaceFileW`, `FlushFileBuffers`
- Windows reparse points: https://learn.microsoft.com/en-us/windows/win32/fileio/reparse-point-operations
- Tokio `spawn_blocking`: https://docs.rs/tokio/latest/tokio/task/fn.spawn_blocking.html
- Axum responses: https://docs.rs/axum/latest/axum/response/trait.IntoResponse.html

## 15. Não objetivos

A Etapa 11 não autoriza:

- scaffold/runtime oficial;
- migrations;
- implementação do renderer;
- implementação do fluxo de impressão;
- instalador obrigatório por estação;
- serviço/daemon/watchdog;
- novo formato de exportação;
- alteração de UX sem bloqueador técnico;
- sincronização destrutiva do checkout local;
- valores arbitrários de performance tratados como contrato.
