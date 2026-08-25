# Tela 12 — Configurações + Categorias

## 1. Identificação

- código/nome da tela: Tela 12 — Configurações + Categorias;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX);
- data da consolidação: 2026-08-25.

## 2. Objetivo

Centralizar configurações administrativas simples que pertencem à empresa, sem transformar o StepFlow em um painel técnico de servidor.

Nesta primeira versão, a Tela 12 cobre somente:

1. **Empresa** — identidade corporativa utilizada no Shell e em documentos gerados;
2. **Categorias** — criação, edição, consulta, arquivamento e reativação das categorias configuráveis dos procedimentos.

Backup/restauração permanece na Tela 13. Exportação/impressão e template final da ficha compacta permanecem na Tela 14/Bloco 10.

## 3. Princípios consolidados

- `Configurações` aparece na sidebar somente quando a sessão possui alguma capacidade administrativa aplicável;
- Host é a autoridade final de autorização;
- identidade da empresa é configurável e não deve ser duplicada manualmente em cada documento;
- categorias são configuráveis pela empresa, não hardcoded;
- um procedimento pode usar múltiplas categorias;
- categorias são simples, sem hierarquia/taxonomia avançada inicialmente;
- categoria pode ser arquivada preservando histórico;
- criação/arquivamento de categorias acontece fora do Editor de Processo;
- não existe exclusão física normal de categoria como primeira opção;
- parâmetros ainda marcados como `PENDENTE` não são convertidos silenciosamente em autorização.

## 4. Atores e autorização

A Tela 12 é controlada por capacidades, não apenas pelo nome do preset.

### ADM

Possui capacidade para alterar configuração da empresa conforme matriz já consolidada.

### Gerência

A possibilidade de alterar configuração da empresa continua **PENDENTE** no contrato de autenticação/autorização.

A Tela 12 não assume que Gerência pode nem que não pode editar a seção `Empresa` até essa decisão ser fechada.

### Gestão de categorias

A capacidade exata/preset responsável por criar, editar, arquivar e reativar categorias permanece pendente de fechamento da matriz correspondente.

A UX deve suportar:

- sessão com acesso somente à seção `Empresa`;
- sessão com acesso somente a `Categorias`;
- sessão com acesso às duas seções;
- sessão sem nenhuma dessas capacidades, caso em que `Configurações` não aparece na sidebar.

Funcionário não recebe essas capacidades por padrão.

Ocultar controles no Client não substitui validação Host-side.

## 5. Entrada e navegação

Fluxo:

```text
Shell/sidebar
→ Configurações
→ Tela 12
```

A Tela 12 usa navegação local simples:

```text
Configurações

[ Empresa ] [ Categorias ]
```

Regras:

- não criar nova sidebar global;
- a aba/seção não autorizada fica oculta;
- se a sessão possuir apenas uma das capacidades, abrir diretamente a seção disponível;
- mudança de aba não salva automaticamente alterações.

## 6. Estrutura visual aprovada

### 6.1 Empresa

```text
Configurações
[ Empresa ] [ Categorias ]

IDENTIDADE DA EMPRESA

[ logo atual / placeholder ]
[ Escolher logo ] [ Remover logo* ]

Nome da empresa      [ Empresa Exemplo Ltda.          ]
Contato              [ (11) 3333-4444 / Ramal 123    ]
Site                 [ www.empresa.com.br             ]
E-mail               [ suporte@empresa.com.br         ]

Como será utilizado:
• sidebar do StepFlow, quando aplicável;
• cabeçalhos de exportações/documentos;
• ficha compacta de atendimento/equipamento.

                                      [ Salvar alterações ]
```

`*` aparece somente quando há logo configurado.

### 6.2 Categorias

```text
Configurações
[ Empresa ] [ Categorias ]

CATEGORIAS                                      [ Nova categoria ]

[ Buscar categoria... ]          [ Estado: Ativas ▾ ]

Categoria                    Estado
Infraestrutura               Ativa                         [ ⋯ ]
Manutenção                   Ativa                         [ ⋯ ]
Service Desk                 Ativa                         [ ⋯ ]
Categoria antiga             Arquivada                     [ ⋯ ]
```

