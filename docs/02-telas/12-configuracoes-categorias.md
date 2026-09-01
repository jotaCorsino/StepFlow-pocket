# Tela 12 — Configurações + Categorias

## 1. Identificação

- código/nome da tela: Tela 12 — Configurações + Categorias;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX), sincronizado com D12.62–D12.65;
- atualização: 2026-09-01.

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
- limites e capacidades consolidados no Bloco 12 devem ser aplicados no Host, não inferidos pela UI.

## 4. Atores e autorização

A Tela 12 é controlada por capacidades, não apenas pelo nome do preset.

### Empresa — D12.62

| Preset | Alterar configuração da empresa |
|---|---:|
| ADM | sim |
| Gerência | sim |
| Funcionário | não |

### Gestão de categorias

Conforme matriz consolidada de autenticação/autorização:

| Preset | Gerir categorias |
|---|---:|
| ADM | sim |
| Gerência | sim |
| Funcionário | não |

A UX continua preparada para capacidades granulares/personalizadas:

- sessão com acesso somente à seção `Empresa`;
- sessão com acesso somente a `Categorias`;
- sessão com acesso às duas seções;
- sessão sem nenhuma dessas capacidades, caso em que `Configurações` não aparece na sidebar.

Ocultar controles no Client não substitui validação Host-side.

## 5. Entrada e navegação

```text
Shell/sidebar
→ Configurações
→ Tela 12
```

Navegação local:

```text
Configurações

[ Empresa ] [ Categorias ]
```

Regras:

- não criar nova sidebar global;
- aba/seção não autorizada fica oculta;
- se a sessão possuir apenas uma capacidade, abrir diretamente a seção disponível;
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

Campos iniciais:

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

## 8. Logo da empresa — D12.64

### 8.1 Visualização

Mostrar preview preservando proporção e sem distorção.

Na ausência de logo:

- usar placeholder discreto;
- Shell/documentos continuam funcionais sem imagem quebrada.

### 8.2 Escolher ou substituir

`Escolher logo` abre seletor local do Client.

Após escolha:

- mostrar preview local;
- não considerar o arquivo persistido antes do save;
- aplicar oficialmente somente após confirmação do Host.

### 8.3 Remover

Quando houver logo configurado, `Remover logo` marca a remoção para o próximo `Salvar alterações`.

### 8.4 Segurança e limites

Baseline:

- PNG ou JPEG;
- máximo 2 MiB no upload;
- máximo 2048 × 2048 pixels após decode;
- Host valida magic/content, não apenas extensão;
- Host decodifica e reencoda para arquivo administrado seguro, removendo metadata desnecessária;
- rejeitar arquivo que não decodifica ou excede limites;
- SVG/conteúdo ativo não entra no baseline;
- não aceitar caminho arbitrário controlado pelo usuário como fonte persistente;
- ausência de logo é válida.

Editor avançado/cropper não é requisito inicial.

## 9. Nome, contato, site e e-mail — D12.63

| Campo | Regra |
|---|---|
| Nome da empresa | obrigatório, trim, 1–120 caracteres |
| Contato | opcional, trim, até 160 caracteres |
| Site | opcional, trim, até 200 caracteres |
| E-mail | opcional, trim, até 254 caracteres |

Regras:

- nenhum campo é truncado silenciosamente;
- `site` é identidade textual no baseline; se futuramente virar link, aceitar somente HTTP/HTTPS validado;
- e-mail usa validação prática de formato, sem regex desnecessariamente complexa;
- limites devem continuar coerentes com a Ficha compacta A4.

## 10. Uso centralizado da identidade

A identidade confirmada pelo Host é a fonte única para:

- logo discreto no topo esquerdo do Shell;
- cabeçalho da ficha compacta;
- identidade de exportações/documentos quando aplicável.

A Tela 12 não define o template de impressão. Após save confirmado, o Shell pode refazer consulta e atualizar o logo sem reiniciar o Client quando tecnicamente aplicável.

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

- botão habilitado somente com mudança válida;
- sair com alteração não salva pede confirmação;
- nenhuma alteração aparece como oficial antes do commit;
- conflito concorrente não usa `last write wins` silencioso.

## 12. Gestão de Categorias

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
- cor/ícone obrigatório;
- regras automáticas de classificação;
- exclusão física normal;
- ações em massa.

## 13. Lista de Categorias

Colunas iniciais:

- nome;
- estado (`Ativa` / `Arquivada`).

Busca por nome e filtro por estado. Direção inicial:

- ativas em ordem alfabética;
- arquivadas conforme filtro/estado selecionado.

Ordenação exata é detalhe visual de baixa criticidade.

## 14. Nova categoria

`Nova categoria` aparece somente com capacidade correspondente.

```text
Nova categoria

Nome  [ Infraestrutura                  ]

                     [ Cancelar ] [ Criar categoria ]
```

Não criar categoria diretamente dentro do Editor de Processo. A categoria só aparece como disponível após commit confirmado pelo Host.

## 15. Editar categoria

A edição inicial altera somente o nome.

Validações:

- nome obrigatório;
- trim/normalização pelo Host;
- limite textual explícito conforme schema/tela proprietária;
- impedir duplicidades equivalentes após normalização.

