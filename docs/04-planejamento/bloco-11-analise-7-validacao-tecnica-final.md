# Bloco 11 — Análise 7 — Validação técnica final

**Status:** APROVADA PELO PO / CONSOLIDADA em 2026-09-01  
**Bloco:** 11 — Backup / Restauração técnico  
**Data:** 2026-09-01

## Objetivo

Executar a revisão cruzada final de D11.1–D11.103, validar as premissas técnicas principais e fechar lacunas arquiteturais antes de declarar o Bloco 11 implementável.

Esta análise não cria implementação, migration oficial, nova UX ou infraestrutura pesada.

## Resultado geral

A arquitetura de Backup/Restore permanece tecnicamente viável e coerente com o contrato Pocket.

Foram mantidos:

- SQLite/`rusqlite`;
- SQLite Online Backup API;
- pacote ZIP `Stored`;
- captura Host-side;
- barrier lógico de mutações;
- Restore por staging + troca de `data/`;
- journal externo ao `data/`;
- fresh Host após Restore destrutivo;
- Recovery local/transitório pelo Controller.

A revisão encontrou quatro lacunas que foram fechadas nesta etapa:

1. continuidade exata entre safety backup e início destrutivo;
2. revalidação final do candidato após o safety backup;
3. paths seguros segundo semântica Windows;
4. provenance e limites estruturais explícitos do pacote.

Após os refinamentos abaixo, **não existe bloqueador arquitetural conhecido para o Bloco 11**.

## Validação das tecnologias

- Online Backup API permanece baseline para snapshot SQLite consistente;
- o barrier StepFlow continua necessário para alinhar SQLite com `company/**` e `avatars/**`;
- Restore/Recovery mantêm `integrity_check = ok` + `foreign_key_check` vazio;
- ZIP `Stored` permanece viável sem ferramenta externa;
- renames/moves de diretório permanecem same-volume/no-replace sob adapter Windows explícito;
- `sync_all`/flush permanecem parte do protocolo, sem promessa absoluta contra toda falha física.

## Safety backup pré-Restore

O `pre_restore` usa uma variante especial da pipeline de Backup:

```text
candidato data-next validado
→ RESTORE_PRE_DESTRUCTIVE_MAINTENANCE
→ suspender novas mutações
→ drenar mutações aceitas
→ ponto quiescente
→ capturar SQLite + company + avatars
→ MANTER barrier
→ finalizar/hash/ZIP/verificar/promover safety backup
→ confirmar safety backup
→ revalidar data-next
→ DESTRUCTIVE_STARTED
→ fechar handles necessários
→ primeiro rename de data/
```

Regras:

- D11.18 continua válido para Backup normal;
- no safety backup `pre_restore`, o barrier não é liberado entre captura e primeiro rename;
- nenhuma mutação de negócio ou mutação interna em `data/` ocorre após a captura;
- journal/admin-audit externos continuam permitidos;
- falha/cancelamento antes do primeiro rename libera o barrier e mantém o estado ativo intocado;
- safety backup já confirmado continua válido se o Restore for cancelado antes da fase destrutiva.

## Revalidação final de `data-next/`

Imediatamente antes de `DESTRUCTIVE_STARTED`:

- recalcular/verificar o digest determinístico;
- confirmar correspondência com o conjunto validado após migrations;
- confirmar schema esperado;
- confirmar staging no root/volume controlado;
- diferença aborta antes do primeiro rename e exige nova validação.

Se o digest byte-a-byte permanecer idêntico, não é necessário repetir `integrity_check` completo; digest diferente exige nova validação integral.

## Segurança de paths no Windows

Paths do pacote são lógicos e materializados somente após validação Windows estrita.

Rejeitar:

