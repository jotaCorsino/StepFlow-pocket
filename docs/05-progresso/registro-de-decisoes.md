# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-31

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

- experiência de livro/manual;
- `Visão geral` antes da Etapa 1;
- uma Etapa por página lógica;
- stepper horizontal compacto/navegável representa navegação, não conclusão;
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
- Restore = ADM sim, Gerência não, Funcionário não;
- Gerência × Backup permanece **PENDENTE** até aprovação da Análise 6;
- Gerência × configuração da empresa permanece **PENDENTE**.

Parâmetros finais pendentes: custo Argon2id, senha mínima, duração/expiração de sessão e tamanho/entropia numérica do token.

### Restore e sessões — D11.74–D11.77

- Restore que entra na fase destrutiva invalida todas as sessões/tokens anteriores;
- isso vale também se o Restore terminar em rollback;
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

## 10. Backup / Restore — Bloco 11 em análise

### D11.1–D11.10 — estado e envelope

- Backup normal protege `stepflow.sqlite + company/** + avatars/**`, não a implantação inteira;
- binários/config/logs/backups/exportações/temporários/Client local ficam fora;
- pacote confirmado é único e imutável `.stepflow-backup`;
- ZIP `Stored` no baseline;
- `manifest.json` versionado + SHA-256 por entrada;
- paths lógicos allowlisted;
- staging precede promoção;
- SQLite usa Online Backup API.

### D11.11–D11.25 — consistência e promoção

- consistência = SQLite + arquivos administrados;
- barrier curto sobre mutações até snapshot bruto completo;
- `-wal`/`-shm` fora do pacote;
- `quick_check = ok` + `foreign_key_check` vazio na criação;
- hash/ZIP/verificação/promoção fora do barrier;
- flush explícito;
- promoção same-volume/no-replace;
- sucesso só após reabertura/confirmação;
- parcial/crash nunca vira backup válido;
- números de performance/timeout/tamanho ficam para benchmark.

### D11.26–D11.42 — catálogo, retenção e coordenação

- catálogo reconstruído dos pacotes finais, sem depender do banco ativo;
- `backup_id` é identidade; filename não é identidade canônica;
- cache de verificação somente em memória; Restore sempre revalida integralmente;
- retenção sem scheduler e por quantidade;
- `retention_max_confirmed_backups` terá valor/default final no Bloco 12;
- nunca apagar backup antigo antes de confirmar o novo apenas para abrir espaço;
- source/safety/pre-migration em uso e resultado incerto ficam protegidos;
- pacote inválido/corrompido não é apagado silenciosamente;
- lease exclusivo do Host coordena `BACKUP`, `RESTORE`, `MIGRATION`;
- safety/pre-migration backup são suboperações do lease raiz;
- `uncertain` suspende retenção/cleanup destrutivo.

### D11.43–D11.61 — Restore normal e compatibilidade

- Restore revalida envelope/hashes/banco;
- extrai para `data-next/` same-volume;
- pré-Restore exige `integrity_check = ok` + `foreign_key_check` vazio;
- compatibilidade = `format_version + schema/migration path`;
- schema mais antigo só com cadeia completa de migrations forward no staging;
- schema mais novo/sem cadeia = incompatível;
- sem down migration automática;
- safety backup confirmado antes da fase destrutiva;
- cancelamento termina antes da primeira alteração física do `data/`;
- ativação por troca lógica `data → old`, `data-next → data`;
- `old` permanece até validação final;
- rollback conhecido restaura estado anterior; impossibilidade de provar/reverter = `uncertain`.

### D11.62–D11.82 — restart, sessões, reconexão e falhas

- Restore usa journal persistente fora de `data/` (`backups/.operations/restore-active.json` baseline);
- journal registra fase/IDs/schema/digest sem segredos e é atualizado durable-before-action;
- fresh Host reconcilia Restore antes de migrations/readiness;
- digest determinístico identifica o candidato preparado;
- queda antes da primeira troca preserva estado original;
- queda entre os dois renames causa rollback para `old`, não conclusão automática;
- estado não comprovável/journal inconsistente = `RECOVERY_REQUIRED/uncertain`;
- Restore aplicado ou rollback após fase destrutiva exige fresh Host;
- Controller pode realizar relaunch bounded de recovery, sem watchdog/loop infinito;
- WebSocket de manutenção é best-effort e desconexão não indica resultado;
- `restore-last.json`/equivalente preserva resultado terminal mínimo;
- active journal só some depois de fresh Host provar estado conhecido;
- `uncertain` bloqueia readiness, mutações, nova operação destrutiva, retenção e cleanup.

Fontes detalhadas:

- `docs/04-planejamento/bloco-11-backup-restauracao.md`;
- `bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- `bloco-11-analise-4-restore-safety-compatibilidade.md`;
- `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`.

### Análise 6 — proposta, não contrato

P11.83–P11.103 propõem:

- disaster recovery local/transitório do Controller, sem listener normal de rede;
- autoridade local/ACL quando o banco não autentica;
- mesma validação/compatibilidade do Restore normal;
- Gerência × Backup = **SIM** para consultar/criar, mantendo Restore = ADM-only;
- trilha administrativa fora de `data/` para atravessar Restore;
- distinção entre journal, admin audit e logs técnicos.

Nada deste subsection vira decisão antes de aprovação explícita do PO.

## 11. Pendências vigentes

### Bloco 11

- revisar P11.83–P11.103;
- executar Análise 7 — validação técnica final;
- sincronizar decisões finais nas fontes específicas.

### Bloco 12

- estrutura oficial do repositório;
- migrations/scripts/testes iniciais;
- parâmetros finais;
- plano da Fase 2;
- sincronização segura do checkout local antes do primeiro trabalho de implementação.

### Ambiente corporativo

- Windows/WebView2 real e PoC do fallback Pocket;
- Launcher pelo compartilhamento;
- Word/impressoras;
- SMB/permissões/falhas;
- EDR/firewall/políticas;
- long paths quando aplicável.

## 12. Precedência

Em divergência:

1. `AGENTS.md`;
2. este registro;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.