# Tela 02 — Shell Principal e Sidebar

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Definir a estrutura visual e funcional persistente da área autenticada do StepFlow, fornecendo navegação simples, identidade visual consistente e acesso ao perfil sem competir com o conteúdo principal.

O Shell não é um dashboard específico. Ele é a moldura comum que permanece enquanto o usuário navega entre as áreas autorizadas.

## 2. Atores

O Shell é usado por:

- ADM;
- Gerência;
- Funcionário.

A estrutura visual é comum; os itens visíveis dependem das capacidades efetivas retornadas pelo Host.

## 3. Entrada no Shell

Fluxo:

```text
Login válido
→ sessão criada pelo Host
→ Client recebe capacidades efetivas
→ Shell principal
→ Início/Dashboard
```

Se a sessão expirar ou for revogada, a área autenticada é abandonada e o usuário retorna ao Login.

## 4. Estrutura geral proposta

```text
┌──────────────────┬──────────────────────────────────────────────┐
│ [logo empresa]   │                                              │
│ StepFlow         │        conteúdo da tela atual                │
│                  │                                              │
│ • Início         │                                              │
│ • Processos      │                                              │
│ • Usuários*      │                                              │
│ • Configurações* │                                              │
│                  │                                              │
│                  │                                              │
│                  │                                              │
│ [avatar] Nome    │                                              │
│          Cargo   │                                              │
└──────────────────┴──────────────────────────────────────────────┘

* somente quando a sessão possuir capacidade correspondente
```

### Direção consolidada já existente

- sidebar à esquerda;
- logo discreto no topo esquerdo;
- perfil/avatar na parte inferior;
- visual corporativo, limpo e clássico;
- conteúdo técnico deve dominar a área útil.

### Proposta para aprovação

- sidebar persistente enquanto autenticado;
- nenhuma barra superior global redundante;
- conteúdo da página ocupa todo o restante da janela;
- sidebar não fica recolhendo/expandindo automaticamente na primeira versão;
- itens sem permissão ficam ocultos em vez de aparecerem desabilitados;
- `Backup/Restauração` não vira item global separado: entra em Configurações/área administrativa quando permitido;
- `Exportar/Imprimir` permanece ação contextual do processo/documento, não item global da sidebar.

## 5. Identidade no topo

Área superior da sidebar deve mostrar:

- logo da empresa, pequeno e sem deformação;
- nome `StepFlow` de forma discreta;
- nenhum texto técnico de Host, banco, versão de schema ou ambiente.

Se o logo não estiver configurado, o Shell deve manter um fallback visual limpo sem quebrar layout.

## 6. Navegação principal

### Itens propostos

#### Início

Disponível para todos os perfis autenticados.

Destino: `Início/Dashboard`.

#### Processos

Disponível para todos.

Destino: lista/pesquisa de processos.

#### Usuários

Visível somente quando a sessão possuir capacidade de ler/gerir usuários.

- ADM: acesso completo conforme autorização;
- Gerência: acesso restrito a usuários não-ADM;
- Funcionário: não vê o item.

#### Configurações

Visível somente quando houver pelo menos uma capacidade administrativa/configurável aplicável.

Pode futuramente agrupar, conforme permissões:

- identidade da empresa;
- backup/restauração;
- outras configurações administrativas aprovadas.

O item não autoriza por si só nenhuma capacidade; o Host continua validando todas as operações.

## 7. Regra de visibilidade por permissão

A sidebar reflete capacidades retornadas pelo Host.

Proposta:

- item não autorizado é **ocultado**, não apenas desabilitado;
- navegação direta para rota não autorizada continua sendo bloqueada pelo Host/Client;
- ocultação serve somente à experiência do usuário e nunca substitui autorização server-side.

Se capacidades mudarem durante a sessão, o Client deve reconciliar a UI quando receber/descobrir o novo estado; mecanismo técnico exato fica fora desta tela.

## 8. Item ativo

O destino atual precisa ser perceptível sem excesso visual.

Direção:

- destaque discreto do item ativo;
- não depender somente de cor;
- ícone e texto permanecem legíveis;
- somente um destino principal fica marcado como ativo por vez.

Detalhes de cor/borda serão fechados no sistema visual.

