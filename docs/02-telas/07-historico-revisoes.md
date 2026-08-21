# Tela 07 — Histórico e Revisões de Processo

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Permitir consultar o histórico imutável de um procedimento, identificar com clareza a revisão atual e a revisão publicada, abrir snapshots autorizados em modo somente leitura e reutilizar conteúdo antigo sem reescrever o passado.

## 2. Atores e permissões

- ADM;
- Gerência;
- outros usuários somente quando houver capacidade explícita para consultar histórico.

Funcionário/Técnico permanece orientado principalmente à revisão publicada/autorizada. O Host é a autoridade final.

## 3. Entrada

```text
Leitor → menu contextual → Histórico
Processos → menu do item → Histórico
Editor → ação contextual → Histórico
```

O histórico sempre pertence a um procedimento identificado de forma estável.

## 4. Estrutura aprovada

```text
← Processo

PR-014  Configuração de VLAN
Histórico de revisões

Revisão   Versão   Criada em          Autor            Estado
r18       2.0      21/08/2026 08:40   Maria            Atual • Publicada
r17       1.9      20/08/2026 15:12   João             —
r16       1.9      20/08/2026 11:03   João             —
r15       1.8      18/08/2026 09:26   Maria            —

[ Abrir revisão ]   [ Criar nova revisão a partir desta* ]
```

A lista é compacta e cronológica em ordem decrescente, com a revisão mais recente primeiro.

## 5. Revisão técnica × versão exibida

- `revision_no` é técnico, monotônico e identifica o snapshot histórico;
- `display_version` é editorial e pode se repetir entre revisões;
- a interface não trata `Versão 2.0` como identificador único do histórico;
- quando necessário, mostrar ambas: `Revisão r18 · Versão 2.0`.

## 6. Indicadores de estado

Exibir de forma discreta:

- **Atual** — revisão apontada por `current_revision_id`;
- **Publicada** — revisão apontada por `published_revision_id`.

As duas podem coincidir ou ser diferentes.

Uma revisão que foi publicada no passado não recebe selo `Publicada` apenas por ter sido publicada anteriormente; o selo representa o ponteiro publicado vigente.

## 7. Abrir revisão histórica

Ao abrir uma revisão histórica:

- reutilizar o Leitor em modo somente leitura;
- manter indicação textual persistente de que é uma revisão histórica;
- exibir algo equivalente a `Revisão histórica r17 · Versão 1.9`;
- não permitir que pareça a revisão vigente;
- voltar retorna ao Histórico preservando a posição/lista quando possível.

## 8. Criar nova revisão a partir de uma antiga

Para usuário com capacidade de edição, usar a ação explícita:

`Criar nova revisão a partir desta`

Fluxo consolidado:

1. o snapshot antigo é usado como conteúdo inicial no Editor;
2. nenhuma revisão histórica é alterada;
3. nada é persistido até `Salvar`;
4. o save usa a revisão atual vigente como base de concorrência;
5. save aceito cria uma **nova** `process_revision` imutável;
6. se a revisão atual mudar antes do save, aplica-se `409 REVISION_CONFLICT` normalmente.

Não usar ação ambígua de `Restaurar` que possa sugerir reescrita destrutiva do histórico.

## 9. Imutabilidade do passado

Na operação normal:

- revisão histórica não pode ser editada in-place;
- revisão histórica não pode ser excluída;
- `revision_no` existente não é alterado;
- não há rollback destrutivo;
- abrir uma revisão nunca move ponteiros silenciosamente.

## 10. Publicação

Revisão histórica não é publicada diretamente.

Fluxo seguro:

```text
revisão histórica
→ Criar nova revisão a partir desta
→ revisar/salvar no Editor
→ nova revisão atual
→ Publicar revisão atual, se autorizado
```

Isso preserva a regra consolidada de que `Salvar` e `Publicar revisão atual` são ações distintas.

## 11. Dados exibidos

Por revisão, quando disponíveis:

- `revision_no`;
- `display_version`;
- data/hora de criação;
- autor da revisão;
- indicação Atual/Publicada;
- acesso ao snapshot completo.

Não adicionar mensagem obrigatória de commit/revisão sem requisito explícito.

## 12. Estados da interface

### Loading

Exibir estrutura da página e skeleton da lista.

### Sem histórico

Procedimento ainda não salvo não abre Histórico. Procedimento persistido possui ao menos sua primeira revisão.

### Revisão indisponível

Informar de forma simples e manter retorno ao histórico.

