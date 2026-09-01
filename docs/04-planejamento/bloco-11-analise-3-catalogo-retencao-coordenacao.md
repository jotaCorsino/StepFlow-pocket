# Bloco 11 — Análise 3 — Catálogo, retenção e coordenação administrativa

**Status:** APROVADA PELO PO / CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Data da aprovação:** 2026-08-31

## Objetivo

Fechar como o Host descobre/classifica backups administrados, limita crescimento do diretório sem scheduler e serializa operações administrativas críticas.

Esta análise parte das Análises 1–2 aprovadas e não altera a UX da Tela 13 nem o contrato Pocket.

## Catálogo

O catálogo é reconstruível diretamente dos pacotes finais administrados.

Regras:

- não depende do `stepflow.sqlite` ativo;
- considera somente `.stepflow-backup` finais;
- `.staging` e resíduos operacionais não entram;
- `backup_id` do manifesto é identidade lógica;
- filename e timestamps do filesystem não são identidade/prova de validade;
- cache de verificação é apenas em memória;
- metadados do filesystem podem invalidar cache, nunca comprovar integridade;
- Restore sempre executa revalidação integral.

Backup inválido/corrompido pode aparecer como inválido para diagnóstico, mas não é apagado silenciosamente.

## Startup e resíduos

Ao iniciar de forma segura/exclusiva:

- reconstruir catálogo dos pacotes finais;
- ignorar staging como backup;
- resíduos antigos só são limpos após provar que não pertencem a operação ativa/incerta;
- `uncertain` suspende cleanup destrutivo.

## Retenção

Baseline:

- sem scheduler;
- por quantidade, não por idade;
- executada após novo backup confirmado ou em startup seguro;
- parâmetro central `retention_max_confirmed_backups`;
- valor/default numérico reservado ao Bloco 12.

Regras de segurança:

- nunca apagar backup antigo antes de confirmar o novo apenas para abrir espaço;
- espaço insuficiente faz o novo backup falhar sem sacrificar silenciosamente ponto de recuperação existente;
- source backup de Restore, safety backup, pre-migration backup e backups associados a estado incerto ficam protegidos;
- pacote inválido/corrompido não é removido silenciosamente pela retenção;
- falha de retenção após backup confirmado gera warning, não invalida o pacote válido.

## Coordinator administrativo

O Host mantém um único lease/coordinator lógico exclusivo para operações raiz incompatíveis:

```text
BACKUP
RESTORE
MIGRATION
```

Regras:

- somente uma operação raiz incompatível por vez;
- safety backup e pre-migration backup são suboperações da operação que já possui o lease;
- suboperação não tenta adquirir segundo lease raiz;
- Backup normal mantém lease durante toda a operação, mas o barrier de mutações existe somente durante a captura;
- fila administrativa crítica não é ilimitada; conflito de operação retorna estado compreensível/busy;
- `uncertain/RECOVERY_REQUIRED` bloqueia nova operação destrutiva até resolução.

## Decisões aprovadas — D11.26 a D11.42

- **D11.26:** catálogo é reconstruível dos pacotes finais e não depende do SQLite ativo;
- **D11.27:** `backup_id` do manifesto é identidade lógica; filename não é identidade canônica;
- **D11.28:** somente pacotes finais entram no catálogo; staging/resíduos operacionais ficam fora;
- **D11.29:** cache de verificação é somente em memória e pode ser invalidado por metadados do filesystem;
- **D11.30:** Restore sempre revalida integralmente o pacote, independentemente do cache;
- **D11.31:** startup pode limpar resíduos somente após provar exclusividade e ausência de operação ativa/incerta;
- **D11.32:** pacote final inválido/corrompido não é apagado silenciosamente;
- **D11.33:** retenção baseline não usa scheduler e é por quantidade;
- **D11.34:** retenção roda após backup confirmado e/ou startup seguro;
- **D11.35:** `retention_max_confirmed_backups` é parâmetro central, com valor/default final no Bloco 12;
- **D11.36:** não apagar backup antigo antes de confirmar novo apenas para abrir espaço;
- **D11.37:** source/safety/pre-migration e backups ligados a resultado incerto ficam protegidos;
- **D11.38:** falha de retenção não invalida backup já confirmado; gera warning/estado administrativo;
- **D11.39:** Host usa lease/coordinator exclusivo para `BACKUP`, `RESTORE` e `MIGRATION` incompatíveis;
- **D11.40:** safety/pre-migration backup são suboperações do lease raiz e não adquirem lease concorrente;
- **D11.41:** Backup mantém lease até terminar, mas o barrier de mutações do Backup normal dura somente a captura;
- **D11.42:** `uncertain/RECOVERY_REQUIRED` suspende retenção, cleanup destrutivo e nova operação administrativa destrutiva.

## Parâmetro ainda externo a esta análise

Somente o **valor numérico/default** de `retention_max_confirmed_backups` permanece para o Bloco 12. A política e o algoritmo já estão consolidados.