- path absoluto;
- drive prefix (`C:` etc.);
- UNC/device namespace;
- `.` ou `..` como segmento;
- `:`/alternate data streams;
- NUL/caracteres de controle;
- nomes reservados Win32 (`CON`, `PRN`, `AUX`, `NUL`, `COM*`, `LPT*`, inclusive com extensão);
- trailing space/dot;
- colisão case-insensitive/Windows-equivalent;
- duplicidade pós-canonicalização;
- symlink, hardlink, junction, reparse point ou entrada não regular/controlada;
- qualquer materialização fora do root de staging.

A criação do Backup aplica a mesma disciplina aos arquivos existentes em `company/` e `avatars/`; entrada não canônica causa falha explícita.

Long paths não são resolvidos por truncamento; permanecem gate do adapter/ambiente real.

## Provenance do pacote

`manifest.json` inclui:

```text
source_deployment_id
```

Regras:

- deriva da identidade não secreta da implantação;
- faz parte do formato inicial;
- Restore normal e Recovery baseline exigem correspondência;
- pacote de outra implantação retorna `source_mismatch` mesmo se íntegro/compatível;
- migração deliberada entre implantações exige contrato futuro próprio.

`backup_id` continua identidade do backup; `source_deployment_id` identifica a origem da implantação.

## Limites estruturais

Parser e extração são bounded.

Devem existir limites para:

- quantidade de entradas;
- tamanho individual;
- total declarado/extraído;
- profundidade/comprimento de path;
- overflow aritmético;
- espaço livre/preflight;
- buffers/alocações durante streaming.

Os valores numéricos pertencem ao Bloco 12/benchmark/fixtures; sua existência é contrato do Bloco 11.

## Criptografia e assinatura

Baseline inicial:

- sem criptografia application-level do `.stepflow-backup`;
- sem assinatura criptográfica application-level;
- SHA-256 representa integridade/corrupção, não autenticidade.

Trust boundary inicial:

- root administrado;
- ACLs Windows;
- proteção/criptografia de volume quando fornecida pela infraestrutura;
- `source_deployment_id`;
- auditoria administrativa.

Criptografia/assinatura futuras exigem novo `format_version` e key management explícito.

## Limite operacional do disaster recovery

O Backup StepFlow protege contra corrupção, Restore de estado anterior, falha de migration/operação crítica quando houver backup adequado e perda de `data/` enquanto `backups/` permanecer disponível.

Não promete, sozinho, proteção contra:

- perda física total do volume/servidor;
- ransomware com acesso ao mesmo storage;
- desastre de site.

Cópia/offsite corporativa de `backups/` pertence à infraestrutura operacional externa ao baseline StepFlow.

## Auditoria durante safety barrier

Após a captura do safety backup e enquanto o barrier permanecer fechado:

- não escrever evento funcional no SQLite ativo;
- não alterar `company/**`/`avatars/**`;
- journal e admin-audit externos podem continuar;
- auditoria funcional pós-Restore ocorre sobre o estado novo, quando aplicável.

## Parâmetros reservados ao Bloco 12

Permanecem deliberadamente pendentes, sem liberdade para o executor inventar valores:

- `retention_max_confirmed_backups`;
- limites de tamanho/entradas/path;
- margem mínima de espaço;
- timeouts de Backup/Restore;
- duração alvo do barrier/manutenção;
- backoff/reconexão;
- limiares de warning;
- rotação física do admin audit/logs;
- versões pinadas de crates/adapters;
- parâmetros finais de autenticação já pendentes.

## Matriz mínima de validação futura

### Backup

- zero/alguns arquivos administrados;
- leituras concorrentes;
- mutações entrando no barrier;
- falha/espaço em staging/ZIP/sync;
- crash antes/depois da promoção;
- resposta perdida após promoção;
- retenção com backups protegidos.

### Restore

- schema igual/antigo compatível/antigo sem cadeia/novo;
- hash, SQLite ou FK inválidos;
- `source_deployment_id` diferente;
- traversal/UNC/device/ADS/nome reservado/case collision;
- pacote acima dos limites;
- safety backup falhando;
- mutação durante safety barrier;
- candidato alterado após validação;
- cancelamento/crash em cada fronteira da troca;
- rollback conhecido/`uncertain`;
- fresh Host + rejeição de token antigo.

