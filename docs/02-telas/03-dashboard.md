# Tela 03 — Início / Dashboard

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Aprovação:** 2026-08-21

## 1. Objetivo

Ser o primeiro destino após login e oferecer um ponto de partida simples para o uso diário do StepFlow.

O Dashboard não é painel executivo nem central de KPIs. Sua prioridade é reduzir o tempo entre abrir o aplicativo e localizar um procedimento útil.

## 2. Entrada

```text
Login válido
→ Shell principal
→ Início/Dashboard
```

O item `Início` da sidebar retorna a esta tela.

## 3. Composição aprovada

```text
Início
Bem-vindo, <nome>.

┌──────────────────────────────────────────────────────────────┐
│ Buscar um processo                                           │
│ [ código, título ou termo...                         ] [🔎]   │
└──────────────────────────────────────────────────────────────┘

Atualizados recentemente
---------------------------------------------------------------
[Código] Título do processo             Categoria/Área   Versão
[Código] Título do processo             Categoria/Área   Versão
[Código] Título do processo             Categoria/Área   Versão

                                      [ Ver todos os processos ]
```

A composição permanece deliberadamente enxuta.

## 4. Busca rápida

- busca por código, título ou termo compatível com a Tela 04;
- Enter/ação de busca encaminha para `Processos` com o termo aplicado;
- filtros completos pertencem à Tela 04;
- resultados respeitam autorização Host-side;
- a busca do Dashboard é de **procedimentos/documentação**, não de OS/equipamentos/atendimentos.

Busca operacional por atendimento, cliente, OS, equipamento, serial/patrimônio/MAC pertence à área `Atendimentos`.

## 5. Atualizados recentemente

Exibir lista curta de procedimentos oficiais recentemente publicados/atualizados e visíveis ao usuário.

Campos úteis:

- Código;
- Título;
- Categoria(s) de forma discreta;
- Área/Departamento quando útil;
- Versão exibida.

Regras:

- somente conteúdo autorizado;
- Funcionário não recebe rascunho administrativo indevido;
- clique abre o leitor;
- `Ver todos os processos` abre a Tela 04;
- quantidade exata de itens é detalhe visual do sistema de design.

## 6. O que não entra no Dashboard inicial

Não incluir por padrão:

- KPIs/gráficos;
- ranking de usuários;
- analytics de produtividade;
- favoritos/bookmarks;
- histórico pessoal de navegação;
- painel gerencial separado;
- duplicação dos atalhos `Atendimentos`, `Usuários` ou `Configurações` já presentes na sidebar.

Se necessidade futura justificar algum desses elementos, a tela poderá ser revisada explicitamente.

## 7. Conteúdo por perfil

O núcleo é comum a ADM, Gerência e Funcionário:

- busca rápida;
- atualizados recentemente;
- acesso à lista completa.

Diferenças vêm da autorização do conteúdo, não de dashboards completamente distintos.

## 8. Estados

### Loading

Mostrar estrutura estável sem múltiplos spinners desconexos.

### Nenhum processo disponível

Mensagem simples. Não sugerir criação para usuário sem capacidade.

### Nenhuma atualização recente

Mensagem curta mantendo `Ver todos os processos` disponível.

### Host indisponível

Seguir o estado transversal do Shell; não apresentar cache como atual sem indicação.

### Sessão expirada

Retornar ao Login.

### Erro parcial

Falha na lista de recentes não precisa derrubar toda a tela quando busca/perfil continuarem válidos.

## 9. Atualização em tempo real

Eventos relevantes podem provocar reconsulta da lista de procedimentos recentes.

O Client não trata payload de evento como nova fonte oficial de verdade; reconsulta o estado autorizado no Host.

## 10. Navegação

- busca → Tela 04 com termo aplicado;
- procedimento recente → leitor;
- `Ver todos os processos` → Tela 04;
- atendimentos → pela sidebar ou fluxos contextuais aprovados posteriormente.

## 11. Acessibilidade

- campo de busca possui label acessível;
- Enter executa busca;
- itens recentes são acessíveis por teclado;
- foco visível;
- estados vazio/erro não dependem só de cor;
- ordem de foco previsível.

## 12. Janelas menores

- busca permanece utilizável;
- lista recente pode reduzir informação secundária sem esconder Código/Título;
- rolagem vertical é aceitável;
- não criar layout mobile sem necessidade comprovada.

## 13. Dados/contratos necessários

- nome de exibição da sessão;
- capacidades efetivas;
- consulta de procedimentos visíveis ordenados por atualização oficial;
- busca/listagem de procedimentos;
- abertura por identidade estável;
- eventos/reconsulta quando documentação relevante mudar.

Não criar novo dado de domínio apenas para enriquecer o Dashboard.

## 14. Segurança

- Host filtra/autoriza dados;
- ocultação no Client não concede autoridade;
- rascunhos/revisões não publicadas respeitam permissões;
- Dashboard não expõe dados administrativos ou operacionais indevidos.

## 15. Sistema visual

Usar o sistema visual compartilhado do Shell:

- área ampla e limpa;
- busca como ação principal;
- poucos blocos;
- categorias como informação secundária discreta;
- cards somente quando ajudarem, não como padrão obrigatório.

## 16. Decisões consolidadas

1. Dashboard é destino pós-login;
2. composição enxuta, sem KPIs/gráficos;
3. busca rápida de processos/procedimentos é o elemento principal;
4. busca encaminha à Tela 04 com termo aplicado;
5. seção curta `Atualizados recentemente`;
6. clique em item recente abre o leitor;
7. sem favoritos/histórico pessoal inicialmente;
8. mesma base de Dashboard para todos os perfis;
9. não duplicar áreas administrativas/operacionais já disponíveis na sidebar;
10. busca do Dashboard não mistura procedimentos com registros de Atendimento.

## 17. Critérios de aceite

- [x] destino pós-login aprovado;
- [x] prioriza consulta de procedimentos;
- [x] sem KPIs/gráficos/painel gerencial separado;
- [x] sem favoritos/recents pessoais não requeridos;
- [x] categorização pode aparecer de forma discreta;
- [x] `Atendimentos` permanece domínio separado;
- [x] autorização continua Host-side;
- [x] nenhum código de produção criado.

## 18. Casos de teste futuros

1. login chega ao Dashboard;
2. busca por Código encaminha à Tela 04;
3. busca por Título/termo mantém termo aplicado;
4. item recente abre o processo correto;
5. lista recente respeita permissões;
6. categoria aparece sem comprometer legibilidade;
7. nenhum processo mostra estado vazio coerente;
8. Host indisponível mostra estado correto;
9. atualização relevante reflete após evento/reconsulta;
10. teclado funciona;
11. layout funciona na janela mínima aprovada;
12. busca não retorna atendimento/equipamento por engano.