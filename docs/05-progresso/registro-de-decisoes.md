# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-09-01

Este arquivo é o **digest ativo de decisões, pendências reais e gates vigentes**. Detalhes técnicos pertencem aos documentos específicos. Proposta não aprovada não é contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade durante a Fase 1;
- fluxo: branch → draft PR → discussão/refino → aprovação PO → consolidação → validação → ready → squash merge → remover branch → verificar remoto limpo;
- merge não encerra etapa/bloco sem remoção da branch e verificação do gate;
- checkout local previsto em `C:\dev\StepFlow` só deve ser sincronizado antes da implementação, preservando alterações locais do PO;
- Fase 1 não autoriza scaffold/runtime/migrations oficiais/código de negócio antes do gate do Bloco 12/Fase 2;
- `AGENTS.md` é a regra operacional superior;
- documentos estáveis não carregam gates Git consumidos nem anunciam próximo bloco.

## 2. Contrato Pocket

```text
pasta publicada no servidor Windows
→ usuário acessa compartilhamento
→ executa StepFlowLauncher.exe
→ Launcher prepara/valida Client local em %LOCALAPPDATA%
→ Client abre localmente
→ Client comunica com Host central
```

Obrigatório:

- sem instalador tradicional por estação;
- sem preparação manual de dependências;
- sem elevação administrativa no uso normal;
- sem toolchain de desenvolvimento em produção;
- sem Office/LibreOffice/Adobe como dependência operacional;
- sem Internet obrigatória;
- Client não roda permanentemente do SMB;
- Controller/Host sob demanda na máquina central;
- fechar Client individual não encerra Host;
- sem Windows Service, Task Scheduler, watchdog, tray ou daemon como baseline.

WebView2 Evergreen compatível já presente é preferível. Fixed Version não roda por UNC/SMB; fallback local só entra após PoC provar preparação automática sem instalação/admin/manualidade.

## 3. Produto e domínio

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação oficial versionada;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo opcional/reutilizável;
- Atendimento pode existir sem Equipamento e usar vários Procedimentos;
- vínculo preserva revisão exata utilizada;
- histórico concluído não é reescrito por alterações posteriores;
- StepFlow não vira CRM, financeiro, estoque, RMM ou help desk/SLA completo por inferência.

## 4. Procedimentos, categorias e revisões

Campos principais: Código, Título, Área/Departamento, Responsável, Status, Versão, Objetivo, Observações, Pré-requisitos, Categorias, Etapas e Histórico.

- categorias configuráveis, múltiplas e sem árvore inicial;
- arquivamento preserva histórico;
- editor usa `Informações` + `Etapas` e painel local `Estrutura`;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- controle otimista com conflito explícito;
- publicar é separado de salvar;
- `revision_no` técnico separado de `display_version` editorial.

Pendente: regra editorial de nova revisão ainda referenciando categoria arquivada.

## 5. Reader e direção visual

- experiência livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa por página lógica;
- stepper horizontal representa navegação, não conclusão;
- comandos preservam whitespace e usam copiar icon-only acessível;
- baixa densidade textual e informação secundária sob demanda;
- Reader standalone não persiste execução;
- Reader em Atendimento persiste checklist e `Observação do serviço` por Etapa conforme lifecycle/autorização.

## 6. Atendimentos e Equipamentos

Lifecycle:

```text
rascunho Client
→ primeiro save
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- primeiro save cria ID/código `AT-000001`;
- conclusão exige responsável + `Resumo do trabalho`;
- checklist incompleto avisa, não bloqueia automaticamente;
- progresso deriva somente do checklist;
- 100% não conclui automaticamente;
- observação de serviço por Etapa é opcional e operacional;
- Equipamento usa código `EQP-000001`, é opcional e reutilizável;
- serial/MAC/patrimônio não são identidade canônica;
- reprodução histórica suficiente é obrigatória.

## 7. Autenticação e capacidades

- Argon2id;
- sessão opaca server-side;
- token somente em memória do Client no baseline;
- autorização Host-side por capacidade;
- `ADM`, `GERENCIA`, `FUNCIONARIO` são presets;
- pelo menos um ADM ativo;
- bootstrap do primeiro ADM é local/controlado;
- Gerência não administra ADM;
- Backup = ADM sim, Gerência sim, Funcionário não;
- Restore = ADM sim, Gerência não, Funcionário não;
- Gerência × configuração da empresa permanece **PENDENTE**.

Parâmetros finais pendentes: custo Argon2id, senha mínima, duração/expiração de sessão e tamanho/entropia numérica do token.

### Restore e sessões

- Restore que entra na fase destrutiva invalida todas as sessões/tokens anteriores;
- isso vale também em rollback;
- backup restaurado nunca ressuscita token reutilizável antigo;
- fresh Host exige novo login antes do uso normal.

## 8. Arquitetura técnica

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite` bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- WAL + writer lógico coordenado + fila bounded + revisão otimista;
- eventos pós-commit;
- implantação central por pasta pronta;
- dados/config/logs/backups separados de binários substituíveis;
- nenhuma toolchain na produção.

## 9. Geração documental — Bloco 10 concluído

