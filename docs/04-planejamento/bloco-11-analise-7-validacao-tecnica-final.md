# Bloco 11 — Análise 7 — Validação técnica final

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Data:** 2026-09-01

## Objetivo

Executar revisão cruzada final de D11.1–D11.103, validar as premissas técnicas principais contra as APIs/plataformas escolhidas e fechar qualquer lacuna arquitetural antes de declarar o Bloco 11 implementável.

Esta análise não cria implementação, migration oficial, nova UX ou novo requisito de infraestrutura pesada.

---

## 7.1 Resultado geral da revisão

A arquitetura de Backup/Restore permanece tecnicamente viável e coerente com o contrato Pocket.

Não foi identificado bloqueador que exija trocar:

- SQLite/Rusqlite;
- Online Backup API;
- pacote ZIP `Stored`;
- captura Host-side;
- barrier lógico de mutações;
- Restore por staging + troca de `data/`;
- journal externo ao `data/`;
- fresh Host após Restore destrutivo;
- Recovery local/transitório pelo Controller.

Entretanto, a revisão cruzada encontrou **quatro lacunas que precisam virar contrato antes do fechamento**:

1. continuidade exata entre safety backup e início destrutivo;
2. revalidação final do candidato após o safety backup;
3. normalização de paths segundo semântica Windows, não apenas path traversal genérico;
4. provenance/limites explícitos do pacote para não deixar decisões de segurança ao executor.

Os demais pontos encontrados podem permanecer como parâmetros/gates do Bloco 12 ou validações de ambiente real.

---

## 7.2 Validação das tecnologias escolhidas

### SQLite Online Backup API

A documentação oficial do SQLite define a Online Backup API como mecanismo capaz de produzir um snapshot consistente do banco ativo. `rusqlite` expõe essa API pela feature `backup`.

Conclusão: **baseline mantida**.

O barrier StepFlow continua necessário não por limitação do SQLite, mas para sincronizar o snapshot do banco com `company/**` e `avatars/**` no mesmo ponto lógico.

### `integrity_check` + `foreign_key_check`

SQLite documenta que `PRAGMA integrity_check` não detecta violações de foreign key. Logo, a dupla já aprovada permanece correta:

```text
integrity_check = ok
foreign_key_check = vazio
```

`quick_check` continua apropriado para criação do backup; `integrity_check` completo continua reservado a Restore/Recovery.

### ZIP `Stored`

A crate Rust `zip` suporta explicitamente `CompressionMethod::Stored`, e `ZipWriter::finish()` retorna erro que pode ser tratado pelo chamador.

Conclusão: pacote sem ferramenta externa continua viável.

### Windows — rename/move de diretórios

A documentação Win32 confirma que movimentação de diretório exige mesma unidade/volume e que destino existente não deve ser tratado como replace silencioso no nosso baseline.

Conclusão: a estratégia same-volume/no-replace permanece adequada, mas precisa de adapter testado em Windows real; a sequência de dois renames não é tratada como transação atômica única.

### Durabilidade

`File::sync_all` tenta sincronizar conteúdo e metadados ao filesystem. Continua válido como parte do protocolo, mas não cria promessa absoluta contra toda combinação de falha física/controladora/EDR/filesystem.

Conclusão: manter flush/sync + journal + reconciliação e validar o adapter no ambiente real.

---

## 7.3 Refinamento crítico — safety backup precisa fechar a janela de mutações

A revisão encontrou uma lacuna entre D11.18 e D11.51–D11.55.

No Backup normal, é correto liberar o barrier assim que o snapshot bruto de SQLite + arquivos estiver completo e finalizar hash/ZIP/verificação fora dele.

No **safety backup `pre_restore`**, porém, liberar mutações após a captura e somente depois aguardar a confirmação do pacote permitiria isto:

```text
captura do safety backup
→ liberar mutações
→ usuário salva nova alteração
→ safety backup termina/promove
→ Restore entra na fase destrutiva
```

Nesse cenário, o safety backup não representaria mais o estado imediatamente anterior ao Restore.

### Proposta

O safety backup de Restore usa uma variante controlada da mesma pipeline:

```text
candidato data-next já validado
→ entrar em RESTORE_PRE_DESTRUCTIVE_MAINTENANCE
→ parar novas mutações
→ drenar mutações aceitas
→ ponto quiescente
→ capturar SQLite + company + avatars para safety backup
→ manter barrier de mutações
→ finalizar/hash/ZIP/verificar/promover safety backup
→ confirmar safety backup
→ revalidar candidato data-next
→ persistir DESTRUCTIVE_STARTED
→ fechar leituras/handles necessários
→ primeiro rename de data/
```

Regras:

