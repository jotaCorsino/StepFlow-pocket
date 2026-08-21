# Tela 02 — Shell Principal e Sidebar

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Aprovação:** 2026-08-21

## 1. Objetivo

Definir a estrutura visual e funcional persistente da área autenticada do StepFlow, fornecendo navegação simples, identidade visual consistente e acesso ao perfil sem competir com o conteúdo principal.

O Shell é a moldura comum da área autenticada; não é um dashboard específico.

## 2. Atores

Usado por:

- ADM;
- Gerência;
- Funcionário.

A estrutura é comum. Itens visíveis dependem das capacidades efetivas retornadas pelo Host.

## 3. Entrada

```text
Login válido
→ sessão criada pelo Host
→ Client recebe capacidades efetivas
→ Shell principal
→ Início/Dashboard
```

Sessão expirada ou revogada encerra a área autenticada e retorna ao Login.

## 4. Estrutura aprovada

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
│ [avatar] Nome    │                                              │
│          Cargo   │                                              │
└──────────────────┴──────────────────────────────────────────────┘

* conforme capacidades efetivas da sessão
```

Decisões aprovadas:

- sidebar persistente à esquerda;
- logo pequeno da empresa + `StepFlow` no topo;
- sem topbar global fixa/redundante;
- conteúdo ocupa o restante da janela;
- sidebar não recolhe/expande automaticamente na primeira versão;
- itens sem permissão ficam ocultos;
- `Backup/Restauração` pertence à área administrativa/Configurações quando autorizado;
- `Exportar/Imprimir` permanece contextual ao processo/documento;
- perfil/avatar permanece no rodapé;
- sem indicador verde permanente de conexão;
- sem modo hamburger/mobile na primeira versão, salvo necessidade real comprovada pela janela mínima suportada.

## 5. Identidade no topo

Exibir:

- logo da empresa pequeno, sem deformação;
- nome `StepFlow` de forma discreta.

Não exibir informações técnicas de Host, banco, porta, schema ou ambiente.

Sem logo configurado, usar fallback visual simples que preserve o layout.

## 6. Navegação principal

### Início

Disponível para todos os usuários autenticados.

Destino: `Início/Dashboard`.

### Processos

Disponível para todos.

Destino: lista/pesquisa de processos.

### Usuários

Visível somente com capacidade correspondente.

- ADM: conforme permissões administrativas;
- Gerência: gestão delegada de usuários não-ADM;
- Funcionário: item oculto.

### Configurações

Visível somente quando houver capacidade administrativa aplicável.

Pode agrupar, conforme permissões aprovadas:

- identidade da empresa;
- backup/restauração;
- demais configurações administrativas documentadas.

A visibilidade nunca substitui autorização Host-side.

## 7. Visibilidade e autorização

- item não autorizado fica oculto;
- rota/ação não autorizada continua bloqueada mesmo se o Client for manipulado;
- capacidades alteradas durante a sessão devem reconciliar a UI;
- o Host é sempre a autoridade final.

## 8. Item ativo

O destino atual deve ser perceptível de forma discreta e acessível:

- somente um item principal ativo;
- não depender apenas de cor;
- texto e ícone legíveis;
- foco de teclado distinto do estado ativo.

Cor/borda exatas pertencem ao sistema visual comum.

## 9. Perfil no rodapé

```text
[avatar]  Nome de exibição
          Cargo