- geração Host-side por snapshot consistente + `DocumentModel`;
- PDF via Typst embutido;
- DOCX OOXML Transitional direto em Rust;
- impressão Windows pelo mesmo PDF oficial via WebView2;
- Procedimento físico A4 multipágina;
- Ficha PDF + preview do mesmo `PagedDocument`, exatamente uma A4;
- `SHEET_OVERFLOW` sem truncamento/segunda página/redução automática;
- soft limits 600/400/300/280 orientativos;
- naming e temporários consolidados;
- Word/impressoras/SMB/Windows/WebView2/EDR são gates de ambiente real;
- limites de performance ficam para benchmark.

## 10. Backup / Restore — Bloco 11 tecnicamente consolidado

Decisões vigentes: **D11.1–D11.116**.

### Estado e envelope — D11.1–D11.10

- Backup protege `stepflow.sqlite + company/** + avatars/**`, não a implantação inteira;
- binários/config/logs/backups/exportações/temporários/Client local ficam fora;
- pacote final único `.stepflow-backup`, ZIP `Stored`;
- `manifest.json` versionado + SHA-256 por entrada;
- staging antes da promoção;
- SQLite via Online Backup API.

### Consistência e promoção — D11.11–D11.25

- consistência = SQLite + arquivos administrados;
- Backup normal usa barrier curto até snapshot bruto;
- `-wal`/`-shm` ficam fora;
- criação exige `quick_check = ok` + `foreign_key_check` vazio;
- hash/ZIP/verificação/promoção fora do barrier normal;
- flush + promoção same-volume/no-replace;
- parcial/crash nunca vira backup válido.

### Catálogo, retenção e coordenação — D11.26–D11.42

- catálogo reconstruível sem depender do banco ativo;
- `backup_id` é identidade canônica;
- Restore sempre revalida integralmente;
- retenção sem scheduler e por quantidade;
- `retention_max_confirmed_backups` terá valor final no Bloco 12;
- backups em uso/resultado incerto ficam protegidos;
- lease exclusivo coordena `BACKUP`, `RESTORE`, `MIGRATION`;
- `uncertain` suspende retenção/cleanup destrutivo.

### Restore e compatibilidade — D11.43–D11.61

- Restore prepara `data-next/` same-volume e revalida pacote/banco;
- exige `integrity_check = ok` + `foreign_key_check` vazio;
- schema antigo somente com migrations forward completas no staging;
- schema novo/cadeia incompleta = incompatível;
- sem down migration automática;
- safety backup confirmado obrigatório;
- ativação troca logicamente `data/` e preserva `old`;
- cancelamento termina antes do primeiro rename;
- falha termina em rollback conhecido ou `uncertain`.

### Restart/recovery — D11.62–D11.82

- journal fora de `data/`;
- fresh Host reconcilia antes de migrations/readiness;
- queda entre renames causa rollback para `old`;
- estado não comprovável = `RECOVERY_REQUIRED/uncertain`;
- relaunch de Restore é bounded, sem watchdog geral;
- fase destrutiva invalida sessões antigas;
- `restore-last.json`/equivalente preserva resultado terminal;
- `uncertain` bloqueia readiness/mutações/cleanup.

### Disaster recovery, capacidades e auditoria — D11.83–D11.103

- Recovery é excepcional, local/transitório pelo Controller e sem listener normal de rede;
- autoridade local/ACL/exclusividade quando o banco não autentica;
- mesma validação/compatibilidade/migrations forward do Restore normal;
- ausência de safety backup só pode ser aceita em disaster recovery real;
- Backup = ADM/Gerência; Restore = ADM-only;
- trilha administrativa estruturada fora de `data/`;
- journal, admin audit e logs técnicos têm finalidades/lifecycles distintos.

### Validação técnica final — D11.104–D11.116

- safety backup `pre_restore` mantém barrier desde a captura até o primeiro rename;
- nenhuma mutação em `data/` ocorre após captura do safety snapshot;
- digest de `data-next/` é revalidado antes de `DESTRUCTIVE_STARTED`;
- paths seguem canonicalização Windows estrita e bloqueiam traversal/drive/UNC/device/ADS/reserved names/trailing dot-space/case collision/reparse/non-regular;
- criação de Backup aplica a mesma disciplina aos arquivos administrados;
- manifesto inclui `source_deployment_id` e bloqueia `source_mismatch` no baseline;
- parser/extração são bounded e fazem preflight de espaço;
- baseline sem criptografia nem assinatura application-level; SHA-256 é integridade, não autenticidade;
- offsite/cópia corporativa é responsabilidade operacional externa ao baseline;
- adapter Windows, ACL/EDR/long paths/crash injection são gates obrigatórios antes de produção;
- não existe bloqueador arquitetural conhecido para o Bloco 11.

Fonte principal: `docs/04-planejamento/bloco-11-backup-restauracao.md`.

## 11. Pendências vigentes

### Bloco 12

- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- parâmetros numéricos finais, inclusive retenção/limites/timeouts;
- plano da Fase 2;
- sincronização segura do checkout local antes do primeiro trabalho de implementação.

### Outras pendências funcionais/técnicas

- parâmetros finais de Argon2id/senha/sessão/token;
- Gerência × configuração da empresa;
- regra editorial de categoria arquivada.

### Ambiente corporativo

- Windows/WebView2 real e PoC do fallback Pocket;
- Launcher pelo compartilhamento;
- Word/impressoras;
- SMB/permissões/falhas;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Win32 e crash injection para Backup/Restore.

## 12. Precedência

Em divergência:

1. `AGENTS.md`;
2. este registro;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão pelo executor.
