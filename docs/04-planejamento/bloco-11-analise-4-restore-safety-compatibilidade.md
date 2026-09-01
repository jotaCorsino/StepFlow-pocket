# Bloco 11 — Análise 4 — Restore normal, safety backup e compatibilidade

**Status:** APROVADA PELO PO / CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Aprovação original:** 2026-08-31  
**Refinamento final:** 2026-09-01

## Objetivo

Fechar o Restore normal pela UI: revalidação integral do pacote, compatibilidade, preparação em staging, safety backup obrigatório, fronteira de cancelamento e troca controlada do estado ativo.

Os refinamentos D11.104–D11.116 da Análise 7 integram este contrato sem reabrir a UX da Tela 13.

## Pré-condições

Antes do Restore, o Host deve:

- possuir lease administrativo exclusivo `RESTORE`;
- revalidar sessão e capacidade;
- proteger o backup de origem contra retenção;
- não possuir Backup/Migration/Restore raiz concorrente;
- tratar o item do catálogo como candidato, nunca como elegível apenas por estar listado.

Restore sempre revalida integralmente o pacote.

## Pipeline pré-destrutivo

```text
Restore confirmado
→ lease RESTORE
→ revalidar pacote + source_deployment_id
→ extrair para data-next/ controlado
→ validar SQLite/integridade/foreign keys
→ avaliar compatibilidade
→ aplicar migrations forward no staging, quando necessárias
→ revalidar staging migrado
→ entrar em RESTORE_PRE_DESTRUCTIVE_MAINTENANCE
→ suspender e drenar mutações
→ capturar safety backup
→ manter barrier de mutações
→ finalizar/verificar/promover safety backup
→ confirmar safety backup
→ revalidar digest/schema/root de data-next/
→ persistir DESTRUCTIVE_STARTED
→ fechar handles necessários
→ primeiro rename
```

Qualquer falha anterior ao primeiro rename encerra o Restore sem alterar `data/` ativo.

## Staging

Baseline conceitual:

```text
StepFlow\
├── data\
├── backups\
└── .restore-staging\
    └── <operation_id>\
        └── data-next\
            ├── stepflow.sqlite
            ├── company\
            └── avatars\
```

Regras:

- `data-next/` representa o conjunto recuperável completo;
- staging de ativação fica no mesmo volume de `data/`;
- nunca extrair diretamente sobre `data/`;
- paths são lógicos, allowlisted e validados segundo a semântica Windows consolidada em D11.107–D11.108;
- reparse points/symlinks/junctions/hardlinks e entradas não regulares não são aceitos;
- staging não é servido aos Clients.

## Revalidação integral do pacote

Antes de usar o payload:

- envelope/ZIP deve abrir corretamente;
- `format_version` deve ser suportado;
- `backup_id`, manifesto e `source_deployment_id` devem ser coerentes;
- origem da implantação deve corresponder ao deployment atual no baseline;
- todas as entradas esperadas devem respeitar o contrato da versão;
- tamanho e SHA-256 de cada entrada devem conferir;
- limites estruturais e preflight de espaço devem passar;
- extração ocorre apenas sob roots controlados.

Falha de hash/envelope/path/provenance/limite bloqueia Restore.

## Validação SQLite pré-Restore

Sobre `data-next/stepflow.sqlite`:

- abrir isoladamente;
- confirmar identidade/schema esperado;
- executar `PRAGMA integrity_check` e exigir `ok`;
- executar `PRAGMA foreign_key_check` e exigir zero violações;
- confirmar coerência schema/migration com o manifesto.

## Compatibilidade

Compatibilidade é decidida pelo Host atual.

Um pacote é elegível somente quando:

1. `format_version` é suportado;
2. `source_deployment_id` é compatível com a implantação atual;
3. o pacote/banco são íntegros;
4. existe caminho de schema suportado.

### Schema igual

Elegível sem migration de Restore.

### Schema antigo

Elegível somente com cadeia completa, determinística e conhecida de migrations forward até o schema corrente.

Migrations rodam no `data-next/`, nunca no banco ativo, e o staging é revalidado depois.

### Schema mais novo ou cadeia incompleta

Incompatível. Não executar down migration automática nem interpretação parcial.

`source_app_version` permanece metadado de diagnóstico; não substitui `format_version + schema/migration path`.

## Safety backup obrigatório

O safety backup é criado depois que o candidato está preparado e antes da fase destrutiva.

Regras consolidadas:

- usa a pipeline de Backup sob o mesmo lease `RESTORE`;
- recebe `origin=system` e `reason=pre_restore`;
- recebe `backup_id` próprio;
- precisa ser pacote final confirmado/verificável;
- qualquer falha bloqueia Restore normal;
- não existe opção de “continuar mesmo assim” na UI normal.

### Continuidade do safety barrier — refinamento final

D11.18 continua válido para Backup normal. Para `pre_restore`, porém:

- novas mutações são suspensas antes da captura;
- mutações aceitas são drenadas;
- o safety snapshot é capturado no ponto quiescente;
- **o barrier permanece fechado até o primeiro rename de `data/`**;
- nenhuma mutação de negócio ou mutação interna em `data/`, `company/**` ou `avatars/**` ocorre depois da captura;
- journal/admin-audit externos a `data/` podem continuar;
- se o safety backup falhar ou o usuário cancelar antes do primeiro rename, o barrier é liberado e `data/` permanece intacto.

