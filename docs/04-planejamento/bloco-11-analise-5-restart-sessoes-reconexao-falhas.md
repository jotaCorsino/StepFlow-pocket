# Bloco 11 — Análise 5 — Restart, sessões, reconexão e falhas

**Status:** APROVADA PELO PO EM 2026-08-31  
**Bloco:** 11 — Backup / Restauração técnico  
**Data:** 2026-08-31

## Objetivo

Fechar a continuidade técnica do Restore quando o processo Host é reiniciado ou interrompido: marcador persistente, reconciliação do filesystem, reinicialização controlada, invalidação de sessões, reconexão dos Clients e classificação determinística entre sucesso, rollback conhecido e resultado `uncertain`.

Esta análise parte das Análises 1–4 aprovadas. Não altera a UX da Tela 13 nem o contrato Pocket.

## 5.1 Princípio de recuperação

O Restore não depende da memória do processo para saber em que estado está.

Antes da primeira alteração física do `data/`, o Host persiste estado operacional mínimo fora de `data/`. Se o Host cair, o próximo processo deve reconciliar o que realmente existe no filesystem antes de abrir o sistema para uso normal.

Regras:

- marcador é evidência de intenção/fase, não prova isolada de que um rename ocorreu;
- disposição real de `data/`, `data-next/` e `.restore-old-<id>/` também é observada;
- candidato preparado possui digest determinístico registrado no marcador;
- nenhuma inferência ambígua autoriza mutações normais;
- estado não comprovável entra em `uncertain`.

## 5.2 Local do journal operacional

Baseline conceitual:

```text
StepFlow\
├── data\
├── backups\
│   └── .operations\
│       ├── restore-active.json
│       └── restore-last.json
├── logs\
└── .restore-staging\
```

`backups/.operations/`:

- não entra no catálogo de `.stepflow-backup`;
- não entra em retenção de backups;
- não faz parte do payload restaurável;
- permanece fora de `data/` para sobreviver à troca;
- é administrado somente pelo Host/Controller;
- não contém senha, token reutilizável ou conteúdo de negócio.

O path físico final pode ser encapsulado em configuração interna, mas precisa permanecer no mesmo deployment e fora do conjunto substituído pelo Restore.

## 5.3 Conteúdo mínimo de `restore-active.json`

Registrar pelo menos:

- `journal_format_version`;
- `operation_id`;
- `source_backup_id`;
- `safety_backup_id` após confirmação;
- `phase`;
- `phase_seq` monotônico;
- `started_at` UTC;
- paths lógicos controlados da operação;
- schema esperado do candidato final;
- digest determinístico do `data-next/` preparado após migrations;
- flags técnicas estritamente necessárias à reconciliação.

Não registrar:

- token de sessão;
- senha;
- conteúdo integral de manifesto/payload;
- paths arbitrários fornecidos pelo Client.

## 5.4 Escrita do journal

Mudança de fase crítica usa escrita substitutiva controlada:

```text
montar novo journal
→ escrever arquivo temporário no mesmo diretório
→ flush/sync do temporário
→ promover/substituir restore-active.json por adapter de filesystem
→ somente então executar a ação física correspondente
```

A implementação futura deve validar a primitive Windows escolhida e sua durabilidade no filesystem real. `sync`/rename reduzem risco, mas não justificam promessa de resistência absoluta a qualquer falha de hardware.

Journal ausente, corrompido ou incompatível na presença de artefatos de Restore é condição de recuperação, não autorização para cleanup destrutivo.

## 5.5 Fases persistentes

Fases conceituais mínimas:

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

A implementação pode agrupar estados não críticos, mas não pode perder as fronteiras necessárias para recovery.

Regras:

- `PREPARED`: candidato e safety backup confirmados; `data/` ainda não alterado;
- `DESTRUCTIVE_STARTED`: usuário já não pode cancelar; troca física será/está sendo executada;
- `OLD_MOVED`: `data/` anterior foi retirado da posição ativa;
- `NEW_ACTIVATED`: candidato foi colocado como novo `data/`;
- `VALIDATED`: novo estado passou na validação pós-ativação;
- `RESTART_REQUIRED`: estado escolhido foi validado e o processo precisa reinicialização limpa;
- `COMPLETED`: fresh Host confirmou estado restaurado e readiness aplicável;
- `ROLLED_BACK`: estado anterior foi recolocado e validado após falha do Restore;
- `UNCERTAIN`: não foi possível provar/concluir/reverter deterministicamente.

## 5.6 Digest do candidato

Depois de migrations e validação de `data-next/`, antes da fase destrutiva, calcular uma identidade determinística do conjunto preparado.

Pode ser uma raiz/hash derivada da lista ordenada de paths allowlisted + tamanho + SHA-256 de cada arquivo após transformação/migration.

Objetivo:

- permitir ao startup distinguir se `data/` atual corresponde ao candidato preparado;
- não depender somente de nomes de diretório ou timestamps;
- detectar alteração externa entre preparação e reconciliação.

Esse digest não substitui `integrity_check`/`foreign_key_check`; é identidade operacional do conjunto físico preparado.

## 5.7 Ordem no startup

Ao iniciar, o Host/Controller segue esta precedência:

```text
resolver deployment + adquirir exclusividade da instância
→ verificar restore-active.json e artefatos de Restore
→ se houver recovery pendente, reconciliar Restore
→ somente após resolução conhecida abrir fluxo normal de migrations/readiness
→ reconstruir catálogo/caches
→ aceitar login/uso normal
```

Enquanto a reconciliação estiver pendente:

- não aplicar migration normal por conveniência sobre estado ainda ambíguo;
- não aceitar mutações;
- não anunciar readiness normal;
- não executar retenção/cleanup destrutivo;
- preservar source backup, safety backup, staging e old necessários ao diagnóstico.

## 5.8 Matriz de reconciliação do filesystem

### Caso A — `data/` existe, `data-next/` existe, `old/` não existe

Interpretação baseline: primeira troca física não ocorreu.

Ações:

- validar `data/` ativo;
- se válido, classificar Restore como interrompido antes da troca;
- preservar/limpar `data-next/` somente após classificação conhecida;
- não aplicar candidato;
- Restore termina como falha conhecida, estado original preservado.

### Caso B — `data/` ausente, `old/` existe, `data-next/` existe

Interpretação baseline: queda entre `data → old` e `data-next → data`.

Política: **rollback para o estado anterior**, não completar automaticamente o Restore.

```text
old → data
→ validar estado anterior
→ se válido: ROLLED_BACK
→ se falhar: UNCERTAIN
```

Motivo: o candidato ainda não entrou na posição ativa; restaurar o estado previamente ativo minimiza mudança após falha intermediária.

### Caso C — `data/` existe, `old/` existe, `data-next/` ausente

Possível estado depois da ativação do candidato.

- recalcular digest controlado de `data/`;
- comparar com digest do candidato registrado;
- executar validação SQLite/files;
- se corresponder e for válido: continuar finalização do Restore;
- se não corresponder, tentar rollback controlado com `old/`;
- se nenhum estado puder ser comprovado: `UNCERTAIN`.

### Caso D — `data/` existe, `old/` ausente, `data-next/` ausente

Se journal indica fase terminal pós-validação/restart:

- validar `data/`;
- confirmar digest do candidato quando Restore deveria ter sido aplicado;
- finalizar `COMPLETED` se tudo corresponder.

Se journal/fase não sustentar esse estado, classificar como `UNCERTAIN` em vez de inventar conclusão.

### Caso E — combinações adicionais/inesperadas

Exemplos: múltiplos `old`, `data/` ausente sem `old`, journal incompatível, paths alterados manualmente.

- nenhuma escolha destrutiva automática;
- classificar `UNCERTAIN`/`RECOVERY_REQUIRED`;
- preservar artefatos;
- bloquear readiness normal;
- encaminhar para disaster recovery da Análise 6.