Menu contextual conforme estado/capacidade:

```text
Editar
Arquivar
```

ou:

```text
Editar
Reativar
```

## 7. Seção Empresa

A seção `Empresa` administra somente a identidade institucional necessária ao produto.

Campos iniciais aprovados:

- logo;
- nome da empresa;
- contato;
- site;
- e-mail.

Não incluir sem requisito posterior:

- CNPJ/CPF;
- endereço fiscal completo;
- dados bancários;
- faturamento;
- inscrição estadual/municipal;
- dados de CRM;
- campos jurídicos genéricos;
- configurações técnicas de Host/rede.

O StepFlow não deve crescer para ERP/CRM por conveniência da tela de configurações.

## 8. Logo da empresa

### 8.1 Visualização

Mostrar preview preservando proporção e sem distorção.

Na ausência de logo:

- usar placeholder discreto na Tela 12;
- Shell/documentos continuam funcionais sem imagem quebrada.

### 8.2 Escolher ou substituir

`Escolher logo` abre seletor local do Client.

Após escolha:

- mostrar preview local;
- não considerar o arquivo persistido antes do save;
- aplicar oficialmente somente após confirmação do Host.

### 8.3 Remover

Quando houver logo configurado, `Remover logo` marca a remoção para o próximo `Salvar alterações`.

### 8.4 Segurança

- Host valida conteúdo/tipo real do arquivo;
- não confiar apenas na extensão;
- aceitar somente formatos explicitamente suportados;
- limites de bytes/dimensões serão fechados antes da implementação;
- não aceitar caminho arbitrário controlado pelo usuário como fonte persistente;
- armazenar como arquivo controlado pelo Host;
- não executar conteúdo ativo.

Editor avançado/cropper não é requisito da primeira versão.

## 9. Nome, contato, site e e-mail

### Nome da empresa

Texto curto destinado à identidade visual e aos documentos.

### Contato

Campo curto para uma ou mais formas de contato operacionais, como telefone, ramal ou WhatsApp, sem criar cadastro complexo de canais na primeira versão.

### Site

Campo textual curto. Validação final de formato será definida antes da implementação.

### E-mail

Campo textual curto com validação de formato adequada antes da implementação.

### Limites

Limites numéricos finais devem ser fechados antes da implementação correspondente.

Para campos que entram na ficha compacta, o Bloco 10 deve confirmar limites coerentes com o contrato de **no máximo uma página A4**.

## 10. Uso centralizado da identidade

A identidade confirmada pelo Host é a fonte única para as superfícies que a utilizam.

Usos já aprovados:

- logo discreto no topo esquerdo do Shell;
- cabeçalho da ficha compacta;
- identidade de exportações/documentos quando aplicável.

A Tela 12 não define o template de impressão. Ela administra os dados que serão consumidos pelo Bloco 10.

Após save confirmado, o Shell pode refazer consulta e atualizar o logo sem reiniciar o Client quando tecnicamente aplicável.

## 11. Salvamento da Empresa

Salvamento explícito, sem autosave inicial.

```text
editar identidade
→ Salvar alterações
→ Host valida sessão + capacidade + campos/arquivo
→ writer/transaction quando aplicável
→ commit
→ evento/refetch
→ UI confirma sucesso
```

Regras:

- botão habilitado somente com mudança válida;
- sair com alteração não salva pede confirmação;
- nenhuma alteração aparece como oficial antes do commit;
- conflito concorrente não usa `last write wins` silencioso.

## 12. Gestão de Categorias

Categorias permanecem entidades simples e configuráveis.

A seção oferece:

- listar;
- buscar por nome;
- filtrar por estado;
- criar;
- editar nome;
- arquivar;
- reativar.

Não incluir inicialmente:

- árvore hierárquica;
- categoria pai/filha;
- cor obrigatória;
- ícone obrigatório;
- regras automáticas de classificação;
- exclusão física normal;
- ações em massa.

## 13. Lista de Categorias

Tabela/lista compacta, coerente com as demais superfícies administrativas.

