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
- Restore de Gerência não autorizado; Gerência × Backup permanece pendente;
- contrato Pocket como gate superior.

## Estado das análises

| Análise | Tema | Estado |
|---|---|---|
| 1 | Estado recuperável + envelope | ✅ Aprovada pelo PO |
| 2 | Consistência + escrita/promoção/verificação | ✅ Aprovada pelo PO |
| 3 | Catálogo + retenção + coordenação administrativa | ✅ Aprovada pelo PO |
| 4 | Restore + safety backup + compatibilidade | ⏳ Proposta para revisão |
| 5 | Restart + sessões + reconexão + falhas | Pendente |
| 6 | Disaster recovery + capacidades + auditoria | Pendente |
| 7 | Validação técnica final | Pendente |

Detalhes:

- Análise 3: `bloco-11-analise-3-catalogo-retencao-coordenacao.md`;
- Análise 4: `bloco-11-analise-4-restore-safety-compatibilidade.md`.

---

# Análise 1 — Estado recuperável + envelope

**Status:** APROVADA PELO PO em 2026-08-29.

## Fronteira do estado recuperável

O Backup normal protege **estado da aplicação**, não a implantação inteira.

Entram:

```text
data\stepflow.sqlite
data\company\**
data\avatars\**
```

Ficam fora:

- `app/`;
- `config/`;
- `logs/`;
- `backups/`;
- exportações salvas pelo usuário;
- temporários;
- Client local em `%LOCALAPPDATA%`.

Novo tipo de arquivo persistente só entra após allowlist/contrato explícito do Host.

## Envelope físico

Um backup confirmado é um único arquivo imutável:

```text
backup-<utc>-<backup_id>.stepflow-backup

manifest.json
payload/
├── stepflow.sqlite
├── company/
└── avatars/
```

Baseline:

- ZIP padrão;
- método `Stored` sem compressão;
- extensão própria `.stepflow-backup`;
- nenhuma ferramenta externa como dependência operacional.

## Manifesto e paths

`manifest.json` registra `format_version`, `backup_id`, data UTC, origem, ator/motivo quando aplicável, versão StepFlow, schema/migration de origem, entradas ordenadas, path lógico, tamanho e SHA-256 por entrada.

SHA-256 verifica integridade/corrupção; não é assinatura. Paths são relativos/normalizados e Restore não segue reparse points/symlinks/junctions.

## SQLite

A **SQLite Online Backup API**, via `rusqlite` feature `backup`, é o mecanismo baseline. Não copiar `stepflow.sqlite` cru e não usar `VACUUM INTO` como baseline.

## Decisões aprovadas — D11.1 a D11.10

- **D11.1:** Backup normal protege estado da aplicação, não implantação inteira;
- **D11.2:** payload = `stepflow.sqlite` + `company/**` + `avatars/**`;
- **D11.3:** `app/`, `config/`, `logs/`, `backups/`, exportações, temporários e Client local ficam fora;
- **D11.4:** novo arquivo persistente exige allowlist/contrato explícito;
- **D11.5:** backup confirmado é pacote único imutável `.stepflow-backup`;
- **D11.6:** container = ZIP padrão `Stored`;
- **D11.7:** pacote contém `manifest.json` + `payload/` controlado;
- **D11.8:** manifesto versionado registra origem, compatibilidade, tamanho e SHA-256 por entrada;
- **D11.9:** staging precede promoção e parcial nunca é válido;
- **D11.10:** Online Backup API é baseline do snapshot SQLite.

---

# Análise 2 — Consistência + escrita/promoção/verificação

**Status:** APROVADA PELO PO em 2026-08-29.

## Consistência lógica

Consistência é definida sobre **SQLite + arquivos administrados**. O Host usa barrier curto sobre mutações:

```text
Backup aceito
→ BACKUP_CAPTURE
→ parar novas mutações temporariamente
→ drenar mutações já aceitas
→ ponto quiescente
→ capturar SQLite + arquivos em staging
→ liberar mutações
→ hash/ZIP/verificação/promoção fora do barrier
```

Consultas read-only podem continuar quando seguras.

## Staging e WAL

Baseline:

```text
backups/
├── .staging/<backup_id>/
│   ├── snapshot.sqlite
│   ├── company/
│   └── avatars/
└── <backups confirmados>
```

