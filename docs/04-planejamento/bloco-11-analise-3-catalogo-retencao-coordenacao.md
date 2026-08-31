# Bloco 11 — Análise 3 — Catálogo, retenção e coordenação administrativa

**Status:** APROVADA PELO PO  
**Bloco:** 11 — Backup / Restauração técnico  
**Data da aprovação:** 2026-08-31

## Objetivo

Fechar como o Host descobre e classifica backups administrados, como limita crescimento do diretório de backups sem scheduler e como serializa operações administrativas críticas.

Esta análise parte das decisões aprovadas nas Análises 1 e 2. Não altera a UX da Tela 13 nem o contrato Pocket.

## 3.1 Fonte de verdade do catálogo

O catálogo normal não depende do `stepflow.sqlite`, porque o próprio Restore substitui esse banco.

Fonte física de verdade:

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
- pacote colocado localmente no diretório administrado só aparece após passar pelo mesmo parser/classificador; isso não cria upload/importação pela UX.

## 3.2 Estados de catálogo

A projeção técnica suporta pelo menos:

- `verifying`;
- `valid`;
- `invalid_or_corrupt`;
- `incompatible`;
- `temporarily_unavailable`;
- `verification_failed`.

Integridade e compatibilidade são estados distintos. Pacote íntegro pode ser incompatível; pacote corrompido nunca é elegível para Restore.

## 3.3 Verificação e cache

- cache de verificação é somente em memória;
- tamanho/mtime servem apenas para invalidar cache, nunca provar integridade;
- mudança observada invalida verificação anterior;
- verificação pesada é bounded;
- refresh pode expor `verifying`;
- Restore sempre revalida integralmente o pacote.

## 3.4 Reconciliação no startup

Depois de adquirir exclusividade da instância sobre os dados, o Host pode:

1. ignorar `.staging` como fonte válida;
2. tratar staging herdado como órfão de operação interrompida;
3. executar cleanup best-effort/conservador sem sair do root administrado;
4. enumerar pacotes finais;
5. classificar envelope/manifesto;
6. iniciar verificação bounded;
7. aplicar retenção somente quando não houver operação crítica ou resultado de Restore pendente/incerto.

Pacote final inválido/corrompido não é apagado automaticamente durante reconciliação.

## 3.5 Retenção inicial

Não existe scheduler periódico. Retenção é housekeeping acionado:

- após novo backup confirmado;
- no startup seguro.

A política inicial é **por quantidade de backups confirmados**, não por idade.

Motivos:

- Host Pocket pode permanecer desligado por períodos longos;
- não há periodicidade garantida de backup;
- quantidade é determinística e simples de auditar.

## 3.6 Parâmetro de retenção

O algoritmo usa:

```text
retention_max_confirmed_backups
```

O valor/default numérico final pertence ao Bloco 12.

Regras:

- valor positivo e validado;
- configuração central, sem nova tela nesta fase;
- mudança de limite só repercute no próximo housekeeping seguro;
- configuração ausente/inválida usa default oficial futuro, nunca valor improvisado.

## 3.7 Algoritmo de retenção

Após um backup novo estar confirmado:

```text
listar backups finais confirmados
→ retirar do conjunto os protegidos
→ ordenar elegíveis por created_at + backup_id
→ enquanto quantidade > limite
     remover o elegível mais antigo
→ atualizar catálogo
```

Proteções:

- nunca apagar backup antigo antes de o novo estar confirmado apenas para abrir espaço;
- nunca apagar origem de Restore ativa;
- nunca apagar safety backup durante Restore ou resultado incerto;
- nunca apagar pre-migration backup enquanto a migration não terminou;
- retenção não roda em paralelo com Restore/migration;
- pacote inválido/corrompido não é apagado silenciosamente;
- falha ao remover backup antigo gera warning/auditoria e pode exceder temporariamente o limite sem rebaixar o backup novo.

Backup íntegro mas atualmente incompatível continua sujeito à política normal de antiguidade quando não estiver protegido.

## 3.8 Espaço insuficiente

- retenção não remove antecipadamente backups confirmados para tentar abrir espaço;
- se não houver espaço para staging/pacote novo, a criação falha explicitamente;
- nenhum ponto de recuperação existente é sacrificado automaticamente;
- política futura de quota/limpeza antecipada exige nova decisão.

## 3.9 Coordinator administrativo

O Host possui coordinator lógico exclusivo para operações críticas:

```text
BACKUP
RESTORE
MIGRATION
```

Cada operação raiz adquire um lease exclusivo.

