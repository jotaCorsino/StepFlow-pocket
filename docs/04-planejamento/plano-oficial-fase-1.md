# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-29

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e PoCs descartáveis quando necessárias. Não autoriza scaffold/runtime oficial nem código de negócio definitivo antes do gate correspondente do Bloco 12/Fase 2.

## Estado dos blocos

| Bloco | Tema | Estado | Fonte principal |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório |
| 1 | Client Windows/Tauri | CONCLUÍDO | `../03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `../03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `../03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `../03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | NÚCLEO CONCLUÍDO / PARÂMETROS FINAIS PENDENTES | `../03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS | `../03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | NÚCLEO CONCLUÍDO | `../03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | CONCLUÍDO | `../02-telas/README.md` |
| 9 | Execução operacional/Atendimentos | CONCLUÍDO | `bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão + Ficha compacta | CONCLUÍDO | `bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração técnico | EM ANÁLISE | `bloco-11-backup-restauracao.md` |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | a abrir |

## Extensão de produto consolidada

Fazem parte do contrato atual:

- categorias configuráveis/múltiplas;
- domínio `Procedimento × Atendimento/Execução × Equipamento`;
- Atendimentos como área operacional própria;
- Equipamento opcional/reutilizável;
- múltiplos Procedimentos por Atendimento;
- revisão exata utilizada preservada;
- checklist persistente em contexto de execução;
- `Observação do serviço` opcional por Etapa;
- reprodução histórica após conclusão/reabertura;
- Ficha compacta com ou sem Equipamento;
- identidade central da empresa;
- PDF/DOCX/impressão de Procedimentos;
- PDF + preview + impressão da Ficha;
- template físico e política de overflow;
- naming persistente e temporários;
- estados transversais;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`;
- clareza e baixa densidade textual como princípio visual;
- contrato Pocket sem instalação/preparação manual por estação.

## Contrato Pocket da Fase 1

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
- exigir elevação administrativa;
- exigir configuração manual de dependência;
- executar permanentemente o Client pelo SMB.

WebView2 Evergreen existente é preferível quando compatível. Fixed Version não roda por UNC/SMB; fallback local só pode ser adotado após PoC provar preparação automática sem instalação/elevação/manualidade.

## Bloco 8 — UI/UX

**CONCLUÍDO.**

Telas 01–15 estão consolidadas/aprovadas. Nenhuma UI de produção foi criada.

Direção transversal:

- Reader em formato livro/manual;
- `Visão geral` como primeira página lógica;
- uma Etapa por página lógica;
- stepper horizontal compacto e navegável;
- stepper representa navegação, não conclusão operacional;
- informação secundária sob demanda e baixa densidade textual.

## Bloco 9 — Execução operacional / Atendimentos

**CONCLUÍDO.**

Consolidado:

- lifecycle `Em andamento / Concluído / Cancelado`, com reabertura explícita;
- primeiro save cria Atendimento;
- responsável + resumo obrigatórios para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- Funcionário opera por responsabilidade;
- revisão exata preservada;
- checklist persistente somente em Atendimento;
- observação de serviço opcional/persistente por Etapa;
- progresso deriva somente do checklist;
- Equipamento opcional/reutilizável;
- reprodução histórica do estado relevante;
- códigos legíveis Host-only;
- Ficha disponível conforme lifecycle/capacidade.

## Bloco 10 — Exportação / impressão / Ficha compacta

**CONCLUÍDO.**

Fontes:

- `bloco-10-exportacao-impressao-ficha.md` — mapa técnico;
- `bloco-10-etapa-11-validacao-tecnica-final.md` — matriz final;
- `../02-telas/14-exportacao-impressao-ficha.md` — UX;
- documentos Pocket/Windows — distribuição e compatibilidade.

Resultado:

- geração Host-side por snapshot consistente + `DocumentModel`;
- PDF via Typst embutido;
- DOCX OOXML direto em Rust;
- impressão Windows pelo mesmo PDF oficial via WebView2;
- Procedimento físico A4 multipágina;
- Ficha PDF + preview do mesmo `PagedDocument`, exatamente uma A4;
- `SHEET_OVERFLOW` sem truncamento/segunda página/redução automática;
- soft limits 600/400/300/280 orientativos;
- naming e temporários consolidados;
- nenhum bloqueador arquitetural identificado na validação final;
- Word, impressoras, SMB, Windows/WebView2 e EDR mantidos como gates de ambiente real;
- limites de performance definidos por benchmark na fase executável;
- contrato Pocket preservado como gate superior.

## Bloco 11 — Backup / Restore técnico

**EM ANÁLISE.**

Fonte de trabalho: `bloco-11-backup-restauracao.md`.

A UX já está consolidada em `../02-telas/13-backup-restauracao.md` e não será reaberta sem bloqueador técnico concreto.

O Bloco 11 deve fechar tecnicamente:

- conjunto exato do estado recuperável;
- snapshot consistente de SQLite + arquivos administrados;
- formato/identidade do backup;
- manifesto, verificação e compatibilidade;
- escrita/promoção e tratamento de backup parcial;
- retenção;
- coordenação com operações e mutações;
- safety backup e Restore normal;
- restart/reconexão/sessões;
- falhas parciais/resultado incerto;
- disaster recovery local quando Host não inicia;
- capacidades/auditoria;
- validação técnica final.

Alternativas técnicas listadas durante a análise não são contrato até aprovação explícita e sincronização documental.

## Bloco 12 — Estrutura oficial + Fase 2

**PENDENTE.**

Fechará:

- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- parâmetros finais ainda abertos;
- plano da Fase 2;
- sincronização segura do checkout local antes do primeiro Codex de implementação.

## Pendências restantes da Fase 1

### Segurança/configuração

- Argon2id exato;
- senha mínima final;
- duração de sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada.

### Ambiente real

- Windows/WebView2 nas estações reais;
- PoC do fallback Pocket WebView2;
- execução do Launcher pelo share corporativo;
- Word/impressoras;
- SMB real;
- EDR/firewall/políticas.

## Regras finais

- não criar scaffold/runtime definitivo ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência técnica sem retorno explícito ao PO;
- gates Git já consumidos não permanecem como “estado atual” em documentos técnicos estáveis.
