# Bloco 11 — Análise 3 — Catálogo, retenção e coordenação administrativa

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Data:** 2026-08-29

## Objetivo

Fechar como o Host descobre e classifica backups administrados, como limita crescimento do diretório de backups sem scheduler e como serializa operações administrativas críticas.

Esta análise parte das decisões já aprovadas nas Análises 1 e 2. Não altera a UX da Tela 13 nem o contrato Pocket.

## 3.1 Fonte de verdade do catálogo

O catálogo normal não depende do `stepflow.sqlite`, porque o próprio Restore substitui esse banco.

Fonte física de verdade proposta:

```text
backups/
├── *.stepflow-backup      ← candidatos finais
└── .staging/              ← nunca entra no catálogo
```

O Host deriva o catálogo dos pacotes finais administrados:

```text
enumerar candidatos finais
→ abrir envelope com parser seguro
→ ler manifest.json
→ obter backup_id/origem/data/schema/tamanho
→ verificar/classificar
→ expor projeção de catálogo ao Client
```

Regras:

- nenhum banco de catálogo persistente adicional é obrigatório;
- cache em memória por ciclo do Host é permitido;
- filename é apresentação/localização, nunca identidade canônica;
- `backup_id` do manifesto é a identidade lógica;
- `.staging` nunca é fonte de backup conhecido;
- arquivo desconhecido não é interpretado por extensão arbitrária;
- reparse point/symlink/junction não é seguido durante enumeração;
- pacote final colocado localmente no diretório administrado só pode aparecer após passar pelo mesmo parser/classificador; isso não cria upload/importação pela UX.

Motivo: o catálogo precisa poder ser reconstruído mesmo quando o banco ativo está ausente, foi restaurado ou ainda não abriu.

## 3.2 Estados de catálogo

A projeção técnica deve permitir pelo menos:

- `verifying`;
- `valid`;
- `invalid_or_corrupt`;
- `incompatible`;
- `temporarily_unavailable`;
- `verification_failed`.

A UI traduz esses estados conforme a Tela 13.

Compatibilidade é diferente de integridade:

- pacote pode ser íntegro e incompatível com a versão/schema atual;
- pacote corrompido nunca é elegível para Restore;
- decisão exata de compatibilidade será fechada na Análise 4.

## 3.3 Verificação e cache

Listar backups não deve recalcular SHA-256 de todos os pacotes em toda requisição.

Proposta:

- Host mantém cache de verificação somente em memória;
- tamanho/mtime podem ser usados apenas como **sinal para invalidar cache**, nunca como prova de integridade;
- mudança observada no arquivo invalida a verificação anterior;
- verificação pesada ocorre de forma bounded, priorizando backups mais recentes quando necessário;
- refresh pode expor temporariamente `verifying`;
- Restore sempre revalida o pacote integralmente, independentemente do cache da sessão.

O catálogo persistente continua reconstruível somente a partir dos pacotes.

## 3.4 Reconciliação no startup

Depois de adquirir a exclusividade da instância sobre os mesmos dados, o Host pode reconciliar `backups/`:

1. ignorar `.staging` como fonte válida;
2. tratar staging herdado de processo anterior como órfão de operação interrompida;
3. realizar cleanup best-effort/conservador desses resíduos sem sair do root administrado;
4. enumerar pacotes finais;
5. classificar envelope/manifesto;
6. iniciar verificação bounded quando aplicável;
7. aplicar retenção somente se não existir operação administrativa crítica ou resultado de Restore pendente/incerto.

Pacote final inválido/corrompido **não é apagado automaticamente** durante reconciliação; permanece classificável para diagnóstico.

Arquivo final desconhecido que não pertença ao formato suportado é ignorado pela UX normal e registrado apenas quando relevante.

## 3.5 Retenção inicial — princípio

Não haverá scheduler periódico. Retenção é housekeeping acionado por eventos do próprio Host:

- após um novo backup ter sido confirmado com sucesso;
- na reconciliação de startup quando seguro.

A política inicial proposta é **por quantidade de backups confirmados**, não por idade.

Motivos:

- Host é Pocket e pode ficar desligado por períodos longos;
- não existe periodicidade garantida de criação de backup;
- política por dias/semanas criaria comportamento pouco previsível sem scheduler;
- quantidade é determinística e simples de auditar.