Colunas iniciais:

- nome;
- estado (`Ativa` / `Arquivada`).

Busca:

- nome da categoria.

Filtro inicial:

- estado.

Direção inicial de ordenação:

- categorias ativas em ordem alfabética;
- arquivadas conforme filtro/estado selecionado.

A ordenação exata é detalhe visual de baixa criticidade e pode ser ajustada na implementação sem alterar o modelo.

## 14. Nova categoria

Ação `Nova categoria` aparece somente com capacidade correspondente.

Fluxo:

```text
Nova categoria

Nome  [ Infraestrutura                  ]

                     [ Cancelar ] [ Criar categoria ]
```

Não criar categoria diretamente dentro do Editor de Processo.

A categoria só aparece como disponível após commit confirmado pelo Host.

## 15. Editar categoria

A edição inicial altera somente o nome da categoria.

Não inventar descrição, cor, ícone ou taxonomia sem requisito.

Validações consolidadas como princípio:

- nome obrigatório;
- trim/normalização pelo Host;
- limite textual explícito antes da implementação;
- impedir duplicidades equivalentes após normalização para evitar duas categorias visualmente indistinguíveis.

A regra técnica exata de comparação/case-folding será fechada na implementação do modelo de dados, preservando o princípio de não criar duplicatas ambíguas.

## 16. Arquivar categoria

Arquivar é a ação normal para retirar uma categoria de uso futuro sem destruir histórico.

Confirmação:

`Arquivar esta categoria? Ela deixará de estar disponível para novas associações, mas continuará preservada no histórico.`

Regras:

- não apagar revisões históricas;
- não reescrever procedimentos antigos;
- categoria arquivada continua consultável na gestão;
- categoria arquivada não aparece como opção normal para nova associação;
- histórico que já utiliza a categoria permanece legível.

A regra exata para um procedimento atual que já carrega uma categoria posteriormente arquivada ao criar **nova revisão** deve ser fechada junto do contrato de dados/Editor antes da implementação. A Tela 12 não autoriza migração silenciosa nem remoção automática.

## 17. Reativar categoria

Categoria arquivada pode ser reativada quando a sessão possuir capacidade correspondente.

Após commit:

- volta ao estado `Ativa`;
- torna-se novamente elegível para novas associações;
- histórico anterior permanece inalterado.

## 18. Exclusão física

Não oferecer `Excluir categoria` como operação administrativa normal.

Motivos:

- preservar histórico;
- evitar quebrar revisões e filtros antigos;
- manter rastreabilidade.

Qualquer manutenção física excepcional de dados, se algum dia necessária, não pertence à UX normal desta tela.

## 19. Concorrência

Empresa e categorias são mutações oficiais do Host.

Regras:

- controle otimista equivalente quando aplicável;
- sem overwrite silencioso;
- edição aberta não é substituída por evento remoto;
- em conflito, preservar conteúdo local e oferecer reconsulta;
- ações só confirmam sucesso após commit.

Conflitos de identidade da empresa e conflitos de categoria são tratados como entidades/ações independentes.

## 20. Eventos em tempo real

Eventos pós-commit podem sinalizar:

- identidade da empresa alterada;
- logo alterado/removido;
- categoria criada;
- categoria renomeada;
- categoria arquivada/reativada.

Clients relevantes fazem refetch/reconciliação.

Exemplos:

- Shell atualiza logo confirmado;
- Editor aberto pode atualizar a lista de categorias quando não houver conflito local;
- lista de Processos pode refazer filtros sem substituir silenciosamente o contexto do usuário.

## 21. Auditoria

Alterações administrativas devem ser auditáveis de forma proporcional:

- mudança da identidade da empresa;
- alteração/remoção de logo;
- criação de categoria;
- renomeação;
- arquivamento/reativação.

Auditoria não armazena conteúdo binário desnecessário nem dados secretos.

## 22. Estados da interface

### Carregando

Preservar estrutura da seção enquanto os dados são consultados.

### Sem permissão

A aba/seção correspondente fica oculta. Acesso direto manipulado retorna permissão negada sem expor dados administrativos.

### Host indisponível

