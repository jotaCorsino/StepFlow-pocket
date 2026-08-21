# Tela 04 — Lista e Pesquisa de Processos

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Aprovação:** 2026-08-21

## 1. Objetivo

Permitir localizar rapidamente procedimentos oficiais por código, título, termo, área e categoria, com acesso direto ao leitor e às ações de manutenção documental quando autorizadas.

Esta tela pesquisa **documentação/procedimentos**. Registros de serviço, OS, equipamentos, cliente, serial, patrimônio e MAC pertencem à área `Atendimentos`.

## 2. Atores

- ADM;
- Gerência;
- Funcionário.

Todos podem consultar procedimentos autorizados. Criação, edição, publicação e arquivamento dependem das capacidades efetivas da sessão.

## 3. Entrada

- item `Processos` da sidebar;
- busca rápida do Dashboard com termo já aplicado;
- retorno contextual de outras telas.

## 4. Estrutura aprovada

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

A visualização padrão usa **lista/tabela compacta**, não cards grandes.

## 5. Busca textual

A busca aceita, no mínimo:

- Código;
- Título;
- termos compatíveis com os campos indexados definidos pelo Host.

Regras:

- termo vindo do Dashboard aparece preenchido e aplicado;
- busca vazia mostra listagem padrão autorizada;
- execução por Enter/ação explícita ou debounce controlado, sem tempestade de requisições;
- não pesquisar dados operacionais de `Atendimentos` nesta tela.

## 6. Filtro por categorias

Categorias são configuráveis e um procedimento pode possuir múltiplas.

Aprovado:

- seleção de uma ou mais categorias;
- sem árvore hierárquica na primeira versão;
- múltiplas categorias usam semântica **OU** inicialmente: o procedimento aparece se pertencer a qualquer categoria selecionada;
- filtros ativos ficam visíveis e removíveis;
- categoria arquivada não aparece como opção normal de novo filtro.

Não criar construtor avançado `E/OU` sem necessidade futura aprovada.

## 7. Outros filtros

### Área/Departamento

Filtro simples pelas áreas existentes nos procedimentos visíveis.

### Status

Somente para perfis que realmente precisem trabalhar com estados documentais diferentes. Funcionário recebe essencialmente conteúdo publicado/disponível; ADM/Gerência podem receber filtro adicional conforme capacidades.

## 8. Ordenação

Direção consolidada:

- ordem estável e previsível;
- sem busca textual: Código ou Título como ordenação padrão final a fechar visualmente;
- com busca: relevância do mecanismo + desempate estável;
- Código, Título e atualização podem ser opções se úteis.

A escolha exata da ordenação padrão não altera o domínio e pode ser fechada durante validação visual.

## 9. Colunas prioritárias

- Código;
- Título;
- Categoria(s);
- Área/Departamento;
- Versão exibida.

Secundários, quando úteis:

- Responsável;
- data de atualização/publicação;
- status para perfis administrativos.

Evitar excesso de colunas.

## 10. Exibição das categorias

- labels/chips discretos;
- excesso pode aparecer como `+N`/expansão acessível;
- cor não é a única forma de distinção;
- não exigir uma cor fixa exclusiva por categoria.

## 11. Abertura do procedimento

A ação principal da linha abre o **Leitor em formato livro**.

- clique/título abre o leitor;
- teclado oferece ação equivalente;
- usuário com permissão de edição não é jogado diretamente no editor.

Leitura continua sendo o caminho principal.

## 12. Ações administrativas

### Novo processo

Aparece somente com capacidade de criação.

### Menu contextual por item

Quando autorizado, pode incluir:

- Editar;
- Histórico;
- Publicar, quando aplicável;
- Arquivar.

Na operação normal usar **Arquivar**, não `Excluir`. Ações destrutivas ou irreversíveis exigem confirmação proporcional.

## 13. Preservação do contexto

Ao retornar do leitor, preservar preferencialmente:

- termo de busca;
- categorias selecionadas;
- área/status;
- ordenação;
- página/posição quando aplicável.

Não obrigar o técnico a refazer a pesquisa.

## 14. Estados

### Loading

Preservar estrutura da tela; evitar spinner por linha.

### Nenhum resultado

`Nenhum processo encontrado com os filtros atuais.`