`stepflow.sqlite-wal` e `stepflow.sqlite-shm` não entram no pacote.

## Verificação e promoção

Candidato exige:

- envelope válido;
- SHA-256 por entrada;
- `PRAGMA quick_check = ok`;
- `PRAGMA foreign_key_check` vazio;
- flush explícito;
- promoção same-volume/no-replace;
- reabertura/confirmação antes de `BACKUP_CONFIRMED`.

Crash/falha nunca transforma staging/parcial em backup válido.

## Decisões aprovadas — D11.11 a D11.25

- **D11.11:** consistência = SQLite + arquivos administrados;
- **D11.12:** Host usa barrier de captura para mutações; leituras seguras podem continuar;
- **D11.13:** mutações aceitas drenam; novas mutações no barrier recebem estado temporário/retryable;
- **D11.14:** ponto quiescente inclui writer, transações e promoções de arquivos;
- **D11.15:** captura bruta usa staging privado no mesmo volume de `backups/`;
- **D11.16:** SQLite usa Online Backup API para banco novo; `-wal`/`-shm` ficam fora;
- **D11.17:** `company/**` e `avatars/**` são capturados sob o mesmo barrier/allowlist;
- **D11.18:** barrier termina após snapshot bruto; hash/ZIP/verificação/promoção ficam fora;
- **D11.19:** candidato exige envelope válido + SHA-256 por entrada;
- **D11.20:** criação exige `quick_check = ok` + `foreign_key_check` vazio; `integrity_check` fica para Restore;
- **D11.21:** candidato recebe flush explícito antes da promoção;
- **D11.22:** promoção final é same-volume e no-replace;
- **D11.23:** sucesso só após reabertura/confirmação do arquivo final;
- **D11.24:** crash/falha nunca transforma staging/parcial em válido;
- **D11.25:** números de performance/timeout/tamanho ficam para benchmark.

---

# Análise 3 — Catálogo, retenção e coordenação administrativa

**Status:** APROVADA PELO PO em 2026-08-31.

Detalhe: `bloco-11-analise-3-catalogo-retencao-coordenacao.md`.

Contrato resumido:

- catálogo reconstruível dos pacotes finais, sem depender do SQLite ativo;
- `backup_id` é identidade lógica; filename não é identidade canônica;
- cache de verificação somente em memória;
- Restore sempre revalida integralmente;
- retenção sem scheduler e por quantidade;
- `retention_max_confirmed_backups` terá valor/default final no Bloco 12;
- nunca remover backup antigo antes de confirmar o novo apenas para abrir espaço;
- source/safety/pre-migration em uso ou resultado incerto ficam protegidos;
- pacote inválido/corrompido não é apagado silenciosamente pela retenção;
- coordinator/lease exclusivo para `BACKUP`, `RESTORE` e `MIGRATION`;
- safety/pre-migration backup são suboperações do lease raiz;
- Backup mantém lease completo, mas barrier de mutações apenas durante captura;
- `uncertain` suspende retenção e cleanup destrutivo.

Decisões aprovadas: **D11.26–D11.42**.

---

# Análise 4 — Restore normal, safety backup e compatibilidade

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA.

Detalhe: `bloco-11-analise-4-restore-safety-compatibilidade.md`.

Direção proposta:

- revalidação integral do pacote antes de Restore;
- extração para `data-next/` same-volume, nunca sobre `data/` ativo;
- `integrity_check` completo + `foreign_key_check` antes da fase destrutiva;
- compatibilidade por `format_version + schema/migration path`;
- schema mais antigo somente com cadeia completa de migrations forward aplicada no staging;
- schema mais novo ou sem cadeia completa = incompatível;
- nenhuma down migration automática;
- safety backup confirmado imediatamente antes da fase destrutiva;
- cancelamento somente até antes da primeira alteração física do `data/`;
- ativação por troca lógica do conjunto `data/`, não overwrite arquivo a arquivo;
- `.restore-old-<id>` preservado até validação final;
- falha reversível volta ao estado anterior; impossibilidade de provar/reverter = `uncertain`.

Propostas numeradas: **P11.43–P11.61**.

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
- decisões aprovadas forem sincronizadas nas fontes específicas.

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

Após aprovação de P11.43–P11.61, seguir para **Análise 5 — restart, sessões, reconexão, falhas e resultado incerto**.
