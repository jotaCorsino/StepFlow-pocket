# Tela 06 — Editor de Processo + Categorias

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Permitir que usuários autorizados criem e mantenham procedimentos oficiais de forma estruturada, sem editor HTML livre e sem transformar a atividade documental em workflow burocrático.

O Editor deve refletir o mesmo modelo que o Leitor consome: metadados do procedimento, categorias, etapas ordenadas e blocos tipados.

## 2. Atores e permissões

Usuários com capacidades documentais apropriadas, normalmente:

- ADM;
- Gerência.

Funcionário/Técnico não recebe edição oficial por padrão.

O Client pode ocultar controles sem permissão, mas todas as operações são validadas novamente pelo Host.

## 3. Entrada

Fluxos principais:

```text
Processos → Novo processo
Processos → menu do item → Editar
Leitor → menu contextual → Editar
```

Ao abrir edição existente, o Editor recebe a revisão-base usada para controle otimista.

## 4. Layout proposto

```text
← Processos                       Editando PR-014
                                  [ Visualizar ] [ Salvar alterações ]

[ Informações ] [ Etapas ]

────────────────────────────────────────────────────────────

INFORMAÇÕES
Código              [ PR-014                         ]
Título               [ Configuração de VLAN          ]
Área/Departamento    [ TI ▾                          ]
Responsável          [ ...                           ]
Categorias           [ Redes ] [ Infraestrutura ] [+]
Versão exibida       [ 2.0                           ]

Objetivo
[ .................................................. ]

Pré-requisitos
[ .................................................. ]

Observações
[ .................................................. ]
```

Na área `Etapas`:

```text
Estrutura                         Etapa 3 — Configurar VLAN
────────────────────             ───────────────────────────
1. Preparação                     Título [..................]
2. Acesso ao switch               Introdução [.............]
3. Configurar VLAN  ← atual
4. Validar
[ + Nova etapa ]                  Blocos
                                  ┌ Passos numerados       ┐
                                  │ ...                    │
                                  └────────────────────────┘
                                  ┌ Comando             ⋮  ┐
                                  │ switchport ...         │
                                  └────────────────────────┘

                                  [ + Adicionar bloco ]
```

A sidebar global do StepFlow permanece. O painel `Estrutura` é contextual ao Editor e não vira nova navegação global.

## 5. Separação `Informações` × `Etapas`

### Proposta

Usar duas áreas internas simples:

- **Informações** — metadados gerais do procedimento;
- **Etapas** — estrutura e conteúdo técnico.

Isso evita uma tela única muito longa e mantém clara a separação entre identidade/metadados e conteúdo executável.

Trocar entre as duas áreas não salva automaticamente.

## 6. Campos de Informações

Exibir/editar conforme capacidade e contrato:

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Categorias;
- Versão exibida;
- Objetivo;
- Observações;
- Pré-requisitos.

`Status` editorial pode ser exibido quando relevante, mas não deve virar dropdown genérico se o lifecycle editorial não exigir edição direta do campo.

## 7. Categorias

Categorias são configuráveis e múltiplas.

### Proposta de UX

- multi-select simples com busca quando necessário;
- categorias já associadas aparecem como chips removíveis;
- editor seleciona categorias existentes autorizadas;
- não criar taxonomia/hierarquia;
- não criar categoria inline dentro do Editor na primeira versão;
- gestão/arquivamento de categorias fica em Configurações, conforme capacidade futura.

Isso evita misturar edição do procedimento com administração da taxonomia.

## 8. Área `Etapas`

O Editor apresenta uma lista ordenada de etapas e o conteúdo da etapa selecionada.

Ações propostas:

- `Nova etapa`;
- renomear título;
- editar introdução;
- reordenar etapa;
- remover etapa com confirmação quando possuir conteúdo;
- selecionar etapa sem salvar automaticamente.

A numeração exibida deriva da ordem das etapas, não precisa ser digitada manualmente.

## 9. Reordenação de etapas

Proposta:

- drag-and-drop pode existir como atalho visual;
- deve haver alternativa acessível `Mover para cima` / `Mover para baixo`;
- ordem só vira oficial após salvamento aceito pelo Host;
- reordenar localmente não cria lock.

## 10. Blocos tipados

`Adicionar bloco` oferece somente tipos aprovados:

- Parágrafo (`paragraph`);
- Passos numerados (`numbered_steps`);
- Checklist (`checklist`);
- Observação (`note`);
- Alerta (`warning`);
- Comando (`command`);
- Código (`code`).

Não oferecer HTML arbitrário como fonte de verdade.

## 11. Edição dos blocos

Cada bloco possui:

- tipo claramente identificável;
- conteúdo próprio do tipo;
- ordem dentro da etapa;
- menu contextual discreto;
- ações de mover para cima/baixo;
- remover com confirmação proporcional quando houver risco de perda.

Drag-and-drop pode complementar, nunca substituir, as ações acessíveis.

## 12. Passos numerados

