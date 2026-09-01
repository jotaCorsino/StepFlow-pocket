# Bloco 11 — Análise 5 — Restart, sessões, reconexão e falhas

**Status:** APROVADA PELO PO / CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Data da aprovação:** 2026-08-31

## Objetivo

Fechar a continuidade técnica do Restore quando o Host é reiniciado/interrompido: journal persistente, reconciliação do filesystem, fresh Host, invalidação de sessões, reconexão dos Clients e classificação entre sucesso, rollback conhecido e `uncertain`.

## Princípio de recuperação

Restore não depende da memória do processo para saber em que estado está.

Antes da primeira alteração física de `data/`, o Host persiste journal operacional fora de `data/`. No startup seguinte, journal + realidade do filesystem são reconciliados antes de migrations/readiness.

Baseline:

```text
backups/.operations/
├── restore-active.json
└── restore-last.json
```

O journal:

- não entra em catálogo/retenção/payload;
- não contém senha/token/conteúdo de negócio;
- registra operation/source/safety IDs, fase monotônica, schema e digest do candidato;
- é atualizado por temp + flush/sync + promoção controlada antes da ação física correspondente.

## Fases persistentes mínimas

```text
PREPARED
DESTRUCTIVE_STARTED
OLD_MOVED
NEW_ACTIVATED
VALIDATED
RESTART_REQUIRED
COMPLETED
ROLLED_BACK
UNCERTAIN
```

A implementação pode agrupar estados não críticos, mas não pode perder fronteiras necessárias para recovery.

## Digest do candidato

Depois de migrations e validação do `data-next/`, calcular identidade determinística baseada em paths allowlisted/tamanhos/SHA-256.

Objetivo:

- distinguir candidato preparado de outro conteúdo após restart;
- não depender de nomes/timestamps;
- detectar alteração externa;
- complementar, não substituir, `integrity_check`/`foreign_key_check`.

D11.106 acrescenta revalidação desse digest imediatamente antes de `DESTRUCTIVE_STARTED`.

## Ordem no startup

```text
resolver deployment + exclusividade
→ verificar journal/artefatos de Restore
→ reconciliar recovery pendente
→ somente após estado conhecido seguir migrations/readiness normais
→ reconstruir caches
→ aceitar login/uso
```

Enquanto recovery estiver pendente:

- sem migration normal por conveniência;
- sem mutações;
- sem readiness normal;
- sem retenção/cleanup destrutivo;
- preservar source/safety backup, staging e old relevantes.

## Matriz de reconciliação

### `data/` + `data-next/`, sem `old`

Primeira troca física não ocorreu. Validar `data/`; se válido, Restore termina como falha pré-destrutiva e candidato não é aplicado automaticamente.

### sem `data/`, com `old/` + `data-next/`

Queda entre `data → old` e `data-next → data`.

```text
old → data
→ validar estado anterior
→ ROLLED_BACK ou UNCERTAIN
```

Não completar automaticamente o Restore nessa condição.

### `data/` + `old/`, sem `data-next/`

Pode representar candidato ativado. Comparar digest de `data/` com candidato + validar SQLite/files. Se não corresponder, tentar rollback conhecido; se nada for comprovável, `UNCERTAIN`.

### somente `data/`

Se journal/fase terminal sustentarem o estado, validar e finalizar. Caso contrário, `UNCERTAIN`.

### combinação inesperada/journal inválido

Não escolher destrutivamente. Entrar em `RECOVERY_REQUIRED/uncertain` e preservar artefatos.

## Reinicialização controlada

Depois de Restore aplicado/validado, ou rollback após fase destrutiva:

```text
persistir RESTART_REQUIRED
→ encerrar listeners/WebSockets
→ fechar SQLite/handles
→ Host sai com motivo controlado
→ Controller relança fresh Host de forma bounded
→ fresh Host reconcilia journal
→ readiness normal
```

O relaunch não é watchdog geral nem loop infinito. Se a tentativa de recovery falhar, exige intervenção local/controlada.

