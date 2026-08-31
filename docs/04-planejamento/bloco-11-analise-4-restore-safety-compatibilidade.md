# Bloco 11 — Análise 4 — Restore normal, safety backup e compatibilidade

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Data:** 2026-08-31

## Objetivo

Fechar o fluxo técnico do Restore normal pela UI: validação integral do pacote, regra de compatibilidade, preparação do estado restaurado, safety backup obrigatório e ponto exato em que a operação deixa de ser cancelável e passa a substituir o estado ativo.

Esta análise parte das Análises 1–3 aprovadas. Não altera a UX consolidada da Tela 13 nem o contrato Pocket.

## 4.1 Pré-condições

Antes de preparar um Restore, o Host deve:

- possuir o lease administrativo exclusivo `RESTORE`;
- validar novamente sessão/capacidade aplicável;
- proteger o backup de origem contra retenção durante a operação;
- não possuir Backup/Migration/Restore raiz concorrente;
- tratar o pacote selecionado como candidato, nunca como elegível apenas por aparecer no catálogo.

O catálogo pode acelerar a apresentação, mas **Restore sempre revalida integralmente o pacote**.

## 4.2 Pipeline pré-destrutivo

O estado ativo não é alterado durante esta fase.

```text
Restore confirmado pelo usuário
→ adquirir/confirmar lease RESTORE
→ revalidar pacote de origem
→ extrair para restore staging controlado
→ validar SQLite integralmente
→ avaliar compatibilidade
→ aplicar migrations forward no staging, se necessárias e suportadas
→ revalidar staging migrado
→ criar safety backup do estado ativo
→ confirmar safety backup
→ entrar em manutenção destrutiva
```

Qualquer falha anterior à manutenção destrutiva encerra o Restore sem alterar `data/` ativo.

## 4.3 Restore staging

Restore usa staging próprio, diferente do staging de criação de backup.

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

- `data-next/` representa o conjunto recuperável completo suportado pelo formato;
- staging de ativação fica no **mesmo volume** do `data/` ativo;
- não extrair diretamente sobre `data/`;
- paths continuam allowlisted/normalizados;
- não seguir reparse points, symlinks ou junctions;
- não confiar em timestamps/permissões externas como prova de integridade;
- staging nunca é servido aos Clients.

A localização física final pode variar, mas a implementação deve preservar same-volume para a troca de diretórios.

## 4.4 Revalidação integral do pacote

Antes de usar qualquer payload:

- ZIP/envelope deve abrir corretamente;
- `format_version` deve ser suportado;
- `backup_id` e manifesto devem ser coerentes;
- todas as entradas esperadas devem existir e nenhuma entrada não autorizada pode escapar do contrato da versão;
- tamanho e SHA-256 de **cada entrada** devem conferir;
- extração deve ser feita somente para paths controlados pelo Host.

Falha de envelope/hash torna o pacote `invalid_or_corrupt` e bloqueia Restore.

## 4.5 Validação SQLite pré-Restore

Sobre `data-next/stepflow.sqlite`, antes de qualquer troca destrutiva:

- abrir o banco isoladamente;
- confirmar identidade/schema esperado pelo StepFlow conforme mecanismo físico que o Bloco 12 materializar;
- executar `PRAGMA integrity_check` e exigir resultado `ok`;
- executar `PRAGMA foreign_key_check` e exigir zero violações;
- confirmar que a versão de schema/migration observada no banco corresponde ao manifesto.

`integrity_check` é deliberadamente mais forte aqui que o `quick_check` usado na criação do backup. O SQLite documenta que `integrity_check` não cobre erros de foreign key; por isso `foreign_key_check` é obrigatório separadamente.

## 4.6 Regra de compatibilidade

Compatibilidade é decidida pelo **Host atual**, não pelo filename nem somente por `source_app_version`.

Um pacote é elegível somente se:

1. o `format_version` do envelope é suportado pelo leitor atual; e
2. o banco é íntegro; e
3. existe compatibilidade de schema.

### Schema igual ao schema corrente

Elegível sem migration de Restore.

### Schema mais antigo

Elegível somente se o Host atual possuir uma **cadeia completa e determinística de migrations forward** desde o schema do backup até o schema corrente.

Nesse caso:

```text
extrair data-next
→ validar banco de origem
→ aplicar migrations forward em data-next
→ validar schema final
→ integrity_check
→ foreign_key_check
→ somente então permitir etapa destrutiva
```

Migration de Restore ocorre no staging, nunca no banco ativo.

Se uma migration futura também transformar arquivos administrados, ela deve operar sobre o `data-next/` inteiro e permanecer compatível com essa pipeline.