- o barrier **não é liberado** entre a captura do safety backup e o primeiro rename;
- isso é exceção específica do `pre_restore`, não muda o comportamento do Backup normal;
- leituras podem continuar somente enquanto não impedirem a transição final de manutenção;
- nenhuma mutação de negócio ou mutação interna em `data/` pode ocorrer depois da captura do safety backup;
- auditoria/journal externo a `data/` pode continuar sendo escrito;
- se safety backup falhar ou o Restore for cancelado antes do primeiro rename, liberar o barrier e manter `data/` original intacto;
- safety backup já confirmado permanece backup válido mesmo se o Restore for cancelado antes da fase destrutiva.

Isso torna verdadeira a expressão já aprovada “safety backup do estado imediatamente anterior ao Restore”.

---

## 7.4 Refinamento crítico — revalidar o candidato depois do safety backup

O `data-next/` é preparado antes do safety backup. A confirmação do safety backup pode consumir tempo relevante.

Mesmo que staging seja privado, a implementação não deve assumir que um arquivo não pode ser alterado externamente entre preparação e ativação.

### Proposta

Imediatamente antes de `DESTRUCTIVE_STARTED`:

- recalcular/verificar o digest determinístico de `data-next/`;
- exigir correspondência com o digest aprovado após migrations;
- confirmar schema esperado;
- confirmar que staging continua dentro do root/volume esperado;
- se houver diferença, abortar antes do primeiro rename, liberar manutenção e não alterar `data/` ativo.

Não é necessário repetir `integrity_check` completo se o digest byte-a-byte do conjunto validado permanecer idêntico; mudança de digest exige nova validação completa.

---

## 7.5 Refinamento crítico — paths precisam ser seguros sob semântica Windows

“Path relativo sem `..`” não é suficiente no Windows.

O parser/materializador deve tratar os nomes do pacote como **paths lógicos**, não como strings diretamente confiadas ao filesystem.

### Regras propostas

Além das regras já aprovadas, rejeitar:

- path absoluto;
- prefixo de drive (`C:` etc.);
- UNC/device namespace (`\\server`, `\\?\\`, `\\.\\`);
- `.` e `..` como segmentos;
- `:` em segmento, evitando alternate data streams;
- caracteres de controle/NUL;
- nomes reservados Win32 como `CON`, `PRN`, `AUX`, `NUL`, `COM1…`, `LPT1…`, inclusive com extensão;
- segmento terminando em espaço ou ponto;
- colisão case-insensitive/Windows-equivalent entre duas entradas;
- entrada duplicada após canonicalização;
- symlink, hardlink, junction, reparse point ou tipo de entrada diferente de arquivo regular/diretório controlado;
- qualquer materialização que, após canonicalização, saia do root de staging.

A criação do backup deve aplicar a mesma disciplina aos arquivos administrados existentes. Arquivo inesperado/não canônico em `company/` ou `avatars/` causa falha explícita em vez de ser empacotado de forma ambígua.

Long paths continuam gate de ambiente/adapter; não devem ser “resolvidos” truncando nome.

---

## 7.6 Provenance do pacote

O manifesto atual possui versão, schema, origem e hashes, mas ainda não fixa identidade da implantação de origem.

Para reduzir risco de Restore acidental de pacote de outra implantação, propõe-se acrescentar:

```text
source_deployment_id
```

Regras:

- valor deriva da identidade não secreta da implantação vigente;
- faz parte do manifesto desde o `format_version` inicial;
- Restore normal exige correspondência com a implantação atual;
- Recovery local também exige correspondência no baseline;
- pacote de outra implantação é `source_mismatch`, mesmo que íntegro e tecnicamente compatível;
- migração/cópia intencional entre implantações exige contrato futuro próprio e não é inferida pelo executor.

`backup_id` continua sendo identidade do backup; `source_deployment_id` é provenance da implantação.

---

## 7.7 Limites estruturais contra pacote patológico

Mesmo com ZIP `Stored` no baseline, parser e extração precisam ser bounded.

Antes de materializar candidato:

- validar número máximo de entradas;
- validar tamanho individual declarado;
- validar total de bytes declarados;
- verificar espaço livre suficiente para staging/operação;
- limitar profundidade/comprimento lógico de path;
- recusar overflow aritmético ao somar tamanhos;
- nunca alocar buffer proporcional ao pacote inteiro quando streaming for suficiente;
- abortar ao exceder limite configurado.

Os **valores numéricos** desses limites ficam para Bloco 12/benchmark/fixtures. A existência dos limites é contrato do Bloco 11.

---

## 7.8 Criptografia e assinatura do pacote

