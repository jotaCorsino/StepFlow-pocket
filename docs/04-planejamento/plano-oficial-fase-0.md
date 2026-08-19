# Plano Oficial — Fase 0: Fundação Documental e Governança

**Status:** CONCLUÍDA EM 2026-08-19

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
- [x] revisão cruzada final dos documentos da Fase 0;
- [x] registrar formalmente o encerramento da Fase 0.

## Checklist de coerência para encerramento

- [x] Nenhum documento trata proposta técnica pendente como decisão consolidada por engano.
- [x] Requisitos de produto discutidos estão refletidos no guia mestre e visão geral.
- [x] Multiusuário e concorrência estão registrados como requisitos estruturais.
- [x] O cenário de duplo clique pela rede está registrado como requisito de UX, sem amarrar prematuramente a implementação.
- [x] O modelo simplificado de processos está consistente em todos os documentos.
- [x] Perfis ADM/Gerência/Funcionário estão consistentes.
- [x] A metáfora de livro/páginas está preservada.
- [x] O roadmap não libera código antes dos gates.
- [x] AGENTS.md corresponde ao método genérico e às regras específicas do projeto.
- [x] Pendências da Fase 1 estão identificadas.

A evidência da revisão está registrada em `docs/05-progresso/revisao-cruzada-fase-0.md`.

## Critério de saída

A Fase 0 é considerada concluída porque a base documental permite que uma nova sessão do Assistente ou do Codex responda, sem depender do histórico do chat:

- o que é o StepFlow;
- para quem ele existe;
- como deve funcionar no nível atual de decisão;
- quais requisitos são obrigatórios;
- quais propostas técnicas ainda precisam de validação;
- como o projeto será conduzido;
- qual é a próxima fase autorizada;
- o que ainda não pode ser implementado.

## Resultado

**Gate aprovado.**

Nenhum código de negócio, banco real, dependência de aplicação ou executável foi criado durante a Fase 0.

## Próxima fase autorizada

**Fase 1 — Fechamento arquitetural e especificação.**

A Fase 1 começa por investigação e validação técnica, especialmente compatibilidade Windows, Tauri/alternativas, Host, launcher SMB, comunicação, concorrência, modelo de dados e telas críticas.