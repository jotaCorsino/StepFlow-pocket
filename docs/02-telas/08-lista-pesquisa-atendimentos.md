# Tela 08 — Lista e Pesquisa de Atendimentos

## 1. Identificação

- código/nome da tela: Tela 08 — Lista e Pesquisa de Atendimentos;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco original: Fase 1 — Bloco 8 (UI/UX);
- atualização operacional: Bloco 9;
- última consolidação: 2026-08-25.

## 2. Objetivo

Ser a superfície compacta de consulta e entrada para Atendimentos reais, mantendo busca operacional separada da busca documental de Processos.

A tela não vira dashboard, fila de chamados, kanban, SLA ou CRM.

## 3. Ator(es) e autorização

Pode consultar quem possuir capacidade de `Consultar Atendimentos`.

Preset inicial:

- ADM: sim;
- Gerência: sim;
- Funcionário: sim.

`Novo atendimento` aparece somente para sessão com capacidade de criação; preset inicial sim para os três perfis.

Autorização real é Host-side.

## 4. Layout consolidado

```text
Atendimentos                                      [ Novo atendimento* ]

[ Buscar por atendimento, OS, cliente, equipamento,
  serial, patrimônio ou MAC...                                 ]

[ Responsável ▾ ] [ Status ▾ ] [ Período ▾ ]     [ Limpar filtros ]

Atendimento   Status        OS/Ref.   Cliente      Equipamento      Responsável   Data
AT-00142      Em andamento  OS-4587   João Silva   NOTE-15 · EQP-31 Maria         21/08
AT-00141      Concluído     —         Financeiro   —                Carlos        21/08
AT-00140      Cancelado     CH-991    Empresa Alfa PC-ADM · EQP-28 Maria         20/08
```

`*` depende de capacidade.

Visual continua em lista/tabela compacta, não cards.

## 5. Lifecycle exibido

Estados consolidados pelo Bloco 9:

```text
Em andamento
Concluído
Cancelado
```

A lista mostra estado de forma discreta e permite filtro `Status`.

Não inventar outros estados sem decisão futura explícita.

## 6. Busca

Um único campo operacional combina referências úteis.

Busca pode localizar por:

- código do Atendimento (`AT-000001`);
- OS/referência externa;
- cliente/solicitante;
- código/nome do Equipamento;
- serial;
- patrimônio;
- MAC normalizado.

Não misturar conteúdo documental de Procedimentos nessa busca.

## 7. Filtros

Filtros iniciais consolidados:

- `Responsável`;
- `Status`;
- `Período`.

Busca e filtros são combináveis.

### Responsável

Filtra por responsável/técnico atual do Atendimento.

A visibilidade dos registros continua limitada pela autorização da sessão; filtro não amplia acesso.

### Status

Pode selecionar um ou mais estados conforme componente final, preservando semântica clara.

Estados disponíveis inicialmente são apenas os três consolidados.

### Período

Usa `started_at` como referência temporal inicial.

- `Data` na tabela representa `started_at`;
- intervalo inválido/reverso é rejeitado;
- ausência de filtro significa todos os períodos acessíveis conforme paginação/consulta.

## 8. Ordenação

Ordenação padrão:

```text
started_at DESC
```

Atendimentos iniciados mais recentemente aparecem primeiro.

A implementação pode oferecer outras ordenações depois, sem alterar o default consolidado.

## 9. Resultado e navegação

Clique na linha/título/código abre Tela 09.

```text
Tela 08
→ Atendimento AT-...
→ Tela 09
```

Ao voltar, preservar quando possível:

- busca;
- filtros;
- ordenação;
- página;
- posição de rolagem.

## 10. Novo Atendimento

`Novo atendimento` abre Tela 09 em rascunho somente em memória.

- abrir não cria registro;
- código ainda não existe;
- primeiro save aceito pelo Host cria o Atendimento, gera `AT-......`, define `started_at` e `Em andamento`;
- sair antes do primeiro save segue proteção de alterações não salvas.

## 11. Colunas

Colunas principais:

- Atendimento;
- Status;
- OS/Ref.;
- Cliente;
- Equipamento;
- Responsável;
- Data.

Equipamento aparece apenas como resumo, por exemplo:

```text
NOTE-15 · EQP-000031
```

Sem equipamento:

```text
—
```

Não transformar a lista em ficha técnica.

## 12. Estados vazios

### Nenhum Atendimento existente/acessível

`Nenhum atendimento disponível.`

Se a sessão puder criar, pode aparecer CTA contextual.

### Busca/filtros sem resultado

`Nenhum resultado encontrado com os filtros atuais.`

Oferecer `Limpar filtros` quando aplicável.

Não confundir os dois estados.

## 13. Loading e Host indisponível

Segue padrão da Tela 15:

- manter busca/filtros estáveis durante reconsulta;
- não mostrar dados antigos como atuais;
- Host indisponível não oferece IP/porta/path;
- WebSocket degradado pode permitir consultas HTTP quando o Host continuar saudável.

## 14. Atualizações em tempo real

Eventos pós-commit relevantes:

- Atendimento criado/alterado;
- status alterado;
- responsável alterado;
- Equipamento vinculado/alterado;
- Atendimento concluído/reaberto/cancelado.

Client reconsulta lista quando necessário.

Preservar contexto e evitar deslocamento abrupto durante leitura.

## 15. Concorrência

A lista é somente leitura.

Conflitos de edição/lifecycle pertencem à Tela 09.

Evento remoto não transforma item local em estado oficial sem reconsulta.

## 16. Segurança

- área/ações aparecem por capacidade;
- acesso direto não autorizado é rejeitado pelo Host;
- filtros não revelam registros fora do escopo autorizado;
- mensagens não expõem dados protegidos de registros sem acesso.

## 17. Janelas suportadas

Desktop Windows é o alvo principal.

Em janelas menores:

- colunas secundárias podem ser ocultadas progressivamente;
- Atendimento, Status e contexto principal permanecem legíveis;
- evitar scroll horizontal; usar como último recurso;
- sem transformação mobile/hamburger inicial.

## 18. Decisões consolidadas

- lista/tabela compacta;
- busca operacional única;
- filtros Responsável + Status + Período;
- estados `Em andamento`, `Concluído`, `Cancelado`;
- Data/Período = `started_at`;
- default mais recente primeiro;
- Equipamento apenas como resumo;
- linha abre Tela 09;
- retorno preserva contexto;
- `Novo atendimento` depende de capacidade;
- busca de Atendimentos permanece separada de Processos;
- sem KPIs/cards/kanban/SLA.

## 19. Fora do escopo

- dashboard operacional;
- SLA/prioridade/severidade;
- fila por equipe;
- kanban;
- ações em massa;
- edição inline;
- CRM;
- estoque/RMM/faturamento;
- implementação funcional nesta fase.