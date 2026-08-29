# Bloco 10 — Etapa 11 — Validação técnica final — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Base:** `main` em `d74459b2b342a9fda2ccc4e0645c02613edc4fc8`

## Objetivo

Fechar o Bloco 10 por validação técnica das decisões já aprovadas, sem reabrir requisitos de produto nem antecipar implementação da Fase 2.

A Etapa 11 deve transformar os contratos documentais das Etapas 1–10 em uma matriz de viabilidade, limites e critérios de aceitação concretos para Windows 10/11, Tauri 2, WebView2, Typst, DOCX/OOXML e filesystem local/SMB.

## Princípio

A validação final não deve inventar funcionalidades novas. Seu papel é responder, para cada contrato já aprovado:

```text
é tecnicamente viável?
→ quais APIs/mecanismos concretos serão usados?
→ quais limites precisam ser definidos?
→ quais falhas precisam ser tratadas?
→ qual evidência será exigida antes da implementação?
```

Quando uma decisão já aprovada for tecnicamente viável, ela permanece. Somente um bloqueio técnico concreto e demonstrável pode exigir retorno ao PO.

## Escopo proposto

### 1. Pipeline documental Host-side

Validar:

- integração Typst embutida no Host Rust;
- geração de PDF sem processo externo;
- geração DOCX OOXML direta em Rust;
- isolamento entre snapshot do domínio e renderer;
- limites de concorrência/backpressure para geração;
- tamanho máximo operacional aceitável de artefatos retornados pela API;
- comportamento de erro sem artefato parcial tratado como sucesso.

### 2. PDF de Procedimentos

Validar:

- crates/API Typst concretas e compatíveis com a toolchain escolhida;
- PDF 1.7 e suporte efetivo ao baseline estrutural pretendido;
- embedding/subsetting de Noto Sans/Noto Sans Mono;
- PNG/JPEG/SVG controlados;
- texto selecionável/pesquisável;
- paginação multipágina;
- ausência de dependência de Internet ou recursos remotos;
- comportamento com conteúdo extenso e caracteres Unicode.

### 3. DOCX de Procedimentos

Validar:

- biblioteca Rust concreta sob adaptador interno;
- geração de pacote OOXML Transitional válido;
- texto, listas, numeração, comandos/código e imagens;
- abertura em versões corporativas suportadas do Microsoft Word;
- comportamento quando Arial/Consolas não estiverem disponíveis;
- limites de tamanho/complexidade e erros de empacotamento.

### 4. Impressão Windows / WebView2

Validar o fluxo já aprovado:

```text
PDF oficial
→ Client Windows
→ recurso local transitório quando necessário
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo do Windows
```

Confirmar:

- API WebView2 concreta disponível no stack Tauri/Windows adotado;
- carregamento confiável do PDF local;
- momento seguro para remover o arquivo transitório;
- comportamento de cancelamento pelo usuário;
- Host/Client offline da Internet;
- ausência de dependência de Adobe Reader/browser externo;
- impressoras físicas e PDF printers do Windows tratadas pelo diálogo do sistema.

### 5. Ficha A4 e `SHEET_OVERFLOW`

Validar com casos representativos e extremos:

- exatamente uma página A4;
- margens 15 mm e tipografia aprovada;
- seções opcionais colapsando corretamente;
- sem Equipamento;
- 0, 1, 2 e 3+ MACs;
- muitas observações por Etapa;
- Unicode, strings estruturadas longas e quebras de linha;
- detecção real de 2+ páginas pelo renderer;
- nenhum truncamento, redução automática de fonte ou segunda página;
- diagnóstico semântico de contributors suficiente para orientar revisão.

Os soft limits 600 / 400 / 300 / 280 devem ser avaliados por evidência. Ajuste só deve ocorrer se os casos reais mostrarem necessidade; eles continuam orientação, não hard limit de domínio.

### 6. Naming e filesystem Windows

Validar:

- sanitização de filenames Windows;
- nomes reservados;
- Unicode/acentos;
- paths longos no Windows 10/11 alvo;
- conflito de nome conduzido pelo diálogo nativo;
- escrita completa antes de sucesso;
- helper opaco no mesmo diretório de destino quando aplicável;
- promoção/replace em NTFS;
- comportamento em destino SMB;
- falhas de permissão, rede interrompida e espaço insuficiente;
- garantia de não apagar arquivo preexistente do usuário em falha.

Não prometer atomicidade idêntica entre NTFS local e qualquer servidor SMB sem evidência.

### 7. Temporários e lifecycle local

Validar:

- API concreta do Tauri/OS para diretório temporário por usuário;
- namespace `StepFlow/artifacts/<client-instance-id>`;
- isolamento entre múltiplos Clients;
- filenames opacos;
- cleanup após preview/impressão;
- locks mantidos por WebView2, Windows ou EDR;
- retry seguro;
- crash/orphan scavenging;
- proteção contra symlink/reparse point;
- ausência de daemon, serviço ou tarefa agendada para limpeza.

### 8. Memória, tamanho e concorrência

Definir por medição/PoC descartável quando necessário:

- tamanho máximo razoável de PDF/DOCX em memória;
- número de renderizações simultâneas;
- fila/backpressure de geração;
- timeout operacional;
- comportamento do Client com artefatos grandes;
- limites técnicos de payload/API independentes da A4;
- resposta `SERVER_BUSY` ou equivalente quando capacidade estiver saturada.

Não congelar números arbitrários sem evidência.

### 9. Compatibilidade real

Matriz mínima proposta:

- Windows 10 x64 suportado pelo projeto;
- Windows 11 x64;
- WebView2 Runtime corporativo;
- Microsoft Word nas versões efetivamente existentes no ambiente quando disponível para teste;
- impressora física representativa;
- Microsoft Print to PDF;
- NTFS local;
- compartilhamento SMB representativo;
- cenário sem Internet;
- antivírus/EDR corporativo quando o ambiente real estiver disponível.

Itens impossíveis de testar fora da LAN corporativa devem ser marcados `NÃO APLICÁVEL NESTE AMBIENTE` e permanecer como gate de validação no ambiente real, sem virar requisito inventado.

## Evidência esperada

A Etapa 11 pode usar documentação oficial atual e PoCs estritamente descartáveis para confirmar APIs e comportamento. Nenhuma PoC vira scaffold ou código de produção por inferência.

Para cada item, registrar um dos estados:

- **VALIDADO** — evidência suficiente;
- **VALIDADO COM LIMITE** — viável com limite/condição explícita;
- **PENDENTE DE AMBIENTE REAL** — depende de LAN, impressora, Word, SMB ou EDR corporativo;
- **BLOQUEADOR** — decisão aprovada não é tecnicamente sustentável e precisa retornar ao PO.

## Critério de encerramento do Bloco 10

O Bloco 10 pode ser considerado tecnicamente fechado quando:

1. cada contrato das Etapas 1–10 estiver coberto pela matriz final;
2. APIs/mecanismos concretos necessários à implementação estiverem identificados;
3. limites técnicos indispensáveis estiverem definidos por evidência ou explicitamente adiados para ambiente real;
4. falhas relevantes e comportamento esperado estiverem documentados;
5. não houver bloqueador técnico silenciosamente ignorado;
6. nenhuma decisão de produto nova tiver sido introduzida por inferência;
7. pendências de ambiente real estiverem claramente separadas das decisões de arquitetura.

## Não objetivos

Esta Etapa não autoriza:

- scaffold/runtime oficial;
- migrations;
- implementação funcional do renderer;
- implementação do fluxo de impressão;
- criação de serviço/daemon/watchdog;
- alteração de UX aprovada sem bloqueador técnico;
- inclusão de formatos de exportação adicionais;
- reabertura arbitrária das Etapas 1–10;
- sincronização destrutiva do checkout local.

## Resultado esperado para análise do PO

Se aprovada esta direção, a próxima atividade dentro da própria Etapa 11 será executar a matriz de validação usando documentação oficial atual, evidências técnicas e, quando necessário, PoCs descartáveis claramente isoladas. O resultado consolidado fechará o Bloco 10 ou destacará somente bloqueadores/pontos que dependam do ambiente corporativo real.