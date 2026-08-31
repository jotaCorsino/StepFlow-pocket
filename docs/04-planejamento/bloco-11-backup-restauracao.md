# Bloco 11 — Backup / Restauração técnico

**Status:** EM ANÁLISE — BLOCO NÃO CONSOLIDADO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Atualização:** 2026-08-31

## Objetivo

Fechar o contrato técnico de Backup/Restore antes da estrutura oficial e da Fase 2, sem criar implementação funcional nesta etapa.

## Contratos já consolidados

A UX permanece definida em `../02-telas/13-backup-restauracao.md`.

Este bloco não reabre, sem bloqueador técnico concreto:

- Backup/Restore coordenado pelo Host;
- Client sem acesso direto ao SQLite;
- autorização Host-side;
- Restore normal somente com backup elegível e confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva do Restore normal;
- falha do safety backup bloqueia o Restore normal;
- recuperação sem Host funcional pertence a disaster recovery local/controlado;
- sucesso somente após confirmação do Host;
- Backup separado de exportação documental;
- ausência de scheduler periódico por inferência;
- Restore de Gerência não autorizado; Gerência × Backup permanece pendente até aprovação da Análise 6;
- contrato Pocket como gate superior.

## Estado das análises

| Análise | Tema | Estado |
|---|---|---|
| 1 | Estado recuperável + envelope | ✅ Aprovada pelo PO |
| 2 | Consistência + escrita/promoção/verificação | ✅ Aprovada pelo PO |
| 3 | Catálogo + retenção + coordenação administrativa | ✅ Aprovada pelo PO |
| 4 | Restore + safety backup + compatibilidade | ✅ Aprovada pelo PO |
| 5 | Restart + sessões + reconexão + falhas | ✅ Aprovada pelo PO |
| 6 | Disaster recovery + capacidades + auditoria | ⏳ Proposta para revisão |
| 7 | Validação técnica final | Pendente |

Detalhes:

- Análise 3: `bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- Análise 4: `bloco-11-analise-4-restore-safety-compatibilidade.md`;
- Análise 5: `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`;
- Análise 6: `bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md`.

---

# Análise 1 — Estado recuperável + envelope

**Status:** APROVADA PELO PO em 2026-08-29.

O Backup normal protege estado da aplicação, não a implantação inteira.

Entram:

```text
data\stepflow.sqlite
data\company\**
data\avatars\**
```

Ficam fora `app/`, `config/`, `logs/`, `backups/`, exportações, temporários e Client local. Novo tipo persistente exige allowlist explícita.

Um backup confirmado é pacote imutável `.stepflow-backup`, ZIP padrão `Stored`, com `manifest.json`, payload controlado e SHA-256 por entrada. Paths são relativos/normalizados e não seguem reparse points/symlinks/junctions.

SQLite usa **Online Backup API** via `rusqlite`; cópia crua do banco ativo e `VACUUM INTO` não são baseline.

Decisões aprovadas: **D11.1–D11.10**.

---

# Análise 2 — Consistência + escrita/promoção/verificação

**Status:** APROVADA PELO PO em 2026-08-29.

Consistência é definida sobre **SQLite + arquivos administrados**. O Host aplica barrier curto sobre mutações, drena mutações aceitas, captura banco + `company/**` + `avatars/**` no mesmo ponto quiescente e libera mutações antes de hash/ZIP/verificação/promoção.

`-wal`/`-shm` não entram no pacote. Candidato exige envelope válido, SHA-256, `quick_check = ok`, `foreign_key_check` vazio, flush explícito, promoção same-volume/no-replace e reabertura antes de `BACKUP_CONFIRMED`.

Crash/falha nunca transforma staging/parcial em backup válido.

Decisões aprovadas: **D11.11–D11.25**.

---

# Análise 3 — Catálogo, retenção e coordenação administrativa

**Status:** APROVADA PELO PO em 2026-08-31.

Detalhe: `bloco-11-analise-3-catalogo-retencao-coordenacao.md`.

Contrato resumido:

- catálogo reconstruível dos pacotes finais, sem depender do SQLite ativo;
- `backup_id` é identidade lógica; filename não é identidade canônica;
- cache de verificação somente em memória; Restore sempre revalida integralmente;
- retenção sem scheduler e por quantidade;
- `retention_max_confirmed_backups` terá valor/default final no Bloco 12;
- nunca remover backup antigo antes de confirmar o novo apenas para abrir espaço;
- source/safety/pre-migration em uso ou resultado incerto ficam protegidos;
- pacote inválido/corrompido não é apagado silenciosamente pela retenção;
- coordinator/lease exclusivo para `BACKUP`, `RESTORE` e `MIGRATION`;
- safety/pre-migration backup são suboperações do lease raiz;
- `uncertain` suspende retenção e cleanup destrutivo.

Decisões aprovadas: **D11.26–D11.42**.

---

# Análise 4 — Restore normal, safety backup e compatibilidade

**Status:** APROVADA PELO PO em 2026-08-31.

Detalhe: `bloco-11-analise-4-restore-safety-compatibilidade.md`.

Contrato resumido:

- Restore sempre revalida envelope, hashes e banco;
- extração para `data-next/` same-volume, nunca sobre `data/` ativo;
- pré-Restore exige `integrity_check = ok` + `foreign_key_check` vazio;
- compatibilidade por `format_version + schema/migration path`;
- schema mais antigo somente com cadeia completa de migrations forward aplicada e revalidada no staging;
- schema mais novo ou sem cadeia completa = incompatível;
- nenhuma down migration automática;
- safety backup confirmado imediatamente antes da fase destrutiva;
- cancelamento somente até antes da primeira alteração física do `data/`;
- ativação por troca lógica do conjunto `data/`, não overwrite arquivo a arquivo;
- `.restore-old-<id>` preservado até validação final;
- falha reversível volta ao estado anterior; impossibilidade de provar/reverter = `uncertain`.

Decisões aprovadas: **D11.43–D11.61**.

---

# Análise 5 — Restart, sessões, reconexão e falhas

**Status:** APROVADA PELO PO em 2026-08-31.

Detalhe: `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`.

Contrato resumido:

- journal persistente fora de `data/` antes da troca física;
- `backups/.operations/restore-active.json` é baseline e não entra em catálogo/retenção/payload;
- fresh Host reconcilia Restore antes de migrations/readiness normais;
- digest determinístico identifica o candidato preparado após restart;
- queda antes da primeira troca preserva estado original;
- queda entre `data→old` e `data-next→data` causa rollback para `old`, não conclusão automática;
- combinação filesystem/journal não comprovável = `RECOVERY_REQUIRED/uncertain`;
- Restore aplicado ou rollback após fase destrutiva exige reinicialização controlada do Host;
- Controller pode realizar relaunch bounded de recovery, sem watchdog/loop infinito;
- fase destrutiva invalida todas as sessões/tokens anteriores;
- conteúdo restaurado nunca ressuscita token antigo;
- WebSocket de manutenção é best-effort; Clients fazem novo login e reconsultam estado após fresh Host;
- `restore-last.json`/equivalente preserva resultado terminal mínimo;
- active journal só é removido após fresh Host provar estado conhecido;
- `uncertain` bloqueia readiness normal, mutações, nova operação destrutiva, retenção e cleanup.

Decisões aprovadas: **D11.62–D11.82**.

---

# Análise 6 — Disaster recovery, capacidades e auditoria

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA.

Detalhe: `bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md`.

Direção proposta:

- disaster recovery somente quando Restore normal seguro não está disponível;
- Recovery local/transitório pelo Controller na máquina central, sem listener normal de rede;
- exclusividade da implantação e autoridade baseada em acesso local/ACL, pois o banco pode não autenticar;
- candidatos permanecem pacotes finais administrados em `backups/` e passam pela mesma validação/compatibilidade do Restore normal;
- ausência de safety backup válido pode ser aceita somente no disaster recovery real;
- preservar o estado ativo como `.recovery-old-<id>` quando possível, sem fingir que ele é backup íntegro;
- reutilizar journal/staging/troca same-volume e recovery determinístico já aprovados;
- Gerência × Backup proposta como **SIM** para consultar/criar; Restore continua **ADM-only**;
- trilha administrativa estruturada fora de `data/` para sobreviver ao Restore;
- journal, admin audit e logs técnicos permanecem mecanismos distintos.

Propostas numeradas: **P11.83–P11.103**.

---

## Critérios de fechamento do Bloco 11

O bloco só pode ser considerado concluído quando:

- decisões permitirem implementação sem escolhas críticas deixadas ao executor;
- UX da Tela 13 continuar coerente;
- modelo de dados/migrations conhecer os impactos aplicáveis;
- contrato Pocket permanecer intacto;
- nenhum backup parcial puder ser tratado como válido;
- Restore tiver estados de falha e recuperação definidos;
- disaster recovery possuir fronteira clara em relação ao Restore normal;
- capacidades e auditoria estiverem fechadas;
- decisões aprovadas forem sincronizadas nas fontes específicas;
- validação técnica final não identificar bloqueador arquitetural.

## Fora do escopo

- implementação funcional;
- migrations oficiais;
- scheduler periódico;
- serviço persistente de backup;
- backup em nuvem;
- integração com destino externo específico;
- nova UX sem bloqueador técnico;
- números finais de performance sem evidência.

## Próxima análise

Após aprovação de P11.83–P11.103, seguir para **Análise 7 — validação técnica final do Bloco 11**.