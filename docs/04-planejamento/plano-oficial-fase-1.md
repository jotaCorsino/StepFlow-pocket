# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-29

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e PoCs descartáveis quando necessárias. Não autoriza scaffold/runtime oficial nem código de negócio definitivo antes do gate correspondente do Bloco 12/Fase 2.

## Estado dos blocos

| Bloco | Tema | Status | Fonte vigente |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório validado |
| 1 | Client Windows/Tauri | CONCLUÍDO | `03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | NÚCLEO CONCLUÍDO / PARÂMETROS FINAIS PENDENTES | `03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS CONCEITUALMENTE | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | NÚCLEO CONCLUÍDO | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` + Bloco 9 |
| 8 | UI/UX | CONCLUÍDO | `02-telas/README.md` |
| 9 | Execução operacional/Atendimentos | **CONCLUÍDO** | `04-planejamento/bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão + ficha compacta | **ETAPAS 1–11 CONSOLIDADAS / GATE REMOTO PENDENTE** | `04-planejamento/bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Extensão de produto consolidada

Fazem parte da Fase 1:

- categorias configuráveis/múltiplas;
- domínio `Procedimento × Atendimento/Execução × Equipamento`;
- Atendimentos como área operacional própria;
- Equipamento opcional/reutilizável;
- múltiplos Procedimentos por Atendimento;
- revisão exata utilizada preservada;
- checklist persistente em contexto de execução;
- `Observação do serviço` opcional por Etapa no Reader operacional;
- estado final historicamente reproduzível após conclusão/reabertura;
- Ficha compacta como prestação de contas resumida ao cliente, com ou sem Equipamento;
- identidade central da empresa;
- PDF/DOCX/impressão contextual de Procedimentos;
- PDF próprio + preview da Ficha;
- template físico A4 da Ficha;
- limites textuais orientativos e política de overflow;
- projeção compacta de MACs e regras de dados excepcionais;
- naming persistente e política de artefatos transitórios;
- estados transversais;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`;
- princípio visual de clareza com baixa densidade textual permanente;
- contrato Pocket sem instalação/preparação manual por estação.

## Contrato Pocket da Fase 1

A implantação do Client deve preservar:

```text
pasta pronta no servidor Windows
→ usuário acessa o compartilhamento
→ executa StepFlowLauncher.exe
→ Launcher prepara versão local validada
→ Client abre localmente
```

Não é aceitável como baseline:

- instalar StepFlow em cada estação;
- exigir MSI/MSIX/NSIS;
- exigir Rust/Node/npm/Cargo/Office/LibreOffice/Adobe;
- exigir Internet para uso normal;
- exigir elevação administrativa no fluxo normal;
- exigir configuração manual de dependência por computador;
- executar permanentemente o Client pelo SMB.

WebView2 Evergreen existente é preferível quando compatível. Fixed Version não roda por UNC/SMB; fallback autocontido só pode ser adotado se PoC provar preparação local automática sem instalação/elevação/manualidade, inclusive no Windows 10 alvo.

## Bloco 8 — UI/UX

Telas 01–15 estão consolidadas/aprovadas. Nenhuma UI de produção foi criada.

Direção transversal:

- Reader em formato livro/manual;
- `Visão geral` como primeira página lógica;
- uma Etapa por página lógica;
- stepper horizontal compacto de círculos/linhas, navegável diretamente;
- stepper representa navegação, não conclusão operacional;
- informação secundária sob demanda e baixa densidade textual quando possível.

A Tela 05 incorpora `Observação do serviço` por Etapa somente no contexto operacional. A Tela 14 consolida a Ficha como prestação de contas resumida, PDF/preview, template físico A4, política de limites/overflow, regras de dados excepcionais, naming e temporários do Client.

## Bloco 9 — Execução operacional / Atendimentos

Consolidado:

- lifecycle `Em andamento / Concluído / Cancelado`, com reabertura explícita;
- primeiro save cria Atendimento; responsável + resumo são necessários para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- Funcionário opera por responsabilidade;
- revisão exata do Procedimento é preservada;
- checklist persistente somente em Atendimento;
- observação de serviço opcional/persistente por Etapa;
- progresso deriva somente do checklist;
- Equipamento opcional/reutilizável;
- conclusão preserva estado histórico relevante;
- códigos `AT-000001` e `EQP-000001` Host-only;
- Ficha disponível conforme lifecycle/capacidade.

## Bloco 10 — Exportação e impressão

**Status: ETAPAS 1–11 CONSOLIDADAS / APROVADAS PELO PO.**

Fontes:

- `bloco-10-exportacao-impressao-ficha.md` — mapa técnico;
- `bloco-10-etapa-11-validacao-tecnica-final.md` — matriz final;
- `../02-telas/14-exportacao-impressao-ficha.md` — UX;
- `../03-arquitetura/launcher-distribuicao-client.md` e `compatibilidade-windows-client.md` — contrato Pocket/Windows.

### Etapas

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

### Resultado da Etapa 11

A validação final não encontrou bloqueador arquitetural para as decisões documentais das Etapas 1–10.

Consolidado:

- Typst/PDF/PagedDocument viáveis;
- DOCX Rust direto viável, com Word real como gate corporativo;
- impressão Windows viável por WebView2 nativo + `ShowPrintUI(System)`;
- Tauri/Wry/WebView2 do adapter devem ser pinados/testados;
- save local/naming/temporários/scavenging viáveis com limites explícitos;
- SMB, Word, impressoras e EDR permanecem validação de ambiente real;
- memória/tamanho/concorrência/fila/timeout serão definidos por benchmark, sem números arbitrários;
- contrato Pocket é gate superior;
- WebView2 Fixed Version nunca roda do UNC/SMB e fallback local exige PoC sem instalação/elevação/manualidade.

## Gate atual

A consolidação do Bloco 10 só está operacionalmente encerrada após:

```text
PR #24
→ validação do diff
→ ready
→ squash merge em main
→ apagar branch remota
→ verificar remoto somente com main
→ verificar zero PRs abertos
```

Somente depois disso abrir o Bloco 11.

## Pendências restantes da Fase 1

### Segurança/configuração

- Argon2id exato;
- senha mínima final;
- duração de sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão.

### Ambiente real

- Windows/WebView2 nas estações reais;
- PoC do fallback Pocket WebView2;
- execução do Launcher pelo share corporativo;
- Word/impressoras;
- SMB real;
- EDR/firewall/políticas.

### Bloco 11

- mecanismo/pacote de Backup/Restore;
- atomicidade/checksums/retenção;
- restart/reconexão/sessões;
- disaster recovery local.

### Bloco 12

- estrutura oficial do repositório;
- migrations/scripts/testes;
- parâmetros finais;
- plano da Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

## Regras finais

- não criar scaffold, runtime definitivo ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex futura que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência pode virar decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO.