## 5.9 Orfandade sem journal

Se houver `.restore-old-*` ou restore staging relevante sem `restore-active.json` válido:

- não apagar automaticamente;
- validar `data/` ativo quando existir;
- registrar anomalia;
- se não for possível provar que o estado ativo é seguro e único, entrar em `RECOVERY_REQUIRED`;
- resolução manual/controlada pertence à Análise 6.

A ausência do journal não transforma artefato desconhecido em lixo seguro.

## 5.10 Reinicialização controlada do Host

Após Restore aplicado e validado, ou após rollback técnico depois de a fase destrutiva ter começado, o processo Host deve passar por **reinicialização controlada** antes de voltar ao uso normal.

Razões:

- reabrir SQLite a partir do path oficial;
- descartar pools/conexões anteriores;
- descartar caches/projeções em memória;
- recriar WebSockets e estado runtime;
- estabelecer fronteira inequívoca de sessão.

Baseline:

```text
estado físico escolhido validado
→ persistir RESTART_REQUIRED
→ encerrar listeners/WebSockets de forma coordenada
→ fechar SQLite/handles
→ Host sai com motivo/exit code controlado
→ Controller relança um Host fresco
→ fresh Host reconcilia journal antes de readiness
```

Isso não cria Windows Service, watchdog ou daemon. O Controller já está ativo no ciclo central e executa uma transição bounded da operação Restore.

## 5.11 Relaunch bounded pelo Controller

O Controller pode relançar o Host para recovery em duas situações:

1. saída controlada `RESTORE_RESTART_REQUIRED`/equivalente;
2. queda inesperada enquanto existe `restore-active.json` válido indicando Restore em andamento.

Regras:

- relaunch é bounded; não criar loop infinito;
- uma tentativa de recovery que também falhe encerra o ciclo automático e exige intervenção local/controlada;
- não transformar isso em watchdog geral para crashes normais;
- ausência de Restore ativo não autoriza restart automático ilimitado do Host;
- no próximo início manual do Controller, a mesma reconciliação de journal continua disponível.

O número exato de tentativas além do baseline mínimo não é parâmetro a improvisar; a primeira implementação deve privilegiar uma única recuperação automática controlada e falhar fechado se ela não conseguir atingir estado conhecido.

## 5.12 Sessões após Restore

Qualquer Restore que **entre na fase destrutiva** cria uma fronteira de segurança de sessão.

Regra proposta:

- todos os tokens/sessões existentes antes dessa fronteira tornam-se inválidos;
- isso vale tanto para Restore concluído quanto para rollback após fase destrutiva;
- nenhum token do estado anterior ou restaurado pode ser “ressuscitado” pelo conteúdo do backup;
- após fresh Host ficar ready, usuário precisa autenticar novamente.

A implementação física pode manter sessões somente em memória ou usar mecanismo equivalente, mas deve garantir que token pré-Restore nunca volte a ser aceito por ter reaparecido em um banco restaurado.

Restore que falha/cancela antes da fase destrutiva não exige revogação global apenas por ter preparado staging.

## 5.13 Relação com sessão server-side

O contrato existente continua:

- token opaco;
- validação Host-side;
- Client guarda token somente em memória.

A Análise 5 acrescenta uma regra específica de Restore: a transição destrutiva invalida a geração de sessões anterior.

Se a implementação futura persistir metadados de sessão por qualquer motivo, eles não podem tornar reutilizável um token restaurado de snapshot antigo. A estrutura física deve possuir revogação/epoch/armazenamento runtime equivalente que satisfaça essa regra.

## 5.14 WebSocket e Clients conectados

Antes da troca física, o Host tenta emitir estado/evento de manutenção aos Clients autorizados quando o canal ainda estiver disponível.

Depois:

- WebSockets são encerrados como parte da manutenção/restart;
- queda de conexão durante Restore não é interpretada como sucesso nem falha;
- Client entra no estado transversal de manutenção/reconexão;
- tentativas de reconexão usam backoff bounded vigente;
- enquanto recovery não termina, não há readiness normal.

