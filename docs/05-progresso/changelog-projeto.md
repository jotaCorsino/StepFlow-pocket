# Changelog do Projeto — StepFlow

Este arquivo registra apenas marcos ainda relevantes para a árvore documental atual.

## 2026-09-02 — Rebaseline web estático

- infraestrutura definida como cPanel gerenciado sem executáveis próprios;
- produto redefinido como aplicação web estática single-user/local-first;
- persistência movida para IndexedDB no navegador;
- PWA/offline adotada como distribuição preferencial sem rede;
- variante `StepFlow.html` autocontida admitida como saída portátil opcional;
- contas, login, sessão, permissões, sincronização e concorrência distribuída removidos do baseline;
- preservação de dados passa a usar Exportar/Importar;
- impressão/PDF passa a usar recursos nativos do navegador;
- arquitetura e documentação ativas estão sendo reconstruídas para remover contratos substituídos.

O histórico completo de decisões anteriores permanece no Git, não na documentação ativa.
