# Bloco 11 — Backup / Restauração técnico

**Status:** EM ANÁLISE — BLOCO NÃO CONSOLIDADO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Atualização:** 2026-08-29

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
| 3 | Catálogo + retenção + coordenação administrativa | ⏳ Proposta para revisão |
| 4 | Restore + safety backup + compatibilidade | Pendente |
| 5 | Restart + sessões + reconexão + falhas | Pendente |
| 6 | Disaster recovery + capacidades + auditoria | Pendente |
| 7 | Validação técnica final | Pendente |

Detalhe da Análise 3: `bloco-11-analise-3-catalogo-retencao-coordenacao.md`.

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

`config/` fica fora para um Restore de dados não reverter silenciosamente rede, paths ou escolhas específicas da implantação.

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

- container ZIP padrão;
- método `Stored` sem compressão;
- extensão própria `.stepflow-backup`;
- nenhuma ferramenta externa é dependência operacional.

## Manifesto

`manifest.json` registra pelo menos:

- `format_version`;
- `backup_id` opaco;
- `created_at` UTC;
- `origin` (`manual`/`system`);
- ator quando aplicável;
- motivo técnico para `system` quando aplicável;
- versão StepFlow;
- schema/migration de origem;
- entradas ordenadas do payload;
- path lógico normalizado;
- tamanho;
- SHA-256 por entrada.

SHA-256 verifica integridade/corrupção; não é assinatura de autenticidade.

Paths são relativos/normalizados. Restore nunca confia em path arbitrário do pacote e não segue reparse point/symlink/junction.

## Lifecycle físico

```text
staging
→ snapshot
→ pacote candidato
→ finalizar escrita
→ verificar
→ promover para filename final
→ reabrir/confirmar
→ listar como válido
```

Pacote parcial nunca é backup válido e colisão nunca sobrescreve backup confirmado.

## SQLite

A **SQLite Online Backup API**, via `rusqlite` feature `backup`, é o mecanismo baseline.

Não copiar `stepflow.sqlite` cru e não usar `VACUUM INTO` como baseline.

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

Consistência é definida sobre **SQLite + arquivos administrados**. Banco e `company/**`/`avatars/**` precisam pertencer ao mesmo ponto lógico.

O Host usa barrier curto sobre mutações:

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

Consultas read-only podem continuar quando seguras. Nova mutação durante o barrier recebe resultado temporário/retryable em vez de acumular indefinidamente.

Ponto quiescente exige:

- writer sem mutação aceita pendente;
- nenhuma transação mutante ativa;
- nenhuma promoção de arquivo administrado ativa;
- nenhuma operação administrativa incompatível em paralelo.

## Staging

Baseline:

```text
backups/
├── .staging/<backup_id>/
│   ├── snapshot.sqlite
│   ├── company/
│   └── avatars/
└── <backups confirmados>
```

Staging é privado, opaco e preferencialmente no mesmo volume do destino final.

## SQLite / WAL

Durante o barrier:

1. criar banco destino novo em staging;
2. executar Online Backup API;
3. finalizar backup;
4. fechar destino;
5. usar banco autocontido no payload.

Não entram no pacote:

- `stepflow.sqlite-wal`;
- `stepflow.sqlite-shm`.

WAL/SHM são runtime, não estado lógico de backup.

## Arquivos administrados

`company/**` e `avatars/**` são copiados ainda no mesmo barrier, somente por roots allowlisted.

Erro de leitura em arquivo necessário invalida a captura completa. Reparse points não são seguidos.

## Fim do barrier

Assim que `snapshot.sqlite` fechado e cópias de `company/**`/`avatars/**` estiverem completas em staging, mutações podem voltar.

SHA-256, ZIP, checks e promoção trabalham somente no snapshot imutável de staging.

## Verificação antes da promoção

Envelope exige:

- ZIP válido/finalizado;
- um único `manifest.json` válido;
- `format_version` suportado;
- `backup_id` consistente;
- paths allowlisted/normalizados;
- nenhuma entrada absoluta, `..`, duplicada ou desconhecida para a versão;
- tamanhos coerentes;
- SHA-256 correto para todas as entradas.

SQLite exige na criação:

```text
PRAGMA quick_check = ok
PRAGMA foreign_key_check = vazio
```

`integrity_check` completo fica para validação pré-Restore.

## Flush e promoção

Antes da promoção:

- finalizar ZIP;
- propagar erro de fechamento;
- `sync_all`/equivalente;
- fechar handle;
- reabrir/verificar candidato.

Promoção:

```text
candidato íntegro
→ move/rename same-volume
→ no-replace
→ reabrir destino final
→ confirmar identidade
→ BACKUP_CONFIRMED
```

No Windows, usar adapter com semântica no-replace; não depender de comportamento de replace de `std::fs::rename`.

## Falhas

- falha antes da captura: nenhum backup;
- falha durante captura: liberar barrier e deixar apenas staging não confirmado;
- falha após captura e antes da promoção: estado ativo segue normal; parcial não é válido;
- filename final sozinho nunca prova validade;
- pacote final inválido/corrompido permanece classificável para diagnóstico;
- retry cego após I/O incerto é proibido;
- nenhum timeout/tamanho máximo/duração de barrier é congelado antes de benchmark.

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

## Referências técnicas

- SQLite Online Backup API: `https://www.sqlite.org/backup.html`
- SQLite backup C API: `https://www.sqlite.org/c3ref/backup_finish.html`
- SQLite isolation/WAL: `https://www.sqlite.org/isolation.html`
- SQLite `quick_check`: `https://www.sqlite.org/pragma.html#pragma_quick_check`
- Rust `File::sync_all`: `https://doc.rust-lang.org/std/fs/struct.File.html#method.sync_all`
- Windows `MoveFileExW`: `https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw`

---

# Análise 3 — Catálogo, retenção e coordenação administrativa

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA.

A proposta detalhada está em:

`bloco-11-analise-3-catalogo-retencao-coordenacao.md`

Resumo:

- catálogo reconstruível dos pacotes finais, sem depender do SQLite ativo;
- cache de verificação apenas em memória;
- retenção por quantidade e sem scheduler;
- parâmetro numérico final de retenção reservado ao Bloco 12;
- nunca remover backup antigo antes de confirmar o novo apenas para abrir espaço;
- coordinator administrativo exclusivo para `BACKUP`, `RESTORE` e `MIGRATION`;
- safety/pre-migration backup reutilizam a pipeline como suboperações, sem segundo lease;
- resultado incerto suspende cleanup destrutivo e retenção.

Propostas numeradas: **P11.26–P11.42**.

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

Após aprovação de P11.26–P11.42, seguir para **Análise 4 — Restore normal + safety backup + compatibilidade**.
