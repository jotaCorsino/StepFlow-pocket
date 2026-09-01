# Bloco 11 — Backup / Restauração técnico

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Consolidação técnica:** 2026-09-01

## Objetivo

Fechar o contrato técnico de Backup/Restore antes da estrutura oficial e da Fase 2, sem criar implementação funcional nesta etapa.

A UX permanece definida em `../02-telas/13-backup-restauracao.md`.

## Estado das análises

| Análise | Tema | Estado |
|---|---|---|
| 1 | Estado recuperável + envelope | ✅ Aprovada |
| 2 | Consistência + escrita/promoção/verificação | ✅ Aprovada |
| 3 | Catálogo + retenção + coordenação administrativa | ✅ Aprovada |
| 4 | Restore + safety backup + compatibilidade | ✅ Aprovada |
| 5 | Restart + sessões + reconexão + falhas | ✅ Aprovada |
| 6 | Disaster recovery + capacidades + auditoria | ✅ Aprovada |
| 7 | Validação técnica final | ✅ Aprovada |

Decisões vigentes: **D11.1–D11.116**.

## 1. Estado recuperável e envelope — D11.1–D11.10

Backup normal protege estado da aplicação, não a implantação inteira:

```text
data\stepflow.sqlite
data\company\**
data\avatars\**
```

Ficam fora `app/`, `config/`, `logs/`, `backups/`, exportações, temporários e Client local.

Baseline:

- pacote único imutável `.stepflow-backup`;
- ZIP padrão `Stored`;
- `manifest.json` versionado;
- payload allowlisted;
- SHA-256 por entrada;
- SQLite via Online Backup API;
- staging precede promoção;
- parcial nunca é backup válido.

## 2. Consistência e promoção — D11.11–D11.25

Consistência abrange SQLite + arquivos administrados.

Backup normal:

```text
BACKUP_CAPTURE
→ suspender novas mutações
→ drenar mutações aceitas
→ ponto quiescente
→ capturar SQLite + company/** + avatars/**
→ liberar mutações
→ hash/ZIP/verificação/promoção fora do barrier
```

- leituras seguras podem continuar;
- `-wal`/`-shm` não entram no pacote;
- criação exige `quick_check = ok` + `foreign_key_check` vazio;
- flush explícito;
- promoção same-volume/no-replace;
- sucesso somente após reabertura/confirmação do arquivo final;
- crash/falha nunca promove parcial.

## 3. Catálogo, retenção e coordenação — D11.26–D11.42

- catálogo reconstruível dos pacotes finais, independente do banco ativo;
- `backup_id` é identidade canônica;
- Restore sempre revalida integralmente;
- cache de verificação somente em memória;
- retenção sem scheduler e por quantidade;
- valor final de `retention_max_confirmed_backups` fica para Bloco 12;
- não apagar backup antigo antes de confirmar o novo para “abrir espaço”;
- source/safety/pre-migration protegidos enquanto necessários;
- pacote inválido/corrompido não é removido silenciosamente;
- lease exclusivo coordena `BACKUP`, `RESTORE` e `MIGRATION`;
- safety/pre-migration backup são suboperações do lease raiz;
- `uncertain` suspende retenção e cleanup destrutivo.

## 4. Restore, safety backup e compatibilidade — D11.43–D11.61

- Restore revalida envelope, hashes, paths, provenance e SQLite;
- candidato é preparado em `data-next/` same-volume;
- pré-Restore exige `integrity_check = ok` + `foreign_key_check` vazio;
- compatibilidade = formato suportado + schema/migration path;
- schema antigo somente com migrations forward completas no staging;
- schema mais novo/cadeia incompleta = incompatível;
- nenhuma down migration automática;
- safety backup confirmado é obrigatório;
- cancelamento só antes da primeira alteração física de `data/`;
- ativação por troca lógica `data → old`, `data-next → data`;
- `old` permanece até validação final;
- rollback conhecido ou `uncertain` quando não for possível provar/reverter.

### Safety barrier final

D11.18 vale para Backup normal. No `pre_restore`:

```text
suspender/drain mutações
→ capturar safety backup
→ manter barrier
→ finalizar/verificar/promover safety backup
→ revalidar data-next
→ DESTRUCTIVE_STARTED
→ primeiro rename
```

Nenhuma mutação em `data/` ocorre entre a captura do safety backup e o primeiro rename.

## 5. Restart, sessões, reconexão e falhas — D11.62–D11.82

- journal de Restore fica fora de `data/`;
- baseline: `backups/.operations/restore-active.json`;
- fresh Host reconcilia Restore antes de migrations/readiness;
- digest determinístico identifica o candidato preparado;
- queda antes da primeira troca preserva estado original;
- queda entre os dois renames causa rollback para `old`;
- estado não comprovável = `RECOVERY_REQUIRED/uncertain`;
- Restore aplicado ou rollback após fase destrutiva exige fresh Host;
- Controller pode executar relaunch bounded, sem watchdog geral;
- fase destrutiva invalida todas as sessões/tokens anteriores;
- conteúdo restaurado não ressuscita token antigo;
- Clients fazem novo login após fresh Host;
- `restore-last.json`/equivalente preserva resultado terminal mínimo;
- `uncertain` bloqueia readiness, mutações, nova operação destrutiva, retenção e cleanup.