O editor deve permitir:

- criar passos;
- criar subpassos em hierarquia limitada e compreensível;
- reordenar;
- editar texto;
- remover.

A numeração visual é derivada da estrutura, não texto manual obrigatório.

Não criar editor de árvore genérico além do necessário aos passos/subpassos.

## 13. Checklist documental

O bloco Checklist edita somente a **definição dos itens do procedimento**.

No Editor de Processo não existem campos como:

- concluído;
- executado por;
- data de execução;
- percentual concluído.

Esses conceitos pertencem ao Bloco 9/Atendimento.

## 14. Comando e Código

- fonte monoespaçada;
- preservar espaços/quebras relevantes;
- área de edição apropriada ao tamanho;
- preview final pertence ao Leitor;
- não executar comandos diretamente pelo StepFlow.

## 15. Salvamento — proposta central

Usar **salvamento explícito**, não autosave silencioso inicial.

Razões:

- cada save aceito cria revisão imutável;
- evita gerar revisões excessivas a cada tecla;
- torna o momento da escrita oficial previsível;
- combina com controle otimista de concorrência.

Enquanto houver alterações locais não salvas, indicar `Alterações não salvas` de forma discreta.

## 16. Revisões e `base_revision`

Ao abrir um procedimento existente, o Client mantém a revisão-base recebida.

Ao salvar:

```text
Client envia estado estruturado + base_revision
        ↓
Host valida autorização/estrutura/revisão
        ↓
writer/fila coordenada
        ↓
transação
        ↓
nova process_revision imutável
        ↓
current_revision_id atualizado
        ↓
resposta de sucesso + evento pós-commit
```

Um save aceito nunca altera uma revisão antiga no lugar.

## 17. Conflito `409 REVISION_CONFLICT`

Se outro usuário salvar antes:

- nunca sobrescrever automaticamente;
- manter as alterações locais do usuário visíveis/em memória;
- exibir aviso claro de que existe uma versão mais recente;
- bloquear tentativa de “forçar” overwrite silencioso;
- permitir consultar/recarregar a versão atual conscientemente;
- antes de descartar alterações locais para recarregar, pedir confirmação explícita.

A primeira versão não precisa implementar merge automático de documentos.

## 18. Evento de atualização durante edição

Se WebSocket indicar alteração do mesmo procedimento enquanto há edição aberta:

- não substituir os campos locais;
- mostrar banner discreto `Este processo foi atualizado por outro usuário.`;
- informar que o próximo save pode exigir reconciliação;
- reconsulta pode buscar metadados da nova revisão sem apagar o estado local;
- conflito final continua decidido pelo Host no save.

## 19. Saída com alterações não salvas

Ao tentar:

- voltar para Processos;
- abrir Leitor;
- trocar de rota relevante;
- fechar janela do Client;

com mudanças não salvas, mostrar confirmação simples:

`Há alterações não salvas. Deseja sair e descartá-las?`

Não persistir rascunho local oculto sem requisito aprovado.

## 20. Visualizar

### Proposta

A ação `Visualizar` abre o Leitor usando a **última revisão já salva**.

Não prometer preview de alterações ainda não salvas nesta primeira versão. Isso reduz duplicação de renderer/estado e deixa claro o que já é oficial no Host.

Se futuramente houver valor em preview local antes do save, pode ser adicionado como capacidade separada.

## 21. Publicação — direção mínima

O modelo já distingue revisão atual e revisão publicada.

Proposta de UX:

- `Salvar alterações` cria/atualiza a revisão atual por nova revisão imutável;
- usuários com capacidade de publicação recebem ação separada `Publicar revisão atual`;
- publicação nunca acontece implicitamente só porque houve save;
- não criar fluxo complexo de aprovação/revisores nesta primeira versão;
- versão publicada deve ser identificável no Leitor/Histórico.

Os nomes finais dos estados editoriais permanecem subordinados ao contrato de lifecycle vigente; a Tela 06 não inventa estados extras.

## 22. Novo processo

Ao criar:

- formulário começa sem identidade persistida;
- Código/Título e demais campos obrigatórios seguem validação do Host;
- primeira gravação bem-sucedida cria `process_id` estável e primeira revisão;
- processo não é publicado automaticamente apenas por ser criado;
- `Cancelar` antes do primeiro save não cria registro oficial.

## 23. Validações

Host é autoridade para:

- unicidade de Código;
- campos obrigatórios;
- limites/tipos de conteúdo;
- categorias válidas/ativas;
- capacidade do ator;
- revisão-base;
- integridade estrutural das etapas/blocos.

Client replica validações simples para UX, mas não substitui o Host.

## 24. Mensagens

Preferir mensagens curtas:

- `Alterações salvas.`;
- `Há alterações não salvas.`;
- `Este processo foi atualizado por outro usuário.`;
- `Não foi possível salvar. Revise os campos indicados.`;
- `A revisão atual foi publicada.`