### Disaster recovery

- banco ilegível/`data/` incompleto;
- journal reconciliável ou ambíguo;
- pacote íntegro/mismatch;
- falha de ACL;
- falha ao preservar `.recovery-old-*`;
- Recovery sem listener normal;
- fresh Host após Recovery.

### Windows corporativo

- filesystem suportado;
- rename same-volume/no-replace;
- long paths;
- EDR/antivírus;
- ACLs;
- pressão de espaço;
- crash/reboot nos pontos de journal;
- Recovery sem elevação quando ACLs permitirem.

## Gates de ambiente real

Obrigatórios antes de produção:

- adapter Win32 de rename/journal/promoção;
- filesystem real do servidor;
- ACLs;
- EDR/antivírus;
- long paths;
- espaço/volume;
- performance com base representativa;
- crash injection/restart.

Falha nesses gates retorna à revisão técnica e não autoriza enfraquecer o contrato Pocket.

## Decisões aprovadas — D11.104 a D11.116

- **D11.104:** safety backup `pre_restore` mantém o barrier desde a captura até o primeiro rename; D11.18 permanece para Backup normal;
- **D11.105:** nenhuma mutação de negócio/interna em `data/` ocorre após a captura; journal/admin-audit externos permanecem permitidos;
- **D11.106:** antes de `DESTRUCTIVE_STARTED`, o digest de `data-next/` é revalidado; diferença aborta antes do primeiro rename;
- **D11.107:** paths usam validação/canonicalização Windows estrita, bloqueando drive/UNC/device/ADS/reserved names/trailing dot-space/case collision/non-regular entries/escape do root;
- **D11.108:** criação do Backup aplica a mesma disciplina a `company/**` e `avatars/**`;
- **D11.109:** manifesto inclui `source_deployment_id`; Restore/Recovery baseline bloqueiam origem diferente com `source_mismatch`;
- **D11.110:** parser/extração têm limites estruturais e preflight de espaço; números finais ficam no Bloco 12;
- **D11.111:** pacote inicial não usa criptografia application-level; futura criptografia exige key management + novo formato;
- **D11.112:** pacote inicial não usa assinatura criptográfica; SHA-256 é integridade, não autenticidade;
- **D11.113:** Backup/Recovery local não prometem sobrevivência a perda física total/ransomware/site loss; offsite é responsabilidade operacional externa;
- **D11.114:** parâmetros numéricos de tamanho/espaço/timeout/retention/log rotation ficam para Bloco 12 e não são escolha livre do executor;
- **D11.115:** adapter Windows e comportamento sob ACL/EDR/long paths/crash são gates obrigatórios antes de produção;
- **D11.116:** após D11.104–D11.115, não existe bloqueador arquitetural conhecido para o Bloco 11.

## Referências técnicas verificadas

- SQLite Online Backup API: `https://www.sqlite.org/backup.html`
- SQLite PRAGMA `integrity_check` / `foreign_key_check`: `https://www.sqlite.org/pragma.html`
- `rusqlite` backup module: `https://docs.rs/rusqlite/latest/rusqlite/backup/`
- Rust `File::sync_all`: `https://doc.rust-lang.org/std/fs/struct.File.html#method.sync_all`
- Rust `zip` / `CompressionMethod::Stored`: `https://docs.rs/zip/latest/zip/enum.CompressionMethod.html`
- Windows `MoveFileExW`: `https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw`
- Windows naming/files/paths: `https://learn.microsoft.com/windows/win32/fileio/naming-a-file`

## Conclusão

**Análise 7 aprovada e consolidada.** O Bloco 11 não possui bloqueador arquitetural conhecido e pode ser encerrado documentalmente após sincronização das decisões D11.104–D11.116 nas fontes vigentes e conclusão do gate Git normal.