### Schema mais novo que o Host

**Incompatível.** O Host não tenta downgrade, down migration ou interpretação parcial.

### Cadeia incompleta/ambígua

**Incompatível.** Ausência de migration conhecida não autoriza improvisação.

### Versão StepFlow de origem

`source_app_version` é metadado importante para diagnóstico e regras explícitas de formato/capacidade, mas não substitui `format_version + schema/migration path` como critério técnico principal.

## 4.7 Proibição de down migration

Restore não executa down migration automática.

Se um backup possuir schema mais novo que o Host atual, as opções válidas são usar uma versão StepFlow capaz de compreendê-lo ou seguir procedimento controlado futuro. O Host atual não rebaixa o banco por conveniência.

Isso preserva a regra já consolidada de migrations publicadas imutáveis e rollback por backup compatível.

## 4.8 Safety backup obrigatório

Depois que o candidato restaurado estiver totalmente preparado/validado em staging e imediatamente antes da fase destrutiva, o Restore cria o safety backup do **estado ativo atual**.

```text
RESTORE lease
→ candidato preparado
→ suboperação BACKUP origin=system
→ reason=pre_restore
→ safety backup confirmado
→ somente então fase destrutiva
```

Regras:

- reutiliza exatamente a pipeline de Backup aprovada;
- não adquire segundo lease raiz;
- recebe `backup_id` próprio;
- deve ser pacote final confirmado e verificável;
- falha de captura, validação, promoção ou confirmação do safety backup bloqueia Restore;
- nenhum modo “continuar mesmo assim” existe no Restore normal pela UI.

## 4.9 Lifecycle do safety backup

O safety backup:

- fica protegido contra retenção enquanto o Restore está ativo;
- permanece protegido se o resultado ficar `uncertain`;
- após Restore concluído com sucesso e estado novo validado, perde a proteção operacional especial e permanece como backup `system` normal sujeito à retenção futura;
- não é apagado imediatamente após sucesso;
- falha do Restore antes da fase destrutiva pode deixar safety backup confirmado; ele continua sendo backup válido `system`, não resíduo parcial.

Isso preserva um ponto de retorno auditável sem criar categoria física de backup distinta.

## 4.10 Entrada em manutenção destrutiva

Depois de candidato preparado + safety backup confirmado, o Host entra no subestado destrutivo do Restore.

Antes da primeira troca física:

- parar aceitação de novas mutações;
- interromper/fechar novas operações read-only que dependam do estado ativo quando necessário;
- drenar operações já aceitas até ponto seguro;
- impedir novos artefatos/documentos baseados no estado antigo;
- fechar conexões SQLite e handles administrados que impeçam a troca;
- Clients recebem estado de manutenção/desconexão conforme contrato transversal;
- persistir marcador de operação fora de `data/` suficiente para recuperação após restart.

O formato/persistência exata desse marcador e a reconciliação de restart serão fechados na Análise 5.

## 4.11 Ponto de não cancelamento

O Restore permanece cancelável **até imediatamente antes da primeira alteração física do `data/` ativo**.

Após o Host iniciar o primeiro rename/move que retira `data/` de sua posição ativa:

- não existe cancelamento pelo usuário;
- a UI não oferece falso botão de cancelar;
- a operação precisa concluir, reverter tecnicamente ou entrar em estado `uncertain`.

Se o usuário cancelar antes desse ponto, `data/` permanece intocado. Safety backup já confirmado, se existir, permanece válido.

## 4.12 Troca do conjunto recuperável

Não copiar arquivos restaurados “por cima” do estado ativo.

A proposta é ativar o conjunto recuperável como unidade lógica:

```text
estado inicial
StepFlow\data\
StepFlow\.restore-staging\<id>\data-next\

fase destrutiva
1. data\      → .restore-old-<id>\
2. data-next\ → data\
3. abrir/validar novo data\
4. somente após confirmação, liberar cleanup do old
```

Regras:

- ambos os renames/moves ocorrem no mesmo volume;
- destinos existentes não são sobrescritos silenciosamente;
- `.restore-old-<id>/` permanece disponível durante a validação final;
- nenhuma resposta de sucesso ocorre entre os passos 1 e 4;
- configuração operacional, logs e `backups/` não participam dessa troca.

A operação em múltiplos renames não é tratada como atomicidade mágica. O marcador de Restore + `.restore-old-<id>` existem justamente para permitir reconciliação controlada após falha.

## 4.13 Validação pós-ativação

Depois que o novo `data/` estiver em posição ativa e antes de declarar sucesso:

- abrir `stepflow.sqlite` pelo caminho oficial;
- confirmar schema corrente esperado;
- executar `PRAGMA integrity_check` = `ok`;
- executar `PRAGMA foreign_key_check` = vazio;
- confirmar roots/arquivos administrados esperados;
- confirmar readiness mínima do estado persistente;
- confirmar que o Host consegue reconstruir projeções necessárias sem mutação inesperada.

Somente após isso o Restore pode ser marcado como tecnicamente aplicado.

Restart/sessões/reconexão e momento exato de voltar a aceitar Clients pertencem à Análise 5.

## 4.14 Falha durante ativação

### Antes do primeiro rename

- Restore falha/cancela;
- estado ativo permanece intocado.

### Depois do primeiro rename, com `.restore-old-<id>` íntegro

- Host tenta rollback local controlado para recolocar o `data/` anterior;
- se rollback for concluído e validado, Restore termina como falha conhecida, não como sucesso.

### Estado que não pode ser comprovado/revertido

- resultado = `uncertain`;
- não aceitar novas mutações;
- proteger backup de origem + safety backup + roots de recuperação;
- retenção/cleanup destrutivo permanece suspenso;
- reconciliação pertence à Análise 5/6.

Safety backup continua sendo a proteção durável; `.restore-old-<id>` é proteção operacional de curta duração para a própria troca.

## 4.15 Relação com filesystem Windows

A implementação futura deve encapsular renames/moves em adapter Windows e validar comportamento real no volume corporativo.

Baseline:

- same-volume;
- sem replace silencioso;
- nenhum `MOVEFILE_COPY_ALLOWED` como forma de fingir atomicidade entre volumes;
- erro de rename interrompe a sequência e entra na classificação de falha correspondente;
- gates de SMB/EDR/filesystem real permanecem para validação corporativa quando aplicáveis.

## 4.16 Propostas resultantes — P11.43 a P11.61

- **P11.43:** Restore sempre revalida integralmente envelope, paths, tamanhos e SHA-256, independentemente do cache do catálogo;
- **P11.44:** Restore extrai para `data-next/` controlado e same-volume com `data/`, nunca diretamente sobre o estado ativo;
- **P11.45:** pré-Restore exige `integrity_check = ok` + `foreign_key_check` vazio e coerência de schema com o manifesto;
- **P11.46:** compatibilidade usa `format_version` suportado + integridade + schema/migration path; versão textual do app não decide sozinha;
- **P11.47:** schema igual é elegível; schema mais antigo só é elegível com cadeia completa de migrations forward disponível;
- **P11.48:** migrations necessárias ao Restore são aplicadas no staging e revalidadas antes da fase destrutiva;
- **P11.49:** schema mais novo que o Host ou cadeia incompleta/ambígua é incompatível;
- **P11.50:** Restore não executa down migration automática;
- **P11.51:** safety backup é criado depois do candidato preparado e antes da fase destrutiva, reutilizando a pipeline sob o mesmo lease `RESTORE`;
- **P11.52:** safety backup deve estar confirmado; qualquer falha bloqueia o Restore normal;
- **P11.53:** safety backup permanece protegido durante Restore/uncertain e, após sucesso, vira backup `system` normal sujeito à retenção futura, sem exclusão imediata;
- **P11.54:** fase destrutiva só inicia após drenar operações, fechar handles necessários e persistir marcador de Restore fora de `data/`;
- **P11.55:** cancelamento é permitido até antes da primeira alteração física do `data/`; depois disso não existe cancelamento de usuário;
- **P11.56:** ativação usa troca lógica do conjunto `data/`, não overwrite arquivo a arquivo;
- **P11.57:** baseline de troca = `data → .restore-old-<id>` e `data-next → data`, no mesmo volume e sem replace silencioso;
- **P11.58:** `.restore-old-<id>` permanece até validação final do novo estado e serve como rollback operacional curto;
- **P11.59:** pós-ativação exige nova validação de SQLite/schema/files administrados antes de sucesso;
- **P11.60:** se rollback local puder restaurar e validar o estado anterior, Restore falha de forma conhecida; se não puder, resultado é `uncertain`;
- **P11.61:** detalhes de marcador persistente, restart, sessões, reconexão e resolução de `uncertain` ficam para Análises 5–6, sem reabrir as regras de segurança acima.

## Referências técnicas

- SQLite `PRAGMA integrity_check` / `foreign_key_check`: `https://www.sqlite.org/pragma.html`
- SQLite Online Backup API: `https://www.sqlite.org/backup.html`
- Windows `MoveFileExW`: `https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw`

## Próximo passo

Após aprovação de P11.43–P11.61, seguir para **Análise 5 — restart, sessões, reconexão, falhas e resultado incerto**.