Evento é best-effort; a segurança não depende de todos os Clients receberem o aviso antes da desconexão.

## 5.15 Reconexão após fresh Host

Quando o fresh Host atinge readiness:

```text
Client reconecta
→ revalida compatibilidade/deployment
→ token pré-Restore é rejeitado
→ Client encaminha para Login
→ usuário autentica novamente
→ Client refaz consultas do estado atual
```

Mensagem genérica antes do login pode indicar que o StepFlow foi reiniciado após manutenção. Detalhes de backup/ator não são expostos sem autorização.

O Client que iniciou o Restore pode manter `operation_id` somente em memória e, após nova autenticação autorizada, consultar o resultado confirmado da operação.

## 5.16 Resultado persistente para reconsulta

`restore-last.json` fornece continuidade operacional mínima entre o processo antigo e o fresh Host.

Conteúdo permitido:

- `operation_id`;
- estado terminal (`completed`, `rolled_back`, `failed_pre_destructive`, `uncertain`);
- `source_backup_id`;
- `safety_backup_id` quando houver;
- timestamps;
- código técnico resumido/sanitizado.

Regras:

- não contém token/senha/conteúdo de negócio;
- não substitui auditoria histórica;
- acesso via API continua autorizado;
- pode ser sobrescrito pela próxima operação Restore terminal após o resultado anterior já estar seguro/auditado;
- forma histórica/auditoria completa será fechada na Análise 6.

## 5.17 Quando remover `restore-active.json`

O active journal não é removido apenas porque um rename terminou.

Remoção somente após:

### Restore concluído

- fresh Host abriu e validou o `data/` escolhido;
- schema/readiness de persistência estão corretos;
- estado terminal foi persistido em `restore-last.json`/registro equivalente;
- proteção necessária do `old/` foi resolvida;
- resultado está consultável.

### Rollback conhecido

- `old/` foi recolocado como `data/`;
- estado anterior foi validado;
- fresh Host passou pela reinicialização limpa;
- resultado terminal `rolled_back` foi persistido.

### `uncertain`

- `restore-active.json` **não é apagado** automaticamente;
- permanece evidência de recovery até resolução controlada.

## 5.18 Cleanup de `old/` e staging

- `old/` nunca é removido antes de o estado novo ter sido validado e o journal registrar fase segura;
- após `COMPLETED`, cleanup de `old/` é best-effort e pode ocorrer após persistência do resultado terminal;
- falha de cleanup não rebaixa Restore concluído, mas gera warning e preserva diretório para housekeeping posterior;
- após `ROLLED_BACK`, `data-next/`/staging pode ser limpo conservadoramente;
- `UNCERTAIN` suspende cleanup de todos os artefatos relevantes.

Nenhum cleanup segue reparse points ou sai dos roots controlados.

## 5.19 Taxonomia de resultado do Restore

Estados externos mínimos:

- `preparing`;
- `maintenance`;
- `completed`;
- `failed_pre_destructive`;
- `rolled_back`;
- `uncertain`.

Semântica:

- `failed_pre_destructive`: estado ativo nunca foi alterado;
- `rolled_back`: fase destrutiva começou, mas estado anterior foi restaurado e validado;
- `completed`: candidato restaurado foi ativado, validado e confirmado pelo fresh Host;
- `uncertain`: não existe prova suficiente para afirmar qual estado está seguro.

`uncertain` nunca é convertido em sucesso por timeout, existência de filename ou disponibilidade parcial da API.

## 5.20 Comportamento em `uncertain`

Enquanto `uncertain`/`RECOVERY_REQUIRED`:

- normal readiness fica bloqueado;
- nenhuma mutação de negócio é aceita;
- nenhuma nova operação destrutiva normal é iniciada;
- retenção e cleanup destrutivo permanecem suspensos;
- source backup, safety backup, journal, old e staging relevantes ficam protegidos;
- logs registram o diagnóstico sanitizado;
- recuperação local/controlada segue Análise 6.