## 3.6 Parâmetro de retenção

O algoritmo usa um parâmetro central:

```text
retention_max_confirmed_backups
```

O valor numérico final pertence ao fechamento de parâmetros do Bloco 12, junto aos demais números ainda deliberadamente não congelados.

Regras já fecháveis no Bloco 11:

- valor precisa ser positivo e validado;
- configuração pertence à máquina central, sem nova tela nesta fase;
- mudança do limite não apaga pacote no momento da edição por inferência; retenção ocorre no próximo ciclo seguro de housekeeping;
- falta/invalidade da configuração usa default oficial a ser fixado no Bloco 12, nunca valor improvisado pelo executor.

## 3.7 Algoritmo de retenção

Depois de um backup novo estar **confirmado**:

```text
listar backups finais confirmados
→ excluir do conjunto qualquer pacote protegido pela operação atual
→ ordenar elegíveis por created_at + backup_id determinístico
→ enquanto quantidade > limite
     remover o elegível mais antigo
→ atualizar catálogo
```

Proteções obrigatórias:

- nunca apagar backup antes de o novo backup estar confirmado apenas para “abrir espaço”;
- nunca apagar pacote usado como origem de Restore enquanto a operação estiver ativa;
- nunca apagar o safety backup enquanto o Restore correspondente estiver ativo ou com resultado incerto;
- nunca apagar pacote ligado a migration administrativa ainda não concluída;
- retenção não roda em paralelo com Restore/migration;
- pacote inválido/corrompido não é apagado silenciosamente pela retenção automática;
- falha ao remover backup antigo **não transforma** o backup recém-criado em falha; gera warning administrativo/auditoria e pode deixar quantidade acima do limite.

Compatibilidade atual não determina sozinha exclusão: backup íntegro mas incompatível pode ser útil em rollback/disaster recovery com versão adequada e participa da política normal de antiguidade.

## 3.8 Espaço insuficiente

A política de segurança prefere falhar ao criar um novo backup a destruir antecipadamente um ponto de recuperação existente.

Portanto:

- retenção normal não remove backups antigos **antes** da confirmação do novo apenas para liberar espaço;
- se não houver espaço suficiente para staging/pacote, a criação falha de forma explícita;
- nenhum backup confirmado existente é sacrificado automaticamente como tentativa de recuperação de espaço;
- alertas/diagnóstico de espaço pertencem à operação administrativa, sem criar gerenciador de armazenamento completo.

Eventual política futura de quota/limpeza antecipada exige nova decisão explícita.

## 3.9 Coordinator administrativo do Host

Proposta: um coordenador lógico exclusivo para operações administrativas críticas sobre o estado persistente.

Tipos iniciais:

```text
BACKUP
RESTORE
MIGRATION
```

O coordinator fornece um **lease exclusivo** por operação raiz.

Regras:

- apenas uma operação raiz crítica ativa por vez;
- segundo pedido incompatível recebe resultado semântico `ADMIN_OPERATION_IN_PROGRESS`/equivalente;
- o coordinator não substitui o writer lógico: ele coordena operações de nível superior;
- operações normais read-only não adquirem o lease;
- Backup usa o lease durante sua operação completa, mas só o subestado `BACKUP_CAPTURE` aciona o barrier curto de mutações;
- Restore mantém o lease durante validação, safety backup, fase destrutiva e conclusão;
- migration de startup usa o mesmo modelo antes de readiness quando aplicável.

## 3.10 Suboperações sem deadlock

Safety backup e pre-migration backup são suboperações da operação raiz que já possui o lease:

```text
RESTORE lease
→ criar safety backup interno
→ continuar Restore
```

```text
MIGRATION lease
→ criar backup de segurança quando exigido
→ continuar migration
```

Elas **não tentam adquirir um segundo lease raiz**.

O mecanismo de criação reutiliza a pipeline de Backup, mas recebe contexto administrativo interno (`origin = system` + motivo) e respeita proteções da operação pai.

## 3.11 Coordenação com operações normais

### Durante Backup