### Sem permissão

Não carregar snapshots protegidos de cache. O Host decide o acesso.

### Host indisponível

Histórico oficial fica indisponível; cache local não vira fonte oficial.

### Processo arquivado

Usuário administrativo autorizado pode consultar histórico quando o Host permitir, com indicação clara de `Arquivado`.

## 13. Atualização em tempo real

Se uma nova revisão surgir enquanto a tela estiver aberta:

- WebSocket apenas sinaliza;
- Client reconsulta a lista oficial;
- nova revisão pode aparecer no topo;
- revisão histórica selecionada/aberta não é trocada automaticamente;
- indicadores Atual/Publicada são reconciliados com o Host.

## 14. Concorrência

Consulta de histórico não cria lock.

`Criar nova revisão a partir desta` leva ao Editor e passa a obedecer ao mesmo `base_revision`/controle otimista da Tela 06. O Histórico nunca oferece overwrite forçado.

## 15. Contratos Client ↔ Host necessários

Conceitualmente:

1. listar revisões autorizadas de um procedimento;
2. obter snapshot de revisão específica;
3. informar revisão atual/publicada;
4. receber capacidades da sessão;
5. reconsultar após eventos de processo;
6. iniciar Editor a partir de snapshot histórico sem persistir automaticamente.

Nomes finais dos endpoints pertencem à implementação.

## 16. Paginação e volume

Histórico grande não deve ser carregado indefinidamente de forma ingênua. Primeira versão pode usar paginação simples ou carregamento incremental quando necessário, sem infraestrutura adicional desproporcional.

## 17. Comparação de revisões

Diff/comparação visual lado a lado pode ser útil futuramente, mas **não é requisito da primeira versão**.

Se for adicionado no futuro, deve ser derivado dos snapshots imutáveis, sem persistência adicional apenas para apresentação.

## 18. Acessibilidade e teclado

- tabela/lista navegável por teclado;
- foco visível;
- indicadores Atual/Publicada não dependem apenas de cor;
- ações possuem labels claros;
- seleção não depende de duplo clique;
- modo histórico é indicado textualmente.

## 19. Tamanhos de janela

Desktop Windows é prioridade.

Em janela menor:

- colunas secundárias podem ir para detalhes da linha;
- Revisão, Versão e Estado permanecem visíveis;
- ações continuam acessíveis;
- não criar layout mobile específico sem necessidade demonstrada.

## 20. Fora do escopo

- editar revisão histórica;
- excluir revisão histórica;
- rollback destrutivo;
- merge automático entre revisões;
- diff visual obrigatório na primeira versão;
- workflow complexo de aprovação;
- auditoria de segurança genérica misturada à lista de revisões;
- código Tauri/Host.

## 21. Decisões consolidadas

1. lista cronológica compacta, mais recente primeiro;
2. `revision_no` e `display_version` são mostrados como conceitos distintos;
3. badges `Atual` e `Publicada` representam os ponteiros vigentes;
4. revisão histórica reutiliza o Leitor em somente leitura;
5. revisão histórica é identificada visual/textualmente de forma persistente;
6. snapshots não podem ser editados/excluídos;
7. usar `Criar nova revisão a partir desta` em vez de rollback destrutivo;
8. conteúdo antigo só pode voltar a ser publicado depois de virar uma nova revisão atual;
9. diff visual fica fora da primeira versão.

## 22. Pendências

- microcopy final;
- paginação exata;
- capacidade/perfil padrão para consulta de histórico;
- eventual comparação visual futura.

## 23. Critérios de aceite

- [x] revisões permanecem imutáveis;
- [x] current/published são distinguidos;
- [x] versão editorial não substitui `revision_no`;
- [x] nenhuma operação destrutiva foi introduzida;
- [x] concorrência continua otimista;
- [x] UX da lista aprovada pelo PO;
- [x] `Criar nova revisão a partir desta` aprovado;
- [x] diff visual não obrigatório aprovado;
- [x] nenhum código de produção criado.

## 24. Casos de teste futuros

1. listar procedimento com uma revisão;
2. listar muitas revisões;
3. current = published;
4. current diferente de published;
5. abrir revisão atual;
6. abrir revisão histórica;
7. negar histórico sem permissão;
8. receber nova revisão via evento/reconsulta;
9. iniciar nova revisão a partir de snapshot antigo;
10. salvar com base atual;
11. receber 409 se current mudar antes do save;
12. validar processo arquivado;
13. validar janela menor/acessibilidade.