A documentação anterior deixou “criptografia” como item não decidido. A validação final precisa impedir que o executor escolha silenciosamente uma política de chaves.

### Baseline proposta

O `.stepflow-backup` inicial **não usa criptografia nem assinatura criptográfica application-level**.

Motivos:

- introduzir senha/chave de backup exige lifecycle de chave, Recovery e rotação próprios;
- chave perdida poderia tornar disaster recovery impossível;
- o baseline Pocket já possui trust boundary local administrado;
- nenhuma exigência atual determina transporte hostil ou armazenamento em nuvem.

Proteção baseline:

- ACLs Windows da implantação;
- permissões do diretório `backups/`;
- proteção/criptografia de volume quando fornecida pela infraestrutura corporativa;
- `source_deployment_id`;
- SHA-256 para corrupção/integridade;
- trilha administrativa.

SHA-256 **não** vira assinatura/autenticidade. Se o threat model futuro exigir aceitar pacote de origem não confiável, criptografia/assinatura entram por novo contrato e `format_version`, com key management explícito.

---

## 7.9 Limite do disaster recovery local

O Backup StepFlow protege contra:

- corrupção lógica/arquivo;
- Restore de estado anterior;
- falha de migration/operação crítica quando houver backup apropriado;
- perda do `data/` enquanto `backups/` da implantação continuar disponível.

Ele **não é**, sozinho, proteção contra perda física total do volume/servidor, ransomware com acesso ao mesmo storage ou desastre de site.

A cópia/offsite/backup corporativo do diretório `backups/` pode ser feita pela infraestrutura existente, sem alterar o formato StepFlow. Integração automática com nuvem, NAS específico ou software de backup continua fora do baseline.

Isso é limitação operacional explícita, não bloqueador arquitetural do Bloco 11.

---

## 7.10 Auditoria após a captura do safety backup

Depois que o safety snapshot foi capturado e enquanto o barrier permanece fechado:

- não escrever novo evento funcional no `stepflow.sqlite` ativo;
- não alterar `company/**`/`avatars/**`;
- journal e `admin-audit` externos podem ser atualizados normalmente;
- qualquer auditoria funcional pós-Restore necessária deve ocorrer sobre o estado novo depois da ativação/validação, sem tornar token antigo reutilizável.

Isso evita que a própria auditoria quebre a equivalência entre safety backup e estado pré-destrutivo.

---

## 7.11 Parâmetros corretamente reservados ao Bloco 12

Não são bloqueadores da arquitetura e não devem ser inventados agora:

- `retention_max_confirmed_backups`;
- limites de tamanho total/por arquivo/número de entradas;
- espaço livre mínimo/margem;
- timeout de Backup/Restore;
- duração esperada de barrier/manutenção;
- backoff/reconexão;
- limiares de warning;
- política física de rotação dos logs/admin audit;
- versão exata pinada das crates/adapters;
- parâmetros finais de autenticação já pendentes em outro bloco.

O Bloco 12 deve transformar esses itens em configuração/defaults/fixtures mensuráveis antes da implementação correspondente.

---

## 7.12 Matriz mínima de validação executável futura

### Backup

- backup com zero arquivos administrados opcionais;
- backup com company/avatar;
- leituras concorrentes;
- mutações durante entrada no barrier;
- falha ao copiar arquivo administrado;
- falha/espaço durante staging;
- falha durante ZIP/finalização/sync;
- crash antes/depois da promoção final;
- catálogo reconstruindo backup promovido com resposta original perdida;
- retenção com backups protegidos.

### Restore

- pacote íntegro/schema igual;
- schema antigo com migration completa;
- schema antigo com cadeia incompleta;
- schema novo;
- hash incorreto;
- SQLite corrompido;
- foreign key inválida;
- `source_deployment_id` diferente;
- path traversal/UNC/device/ADS/nome reservado/case collision;
- pacote acima dos limites configurados;
- safety backup falhando;
- tentativa de mutação enquanto safety barrier está mantido;
- candidato alterado depois da validação inicial;
- cancelamento antes do primeiro rename;
- crash antes do primeiro rename;
- crash entre os dois renames;
- crash depois de `NEW_ACTIVATED`;
- rollback conhecido;
- estado `uncertain`;
- fresh Host + rejeição de token antigo.

### Disaster recovery

- banco ativo ilegível;
- `data/` incompleto;
- journal ativo reconciliável automaticamente;
- journal/artefatos ambíguos;
- pacote local íntegro;
- pacote externo copiado ao root mas de deployment diferente;
- falta de permissão/ACL;
- falha ao preservar `.recovery-old-*`;
- Recovery sem listener normal de rede;
- fresh Host após recovery.

### Windows corporativo