```

Clique/acionamento abre menu simples:

- `Meu perfil`;
- `Sair`.

Não mostrar `user_id`, role interno ou detalhes de permissão no rodapé.

### Sair

- solicita logout ao Host;
- revoga/encerra a sessão conforme contrato;
- limpa token em memória;
- retorna ao Login;
- **não encerra o Host central**.

## 10. Área de conteúdo

- Shell não força cards em todas as páginas;
- título, filtros e ações pertencem à tela corrente;
- a própria tela define largura e densidade adequadas;
- não existe segunda navegação global no topo.

Cabeçalho local de página pode conter título, descrição e ações. Breadcrumb só quando realmente necessário.

## 11. Estados de conexão e sessão

### Operação normal

Nenhum indicador decorativo permanente de conexão.

### Host indisponível

- mostrar banner/estado contextual claro;
- não apresentar operação como concluída sem confirmação;
- preservar Shell quando isso ajudar a manter contexto;
- ações dependentes do Host ficam explicitamente indisponíveis.

### Sessão expirada/revogada

- interromper acesso protegido;
- limpar autenticação local;
- retornar ao Login com mensagem curta quando apropriado.

### Permissão revogada

- remover/reconciliar navegação;
- não manter conteúdo protegido aberto;
- encaminhar para destino autorizado seguro, preferencialmente Início.

## 12. Loading inicial do Shell

Após login, montar a navegação somente quando perfil/capacidades necessários estiverem disponíveis.

Evitar piscar itens provisórios que desaparecem em seguida.

## 13. Janelas menores

- sidebar com largura estável em desktop;
- conteúdo central pode rolar;
- não criar hamburger/mobile por antecipação;
- modo compacto só será avaliado se a janela mínima suportada demonstrar necessidade.

## 14. Ícones

- ícone simples + texto nos itens principais;
- texto não é removido na primeira versão;
- ícones não substituem labels;
- estilo consistente e discreto.

## 15. Sistema visual compartilhado

O Shell será a referência visual compartilhada pelo Login e demais telas.

Ainda serão fechados em conjunto no Bloco 8:

- paleta exata;
- família tipográfica;
- largura exata da sidebar;
- raios, sombras e espaçamento;
- estilo final de botões, inputs, cards e banners;
- dimensão mínima suportada da janela.

Direção obrigatória já aprovada:

- corporativa;
- limpa;
- clássica/discreta;
- densidade moderada;
- alto contraste para leitura;
- sem aparência de portal burocrático ou dashboard chamativo.

## 16. Acessibilidade e teclado

- navegação totalmente acionável por teclado;
- foco visível;
- item ativo semanticamente identificável;
- menu de perfil acessível por teclado;
- ícones com texto/nome acessível;
- Escape fecha menu de perfil;
- navegação não depende de hover.

Atalhos globais não são requisito inicial.

## 17. Eventos relevantes

O Shell pode reagir a:

- sessão revogada/expirada;
- alteração de nome/avatar;
- alteração de capacidades;
- Host indisponível/reconectado.

Usar os contratos vigentes; não criar protocolo específico por causa da tela.

## 18. Relação com o Host central

- logout não encerra Host;
- fechar Client individual não encerra Host;
- Shell comum não oferece “desligar servidor” ao técnico;
- encerramento central pertence ao ciclo do Controller e à UX administrativa correspondente.

## 19. Fora do escopo

- conteúdo específico do Dashboard;
- lista/leitor/editor de processos;
- estratégia técnica de backup/exportação;
- implementação do Controller;
- código Tauri/HTML/CSS/JS de produção.

## 20. Decisões consolidadas nesta tela

1. sidebar persistente, não recolhível automaticamente na primeira versão;
2. sem topbar global fixa;
3. navegação `Início`, `Processos`, `Usuários` e `Configurações`, conforme permissão;
4. itens sem autorização ocultos;
5. Backup/Restauração dentro da área administrativa/Configurações;
6. Exportar/Imprimir contextual;
7. perfil no rodapé com `Meu perfil` e `Sair`;
8. sem indicador permanente de conexão em estado normal;
9. sem hamburger/mobile sem necessidade comprovada;
10. logout e fechamento de Client não encerram Host.

## 21. Pendências compartilhadas do sistema visual

- paleta;
- tipografia;
- espaçamento;
- dimensões exatas;
- aparência final de componentes e banners;
- janela mínima suportada.

Essas pendências não reabrem a estrutura funcional aprovada nesta tela.

## 22. Critérios de aceite da especificação

- [x] sidebar/logo/perfil aprovados;
- [x] navegação aprovada;
- [x] comportamento da sidebar aprovado;
- [x] menu de perfil aprovado;
- [x] tratamento normal/degradado aprovado em nível funcional;
- [x] ocultação visual não substitui autorização;
- [x] logout não interfere no Host;
- [x] nenhuma implementação de produção criada.

## 23. Casos de teste futuros

1. ADM vê itens autorizados;
2. Gerência não vê controles exclusivos de ADM;
3. Funcionário vê navegação compatível com leitura/execução;
4. rota não autorizada é bloqueada;
5. item ativo e foco são perceptíveis;
6. perfil abre/fecha corretamente;
7. logout volta ao Login e não encerra Host;
8. sessão expirada retorna ao Login;
9. Host indisponível mostra estado coerente;
10. alteração de nome/avatar é refletida;
11. alteração de permissões remove acesso antigo;
12. layout funciona na janela mínima aprovada.