Pode existir health/status mínimo suficiente ao Controller para informar que recuperação é necessária, sem expor dados do negócio.

## 5.21 Falhas do Controller

Se Controller também for encerrado durante Restore:

- nenhuma garantia depende de ele permanecer vivo;
- journal e filesystem preservam informação para o próximo ciclo;
- ao próximo início manual, Controller/Host executam a reconciliação antes de readiness;
- não instalar serviço ou tarefa agendada para “vigiar” a recuperação.

Isso preserva o contrato Pocket.

## 5.22 Decisões aprovadas — D11.62 a D11.82

- **D11.62:** Restore persiste journal operacional fora de `data/` antes da troca física; memória do processo não é fonte suficiente de recovery;
- **D11.63:** baseline do journal = `backups/.operations/restore-active.json`; ele não entra em catálogo, retenção ou payload;
- **D11.64:** journal registra operation/source/safety IDs, fase sequencial, schema e digest do candidato, sem segredos;
- **D11.65:** atualizações críticas do journal usam temp + flush/sync + promoção controlada antes da ação física correspondente;
- **D11.66:** fresh Host reconcilia Restore antes de migrations/readiness normais;
- **D11.67:** candidato preparado recebe digest determinístico do conjunto `data-next/` para identificação após restart;
- **D11.68:** queda antes do primeiro rename preserva `data/` original e termina como falha conhecida;
- **D11.69:** queda entre `data→old` e `data-next→data` causa rollback automático para `old`, não conclusão automática do Restore;
- **D11.70:** se `data/` ativo corresponde ao digest do candidato e valida, fresh Host pode continuar finalização; caso contrário tenta rollback conhecido ou entra em `uncertain`;
- **D11.71:** combinação de filesystem/journal não comprovável, journal inválido com artefatos relevantes ou interferência externa leva a `RECOVERY_REQUIRED/uncertain`;
- **D11.72:** Restore aplicado ou rollback após fase destrutiva exige reinicialização controlada do Host antes de readiness normal;
- **D11.73:** Controller pode relançar bounded um Host de recovery por saída controlada ou queda com journal ativo; isso não é watchdog geral nem loop infinito;
- **D11.74:** qualquer Restore que entre na fase destrutiva invalida todas as sessões/tokens pré-existentes, inclusive quando termina em rollback;
- **D11.75:** conteúdo restaurado nunca pode ressuscitar token reutilizável de sessão antiga;
- **D11.76:** WebSocket/evento de manutenção é best-effort; desconexão não indica resultado e Clients reconsultam após reconexão;
- **D11.77:** fresh Host rejeita tokens pré-Restore e exige novo login antes do uso normal;
- **D11.78:** `restore-last.json`/equivalente preserva resultado terminal mínimo para reconsulta pós-restart, sem substituir auditoria;
- **D11.79:** `restore-active.json` só é removido após fresh Host confirmar estado conhecido e resultado terminal consultável; em `uncertain`, permanece;
- **D11.80:** cleanup de `old`/staging é posterior e best-effort; falha de cleanup não rebaixa `completed`, mas `uncertain` suspende cleanup;
- **D11.81:** taxonomia mínima = `preparing`, `maintenance`, `completed`, `failed_pre_destructive`, `rolled_back`, `uncertain`;
- **D11.82:** `uncertain/RECOVERY_REQUIRED` bloqueia readiness normal, mutações, nova operação destrutiva, retenção e cleanup até recuperação controlada.

## Pendências para Análise 6

- procedimento local/controlado de disaster recovery quando Host não consegue atingir readiness;
- escolha segura entre source backup, safety backup, `old` e candidato em recovery manual;
- capacidades finais, especialmente Gerência × Backup;
- auditoria completa de Backup/Restore atravessando Restore;
- logs/evidências e fronteira de suporte;
- tratamento de perda física do `data/` sem Restore normal disponível.

## Próximo passo

Seguir para **Análise 6 — disaster recovery, capacidades e auditoria**.