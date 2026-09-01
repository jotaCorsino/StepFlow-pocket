# Bloco 12 — Análise 6 — Validação técnica/documental final da Fase 1

**Status:** PROPOSTA PARA REVISÃO DO PO  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Validar cruzadamente D12.1–D12.98, remover contradições documentais consumidas e fechar o gate que permitirá encerrar a Fase 1 sem iniciar implementação neste PR.

## Resultado da revisão

A sequência F2-T01…F2-T08 permanece coerente. Não foi identificado motivo para trocar Tauri/Rust/SQLite, introduzir Node/bundler, criar serviço residente, antecipar crates vazios ou alterar o contrato Pocket.

A revisão encontrou somente refinamentos operacionais/documentais:

1. algumas fontes estáveis ainda usavam o nome antigo `StepFlowLauncher.exe`;
2. mapas técnicos anteriores ainda diziam que parâmetros numéricos seriam fechados no Bloco 12, embora D12.56–D12.79 já os tenha fechado;
3. D12.66 deixava ambíguo o comportamento de configuração inválida de retenção;
4. o plano de packaging precisava distinguir claramente template/test fixture de `deployment.json` real de produção;
5. a sincronização local precisava de uma regra positiva de fast-forward seguro, além da lista do que não fazer;
6. gates corporativos precisam ser classificados corretamente: não bloqueiam o encerramento documental da Fase 1, mas podem bloquear a saída da Fase 2 ou produção quando forem requisito da etapa.

## Refinamentos propostos

### Configuração de retenção

`retention_max_confirmed_backups`:

- campo ausente → usar default documentado `20`;
- valor presente entre `5` e `100` → aceitar;
- valor presente inválido/fora da faixa → erro de configuração explícito;
- nunca clamp silencioso nem fallback para 20 quando o operador forneceu valor inválido.

Isso remove a escolha “default ou falha conforme o owner decidir”.

### Ownership dos parâmetros

Parâmetro aprovado deve possuir owner único e não virar knob arbitrário:

- política de senha/Argon2/token/sessão → Host/auth;
- retenção/envelope/espaço/timeouts de Backup/Restore → Host/backup-restore;
- connect/request/backoff → Client/serviço de comunicação;
- readiness/relaunch → Controller/Host lifecycle;
- rotação de logs/audit → Host/runtime logging.

Somente parâmetros explicitamente documentados como configuráveis entram em configuração operacional. Os demais são defaults/versioned policy centralizados em código/config tipado do owner e não aparecem espalhados como magic numbers.

### `deployment.json`

O repositório pode versionar schema/template/fixture sem endpoint corporativo real.

Produção exige um `deployment.json` materializado por entrada explícita de implantação, contendo somente dados não sensíveis. `deployment.json.example` ou fixture de teste nunca pode ser usado silenciosamente como configuração de produção.

Se o packaging estiver sendo executado para pacote implantável e faltar input de deployment obrigatório, ele deve falhar claramente em vez de inserir placeholder.

### Sincronização segura do checkout local

Após o gate Git do Bloco 12:

```text
cd C:\dev\StepFlow
git rev-parse HEAD
git status --short --branch
```

Se houver alteração local preexistente ou estado inesperado: **parar e reportar**.

Se o checkout estiver limpo, na branch `main`, e não houver divergência local deliberada:

```text
git fetch --prune origin
git merge --ff-only origin/main
```

Depois confirmar novamente HEAD/status. Não usar reset/clean/stash/rebase para forçar alinhamento.

Se não estiver em `main`, não trocar de branch automaticamente quando isso puder afetar trabalho local; reportar o estado ao PO antes da sincronização.

### Gates corporativos

A Fase 1 é documental/arquitetural e pode ser encerrada com gates de ambiente real ainda reservados.

A Fase 2 pode começar para construir a fundação necessária às PoCs. Porém:

- gate Pocket crítico aplicável à fundação deve ser executado no momento definido em F2-T08;
- requisito que não possa ser validado fora da empresa fica `NÃO APLICÁVEL NESTE AMBIENTE`, nunca PASS;
- falha em estação/ambiente que deva ser suportado bloqueia saída da etapa/produção correspondente;
- o executor não pode “resolver” blocker instalando manualmente dependência que viole Pocket.

### Gate do primeiro scaffold

Mesmo após o merge do Bloco 12, o primeiro scaffold não nasce automaticamente.

Ordem obrigatória:

```text
squash merge Bloco 12
→ delete branch
→ verificar main + zero PRs
→ sincronizar checkout local com segurança
→ PO autoriza início da Fase 2
→ pré-flight F2-T01
→ prompt Codex F2-T01
→ branch feat/f2-01-workspace-host
```

Nenhum prompt executável de F2-T02+ é preparado como autorização antecipada.

## Propostas P12.99–P12.108

- **P12.99:** ausência de `retention_max_confirmed_backups` usa 20; valor explicitamente inválido/fora de 5–100 gera erro de configuração, sem clamp/fallback silencioso;
- **P12.100:** cada parâmetro D12 possui owner funcional único; somente valores explicitamente documentados como configuráveis viram knobs operacionais e nenhum parâmetro fica espalhado como magic number;
- **P12.101:** `deployment.json` de produção é materializado por input explícito de implantação; template/fixture versionado nunca é usado silenciosamente em produção;
- **P12.102:** packaging implantável falha se faltar configuração de deployment obrigatória, em vez de empacotar placeholder como endpoint real;
- **P12.103:** sincronização local limpa usa `fetch --prune` + `merge --ff-only origin/main`; qualquer alteração/divergência inesperada interrompe o fluxo sem reset/stash/clean/rebase corretivo;
- **P12.104:** checkout fora de `main` não é trocado automaticamente quando houver risco de afetar trabalho local; o estado volta ao PO antes da sincronização;
- **P12.105:** gates corporativos reservados não bloqueiam o encerramento documental da Fase 1 nem o início da construção da Fase 2, mas podem bloquear a saída da Fase 2/produção quando aplicáveis;
- **P12.106:** validação corporativa impossível no ambiente disponível é `NÃO APLICÁVEL NESTE AMBIENTE`, nunca PASS presumido;
- **P12.107:** merge do Bloco 12 não autoriza scaffold sozinho; F2-T01 exige remoto limpo, sync local seguro, autorização do PO, pré-flight e prompt Codex próprios;
- **P12.108:** após incorporar P12.99–P12.107 e sincronizar as fontes estáveis, não há bloqueador arquitetural/documental conhecido para encerrar a Fase 1.

## Correções documentais de sincronização

Estas correções não criam novas decisões:

- `StepFlowLauncher.exe` → `StepFlow.exe` nas fontes ativas;
- árvore central antiga `StepFlow\app|config|data|...` → `_internal\server\...` quando o documento representar a publicação completa;
- parâmetros “reservados ao Bloco 12” → referência aos valores D12.56–D12.79;
- remoção de gate histórico do PR #26 e de pendências já consumidas.

## Fora do escopo

- executar a sincronização local agora;
- criar workspace/Cargo/code;
- abrir branch F2-T01;
- executar gates corporativos ainda sem ambiente;
- reabrir contratos funcionais de Fases 3–9 sem blocker objetivo.