Regras:

- apenas uma operação raiz crítica ativa;
- pedido incompatível recebe `ADMIN_OPERATION_IN_PROGRESS`/equivalente;
- coordinator não substitui o writer lógico;
- reads normais não adquirem lease;
- Backup mantém lease durante toda a operação, mas apenas `BACKUP_CAPTURE` cria barrier curto de mutações;
- Restore mantém lease durante validação, safety backup, fase destrutiva e conclusão;
- migration de startup usa o mesmo modelo antes de readiness quando aplicável.

## 3.10 Suboperações sem deadlock

Safety backup e pre-migration backup reutilizam a pipeline sob o lease já adquirido:

```text
RESTORE lease
→ safety backup interno
→ Restore
```

```text
MIGRATION lease
→ pre-migration backup
→ migration
```

Não existe segundo lease raiz para essas suboperações.

## 3.11 Coordenação com operações normais

### Backup

- consultas podem continuar;
- geração documental read-only pode continuar quando segura;
- mutações só sofrem indisponibilidade em `BACKUP_CAPTURE`;
- após snapshot bruto, mutações voltam enquanto hash/ZIP/verificação/promoção terminam;
- outra operação crítica espera o lease ser liberado.

### Restore

É operação administrativa exclusiva; política de manutenção/desconexão/restart/sessões pertence às Análises 4 e 5.

### Migration

Ocorre sob exclusividade administrativa e antes de readiness quando aplicável. Backup prévio usa a mesma pipeline consistente aprovada.

## 3.12 Eventos e estado

O coordinator expõe estado consultável pelo Host/API. Eventos sinalizam transições, mas resultado definitivo continua vindo de refetch/estado confirmado.

Não persistir segredo, token reutilizável ou conteúdo do backup no estado da operação.

## 3.13 Resultado incerto

Se Restore terminar `uncertain`:

- retenção e cleanup destrutivo ficam suspensos;
- safety backup e origem permanecem protegidos;
- nova operação destrutiva fica bloqueada até diagnóstico/recuperação;
- quantidade de backups pode ultrapassar temporariamente o limite.

Persistência e reconciliação desse estado através de restart pertencem às Análises 5–6.

## 3.14 Decisões aprovadas — D11.26 a D11.42

- **D11.26:** catálogo é reconstruível a partir dos pacotes finais; não depende do `stepflow.sqlite` nem de banco de catálogo adicional;
- **D11.27:** `backup_id` do manifesto é identidade; filename não é identidade canônica;
- **D11.28:** `.staging` nunca entra no catálogo; staging herdado é órfão elegível a cleanup conservador após exclusividade da instância;
- **D11.29:** catálogo distingue integridade de compatibilidade e suporta estados de verificação/invalidade/incompatibilidade;
- **D11.30:** cache de verificação é somente em memória; metadados de filesystem apenas invalidam cache;
- **D11.31:** Restore sempre revalida integralmente o pacote;
- **D11.32:** retenção não usa scheduler e roda após backup confirmado e no startup seguro;
- **D11.33:** retenção inicial é por quantidade, não por idade;
- **D11.34:** `retention_max_confirmed_backups` é parâmetro central; valor/default final fica para o Bloco 12;
- **D11.35:** retenção nunca apaga backup antigo antes da confirmação do novo apenas para liberar espaço;
- **D11.36:** source/safety/pre-migration em uso ou resultado incerto ficam protegidos;
- **D11.37:** pacote inválido/corrompido não é apagado silenciosamente por retenção automática;
- **D11.38:** falha de cleanup/retenção gera warning e pode exceder temporariamente o limite sem rebaixar backup novo;
- **D11.39:** Host possui coordinator/lease exclusivo para `BACKUP`, `RESTORE` e `MIGRATION`;
- **D11.40:** safety/pre-migration backup são suboperações da operação raiz e não adquirem segundo lease;
- **D11.41:** Backup mantém lease administrativo completo, mas barrier de mutações só existe no subestado curto de captura;
- **D11.42:** Restore/migration bloqueiam operações administrativas incompatíveis; manutenção/restart/sessões ficam para análises seguintes.

## Pendências para análises seguintes

- Restore/safety backup/compatibilidade;
- persistência de estado `uncertain` através de restart;
- restart/reconexão/sessões;
- disaster recovery quando Host não sobe;
- Gerência × Backup;
- valor numérico final da retenção no Bloco 12.

## Próximo passo

**Análise 4 — Restore normal + safety backup + compatibilidade.**