Oferecer `Limpar filtros` quando aplicável.

### Nenhum processo cadastrado

- Funcionário: informar ausência de procedimentos disponíveis;
- perfil com criação: pode receber ação contextual `Novo processo`.

### Host indisponível

Seguir estado transversal do Shell; não apresentar cache como atual sem indicação.

### Mudança de permissão

Reconciliar resultados e ações; Host continua autoridade.

## 15. Atualização em tempo real

Eventos de alteração/publicação/arquivamento/categoria provocam invalidação e reconsulta do estado relevante.

Preservar filtros e evitar reposicionamento agressivo enquanto o usuário interage.

## 16. Paginação / volume

- não carregar quantidade indefinida de registros;
- paginação simples ou carregamento incremental controlado;
- tamanho final de página será definido após estimativa/medição real;
- filtros e termo persistem entre páginas;
- não exigir virtualização complexa sem evidência.

## 17. Contratos conceituais

A tela depende de:

1. consulta paginada de procedimentos autorizados;
2. busca textual;
3. filtro por categoria;
4. filtro por área/status quando aplicável;
5. lista de categorias disponíveis;
6. capacidades da sessão;
7. abertura por `process_id` estável;
8. eventos/reconsulta após mudanças.

Nomes de rotas ficam para implementação.

## 18. Segurança e autorização

- Host filtra resultados;
- Funcionário não recebe conteúdo não autorizado;
- ocultar ações no Client não substitui autorização;
- categoria não concede acesso por si só;
- manipulação de filtros não permite leitura indevida.

## 19. Concorrência

A lista é leitura predominante. Edição/publicação/arquivamento seguem revisão otimista e regras gerais de concorrência; a lista não cria locking próprio.

## 20. Acessibilidade e teclado

- busca/filtros com labels acessíveis;
- filtros operáveis por teclado;
- tabela/lista semanticamente adequada;
- ação de abrir identificável;
- foco visível;
- menu contextual fecha com Escape;
- estado de filtro não depende só de cor.

## 21. Janelas menores

Prioridade de preservação:

1. Código;
2. Título;
3. Categorias;
4. Área/Versão.

Informações secundárias podem colapsar/reorganizar. Evitar rolagem horizontal extensa quando houver alternativa.

## 22. Separação de Atendimentos

Esta tela não pesquisa:

- código de atendimento;
- ordem de serviço;
- cliente/solicitante operacional;
- equipamento;
- serial/patrimônio/MAC;
- resumo de serviço executado.

Esses campos pertencem à futura lista de `Atendimentos`.

Uma futura ação `Iniciar atendimento com este procedimento` depende do Bloco 9 e não é definida aqui.

## 23. Fora do escopo

- editor detalhado;
- leitor detalhado;
- lifecycle de Atendimento;
- busca operacional por equipamento/OS;
- taxonomia hierárquica;
- escolha de FTS5;
- código de produção.

## 24. Decisões consolidadas nesta tela

1. lista/tabela compacta;
2. filtros principais `Categoria` + `Área`, com `Status` apenas quando útil ao perfil;
3. categoria com seleção múltipla simples;
4. semântica **OU** para múltiplas categorias;
5. categorias como labels/chips discretos;
6. abrir procedimento leva ao leitor, não ao editor;
7. ações administrativas em menu contextual;
8. `Arquivar` em vez de `Excluir` na operação normal;
9. retorno do leitor preserva busca/filtros;
10. busca documental de `Processos` separada da busca operacional de `Atendimentos`.

## 25. Pendências visuais/de implementação

- ordenação padrão final;
- tamanho de página;
- microcopy final;
- aparência exata do multiselect de categorias;
- filtros adicionais apenas se uso real justificar.

## 26. Critérios de aceite

- [x] categorização incorporada;
- [x] múltiplas categorias sem taxonomia complexa;
- [x] consulta documental separada de Atendimento;
- [x] autorização Host-side;
- [x] layout lista/tabela aprovado;
- [x] filtros aprovados;
- [x] semântica OU aprovada;
- [x] navegação/ações aprovadas;
- [x] nenhuma implementação criada.

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
12. evento relevante reconcilia resultado;
13. estado vazio permite limpar filtros;
14. teclado/acessibilidade funcionam;
15. busca não retorna Atendimento/equipamento por engano.
