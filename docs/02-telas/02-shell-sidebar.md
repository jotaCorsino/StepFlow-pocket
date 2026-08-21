# Tela 02 — Shell Principal e Sidebar

**Status:** NÚCLEO CONSOLIDADO / REABERTO PONTUALMENTE PARA NOVO REQUISITO DE ATENDIMENTOS  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Aprovação do núcleo:** 2026-08-21  
**Reabertura pontual:** 2026-08-21

## 1. Objetivo

Definir a estrutura visual e funcional persistente da área autenticada do StepFlow, fornecendo navegação simples, identidade visual consistente e acesso ao perfil sem competir com o conteúdo principal.

O Shell é a moldura comum da área autenticada; não é um dashboard específico.

## 2. Atores

Usado por ADM, Gerência e Funcionário. A estrutura é comum; itens visíveis dependem das capacidades efetivas retornadas pelo Host.

## 3. Entrada

```text
Login válido
→ sessão criada pelo Host
→ Client recebe capacidades efetivas
→ Shell principal
→ Início/Dashboard
```

Sessão expirada ou revogada encerra a área autenticada e retorna ao Login.

## 4. Estrutura consolidada

Permanecem aprovados:

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

## 5. Navegação principal após novo requisito

### Consolidado

- `Início` — todos os usuários autenticados;
- `Processos` — documentação/modelos oficiais;
- `Usuários` — conforme capacidade;
- `Configurações` — conforme capacidade administrativa.

### Proposta nova para aprovação do PO

Adicionar item próprio:

- `Atendimentos` — ocorrências reais de execução/serviço, separado de `Processos`.

Motivo: editar um procedimento oficial e registrar que um serviço foi realizado são atividades diferentes e não devem compartilhar a mesma superfície principal.

Estrutura proposta:

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

`Atendimentos` pode ficar visível para usuários com capacidade operacional correspondente. A matriz exata será fechada no Bloco 9.

## 6. Processos x Atendimentos

### Processos

- localizar documentação;
- ler procedimento;
- editar/publicar quando autorizado;
- filtrar por categoria.

### Atendimentos

- localizar serviço/execução real;
- pesquisar por atendimento, equipamento, cliente/referência, OS, serial/patrimônio/MAC;
- iniciar/continuar/concluir atendimento conforme regras futuras;
- associar equipamento opcional;
- registrar procedimentos utilizados;
- gerar ficha compacta quando aplicável.

A separação não impede iniciar um atendimento a partir do leitor de um procedimento; apenas evita misturar os domínios na navegação principal.

## 7. Configurações

Continua agrupando funções administrativas conforme permissão, incluindo:

- identidade da empresa;
- backup/restauração;
- **gestão de categorias de procedimentos**;
- demais configurações aprovadas.

Categoria não precisa virar item global próprio da sidebar.

## 8. Visibilidade e autorização

- item não autorizado fica oculto;
- rota/ação não autorizada continua bloqueada mesmo com Client manipulado;
- capacidades alteradas durante a sessão reconciliam a UI;
- Host é autoridade final;
- visibilidade de `Atendimentos` não define sozinha permissão para criar, editar, concluir ou reabrir um atendimento.

## 9. Item ativo e área de conteúdo

O destino atual deve ser perceptível de forma discreta e acessível, sem depender só de cor.

A área à direita da sidebar pertence à tela corrente. O Shell não força cards, filtros ou cabeçalhos iguais em todas as páginas.

## 10. Perfil no rodapé

```text
[avatar] Nome de exibição
         Cargo
```

Menu:

- `Meu perfil`;
- `Sair`.

`Sair` revoga/encerra sessão, limpa token em memória, retorna ao Login e não encerra Host central.

## 11. Estados de conexão/sessão

### Normal

Sem indicador decorativo permanente.

### Host indisponível

- banner/estado contextual;
- nenhuma operação é apresentada como concluída sem confirmação;
- ações dependentes do Host ficam indisponíveis;
- Shell pode ser preservado para manter contexto.

### Sessão expirada/revogada

Retornar ao Login e impedir uso de conteúdo protegido.

### Permissão revogada

Remover/reconciliar item e encaminhar para destino autorizado seguro.

## 12. Janelas menores e sistema visual

Mantêm-se as decisões já aprovadas:

- sidebar estável em desktop;
- conteúdo central pode rolar;
- sem hamburger/mobile antecipado;
- sistema visual corporativo, limpo, clássico/discreto e de densidade moderada.

Paleta, tipografia, espaçamento e dimensões exatas continuam pendências compartilhadas do Bloco 8.

## 13. Acessibilidade

- navegação acionável por teclado;
- foco visível;
- item ativo semanticamente identificável;
- ícones acompanhados de texto/nome acessível;
- menu de perfil acessível;
- navegação não depende de hover.

## 14. Relação com Host/Controller

- logout não encerra Host;
- fechar Client não encerra Host;
- Shell comum não oferece desligamento do servidor ao técnico;
- encerramento central pertence ao Controller/UX administrativa correspondente.

## 15. Fora do escopo

- lifecycle detalhado dos atendimentos;
- matriz final de permissões operacionais;
- conteúdo interno da ficha de atendimento/equipamento além do contrato de produto;
- tecnologia de impressão/exportação;
- código de produção.

## 16. Decisões consolidadas que não foram reabertas

1. sidebar persistente;
2. sem topbar global;
3. itens sem autorização ocultos;
4. `Backup/Restauração` em Configurações;
5. exportação contextual;
6. perfil no rodapé com `Meu perfil`/`Sair`;
7. sem indicador permanente de conexão;
8. logout e fechamento do Client não encerram Host.

## 17. Nova decisão pendente

Apenas a extensão de navegação precisa de nova aprovação:

- adicionar `Atendimentos` como item principal próprio entre `Processos` e áreas administrativas.

## 18. Critérios de aceite atualizados

- [x] núcleo do Shell permanece aprovado;
- [x] novo requisito não mistura documentação com execução real;
- [x] categorias ficam administráveis sem virar navegação global artificial;
- [x] nenhuma implementação foi criada;
- [ ] item `Atendimentos` na sidebar aprovado pelo PO.

## 19. Casos de teste futuros adicionais

Além dos casos já previstos para o Shell:

1. usuário autorizado vê `Atendimentos`;
2. usuário sem capacidade não vê o item;
3. manipulação do Client não concede acesso operacional;
4. `Processos` continua separado de `Atendimentos`;
5. navegação entre procedimento e atendimento preserva contexto sem duplicar autoridade.