Não inventar descrição, cor, ícone ou taxonomia.

## 16. Arquivar categoria e nova revisão — D12.65

Arquivar retira a categoria de uso futuro sem destruir histórico.

Confirmação:

`Arquivar esta categoria? Ela deixará de estar disponível para novas associações, mas continuará preservada no histórico.`

Regras:

- não apagar revisões históricas;
- não reescrever procedimentos antigos;
- categoria arquivada continua consultável na gestão;
- não aparece como opção para nova associação;
- histórico que já utiliza a categoria permanece legível;
- se a revisão-base de um Procedimento já utiliza categoria arquivada, a nova revisão **preserva a associação por default**;
- Editor identifica `Arquivada` e informa que foi herdada;
- isso não bloqueia salvar/publicar;
- usuário autorizado pode remover a associação;
- depois de removida, não pode ser adicionada novamente enquanto a categoria permanecer arquivada;
- para voltar a associá-la, reativar primeiro;
- nunca remover/substituir automaticamente por outra categoria.

## 17. Reativar categoria

Após commit:

- volta a `Ativa`;
- torna-se elegível para novas associações;
- histórico anterior permanece inalterado.

## 18. Exclusão física

Não oferecer `Excluir categoria` como operação administrativa normal. Arquivamento preserva histórico e rastreabilidade.

## 19. Concorrência

Empresa e categorias são mutações oficiais do Host:

- controle otimista equivalente quando aplicável;
- sem overwrite silencioso;
- edição aberta não é substituída por evento remoto;
- em conflito, preservar conteúdo local e oferecer reconsulta;
- ações só confirmam sucesso após commit.

## 20. Eventos em tempo real

Eventos pós-commit podem sinalizar:

- identidade/logo alterado/removido;
- categoria criada/renomeada/arquivada/reativada.

Clients relevantes fazem refetch/reconciliação sem substituir silenciosamente edição local.

## 21. Auditoria

Auditar proporcionalmente:

- mudança de identidade;
- alteração/remoção de logo;
- criação/renomeação/arquivamento/reativação de categoria.

Auditoria não armazena binário desnecessário nem segredo.

## 22. Estados da interface

### Carregando

Preservar estrutura da seção.

### Sem permissão

A aba/seção fica oculta; acesso direto manipulado retorna permissão negada sem expor dados.

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

- abas/seções com semântica/foco corretos;
- labels visíveis;
- preview do logo com nome acessível;
- ações de logo e menu contextual acionáveis por teclado;
- estado Ativa/Arquivada não depende apenas de cor;
- erros ligados aos campos;
- diálogos devolvem foco adequadamente.

## 24. Janelas menores

Desktop Windows permanece alvo principal.

- campos da empresa passam para uma coluna;
- preview do logo permanece compacto;
- lista preserva nome + estado;
- ações podem migrar para menu contextual;
- não criar experiência mobile/hamburger antecipadamente.

## 25. Fora do escopo

- Backup/restauração — Tela 13;
- exportação/impressão/preview da ficha — Tela 14/Bloco 10;
- template A4 final;
- IP/porta/hostname/path SMB pela UI comum;
- configuração de SQLite/fila/WAL;
- start/stop técnico do Host;
- parâmetros de autenticação/sessão;
- dados fiscais/financeiros/CRM;
- taxonomia hierárquica;
- exclusão física normal;
- ações em massa;
- implementação funcional neste checkpoint documental.

## 26. Decisões vigentes

1. uma única Tela 12 com navegação `Empresa` + `Categorias`;
2. somente seções autorizadas são exibidas;
3. Empresa: ADM/Gerência sim, Funcionário não;
4. Categorias: ADM/Gerência sim, Funcionário não no preset inicial;
5. `Empresa` contém logo, nome, contato, site e e-mail;
6. limites de campos/logo seguem D12.63–D12.64;
7. logo pode ser escolhido/substituído/removido com preview antes do save;
8. identidade salva alimenta Shell e documentos/ficha;
9. salvamento explícito, sem autosave;
10. categorias usam lista compacta, busca e filtro;
11. categoria possui inicialmente nome + estado, sem cor/ícone/hierarquia;
12. criar/editar categoria ocorre fora do Editor;
13. arquivar substitui exclusão física normal e permite reativação;
14. categoria arquivada herdada em nova revisão segue D12.65;
15. duplicidades equivalentes após normalização são impedidas;
16. Backup/Restore e Exportação/Impressão permanecem fora da Tela 12.

## 27. Critérios de aceite documental

- [x] navegação local `Empresa` + `Categorias` aprovada;
- [x] campos e fluxo de logo aprovados;
- [x] uso centralizado da identidade aprovado;
- [x] gestão simples e arquivamento/reativação de categorias aprovados;
- [x] Gerência × Empresa fechada em D12.62;
- [x] presets de gestão de categorias alinhados à matriz de autenticação;
- [x] limites de identidade/logo fechados em D12.63–D12.64;
- [x] regra de categoria arquivada em nova revisão fechada em D12.65;
- [x] Host permanece autoridade final;
- [x] nenhuma implementação funcional foi criada.