## Sessões e Clients

Qualquer Restore que entre na fase destrutiva:

- invalida todas as sessões/tokens anteriores;
- mantém essa invalidação mesmo em rollback;
- nunca permite que conteúdo restaurado ressuscite token reutilizável antigo;
- exige novo login após fresh Host.

Restore que falha/cancela antes da fase destrutiva não exige revogação global apenas por ter preparado staging.

WebSocket/evento de manutenção é best-effort; desconexão nunca prova sucesso/falha. Client reconecta, revalida implantação/compatibilidade, autentica novamente e refaz consultas.

## Resultado persistente

`restore-last.json`/equivalente preserva continuidade mínima pós-restart:

- `operation_id`;
- estado terminal;
- source/safety backup IDs;
- timestamps;
- código técnico resumido/sanitizado.

Não substitui auditoria histórica.

## Cleanup

- `restore-active.json` só é removido depois que fresh Host prova estado conhecido e resultado terminal consultável;
- em `uncertain`, permanece;
- `old` não é removido antes da validação do estado novo;
- cleanup posterior é best-effort;
- falha de cleanup não transforma `completed` em falha;
- `uncertain` suspende todo cleanup destrutivo relevante.

## Taxonomia externa mínima

```text
preparing
maintenance
completed
failed_pre_destructive
rolled_back
uncertain
```

`uncertain` nunca vira sucesso por timeout, filename ou disponibilidade parcial.

## Decisões aprovadas — D11.62 a D11.82

- **D11.62:** Restore persiste journal operacional fora de `data/` antes da troca física;
- **D11.63:** baseline do journal = `backups/.operations/restore-active.json`, fora de catálogo/retenção/payload;
- **D11.64:** journal registra IDs, fase, schema e digest sem segredos;
- **D11.65:** mudanças críticas do journal usam escrita/flush/promoção antes da ação física correspondente;
- **D11.66:** fresh Host reconcilia Restore antes de migrations/readiness;
- **D11.67:** candidato preparado recebe digest determinístico;
- **D11.68:** queda antes da primeira troca preserva estado original e termina como falha conhecida;
- **D11.69:** queda entre os dois renames causa rollback para `old`, não conclusão automática;
- **D11.70:** candidato ativo só pode ser finalizado quando digest + validação o comprovarem; caso contrário rollback conhecido ou `uncertain`;
- **D11.71:** filesystem/journal não comprovável leva a `RECOVERY_REQUIRED/uncertain`;
- **D11.72:** Restore aplicado ou rollback após fase destrutiva exige fresh Host;
- **D11.73:** Controller pode executar relaunch bounded de recovery, sem watchdog geral;
- **D11.74:** fase destrutiva invalida todas as sessões/tokens anteriores, inclusive em rollback;
- **D11.75:** conteúdo restaurado nunca ressuscita token antigo;
- **D11.76:** WebSocket/evento de manutenção é best-effort; desconexão não indica resultado;
- **D11.77:** fresh Host rejeita token pré-Restore e exige novo login;
- **D11.78:** `restore-last.json`/equivalente preserva resultado terminal mínimo;
- **D11.79:** active journal só é removido depois de fresh Host confirmar estado conhecido; em `uncertain`, permanece;
- **D11.80:** cleanup de old/staging é posterior/best-effort; `uncertain` suspende cleanup;
- **D11.81:** taxonomia mínima = `preparing`, `maintenance`, `completed`, `failed_pre_destructive`, `rolled_back`, `uncertain`;
- **D11.82:** `uncertain/RECOVERY_REQUIRED` bloqueia readiness, mutações, nova operação destrutiva, retenção e cleanup.

## Relação com decisões posteriores

D11.83–D11.103 fecham disaster recovery/auditoria/capacidades e D11.104–D11.116 fecham safety barrier, paths, provenance, limites e gates finais. Nenhuma pendência desta análise permanece aberta fora dos parâmetros/gates explicitamente reservados ao Bloco 12/ambiente real.
