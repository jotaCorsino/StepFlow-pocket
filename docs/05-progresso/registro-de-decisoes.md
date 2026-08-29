# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-29

Este arquivo registra decisões vigentes, pendências e gates. Detalhes ficam nos documentos específicos. Proposta não aprovada não é contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade durante o fechamento documental restante da Fase 1;
- checkout local previsto: `C:\dev\StepFlow`;
- sincronização local fica adiada até antes do primeiro trabalho de implementação e deve preservar alterações preexistentes do PO;
- uma tarefa lógica por vez, uma branch ativa e um PR;
- fluxo: proposta → discussão → aprovação do PO → consolidação canônica → validação → ready → squash merge → apagar branch → verificar somente `main` e zero PRs abertos;
- branch mergeada ainda não está encerrada enquanto existir no remoto;
- Fase 1 não autoriza scaffold/runtime/código de negócio oficial antes do gate correspondente;
- `AGENTS.md` é a regra operacional superior.

## 2. Contrato Pocket

O StepFlow deve ser utilizável a partir de uma pasta pronta publicada num servidor Windows.

Fluxo aprovado:

```text
pasta publicada no servidor
→ estação acessa o compartilhamento
→ usuário executa StepFlowLauncher.exe
→ Launcher prepara/valida versão local em %LOCALAPPDATA%
→ Client abre localmente
→ Launcher encerra
```

Requisitos:

- sem instalador tradicional obrigatório por estação;
- sem configuração manual de dependências;
- sem elevação administrativa no uso normal;
- sem Rust/Node/npm/Cargo/Visual Studio na estação ou máquina central de produção;
- sem Office/LibreOffice/Adobe Reader como dependência operacional;
- sem Internet obrigatória durante uso normal;
- Client não roda permanentemente do SMB;
- Controller/Host é iniciado na máquina central quando o ciclo StepFlow será usado;
- fechar um Client não encerra o Host;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon como baseline.

Se uma dependência exigir setup/admin manual por computador, a solução não atende ao contrato Pocket e deve ser redesenhada ou tratada como bloqueador.

### WebView2

- Tauri 2 usa WebView2 no Windows;
- Evergreen compatível já presente é preferível;
- disponibilidade real precisa ser detectada;
- não baixar/instalar runtime silenciosamente da Internet em produção;
- Microsoft documenta que Fixed Version não roda de localização de rede/UNC;
- fallback Fixed/autocontido, se necessário, deve ser local e só pode virar produção após PoC provar preparação em `%LOCALAPPDATA%` sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo;
- se a PoC falhar numa estação que deva ser suportada, o fallback retorna à arquitetura como bloqueador; o requisito Pocket não é reduzido por conveniência.

Fontes: `docs/03-arquitetura/launcher-distribuicao-client.md`, `compatibilidade-windows-client.md`.

## 3. Produto e domínio

StepFlow é aplicação interna para documentar, consultar, versionar e executar Procedimentos técnicos de forma guiada e registrar Atendimentos reais.

Não transformar por inferência em CRM, financeiro/faturamento, estoque, RMM ou help desk/SLA completo.

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação/modelo oficial;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode existir sem Equipamento e usar zero, um ou vários Procedimentos;
- vínculo preserva a revisão exata utilizada;
- alteração futura do Procedimento/Equipamento não reescreve o histórico concluído.

## 4. Procedimentos, categorias e revisões

Campos principais: Código, Título, Área/Departamento, Responsável, Status, Versão, Objetivo, Observações, Pré-requisitos, Categorias, Etapas e Histórico.

Categorias:

- configuráveis, múltiplas e sem árvore inicial;
- pesquisáveis/filtráveis;
- arquivamento preserva histórico;
- gestão por ADM/Gerência;
- evitar nomes normalizados equivalentes.

Editor/revisões:

- Editor = `Informações` + `Etapas`;
- painel local `Estrutura`, sem segunda sidebar global;
- blocos tipados;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- `base_revision`/controle otimista e `409` para base obsoleta;
- sem merge automático;
- publicar é separado de salvar;
- `revision_no` técnico separado de `display_version` editorial.

Pendente: regra editorial de nova revisão ainda referenciando categoria arquivada.

## 5. Reader e direção visual

- experiência de livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa por página lógica;
- `Sumário` temporário, Anterior/Próxima e `Etapa X de Y` compacto;
- stepper horizontal de círculos/linhas, navegável por clique/teclado;
- stepper representa navegação, nunca conclusão operacional;
- nomes das Etapas permanecem no título/Sumário;
- comandos/código preservam whitespace, nunca executam e usam copiar icon-only acessível;
- cor nunca é o único canal semântico;
- mostrar permanentemente apenas o necessário para entender e agir;
- evitar cards/badges/labels sem ganho de leitura.

Reader standalone não persiste checklist/execução. Reader em Atendimento persiste checklist e `Observação do serviço` por Etapa conforme lifecycle/autorização.

## 6. Atendimentos, checklist e observações

Lifecycle:

```text
rascunho Client
→ primeiro save aceito
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- primeiro save cria ID, `AT-000001` e início aplicável;
- abrir a tela não cria registro oficial;
- conclusão exige responsável + `Resumo do trabalho` + estado confirmado;
- checklist incompleto avisa, mas não bloqueia automaticamente;
- cancelamento exige motivo curto;
- Concluído/Cancelado são read-only até reabertura;
- nova conclusão após reabertura não apaga o estado histórico anterior;
- checklist persiste somente em Atendimento;
- progresso deriva somente de itens marcados/total;
- 100% não conclui automaticamente;
- checklist usa concorrência granular por item/equivalente.

`Observação do serviço` por Etapa:

- opcional e somente em Atendimento;
- ligada ao vínculo da revisão + Etapa;
- não altera Procedimento oficial;
- concorrência granular por Etapa/equivalente;
- evento remoto não sobrescreve texto local em edição;
- somente leitura em Concluído/Cancelado;
- participa da reprodução histórica da Ficha;
- sem autosave por inferência.

## 7. Equipamento e permissões operacionais

Equipamento:

- código `EQP-000001`;
- identidade interna própria;
- serial/MAC/patrimônio são atributos de busca, não identidade canônica;
- múltiplos MACs;
- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- não arquivar se vinculado a Atendimento `Em andamento`;
- conclusão preserva projeção histórica relevante.

Presets:

- ADM/Gerência operam amplamente Atendimentos acessíveis;
- Funcionário opera por padrão Atendimento do qual é responsável;
- Funcionário não cancela/reabre por preset;
- Funcionário usa revisão publicada;
- ADM/Gerência podem selecionar explicitamente revisão histórica/não publicada já autorizada;
- Ficha pode ser gerada/reimpressa pelos três presets em Atendimento acessível;
- autorização real continua granular e Host-side.

Pendentes: Gerência × configuração da empresa e Gerência × Backup.

## 8. Arquitetura técnica consolidada

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- WAL + writer lógico coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- eventos pós-commit;
- sessão opaca server-side, token em memória, Argon2id;
- implantação central por pasta pronta;
- dados/config/logs/backups separados de binários substituíveis;
- nenhuma toolchain na produção.

Parâmetros finais de Argon2id, senha, sessão e token permanecem pendentes.

## 9. Bloco 10 — geração documental

**Etapas 1–11 consolidadas / aprovadas pelo PO.**

Fonte do mapa: `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md`.

### Arquitetura e Procedimentos

- geração documental pertence ao Host;
- Client solicita fonte + revisão esperada, sem documento montado;
- Host captura snapshot consistente e `DocumentModel` antes da renderização;
- renderers não usam DOM/HTML nem reconsultam SQLite;
- renderização ocorre fora da fila de mutações e usa capacidade bounded;
- PDF usa Typst embutido e `World` restrito;
- baseline PDF 1.7 + Tagged PDF, sem promessa formal PDF/A/PDF/UA;
- DOCX é OOXML Transitional direto em Rust sob adaptador;
- impressão usa o mesmo PDF oficial no Client Windows via WebView2 + `ShowPrintUI(System)`;
- Procedimento físico = A4 retrato multipágina, margens-base 18 mm;
- nenhuma truncagem/redução silenciosa.

### Ficha compacta

- prestação de contas resumida ao cliente;
- PDF e preview SVG derivam do mesmo `PagedDocument`;
- exatamente uma página A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- Salvar/Imprimir reutilizam os mesmos bytes PDF da prévia;
- soft limits: Resumo 600, Atendimento 400, Equipamento 300, observação por Etapa 280 caracteres;
- soft limits orientam, não bloqueiam nem truncam;
- correção de overflow acontece nos campos reais, sem editor paralelo/IA/resumo automático/compactação automática;
- Procedimentos vinculados não são listados por padrão;
- MACs: 0 omite; 1–2 valores; 3+ somente quantidade;
- observações legítimas não recebem cap/descarte automático.

### Naming e temporários

- Procedimento: `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`; sem `display_version`, omite esse segmento;
- Ficha: `{service_code} - Ficha.pdf`;
- sanitização segue filename Windows e nunca altera conteúdo;
- conflito não causa overwrite silencioso;
- save só é sucesso após gravação integral;
- temporário só existe quando integração local exige filesystem;
- raiz temporária por usuário, namespace StepFlow e subdiretório opaco por Client;
- cleanup/scavenging best-effort, sem daemon/serviço/tarefa agendada;
- arquivo salvo pelo usuário nunca entra no cleanup normal.

## 10. Etapa 11 — validação técnica final

Fonte: `docs/04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`.

Decisões:

- nenhum bloqueador arquitetural identificado para os contratos documentais das Etapas 1–10;
- Typst/PDF/PagedDocument: validados;
- DOCX Rust direto: validado com limite; teste de Word corporativo pendente;
- impressão Windows: WebView2 nativo + `ShowPrintUI(System)`; adapter e versões pinadas/testadas;
- save local/naming/temporários/scavenging: viáveis com limites explícitos;
- SMB, impressoras, Word e EDR: pendentes de ambiente real;
- memória/tamanho/concorrência/fila/timeout: definir por benchmark, nunca número arbitrário;
- Fixed WebView2 por UNC/SMB: não utilizar;
- fallback WebView2 local: PoC obrigatória sem instalação/elevação/manualidade;
- contrato Pocket: gate superior.

## 11. Estado da Fase 1 e gates

- Blocos 0–4: concluídos;
- Bloco 5: núcleo concluído / parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Blocos 8–9: concluídos;
- **Bloco 10: Etapas 1–11 consolidadas; fechamento operacional do PR/branch pendente**;
- Blocos 11–12: pendentes.

Gate antes de abrir o Bloco 11:

```text
PR #24
→ ready
→ squash merge em main
→ apagar branch remota
→ verificar somente main
→ zero PRs abertos
```

## 12. Pendências vigentes

### Ambiente corporativo

- Windows/WebView2 real e PoC do fallback Pocket;
- execução do Launcher pelo compartilhamento;
- Word/impressoras;
- SMB/permissões/falhas;
- EDR/firewall/políticas;
- long paths.

### Bloco 11

- pacote/mecanismo de Backup/Restore;
- atomicidade/checksums/retenção;
- restart/reconexão/sessões;
- disaster recovery local.

### Bloco 12 / implementação

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- árvore oficial/migrations/scripts/testes;
- plano Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

## 13. Precedência

Em divergência:

1. `AGENTS.md`;
2. este registro de decisões;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.