## 6. Disaster recovery, capacidades e auditoria — D11.83–D11.103

Disaster recovery é excepcional: somente quando Restore normal seguro não está disponível.

Baseline:

```text
Controller local na máquina central
→ modo Recovery transitório
→ exclusividade da implantação
→ sem listener normal HTTP/WebSocket
→ validar backup administrado
→ preparar candidato
→ confirmação local reforçada
→ preservar estado antigo quando possível
→ troca controlada
→ fresh Host normal
→ readiness
```

Regras:

- autoridade vem de acesso local controlado + ACLs + exclusividade quando o banco não autentica;
- sem autoelevação silenciosa;
- candidatos baseline em `backups/*.stepflow-backup`;
- mesma integridade/compatibilidade/migrations forward do Restore normal;
- ausência de safety backup pode ser aceita somente em disaster recovery real;
- estado antigo pode virar `.recovery-old-<id>`, sem ser rotulado como backup íntegro;
- Gerência e ADM podem consultar/criar Backup;
- Restore permanece ADM-only;
- disaster recovery local não é capability de sessão;
- auditoria funcional quando disponível + trilha administrativa fora de `data/`;
- baseline `logs/admin-audit.jsonl`/equivalente append-only pela aplicação, protegido por ACL;
- journal, admin audit e logs técnicos são mecanismos distintos.

## 7. Validação técnica final — D11.104–D11.116

### Revalidação final do candidato

Antes de `DESTRUCTIVE_STARTED`, recalcular/verificar digest de `data-next/`, schema e root/volume. Diferença aborta antes do primeiro rename.

### Paths Windows

Materialização rejeita:

- path absoluto/drive/UNC/device namespace;
- `.`/`..`;
- ADS/`:`;
- NUL/controles;
- nomes reservados Win32;
- trailing dot/space;
- colisões case-insensitive/Windows-equivalent;
- duplicidade pós-canonicalização;
- symlink/hardlink/junction/reparse/non-regular;
- escape do root.

A criação do backup aplica a mesma disciplina a `company/**` e `avatars/**`.

### Provenance

`manifest.json` inclui `source_deployment_id`.

Restore/Recovery baseline bloqueiam pacote de implantação diferente com `source_mismatch`. Migração intencional entre implantações exige contrato futuro.

### Limites estruturais

Parser/extração são bounded por:

- quantidade de entradas;
- tamanho individual/total;
- comprimento/profundidade de path;
- overflow aritmético;
- preflight de espaço;
- estratégia de streaming/alocação.

Valores numéricos ficam para Bloco 12.

### Criptografia/autenticidade

Baseline inicial:

- sem criptografia application-level;
- sem assinatura criptográfica application-level;
- SHA-256 = integridade/corrupção, não autenticidade.

Trust boundary: root administrado + ACLs + infraestrutura de volume + deployment ID + auditoria.

### Limite operacional

Backup local não promete proteção contra perda física total, ransomware com acesso ao mesmo storage ou desastre de site. Offsite/cópia corporativa de `backups/` é responsabilidade operacional externa ao baseline.

### Gates obrigatórios antes de produção

- adapter Win32 de rename/promoção/journal;
- filesystem real;
- ACLs;
- EDR/antivírus;
- long paths;
- pressão de espaço;
- performance representativa;
- crash/restart injection.

Após D11.104–D11.115, **não existe bloqueador arquitetural conhecido para o Bloco 11**.

## Parâmetros deliberadamente reservados ao Bloco 12

- `retention_max_confirmed_backups`;
- limites numéricos de tamanho/entradas/path;
- margem mínima de espaço;
- timeouts;
- duração alvo de barrier/manutenção;
- backoff/reconexão;
- limiares de warning;
- rotação física de logs/admin audit;
- versões pinadas das crates/adapters;
- parâmetros finais de autenticação já abertos em suas fontes.

Esses parâmetros não são escolhas livres do executor.

## Critérios de fechamento

- [x] decisões permitem implementação sem escolhas arquiteturais críticas abertas;
- [x] UX da Tela 13 permanece coerente;
- [x] modelo de dados/migrations conhece a integração aplicável;
- [x] contrato Pocket permanece intacto;
- [x] parcial nunca é backup válido;
- [x] Restore possui estados de falha/recovery;
- [x] disaster recovery está separado do Restore normal;
- [x] capacidades e auditoria estão fechadas;
- [x] validação técnica final não identificou bloqueador arquitetural conhecido;
- [x] parâmetros numéricos foram explicitamente reservados ao Bloco 12.

## Fora do escopo

- implementação funcional;
- migrations oficiais;
- scheduler periódico;
- serviço persistente de backup;
- backup em nuvem/NAS específico;
- import/upload remoto genérico;
- nova UX sem bloqueador técnico;
- números finais sem benchmark/fixtures.

## Fontes detalhadas

- `bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- `bloco-11-analise-4-restore-safety-compatibilidade.md`;
- `bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`;
- `bloco-11-analise-6-disaster-recovery-capacidades-auditoria.md`;
- `bloco-11-analise-7-validacao-tecnica-final.md`;
- `../02-telas/13-backup-restauracao.md`.

## Conclusão

**Bloco 11 tecnicamente consolidado em 2026-09-01.** O fechamento operacional ainda depende do gate Git normal do PR #26: revisão final, squash merge, remoção da branch e verificação remota limpa.