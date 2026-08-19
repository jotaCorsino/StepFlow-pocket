# Plano Oficial — Fase 0: Fundação Documental e Governança

**Status:** EM ANDAMENTO

## Objetivo

Preparar o StepFlow Pocket para decisões técnicas e futuras implementações sem improviso, consolidando fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e padrões de execução.

## O que esta fase autoriza

- criação e revisão de documentação;
- análise de requisitos;
- desenho conceitual de arquitetura;
- levantamento de riscos e pendências;
- organização do roadmap;
- criação de templates;
- inspeção de tecnologias e alternativas para embasar decisões futuras.

## O que esta fase não autoriza

- criação do app funcional;
- instalação da stack definitiva;
- criação de banco real;
- migrations de produção;
- implementação de autenticação;
- implementação de CRUD;
- criação de launcher funcional;
- empacotamento de executável final;
- UI tratada como concluída sem especificação/aprovação.

## Entregáveis obrigatórios

- [x] README inicial;
- [x] AGENTS.md;
- [x] índice documental;
- [x] método genérico de trabalho assistido;
- [x] regras operacionais;
- [x] guia mestre inicial;
- [x] visão geral do produto;
- [x] arquitetura inicial;
- [x] roadmap;
- [x] área de documentação de telas;
- [x] registro inicial de decisões;
- [x] diário de progresso;
- [x] changelog;
- [x] template de análise de tela;
- [x] template de tarefa Codex;
- [ ] revisão cruzada final dos documentos da Fase 0;
- [ ] registrar formalmente o encerramento da Fase 0.

## Checklist de coerência para encerramento

Antes de fechar a fase, verificar:

- [ ] Nenhum documento trata proposta técnica pendente como decisão consolidada por engano.
- [ ] Requisitos de produto discutidos estão refletidos no guia mestre e visão geral.
- [ ] Multiusuário e concorrência estão registrados como requisitos estruturais.
- [ ] O cenário de duplo clique pela rede está registrado como requisito de UX, sem amarrar prematuramente a implementação.
- [ ] O modelo simplificado de processos está consistente em todos os documentos.
- [ ] Perfis ADM/Gerência/Funcionário estão consistentes.
- [ ] A metáfora de livro/páginas está preservada.
- [ ] O roadmap não libera código antes dos gates.
- [ ] AGENTS.md corresponde ao método genérico e às regras específicas do projeto.
- [ ] Pendências da Fase 1 estão identificadas.

## Critério de saída

A Fase 0 será concluída quando a base documental permitir que uma nova sessão do assistente ou do Codex responda, sem depender do histórico do chat:

- o que é o StepFlow;
- para quem ele existe;
- como deve funcionar no nível atual de decisão;
- quais requisitos são obrigatórios;
- quais propostas técnicas ainda precisam de validação;
- como o projeto será conduzido;
- qual é a próxima fase autorizada;
- o que não pode ser implementado ainda.

## Próxima fase

**Fase 1 — Fechamento arquitetural e especificação.**

A Fase 1 deverá começar por investigação e validação técnica, especialmente compatibilidade Windows, Tauri/alternativas, Host, launcher SMB, comunicação, concorrência, modelo de dados e telas críticas.
