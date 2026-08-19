# Diário de Progresso — StepFlow Pocket

## 2026-08-19 — Fundação documental inicial

### Objetivo

Iniciar o StepFlow Pocket no mesmo modelo de governança e execução utilizado anteriormente, adaptando o método para um aplicativo interno Client/Host e registrando as decisões já discutidas antes de qualquer implementação funcional.

### O que foi feito

- repositório oficial confirmado como `jotaCorsino/StepFlow-pocket`;
- repositório confirmado inicialmente vazio;
- pasta local prevista para Codex registrada como `C:\dev\StepFlow`;
- `README.md` criado;
- `AGENTS.md` criado;
- índice documental criado;
- método genérico reutilizável PO + Assistente + Codex criado;
- guia mestre inicial do StepFlow criado;
- visão geral do produto criada;
- arquitetura Client/Host/Data inicial documentada;
- roadmap por fases criado;
- regras operacionais do projeto e do Codex registradas;
- decisões consolidadas separadas de propostas técnicas pendentes.

### Decisões de produto refletidas

- processo documental simplificado;
- etapas como páginas de manual/livro;
- passos, observações, checklist e blocos copiáveis;
- ícone discreto de cópia;
- sidebar com logo pequeno no topo esquerdo;
- autenticação interna simples;
- ADM, Gerência e Funcionário;
- edição de perfil pessoal;
- multiusuário obrigatório;
- atualização entre clientes;
- uso por duplo clique a partir do compartilhamento de rede;
- PDF/DOCX/impressão mantidos no escopo desejado.

### Decisões arquiteturais refletidas

- JavaScript modular e ausência de monólito HTML/JS;
- separação lógica Client/Host/Data;
- SQLite somente pelo Host, não compartilhado diretamente entre Clients;
- fila de escrita combinada com controle de revisão;
- detecção de conflito antes de sobrescrita;
- eventos de atualização entre instâncias.

### Pontos ainda não fechados

- stack final do Client;
- formato/stack final do Host;
- compatibilidade Windows real;
- launcher e atualização;
- protocolo Client/Host;
- sessão/hash;
- schema de dados;
- bibliotecas de exportação;
- backup/restore;
- telas detalhadas.

### Próximo passo recomendado

Concluir a Fase 0 com templates e revisão cruzada da documentação. Em seguida abrir a Fase 1 com validação técnica da stack, compatibilidade Windows e launcher, sem ainda implementar funcionalidades de negócio.