## 9. Perfil no rodapé da sidebar

Área inferior persistente:

```text
[avatar]  Nome de exibição
          Cargo
```

Ao acionar a área do perfil, proposta de menu simples:

- `Meu perfil`;
- `Sair`.

Não mostrar login técnico, `user_id`, preset interno ou matriz de permissões diretamente no rodapé.

### Meu perfil

Navega para a tela de perfil pessoal.

### Sair

- solicita logout ao Host;
- revoga/encerra a sessão conforme contrato;
- limpa o token em memória;
- retorna ao Login.

Logout comum não encerra o Host central.

## 10. Área de conteúdo

A área à direita da sidebar é reservada para a tela atual.

Princípios:

- largura máxima do conteúdo é definida pela própria tela quando necessário;
- Shell não força cards/painéis artificiais em todas as páginas;
- títulos, ações contextuais e filtros pertencem à tela corrente;
- evitar uma segunda navegação global no topo.

## 11. Cabeçalho de página

Proposta: não existe um `topbar` global fixo.

Cada tela pode possuir um cabeçalho local no conteúdo contendo, quando necessário:

- título;
- descrição curta;
- ações da página;
- breadcrumbs apenas se a navegação realmente justificar.

Não transformar breadcrumbs em requisito global sem necessidade.

## 12. Estados de conexão e sessão

### Operação normal

Não exibir indicador verde permanente de “online” apenas por decoração.

### Host temporariamente indisponível

Quando uma tela já autenticada perde conexão com o Host:

- exibir estado/banner contextual claro;
- evitar apresentar operações como concluídas;
- preservar a estrutura do Shell quando isso ajudar o usuário a entender o contexto;
- ações que dependem do Host ficam indisponíveis de modo explícito.

A política detalhada de reconexão segue `comunicacao-client-host.md`.

### Sessão expirada/revogada

- interromper uso de áreas protegidas;
- limpar estado de autenticação local;
- retornar ao Login com mensagem curta quando apropriado.

### Permissão revogada

Se o usuário perder acesso a uma área:

- não continuar exibindo conteúdo protegido como se ainda estivesse autorizado;
- remover/reconciliar item de navegação;
- encaminhar para destino autorizado seguro, preferencialmente Início.

## 13. Estados de loading do Shell

Após login e antes de montar a navegação, o Client pode precisar confirmar perfil/capacidades.

Direção:

- loading curto e estável;
- evitar mostrar primeiro itens que depois desaparecem por permissão;
- não piscar sidebar com opções provisórias.

## 14. Comportamento em janelas menores

A sidebar deve continuar funcional nas dimensões mínimas suportadas.

Proposta inicial:

- sidebar mantém largura estável em desktop;
- conteúdo central pode rolar quando necessário;
- não introduzir modo mobile/hamburger como requisito da primeira versão;
- eventual sidebar compacta só será adicionada se o tamanho mínimo validado realmente exigir.

## 15. Ícones

Proposta:

- cada item principal pode ter ícone simples + texto;
- texto não deve ser removido na primeira versão;
- ícones não substituem labels;
- estilo consistente e discreto;
- nenhuma iconografia decorativa excessiva.

## 16. Cores e tipografia

O Shell será a referência do sistema visual compartilhado pelo Login e demais telas.

Ainda não estão aprovados:

- paleta exata;
- família tipográfica;
- tamanho/largura da sidebar;
- raios de borda;
- sombras;
- escala de espaçamento;
- estilo final do estado ativo;
- aparência exata de botões/inputs/cards.

Direção já consolidada:

- visual corporativo e clássico;
- alto contraste suficiente para leitura;
- densidade moderada;
- não parecer portal burocrático nem dashboard chamativo.

## 17. Acessibilidade e teclado

- todos os itens de navegação acessíveis por teclado;
- foco visível;
- item ativo identificável semanticamente;
- avatar/menu de perfil acessível por teclado;
- ícones com texto ou nome acessível;
- tooltip não é a única forma de identificar uma ação;
- navegação não depende apenas de hover.

## 18. Navegação por teclado

Direção:

- Tab percorre itens interativos em ordem previsível;
- Enter/Espaço ativa item conforme semântica do controle;
- menu de perfil pode ser fechado por Escape;
- foco retorna de modo previsível após fechar menu.

Atalhos globais de teclado não são requisito nesta etapa.

## 19. Responsabilidade do Host

O Shell pode ocultar/exibir itens, mas o Host é sempre responsável pela autorização real.

Tentativa de acessar recurso sem permissão deve ser rejeitada mesmo que ocorra por URL/estado interno/manipulação do Client.

## 20. Eventos em tempo real

O Shell pode precisar reagir a:

- sessão revogada/expirada;
- perfil/nome/avatar alterado;
- capacidades alteradas;
- Host indisponível/reconectado.

Não é necessário criar um protocolo específico nesta tela; usar os contratos/eventos definidos na arquitetura quando correspondentes.

## 21. Concorrência

O Shell não mantém estado concorrente de negócio próprio.

Mudanças de perfil/permissões feitas por outro administrador devem ser reconciliadas com o estado retornado pelo Host; o Client não conserva autoridade antiga por conveniência.

## 22. Relação com o Controller/Host central

- logout de usuário não encerra Host;
- fechar Client individual não encerra Host;
- Shell não oferece ação comum de “desligar servidor” ao técnico;
- eventual comando administrativo de encerramento central pertence à UX do Controller/área operacional correspondente, não à navegação comum do Client sem decisão específica.

## 23. Fora do escopo

- conteúdo do Dashboard;
- layout da lista de processos;
- leitor/editor;
- estratégia técnica de backup/exportação;
- implementação do Controller;
- sistema final de design tokens;
- código Tauri/HTML/CSS/JS de produção.

## 24. Decisões já consolidadas antes desta análise

- sidebar à esquerda;
- logo pequeno no topo esquerdo;
- perfil/avatar na parte inferior;
- usuário vê experiência compatível com suas permissões;
- Funcionário é leitura/execução por padrão;
- Host valida autorização;
- visual corporativo, limpo e discreto.

## 25. Propostas para aprovação do PO

1. sidebar persistente e não recolhível na primeira versão;
2. sem topbar global fixa;
3. navegação principal: `Início`, `Processos`, `Usuários` e `Configurações` conforme permissão;
4. itens sem autorização ficam ocultos;
5. `Backup/Restauração` fica dentro da área de Configurações, não como item global;
6. `Exportar/Imprimir` permanece contextual aos processos/documentos;
7. perfil no rodapé abre menu com `Meu perfil` e `Sair`;
8. não mostrar indicador permanente de conexão quando tudo está normal;
9. não criar modo hamburger/sidebar compacta sem necessidade real de janela mínima.

## 26. Pendências

- identidade visual exata do Shell;
- dimensão mínima suportada da janela;
- largura exata da sidebar;
- microcopy final;
- política visual para banners/erros transversais;
- UX específica de encerramento central quando houver Clients conectados será tratada na superfície administrativa correspondente.

## 27. Critérios de aceite da especificação

- [x] respeita sidebar/logo/perfil já aprovados;
- [x] não confunde ocultação visual com autorização;
- [x] logout não interfere no Host central;
- [x] mantém Backup/Exportação dentro dos limites dos Blocos 10/11;
- [x] não cria código de produção;
- [ ] estrutura de navegação aprovada pelo PO;
- [ ] comportamento de sidebar aprovado pelo PO;
- [ ] menu de perfil aprovado pelo PO;
- [ ] tratamento visual normal/degradado aprovado pelo PO;

## 28. Casos de teste futuros

1. ADM vê itens autorizados;
2. Gerência não vê controles exclusivos de ADM;
3. Funcionário vê somente navegação compatível com leitura/execução;
4. tentativa de rota não autorizada é bloqueada;
5. item ativo é perceptível por teclado/leitor de tela;
6. perfil abre e fecha corretamente;
7. logout revoga sessão e volta ao Login;
8. logout não encerra Host;
9. sessão expirada retorna ao Login;
10. Host indisponível mostra estado coerente;
11. alteração de nome/avatar é refletida no rodapé;
12. alteração de permissões não deixa acesso antigo ativo;
13. layout continua utilizável na janela mínima suportada.