Não expor SQL, stack trace, paths internos ou IDs técnicos desnecessários.

## 25. Estados da interface

### Loading

Estrutura do Editor carregada sem mostrar dados antigos como atuais.

### Novo/vazio

Exibir formulário limpo e primeira ação orientada.

### Erro de validação

Marcar campos/blocos correspondentes e manter todo o restante editável.

### Sem permissão

Não entrar em modo editável; oferecer retorno seguro ao Leitor/Processos.

### Host indisponível

Não permitir salvar nem sugerir que mudanças locais foram confirmadas. Manter estado local enquanto a tela estiver aberta e comunicar indisponibilidade.

### Conflito

Aplicar seção 17.

### Processo arquivado enquanto edita

Não continuar salvando silenciosamente. Reconsultar Host e explicar mudança de estado/permissão.

## 26. Autorização

Capacidades podem separar:

- criar procedimento;
- editar conteúdo/metadados;
- publicar;
- arquivar;
- gerenciar categorias.

Perfil é preset; autorização final é granular e Host-side.

## 27. Persistência

Persistência oficial inclui apenas saves aceitos pelo Host.

- nenhum acesso direto ao SQLite;
- nenhuma fila local offline;
- nenhuma revisão “aceita mas ainda não commitada”;
- evento somente após commit.

## 28. Contratos Client ↔ Host necessários

Conceitualmente:

1. carregar processo/revisão editável;
2. listar categorias autorizadas;
3. validar/salvar revisão com `base_revision`;
4. criar novo processo;
5. publicar revisão atual quando permitido;
6. receber capacidades da sessão;
7. receber/reconsultar eventos de atualização;
8. eventualmente arquivar por contrato próprio.

Nomes finais dos endpoints pertencem à implementação.

## 29. Acessibilidade e teclado

- labels reais para todos os campos;
- ordem de foco previsível;
- ações de mover etapa/bloco por teclado;
- drag-and-drop não é único meio de reordenar;
- menus fecham com Escape;
- erros são associados aos campos;
- chips de categoria possuem remoção acessível;
- foco não é perdido após adicionar/remover bloco.

## 30. Janelas menores

Desktop Windows é prioridade.

- `Informações` pode virar uma coluna;
- painel Estrutura pode reduzir largura, mas não esconder controle essencial;
- conteúdo central preserva área mínima de edição;
- toolbars podem quebrar em duas linhas;
- não criar versão mobile/hamburger sem necessidade demonstrada.

## 31. Fora do escopo

- HTML/WYSIWYG genérico;
- Markdown como fonte oficial obrigatória;
- autosave a cada tecla;
- merge automático de conflitos;
- colaboração caractere a caractere;
- criação inline de categorias;
- execução real de comandos;
- persistência de checklist operacional;
- workflow editorial complexo;
- código Tauri/Host.

## 32. Propostas para aprovação do PO

1. separar Editor em `Informações` e `Etapas`;
2. usar painel contextual `Estrutura` dentro de Etapas;
3. usar salvamento explícito, sem autosave inicial;
4. cada save aceito gera nova revisão imutável;
5. usar blocos tipados, sem HTML livre;
6. categorias são selecionadas no Editor, mas gerenciadas fora dele;
7. reordenação por drag-and-drop + alternativa mover acima/abaixo;
8. conflito preserva edição local e nunca faz overwrite automático;
9. `Visualizar` mostra última revisão salva, não estado local não salvo;
10. `Salvar` e `Publicar revisão atual` são ações distintas, sem workflow editorial complexo.

## 33. Pendências

- campos obrigatórios finais além dos já consolidados;
- limites de tamanho por campo/bloco;
- microcopy final;
- aparência exata do seletor múltiplo de categorias;
- apresentação final do estado publicado/rascunho;
- necessidade futura de preview de alterações ainda não salvas.

## 34. Critérios de aceite da especificação

- [x] modelo estruturado respeitado;
- [x] categorias incorporadas;
- [x] revisão otimista respeitada;
- [x] conflito não sobrescreve;
- [x] checklist documental separado do operacional;
- [x] publicação não cria workflow extra automaticamente;
- [x] nenhum código de produção criado;
- [ ] layout Informações/Etapas aprovado pelo PO;
- [ ] salvamento explícito aprovado;
- [ ] UX de blocos/reordenação aprovada;
- [ ] separação Salvar/Publicar aprovada.

## 35. Casos de teste futuros

1. criar processo e cancelar antes de salvar;
2. criar primeira revisão;
3. editar metadados/categorias;
4. adicionar/reordenar/remover etapas;
5. adicionar cada tipo de bloco;
6. reordenar por teclado;
7. sair com alterações não salvas;
8. salvar com base atual;
9. receber `409` após edição concorrente;
10. receber evento de atualização enquanto edita;
11. visualizar última revisão salva;
12. publicar revisão atual com permissão;
13. negar publicação sem permissão;
14. perder Host antes do save;
15. validar janela menor/acessibilidade.
