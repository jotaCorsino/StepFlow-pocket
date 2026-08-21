# Tela 02 — Shell Principal e Sidebar

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Aprovação do núcleo:** 2026-08-21  
**Extensão operacional aprovada:** 2026-08-21

## 1. Objetivo

Definir a estrutura visual e funcional persistente da área autenticada do StepFlow, fornecendo navegação simples, identidade visual consistente e acesso às áreas autorizadas sem competir com o conteúdo principal.

## 2. Estrutura consolidada

```text
[logo empresa]
StepFlow

• Início
• Processos
• Atendimentos
• Usuários*
• Configurações*

[avatar] Nome
         Cargo
```

`*` conforme capacidades efetivas da sessão.

Decisões consolidadas:

- sidebar persistente à esquerda;
- logo pequeno da empresa + `StepFlow` no topo;
- sem topbar global fixa/redundante;
- conteúdo ocupa o restante da janela;
- sidebar não recolhe/expande automaticamente na primeira versão;
- itens sem permissão ficam ocultos;
- perfil/avatar no rodapé com `Meu perfil` e `Sair`;
- sem indicador verde permanente de conexão;
- sem modo hamburger/mobile sem necessidade comprovada;
- logout/fechamento de Client não encerram o Host.

## 3. Navegação principal

### Início

Disponível para todo usuário autenticado.

Destino: `Início/Dashboard`.

### Processos

Área de documentação/modelos oficiais:

- localizar procedimentos;
- filtrar por categoria;
- ler conteúdo;
- editar/publicar quando autorizado.

### Atendimentos

Área operacional para ocorrências reais de serviço/execução:

- localizar atendimento;
- pesquisar por atendimento, equipamento, cliente/referência, OS, serial/patrimônio/MAC;
- iniciar/continuar/concluir conforme regras do Bloco 9;
- associar equipamento opcional;
- registrar procedimentos utilizados;
- gerar ficha compacta quando aplicável.

A separação entre `Processos` e `Atendimentos` é deliberada: editar documentação oficial e registrar serviço executado são atividades distintas.

Um atendimento poderá ser iniciado a partir de um procedimento quando a UX correspondente for especificada.

### Usuários

Visível somente com capacidade correspondente.

### Configurações

Visível quando houver capacidade administrativa aplicável.

Pode agrupar:

- identidade da empresa;
- gestão de categorias;
- backup/restauração;
- demais configurações aprovadas.

Categoria não vira item global próprio da sidebar.

## 4. Visibilidade e autorização

- item não autorizado fica oculto;
- rota/ação não autorizada continua bloqueada mesmo com Client manipulado;
- capacidades alteradas durante a sessão reconciliam a UI;
- Host é autoridade final;
- ver `Atendimentos` não implica automaticamente permissão para criar, concluir ou reabrir atendimento.

A matriz operacional exata será fechada no Bloco 9.

## 5. Perfil no rodapé

```text
[avatar] Nome de exibição
         Cargo
```

Menu:

- `Meu perfil`;
- `Sair`.

`Sair` revoga/encerra sessão, limpa token em memória, retorna ao Login e não encerra Host central.

## 6. Área de conteúdo

- Shell não força cards em todas as páginas;
- título, filtros e ações pertencem à tela corrente;
- a própria tela define largura/densidade;
- não existe segunda navegação global no topo.

## 7. Estados de conexão e sessão

### Normal

Sem indicador decorativo permanente.

### Host indisponível

- banner/estado contextual claro;
- nenhuma operação apresentada como concluída sem confirmação;
- ações dependentes do Host ficam indisponíveis;
- Shell pode ser preservado para manter contexto.

### Sessão expirada/revogada

Retornar ao Login e impedir uso de conteúdo protegido.

### Permissão revogada

Remover/reconciliar item e encaminhar para destino autorizado seguro.

## 8. Janelas menores e sistema visual

- sidebar estável em desktop;
- conteúdo central pode rolar;
- sem hamburger/mobile antecipado;
- sistema visual corporativo, limpo, clássico/discreto e de densidade moderada.

Paleta, tipografia, espaçamento e dimensões exatas continuam pendências compartilhadas do Bloco 8.

## 9. Acessibilidade

- navegação acionável por teclado;
- foco visível;
- item ativo semanticamente identificável;
- ícones acompanhados de texto/nome acessível;
- menu de perfil acessível;
- navegação não depende de hover.

## 10. Relação com Host/Controller

- logout não encerra Host;
- fechar Client não encerra Host;
- Shell comum não oferece desligamento do servidor ao técnico;
- encerramento central pertence ao Controller/UX administrativa correspondente.

## 11. Fora do escopo

- lifecycle detalhado dos atendimentos;
- matriz final de permissões operacionais;
- tecnologia de impressão/exportação;
- código de produção.

## 12. Critérios de aceite

- [x] sidebar persistente aprovada;
- [x] navegação `Início`, `Processos`, `Atendimentos`, `Usuários`, `Configurações` aprovada;
- [x] `Processos` separado de `Atendimentos`;
- [x] itens sem autorização ocultos sem substituir autorização Host-side;
- [x] gestão de categorias fica em Configurações;
- [x] perfil no rodapé aprovado;
- [x] logout/fechamento do Client não encerram Host;
- [x] nenhuma implementação de produção criada.

## 13. Casos de teste futuros

1. usuário autorizado vê `Atendimentos`;
2. usuário sem capacidade não vê o item;
3. manipulação do Client não concede acesso operacional;
4. `Processos` continua separado de `Atendimentos`;
5. ADM/Gerência/Funcionário veem apenas itens permitidos;
6. logout retorna ao Login e não encerra Host;
7. Host indisponível mostra estado coerente;
8. alteração de permissões remove acesso antigo;
9. navegação por teclado funciona;
10. layout funciona na janela mínima aprovada.