- NTFS/ReFS suportado quando aplicável;
- rename de diretórios same-volume/no-replace;
- long paths;
- EDR/antivírus interferindo em rename/flush;
- ACLs da implantação;
- espaço insuficiente;
- reboot/crash em pontos de journal;
- execução do Controller/Recovery sem elevação quando ACLs já permitem.

---

## 7.13 Gates de ambiente real

Permanecem gates antes de considerar Backup/Restore pronto para produção:

- comportamento do adapter Win32 para rename/journal/promoção;
- filesystem real do servidor;
- ACLs;
- EDR/antivírus;
- long paths;
- volume/disco e pressão de espaço;
- performance com base de tamanho representativo;
- crash/restart em pontos injetados de falha.

Falhar nesses testes volta para revisão técnica; não autoriza substituir o desenho por serviço/watchdog/instalador ou relaxar o contrato Pocket silenciosamente.

---

## 7.14 Propostas resultantes — P11.104 a P11.116

- **P11.104:** safety backup `pre_restore` mantém o barrier de mutações desde o ponto quiescente de captura até o primeiro rename; D11.18 continua válido para Backup normal, não para essa suboperação especial;
- **P11.105:** nenhuma mutação de negócio ou mutação interna em `data/` ocorre após a captura do safety backup; journal/admin-audit externos permanecem permitidos;
- **P11.106:** antes de `DESTRUCTIVE_STARTED`, recalcular/verificar o digest de `data-next/`; diferença aborta antes do primeiro rename e exige nova validação;
- **P11.107:** paths do pacote são materializados sob canonicalização Windows estrita, rejeitando drive/UNC/device/ADS/reserved names/trailing dot-space/case collision/non-regular entries e qualquer escape do root;
- **P11.108:** a criação do backup aplica as mesmas regras de path/tipo aos arquivos administrados existentes e falha explicitamente diante de entrada não canônica;
- **P11.109:** `manifest.json` inclui `source_deployment_id`; Restore/Recovery baseline bloqueiam pacote de implantação diferente com `source_mismatch`;
- **P11.110:** parser/extração possuem limites estruturais de entradas, tamanho, path e total de bytes, além de preflight de espaço; números finais ficam no Bloco 12;
- **P11.111:** pacote inicial não usa criptografia application-level; proteção baseline é ACL/infraestrutura de volume e qualquer criptografia futura exige key management + nova versão de formato;
- **P11.112:** pacote inicial não usa assinatura criptográfica; SHA-256 continua integridade/corrupção, não autenticidade; provenance baseline = root administrado + deployment ID + ACL + auditoria;
- **P11.113:** Backup/Recovery local não prometem sobrevivência a perda física total/ransomware/site loss; cópia/offsite corporativa de `backups/` é responsabilidade operacional externa ao baseline;
- **P11.114:** parâmetros numéricos de tamanho/espaço/timeout/retention/log rotation permanecem deliberadamente para Bloco 12 e não são escolha livre do executor;
- **P11.115:** adapter Windows de rename/promoção/journal e comportamento sob ACL/EDR/long paths/crash são gates obrigatórios de ambiente real antes de produção;
- **P11.116:** após incorporar P11.104–P11.115, não há bloqueador arquitetural conhecido para o Bloco 11; fechamento ainda depende de aprovação do PO, sincronização documental e gate Git normal.

## Referências técnicas verificadas

- SQLite Online Backup API: `https://www.sqlite.org/backup.html`
- SQLite PRAGMA `integrity_check` / `foreign_key_check`: `https://www.sqlite.org/pragma.html`
- rusqlite backup module: `https://docs.rs/rusqlite/latest/rusqlite/backup/`
- Rust `File::sync_all`: `https://doc.rust-lang.org/std/fs/struct.File.html#method.sync_all`
- Rust `zip` / `CompressionMethod::Stored`: `https://docs.rs/zip/latest/zip/enum.CompressionMethod.html`
- Windows `MoveFileExW`: `https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw`
- Windows naming/files/paths: `https://learn.microsoft.com/windows/win32/fileio/naming-a-file`

## Recomendação de fechamento

Aprovar **P11.104–P11.116**.

Após aprovação:

1. promover P11.104–P11.116 para D11.104–D11.116;
2. corrigir Análise 4 para explicitar a continuidade do safety barrier;
3. sincronizar manifesto/path/provenance/limites nas fontes arquiteturais;
4. remover pendências consumidas da Tela 13/modelo de dados/registro;
5. marcar Bloco 11 como tecnicamente validado;
6. atualizar README/plano/roadmap;
7. tornar PR #26 ready para revisão final do PO;
8. somente depois seguir o fluxo normal de squash merge/remoção de branch/verificação remota.
