# Tela 11 — Estados Transversais

Estados comuns devem ser consistentes e leves.

## Vazio

Explica o próximo passo e oferece uma ação principal quando aplicável.

## Carregando

Usar skeleton/spinner apenas quando houver espera perceptível. IndexedDB e operações locais devem evitar bloqueio visual desnecessário.

## Erro

Mensagem clara, código técnico somente quando útil e nenhuma perda silenciosa de dados.

## Offline

No modo PWA, offline é condição normal após os assets estarem disponíveis. Não existe fila de sincronização porque os dados são locais.

## Persistência indisponível

Se IndexedDB não puder ser aberto, bloquear mutações e orientar o usuário. Não cair silenciosamente para armazenamento volátil.

## Atualização do app

Quando um novo service worker estiver pronto, informar atualização disponível de forma discreta e aplicar em momento seguro.
