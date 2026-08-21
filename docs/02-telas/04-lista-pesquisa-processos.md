# Tela 04 — Lista e Pesquisa de Processos

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Permitir localizar rapidamente procedimentos oficiais por código, título, termo, área e categoria, com acesso direto ao leitor e às ações de manutenção documental quando autorizadas.

Esta tela pesquisa **documentação/procedimentos**. Registros de serviço, OS, equipamentos, cliente, serial, patrimônio e MAC pertencem à área `Atendimentos`.

## 2. Atores

- ADM;
- Gerência;
- Funcionário.

Todos podem consultar procedimentos autorizados. Ações de criação/edição/publicação/arquivamento dependem das capacidades efetivas da sessão.

## 3. Entrada

A tela pode ser aberta por:

- item `Processos` da sidebar;
- busca rápida do Dashboard, com termo já aplicado;
- retornos contextuais de outras telas.

## 4. Estrutura proposta

```text
Processos                                      [ Novo processo* ]
Encontre procedimentos por código, título, área ou categoria.

[ Buscar por código, título ou termo...                     ]

[ Categoria ▾ ] [ Área ▾ ] [ Status* ▾ ]          [ Limpar filtros ]

Resultados
──────────────────────────────────────────────────────────────
Código    Título                       Categorias      Área   Versão
PR-001    Manutenção preventiva       Manutenção     TI     1.3
PR-014    Configuração de VLAN        Redes, Infra   TI     2.0
PR-020    Reset de senha              Help Desk      SD     1.1

* somente quando autorizado
```

A tabela/lista deve favorecer leitura rápida e densidade moderada. Cards grandes não são o padrão inicial para esse tipo de consulta.

## 5. Busca textual

A busca deve aceitar, no mínimo:

- Código;
- Título;
- termos compatíveis com os campos indexados definidos pelo Host.

Direção proposta:

- busca executada explicitamente por Enter/ação de busca ou com debounce controlado se a implementação demonstrar benefício;
- não iniciar dezenas de consultas enquanto o usuário digita;
- termo vindo do Dashboard aparece preenchido e aplicado;
- busca vazia mostra listagem padrão autorizada;
- não pesquisar dados operacionais de `Atendimentos` nesta tela.

A estratégia técnica de normalização/índices pertence à implementação do Host, seguindo o modelo de dados vigente.

## 6. Filtro por categorias

Categorias são configuráveis e um procedimento pode possuir múltiplas.

### Proposta de UX

- filtro `Categoria` permite escolher uma ou mais categorias;
- seleção múltipla usa comportamento simples, sem árvore hierárquica;
- quando várias categorias forem selecionadas, a primeira versão pode aplicar semântica **OU**: mostrar procedimento pertencente a qualquer uma das categorias selecionadas;
- filtros ativos ficam visíveis e removíveis;
- categoria arquivada não aparece como opção normal para novos filtros, salvo contexto administrativo/histórico.

A semântica `OU` é proposta para aprovação; não implementar `E`/construtor avançado sem necessidade real.

## 7. Outros filtros

### Área/Departamento

Filtro simples pelas áreas existentes nos procedimentos visíveis.

### Status

Somente quando fizer sentido para o perfil:

- Funcionário vê essencialmente conteúdo publicado/disponível para consulta;
- ADM/Gerência podem precisar diferenciar rascunho/publicado/arquivado conforme o lifecycle documental consolidado posteriormente.

Não transformar `Status` em filtro burocrático se o usuário não puder agir sobre estados diferentes.

## 8. Ordenação

Proposta inicial:

- sem busca textual: ordenar por Código ou Título de forma estável e previsível;
- com busca: relevância compatível com o mecanismo de busca, com desempate estável;
- permitir ordenação por Código, Título e atualização quando isso trouxer valor.

A ordem padrão final pode ser fechada junto da validação visual; não depende de mudança de domínio.

## 9. Colunas/informações da lista

Informações prioritárias:

- Código;
- Título;
- Categoria(s);
- Área/Departamento;
- Versão exibida.

Informações secundárias opcionais quando úteis:

- Responsável;
- data de atualização/publicação;
- status para perfis administrativos.

Evitar excesso de colunas na visualização padrão.

## 10. Exibição das categorias

Proposta:

- categorias aparecem como labels/chips discretos;
- mostrar quantidade limitada por linha quando houver muitas;
- excesso pode aparecer como `+N`/expansão acessível;
- cor não é a única forma de distinguir categoria;
- não usar uma cor exclusiva fixa por categoria como requisito inicial.

## 11. Abertura do procedimento

Ação principal da linha é abrir o **Leitor em formato livro**.

Comportamento proposto:

- clique no título/linha abre o leitor;
- foco/Enter em elemento equivalente também abre;
- não abrir diretamente o editor apenas porque o usuário possui permissão de edição.

Isso mantém a leitura como caminho principal e evita edição acidental.

## 12. Ações para perfis autorizados

### Novo processo

Botão `Novo processo` aparece apenas com capacidade de criação.

### Ações por item

Quando autorizado, menu contextual discreto pode incluir:

- Editar;
- Histórico;
- Publicar, quando aplicável;
- Arquivar.

`Arquivar` é preferível a `Excluir` na UI normal, alinhado ao modelo de dados.

Ação destrutiva exige confirmação apropriada.

## 13. Estados

### Loading

- preservar estrutura da tela;
- evitar spinner por linha;
- filtros não devem parecer aplicados antes da resposta atual.

### Nenhum resultado

Mensagem contextual:

- `Nenhum processo encontrado com os filtros atuais.`
- ação `Limpar filtros` quando filtros estiverem ativos.