Mostrar indisponibilidade sem IP/porta/path e impedir mutações.

### Nenhuma categoria

`Nenhuma categoria cadastrada.`

Se autorizado, manter `Nova categoria` disponível.

### Busca sem resultado

`Nenhuma categoria encontrada com os critérios informados.`

### Alteração concorrente

Informar atualização e exigir reconciliação; não substituir formulário local silenciosamente.

## 23. Acessibilidade

- abas/seções com semântica e foco corretos;
- labels visíveis para campos da empresa;
- preview do logo com nome acessível;
- ações de logo acionáveis por teclado;
- tabela de categorias semanticamente identificada;
- menu contextual acionável por teclado;
- estado Ativa/Arquivada não depende apenas de cor;
- erros ligados aos campos correspondentes;
- diálogos devolvem foco adequadamente.

## 24. Janelas menores

Desktop Windows permanece alvo principal.

Em janela menor suportada:

- campos da empresa passam para uma coluna;
- preview do logo permanece compacto;
- lista de categorias preserva nome + estado;
- ações podem migrar para menu contextual;
- não criar experiência mobile/hamburger antecipadamente.

## 25. Fora do escopo

- Backup/restauração — Tela 13;
- exportação/impressão e preview da ficha — Tela 14/Bloco 10;
- template A4 final;
- configuração de IP, porta, hostname ou caminho SMB pela UI comum;
- configuração de banco SQLite;
- configuração de fila/writer/WAL;
- start/stop técnico do Host;
- parâmetros de autenticação/sessão;
- dados fiscais/financeiros/CRM;
- taxonomia hierárquica de categorias;
- exclusão física normal de categorias;
- ações em massa;
- implementação funcional.

## 26. Pendências preservadas

A consolidação da Tela 12 **não resolve**:

- se Gerência pode alterar configuração da empresa;
- preset exato autorizado a gerir categorias;
- limites numéricos de logo/nome/contato/site/e-mail;
- limite numérico dos textos que entram no template A4;
- regra de nova revisão de procedimento que ainda referencia categoria arquivada;
- template/tecnologia de exportação.

Esses pontos devem ser fechados no bloco/documento apropriado antes da implementação correspondente.

## 27. Decisões consolidadas pelo PO

1. uma única Tela 12 com navegação local `Empresa` + `Categorias`;
2. somente seções autorizadas pelas capacidades da sessão são exibidas;
3. autorização da Gerência para configuração da empresa permanece `PENDENTE`;
4. `Empresa` contém inicialmente logo, nome, contato, site e e-mail;
5. logo pode ser escolhido, substituído ou removido, com preview antes do save;
6. identidade salva alimenta Shell e documentos/ficha, sem duplicação manual;
7. salvamento da identidade é explícito, sem autosave;
8. categorias usam lista compacta com busca por nome e filtro por estado;
9. categoria possui inicialmente somente nome + estado, sem cor/ícone/hierarquia;
10. criar/editar categoria ocorre fora do Editor de Processo;
11. arquivar substitui exclusão física normal e categoria pode ser reativada;
12. categoria arquivada deixa de ser opção normal para novas associações, preservando histórico;
13. categorias duplicadas/visualmente equivalentes após normalização devem ser impedidas;
14. Backup/Restore e Exportação/Impressão permanecem fora da Tela 12.

## 28. Critérios de aceite

- [x] PO aprovou navegação local `Empresa` + `Categorias`;
- [x] PO aprovou campos iniciais da identidade da empresa;
- [x] PO aprovou fluxo de logo;
- [x] PO aprovou uso centralizado da identidade;
- [x] PO aprovou gestão simples de categorias;
- [x] PO aprovou arquivar/reativar em vez de excluir;
- [x] PO aprovou ausência de hierarquia/cor/ícone inicialmente;
- [x] pendência de Gerência permanece pendente;
- [x] permissão exata de categorias permanece pendente;
- [x] Tela 13 não foi antecipada;
- [x] Tela 14/Bloco 10 não foi antecipada;
- [x] Host permanece autoridade final;
- [x] nenhuma implementação funcional foi criada.
