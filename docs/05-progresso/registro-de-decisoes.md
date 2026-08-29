# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-29

Este arquivo registra **decisões vigentes, pendências reais e gates ativos**. Detalhes pertencem aos documentos específicos. Proposta não aprovada não é contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade durante o fechamento da Fase 1;
- checkout local previsto: `C:\dev\StepFlow`;
- sincronização local permanece adiada até antes do primeiro trabalho de implementação e deve preservar alterações do PO;
- fluxo documental: branch → PR → revisão/aprovação → squash merge → remover branch → verificar remoto limpo;
- branch mergeada só está encerrada após remoção do remoto;
- Fase 1 não autoriza scaffold/runtime/código de negócio oficial antes do gate correspondente;
- `AGENTS.md` é a regra operacional superior;
- documento técnico estável não deve carregar próximo bloco nem gate Git histórico já consumido.

## 2. Contrato Pocket

O StepFlow deve ser utilizável a partir de uma pasta pronta publicada num servidor Windows.

```text
pasta publicada no servidor
→ estação acessa o compartilhamento
→ usuário executa StepFlowLauncher.exe
→ Launcher prepara/valida versão local em %LOCALAPPDATA%
→ Client abre localmente
```

Requisitos:

- sem instalador tradicional obrigatório por estação;
- sem configuração manual de dependências;
- sem elevação administrativa no uso normal;
- sem toolchain de desenvolvimento em produção;
- sem Office/LibreOffice/Adobe como dependência operacional;
- sem Internet obrigatória no uso normal;
- Client não roda permanentemente do SMB;
- Controller/Host é iniciado na máquina central quando o ciclo StepFlow será usado;
- fechar um Client não encerra Host;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon como baseline.

Se dependência exigir setup/admin manual por computador, a solução não atende ao Pocket.

### WebView2

- Evergreen compatível já presente é preferível;
- disponibilidade real precisa ser detectada;
- não baixar/instalar runtime silenciosamente da Internet em produção;
- Fixed Version não roda de localização de rede/UNC;
- fallback autocontido, se necessário, deve ser local e só entra após PoC provar `%LOCALAPPDATA%` sem instalação, elevação ou ação manual;
- falha em estação que deva ser suportada é bloqueador do fallback, não autorização para reduzir o requisito Pocket.

Fontes: `docs/03-arquitetura/implantacao-pocket.md`, `launcher-distribuicao-client.md`, `compatibilidade-windows-client.md`.

## 3. Produto e domínio

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação/modelo oficial;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode existir sem Equipamento e usar zero, um ou vários Procedimentos;
- vínculo preserva a revisão exata utilizada;
- alteração futura do Procedimento/Equipamento não reescreve histórico concluído.

O StepFlow não vira por inferência CRM, financeiro/faturamento, estoque, RMM ou help desk/SLA completo.

## 4. Procedimentos, categorias e revisões

Campos principais: Código, Título, Área/Departamento, Responsável, Status, Versão, Objetivo, Observações, Pré-requisitos, Categorias, Etapas e Histórico.

Categorias:

- configuráveis, múltiplas e sem árvore inicial;
- pesquisáveis/filtráveis;
- arquivamento preserva histórico;
- gestão por ADM/Gerência;
- evitar nomes normalizados equivalentes.

Editor/revisões:

- `Informações` + `Etapas`;
- painel local `Estrutura`, sem segunda sidebar global;
- blocos tipados;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- controle otimista e `409` para base obsoleta;
- sem merge automático;
- publicar é separado de salvar;
- `revision_no` técnico separado de `display_version` editorial.

Pendente: regra editorial de nova revisão ainda referenciando categoria arquivada.

## 5. Reader e direção visual

- experiência de livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa por página lógica;
- Sumário temporário + Anterior/Próxima + `Etapa X de Y`;
- stepper horizontal compacto e navegável;
- stepper representa navegação, nunca conclusão operacional;
- comandos/código preservam whitespace, nunca executam e usam copiar icon-only acessível;
- cor nunca é o único canal semântico;
- mostrar permanentemente somente o necessário para entender e agir.

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
- abrir tela não cria registro oficial;
- conclusão exige responsável + `Resumo do trabalho` + estado confirmado;
- checklist incompleto avisa, mas não bloqueia automaticamente;
- cancelamento exige motivo;
- Concluído/Cancelado são read-only até reabertura;
- checklist persiste somente em Atendimento;
- progresso deriva somente do checklist;
- 100% não conclui automaticamente;
- checklist usa concorrência granular por item/equivalente.

`Observação do serviço` por Etapa:

- opcional e somente em Atendimento;
- ligada ao vínculo da revisão + Etapa;
- não altera Procedimento oficial;
- concorrência granular por Etapa/equivalente;
- somente leitura em Concluído/Cancelado até reabertura;
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

## 8. Arquitetura técnica

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- WAL + writer lógico coordenado + fila bounded + revisão otimista;
- eventos pós-commit;
- sessão opaca server-side, token em memória, Argon2id;
- implantação central por pasta pronta;
- dados/config/logs/backups separados de binários substituíveis;
- nenhuma toolchain na produção.