### Nenhum processo cadastrado

- Funcionário: informar ausência de procedimentos disponíveis;
- usuário com criação: pode receber ação contextual `Novo processo`.

### Host indisponível

Seguir estado transversal do Shell; não apresentar cache como resultado atual sem indicação.

### Sessão/permissão alterada

Reconciliar lista/ações conforme novas capacidades; Host continua autoridade.

## 14. Atualização em tempo real

Quando evento indicar alteração/publicação/arquivamento de procedimento ou categoria relevante:

- invalidar/reconsultar resultados afetados;
- preservar termo/filtros atuais;
- evitar reposicionar agressivamente a lista enquanto o usuário interage, salvo necessidade de consistência.

## 15. Paginação / volume

A lista não deve carregar quantidade indefinida de registros de uma vez.

Direção proposta:

- paginação simples ou carregamento incremental controlado;
- tamanho de página definido na implementação após estimativa/medição do volume real;
- filtros/termo permanecem ao navegar entre páginas;
- não exigir virtualização complexa sem necessidade demonstrada.

## 16. URL/estado de navegação

Mesmo em Tauri, o estado da tela deve ser representável internamente de forma previsível:

- termo de busca;
- categorias selecionadas;
- área/status aplicáveis;
- página/ordenação quando necessário.

Voltar do leitor deve preferencialmente restaurar a consulta anterior em vez de zerar o contexto do usuário.

## 17. Dados/contratos necessários

A tela depende conceitualmente de:

1. consulta paginada/listável de procedimentos autorizados;
2. busca textual;
3. filtro por categoria;
4. filtro por área/status quando aplicável;
5. lista de categorias disponíveis;
6. capacidades da sessão;
7. abertura por `process_id` estável;
8. eventos/reconsulta após mudanças.

Nomes finais de rotas ficam para implementação.

## 18. Segurança e autorização

- Host filtra os resultados;
- Funcionário não recebe rascunhos/arquivados sem permissão;
- ocultar `Novo processo`/ações no Client não substitui autorização;
- categoria não concede acesso a procedimento por si só;
- manipulação de filtros/estado não permite leitura indevida.

## 19. Concorrência

A lista é predominantemente leitura.

Ações de edição/publicação/arquivamento seguem revisão otimista e regras do documento de concorrência; a lista não cria locking próprio.

## 20. Acessibilidade e teclado

- busca e filtros possuem labels acessíveis;
- filtros são operáveis por teclado;
- tabela/lista possui estrutura semântica adequada;
- linha/ação de abrir procedimento é identificável;
- foco visível;
- menus contextuais fecham com Escape;
- estado de filtro não depende só de cor.

## 21. Janelas menores

Prioridade de preservação:

1. Código;
2. Título;
3. Categorias;
4. Área/Versão.

Informações secundárias podem colapsar/reorganizar em janelas menores. Rolagem horizontal extensa deve ser evitada quando houver alternativa de layout.

## 22. Relação com Atendimentos

Esta tela **não** pesquisa:

- código do atendimento;
- ordem de serviço;
- cliente/solicitante operacional;
- equipamento;
- serial/patrimônio/MAC;
- resumo de serviço executado.

Esses filtros pertencem à futura lista de `Atendimentos`.

Pode existir futuramente ação contextual `Iniciar atendimento com este procedimento`, mas seu comportamento depende do Bloco 9 e não é consolidado nesta tela.

## 23. Fora do escopo

- editor do procedimento;
- leitor detalhado;
- lifecycle de Atendimento;
- busca operacional por equipamento/OS;
- taxonomia hierárquica;
- mecanismo FTS5 específico;
- código de produção.

## 24. Propostas para aprovação do PO

1. layout em tabela/lista compacta, não cards grandes;
2. filtros principais `Categoria` + `Área`, com `Status` somente para perfis que precisem;
3. categoria com seleção múltipla simples;
4. múltiplas categorias selecionadas usam semântica **OU** inicialmente;
5. categorias exibidas como labels/chips discretos na linha;
6. clicar no procedimento abre o leitor, não o editor;
7. ações administrativas ficam em menu contextual;
8. `Arquivar` em vez de `Excluir` na operação normal;
9. voltar do leitor restaura busca/filtros;
10. busca de `Processos` permanece separada da busca operacional de `Atendimentos`.

## 25. Pendências

- ordem padrão final;
- quantidade/tamanho de página;
- microcopy final;
- comportamento visual exato do multi-select de categorias;
- necessidade futura de filtros adicionais após uso real.

## 26. Critérios de aceite da especificação

- [x] categorização incorporada;
- [x] múltiplas categorias não exigem taxonomia complexa;
- [x] consulta documental separada de Atendimento;
- [x] autorização permanece Host-side;
- [x] nenhuma implementação criada;
- [ ] layout lista/tabela aprovado pelo PO;
- [ ] filtros aprovados pelo PO;
- [ ] semântica OU para categorias aprovada;
- [ ] navegação/ações aprovadas.

## 27. Casos de teste futuros

1. buscar por código;
2. buscar por título/termo;
3. filtrar uma categoria;
4. filtrar múltiplas categorias;
5. combinar categoria + área;
6. termo vindo do Dashboard permanece aplicado;
7. Funcionário não vê conteúdo não publicado;
8. Gerência/ADM veem ações somente quando autorizados;
9. item abre leitor correto;
10. voltar preserva contexto;
11. categoria arquivada não aparece em filtro normal;
12. evento de atualização reconcilia resultado;
13. estado vazio permite limpar filtros;
14. teclado/acessibilidade funcionam;
15. busca não retorna Atendimento/equipamento por engano.