Assim, o safety backup representa efetivamente o último estado pré-destrutivo.

## Revalidação final do candidato

Depois da confirmação do safety backup e antes de `DESTRUCTIVE_STARTED`:

- recalcular/verificar digest de `data-next/`;
- confirmar correspondência byte-a-byte com o conjunto validado;
- confirmar schema esperado;
- confirmar root/volume controlado;
- qualquer diferença aborta antes do primeiro rename e exige nova validação integral.

Se o digest permanecer idêntico, não é necessário repetir `integrity_check` completo apenas por passagem de tempo.

## Lifecycle do safety backup

- protegido enquanto Restore estiver ativo;
- protegido em `uncertain`;
- após sucesso, permanece backup `system` normal sujeito à retenção futura;
- nunca é apagado imediatamente por sucesso;
- se Restore falhar/cancelar antes da fase destrutiva, safety backup já confirmado continua válido.

## Ponto de não cancelamento

Restore permanece cancelável até imediatamente antes do primeiro rename/move que retire `data/` da posição ativa.

Depois disso:

- não existe cancelamento pelo usuário;
- a operação deve concluir, reverter tecnicamente ou entrar em `uncertain`.

## Troca do conjunto recuperável

```text
1. data\      → .restore-old-<id>\
2. data-next\ → data\
3. abrir/validar novo data\
4. liberar cleanup do old somente após confirmação
```

Regras:

- same-volume;
- no-replace;
- sem overwrite arquivo a arquivo;
- nenhuma resposta de sucesso entre os passos intermediários;
- `config/`, `logs/` e `backups/` não participam da troca;
- a sequência de dois renames não é tratada como transação atômica mágica.

## Validação pós-ativação

Antes de sucesso:

- abrir SQLite pelo path oficial;
- confirmar schema corrente;
- `integrity_check = ok`;
- `foreign_key_check` vazio;
- confirmar roots/arquivos administrados;
- confirmar readiness mínima do estado persistente.

Depois da fase destrutiva, a fronteira de sessão/restart segue D11.62–D11.82.

## Falha e rollback

### Antes do primeiro rename

Estado ativo permanece intocado.

### Depois do primeiro rename, com old comprovável

Tentar rollback local controlado e validar o estado anterior.

### Estado não comprovável/reversível

Resultado `uncertain`; bloquear mutações/readiness e preservar artefatos para Recovery.

Safety backup é proteção durável; `.restore-old-<id>` é proteção operacional de curta duração.

## Filesystem Windows

A implementação futura encapsula renames/moves em adapter Windows e deve validar:

- same-volume/no-replace;
- canonicalização e nomes Windows;
- long paths;
- ACLs;
- EDR/antivírus;
- comportamento sob crash/reboot.

Falha desses gates retorna à revisão técnica; não autoriza relaxar o contrato Pocket.

## Decisões aprovadas — D11.43 a D11.61

- **D11.43:** Restore revalida integralmente envelope, paths, tamanhos e SHA-256;
- **D11.44:** Restore extrai para `data-next/` controlado/same-volume, nunca sobre `data/` ativo;
- **D11.45:** pré-Restore exige `integrity_check = ok` + `foreign_key_check` vazio;
- **D11.46:** compatibilidade usa formato suportado + integridade + schema/migration path;
- **D11.47:** schema igual é elegível; schema antigo só com migrations forward completas;
- **D11.48:** migrations de Restore rodam e são revalidadas no staging;
- **D11.49:** schema mais novo ou cadeia incompleta/ambígua é incompatível;
- **D11.50:** Restore não executa down migration automática;
- **D11.51:** safety backup é criado após candidato preparado e antes da fase destrutiva sob o lease `RESTORE`;
- **D11.52:** safety backup deve ser confirmado; qualquer falha bloqueia Restore normal;
- **D11.53:** safety backup permanece protegido durante Restore/uncertain e depois vira backup `system` normal;
- **D11.54:** fase destrutiva só inicia após coordenação de operações/handles e persistência do journal;
- **D11.55:** cancelamento termina antes da primeira alteração física do `data/`;
- **D11.56:** ativação usa troca lógica de `data/`, não overwrite arquivo a arquivo;
- **D11.57:** baseline = `data → .restore-old-<id>` e `data-next → data`, same-volume/no-replace;
- **D11.58:** `old` permanece até validação final;
- **D11.59:** pós-ativação exige nova validação antes de sucesso;
- **D11.60:** rollback conhecido restaura estado anterior; impossibilidade de provar/reverter = `uncertain`;
- **D11.61:** restart/sessões/reconexão/recovery seguem decisões posteriores do Bloco 11.

## Refinamentos finais aplicáveis

Aplicam-se integralmente **D11.104–D11.116**, especialmente:

- safety barrier contínuo até o primeiro rename;
- digest final de `data-next/`;
- canonicalização Windows estrita;
- `source_deployment_id`;
- parser/extração bounded;
- baseline sem criptografia/assinatura application-level;
- gates Win32/ACL/EDR/long paths/crash.

## Referências

- SQLite PRAGMAs: `https://www.sqlite.org/pragma.html`
- SQLite Online Backup API: `https://www.sqlite.org/backup.html`
- Windows `MoveFileExW`: `https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw`
- Windows naming/files/paths: `https://learn.microsoft.com/windows/win32/fileio/naming-a-file`