Parâmetros finais de Argon2id, senha, sessão e token permanecem pendentes.

## 9. Geração documental — Bloco 10 concluído

- geração pertence ao Host;
- Client solicita fonte + revisão esperada;
- Host captura snapshot consistente e `DocumentModel` antes da renderização;
- renderers não usam DOM/HTML nem reconsultam SQLite;
- renderização fica fora da fila de mutações e usa capacidade bounded;
- PDF usa Typst embutido, PDF 1.7 + Tagged PDF baseline;
- DOCX usa OOXML Transitional direto em Rust sob adaptador;
- impressão usa o mesmo PDF oficial no Client Windows via WebView2 + `ShowPrintUI(System)`;
- Procedimento físico = A4 retrato multipágina, margens-base 18 mm;
- nenhuma truncagem/redução silenciosa.

### Ficha compacta

- prestação de contas resumida ao cliente;
- PDF e preview SVG derivam do mesmo `PagedDocument`;
- exatamente uma A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- soft limits 600/400/300/280 orientam, não bloqueiam nem truncam;
- correção ocorre nos campos reais, sem editor paralelo/IA/resumo automático/compactação automática;
- Procedimentos vinculados não são listados por padrão;
- MACs: 0 omite; 1–2 valores; 3+ somente quantidade;
- observações legítimas não recebem cap/descarte automático.

### Naming e temporários

- Procedimento: `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`; sem `display_version`, omite esse segmento;
- Ficha: `{service_code} - Ficha.pdf`;
- sanitização segue filename Windows;
- conflito não causa overwrite silencioso;
- save só é sucesso após gravação integral;
- temporário só existe quando integração local exige filesystem;
- cleanup/scavenging best-effort, sem daemon/serviço/tarefa agendada;
- arquivo salvo pelo usuário nunca entra no cleanup normal.

### Validação técnica final

- nenhum bloqueador arquitetural conhecido;
- DOCX/Word, impressão, SMB, Windows/WebView2 e EDR mantêm gates de ambiente real;
- adapter Tauri/Wry/WebView2 deve ser pinado/testado;
- limites de memória/tamanho/concorrência/fila/timeout serão definidos por benchmark;
- contrato Pocket é gate superior.

Fonte: `docs/04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`.

## 10. Backup / Restore — Bloco 11 em análise

### Decisões aprovadas — estado e envelope

- Backup normal protege o estado da aplicação, não a implantação inteira;
- payload = `stepflow.sqlite` + `company/**` + `avatars/**`;
- `app/`, `config/`, `logs/`, `backups/`, exportações, temporários e Client local ficam fora;
- pacote confirmado é um único `.stepflow-backup`, ZIP `Stored` no baseline;
- `manifest.json` versionado registra identidade, origem, compatibilidade, tamanho e SHA-256 por entrada;
- paths são lógicos/allowlisted; pacote parcial nunca é válido;
- SQLite é capturado pela Online Backup API, não por cópia crua do arquivo ativo.

### Decisões aprovadas — consistência e promoção

- consistência é definida sobre `SQLite + arquivos administrados`;
- Host aplica barrier curto sobre mutações, drena mutações já aceitas e captura banco + `company/**` + `avatars/**` no mesmo ponto quiescente;
- leituras seguras podem continuar durante a captura;
- `-wal`/`-shm` não entram no payload;
- barrier termina após snapshot bruto completo em staging; hash/ZIP/verificação/promoção ocorrem fora dele;
- candidato exige envelope válido, SHA-256 por entrada, `PRAGMA quick_check = ok` e `foreign_key_check` vazio;
- `integrity_check` completo fica para validação pré-Restore;
- pacote recebe flush explícito antes da promoção;
- promoção final é same-volume, no-replace, sem overwrite silencioso;
- sucesso só ocorre após reabertura/confirmação do arquivo final;
- crash/falha nunca converte staging/parcial em backup válido;
- números de timeout/tamanho/duração de barrier ficam para benchmark.

Fonte: `docs/04-planejamento/bloco-11-backup-restauracao.md`.

## 11. Estado da Fase 1

- Blocos 0–10: encerrados nos respectivos escopos documentais;
- Bloco 11: Backup/Restore técnico em análise;
- Bloco 12: estrutura oficial + Fase 2 pendente.

## 12. Pendências vigentes

### Segurança/configuração

- parâmetros finais de Argon2id/senha/sessão/token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada.

### Ambiente corporativo

- Windows/WebView2 real e PoC do fallback Pocket;
- execução do Launcher pelo compartilhamento;
- Word/impressoras;
- SMB/permissões/falhas;
- EDR/firewall/políticas;
- long paths quando aplicável.

### Bloco 11

- catálogo/retenção/coordenação administrativa;
- Restore/safety backup/compatibilidade;
- restart/reconexão/sessões e falhas;
- disaster recovery local;
- capacidades/auditoria;
- validação técnica final.

### Bloco 12

- árvore oficial/migrations/scripts/testes;
- parâmetros finais;
- plano Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

## 13. Precedência

Em divergência:

1. `AGENTS.md`;
2. este registro;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.