- consultas podem continuar;
- geração documental read-only pode continuar quando segura;
- mutações só sofrem indisponibilidade durante `BACKUP_CAPTURE`;
- depois que o snapshot bruto está em staging, mutações voltam ao normal enquanto hash/ZIP/verificação/promoção continuam;
- outro Backup/Restore/migration não começa até o lease ser liberado.

### Durante Restore

A política detalhada de manutenção, desconexão, restart e sessões pertence às Análises 4 e 5. Nesta análise fica fechado apenas que Restore é operação administrativa exclusiva e bloqueia operações incompatíveis.

### Durante Migration

Migration ocorre sob exclusividade administrativa e antes de readiness quando aplicável. Se exigir backup prévio, usa a mesma pipeline consistente aprovada.

## 3.12 Eventos e consulta de estado

O coordinator expõe estado consultável pelo Host/API, suficiente para a UX mostrar operação em andamento e fazer refetch.

Eventos podem sinalizar transições, mas resultado definitivo continua vindo de consulta ao estado confirmado.

Não persistir segredo, token reutilizável ou conteúdo de backup no estado de operação.

## 3.13 Resultado incerto e retenção

Se uma operação Restore terminar em estado `uncertain`:

- retention/cleanup destrutivo de backups fica suspenso;
- safety backup e origem do Restore permanecem protegidos;
- nova operação destrutiva é bloqueada até diagnóstico/recuperação conforme contrato das Análises 5/6;
- detalhes de persistência desse estado através de restart serão fechados junto ao contrato de falhas/recovery.

Essa suspensão pode fazer a quantidade física ultrapassar temporariamente o limite de retenção; segurança prevalece sobre o cap.

## 3.14 Propostas resultantes — P11.26 a P11.42

- **P11.26:** catálogo é reconstruível a partir dos pacotes finais; não depende do `stepflow.sqlite` nem de banco de catálogo adicional;
- **P11.27:** `backup_id` do manifesto é identidade; filename não é identidade canônica;
- **P11.28:** `.staging` nunca entra no catálogo; staging herdado de processo anterior é órfão elegível a cleanup conservador após exclusividade da instância;
- **P11.29:** catálogo distingue integridade de compatibilidade e suporta estados de verificação/invalidade/incompatibilidade;
- **P11.30:** cache de verificação é somente em memória; metadados de filesystem servem apenas para invalidar cache;
- **P11.31:** Restore sempre revalida integralmente o pacote, mesmo que o catálogo o tenha marcado como válido;
- **P11.32:** retenção não usa scheduler e roda após backup confirmado e no startup seguro;
- **P11.33:** política inicial de retenção é por quantidade, não por idade;
- **P11.34:** `retention_max_confirmed_backups` é parâmetro central; valor numérico/default final fica para o Bloco 12;
- **P11.35:** retenção nunca apaga backup antigo antes da confirmação do novo só para liberar espaço;
- **P11.36:** source/safety/pre-migration em uso ou resultado incerto ficam protegidos;
- **P11.37:** pacote inválido/corrompido não é apagado silenciosamente por retenção automática;
- **P11.38:** falha de cleanup/retenção após backup confirmado gera warning e pode exceder temporariamente o limite, sem rebaixar o backup novo;
- **P11.39:** Host possui coordinator/lease exclusivo para `BACKUP`, `RESTORE` e `MIGRATION`;
- **P11.40:** safety backup e pre-migration backup são suboperações da operação raiz, reutilizando pipeline sem adquirir novo lease;
- **P11.41:** Backup mantém lease administrativo durante a operação, mas barrier de mutações só existe no subestado curto de captura;
- **P11.42:** Restore/migration bloqueiam operações administrativas incompatíveis; detalhes de manutenção/restart/sessões ficam para análises seguintes.

## Pendências deliberadas para análises seguintes

- algoritmo exato de compatibilidade por `format_version`, versão StepFlow e schema/migrations;
- proteção/persistência de estado `uncertain` através de restart;
- lifecycle exato do safety backup após Restore concluído;
- restart/reconexão/sessões;
- disaster recovery quando Host não sobe;
- Gerência × Backup;
- valor numérico final de retenção no Bloco 12.

## Próximo passo

Após aprovação de P11.26–P11.42, seguir para **Análise 4 — Restore normal + safety backup + compatibilidade**.
