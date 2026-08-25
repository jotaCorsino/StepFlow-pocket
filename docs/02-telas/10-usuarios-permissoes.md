# Tela 10 — Usuários / Permissões

## 1. Identificação

- código/nome da tela: Tela 10 — Usuários / Permissões;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX);
- data da consolidação: 2026-08-25.

## 2. Objetivo

Permitir administrar contas de usuário e suas autorizações de forma simples, segura e auditável, sem transformar a gestão de acesso em um painel técnico complexo.

A tela reflete o modelo já consolidado:

- autenticação e autorização são decididas pelo Host;
- `ADM`, `GERENCIA` e `FUNCIONARIO` são presets;
- a autorização real é por capacidades;
- Gerência administra usuários não-ADM dentro do escopo permitido;
- Gerência nunca cria, promove, rebaixa ou administra ADM;
- usuário não eleva a própria autoridade;
- contas são preferencialmente desativadas, não excluídas;
- deve existir pelo menos um ADM ativo;
- ações administrativas críticas são auditáveis.

## 3. Atores e acesso

A área `Usuários` aparece na sidebar apenas para sessões com capacidade correspondente.

Perfis conceituais:

- **ADM** — administração completa de usuários dentro das proteções do sistema;
- **Gerência** — administração delegada de usuários não-ADM;
- **Funcionário** — não acessa a gestão administrativa de usuários por padrão.

A ocultação no Client não substitui autorização Host-side.

## 4. Entrada e navegação

Fluxo principal:

```text
Shell/sidebar
→ Usuários
→ Tela 10 — Usuários / Permissões
```

A tela não cria uma segunda navegação global.

## 5. Estrutura visual aprovada

```text
Usuários                                             [ Novo usuário ]

[ Buscar por nome, login ou cargo... ]
[ Perfil ▾ ] [ Estado ▾ ]                           [ Limpar filtros ]

Nome                Login          Cargo              Perfil base     Estado
Maria Souza         maria.souza    Analista de TI     Gerência        Ativo
Carlos Lima         carlos.lima    Técnico            Funcionário     Ativo
Ana Martins         ana.martins    Técnica            Funcionário     Inativo

                                                    [ ⋯ ]
```

Ao criar ou editar:

```text
Editar usuário

IDENTIFICAÇÃO
Nome de exibição      [ Maria Souza              ]
Login                 [ maria.souza              ]
Cargo                 [ Analista de TI           ]
Estado                [ Ativo ▾                  ]

ACESSO
Perfil base           [ Gerência ▾               ]

Permissões específicas
[ ] Personalizar permissões deste usuário

Processos
[x] Ler processos
[x] Criar/editar processos
[x] Publicar/arquivar processos
[x] Exportar/imprimir

Usuários
[x] Gerir usuários não-ADM
[ ] Gerir ADM                    indisponível quando o ator não pode delegar

[ Restaurar padrão do perfil ]

                         [ Cancelar ] [ Salvar alterações ]
```

Os nomes exatos das capacidades finais devem vir do contrato de autorização vigente. Capacidades operacionais de Atendimento não são inventadas nesta tela antes do Bloco 9.

## 6. Lista de usuários

A tela usa tabela/lista compacta, coerente com `Processos` e `Atendimentos`.

Colunas iniciais aprovadas:

- nome;
- login;
- cargo;
- perfil base;
- estado (`Ativo` / `Inativo`).

Não exibir como coluna padrão:

- hash de senha;
- token/sessão;
- IDs internos;
- último IP;
- dados técnicos de autenticação;
- métricas de atividade sem requisito específico.

## 7. Busca e filtros

Busca principal por:

- nome de exibição;
- login;
- cargo.

Filtros iniciais:

- `Perfil`;
- `Estado`.

O Host deve retornar somente usuários dentro do escopo que a sessão pode consultar.

## 8. Escopo de visibilidade por perfil

### ADM

Pode consultar e administrar usuários conforme capacidades, inclusive contas ADM quando permitido.

### Gerência

A área administrativa trabalha apenas com usuários não-ADM dentro de seu escopo de gestão.

Gerência não recebe ações para:

- criar ADM;
- transformar outro perfil em ADM;
- rebaixar ADM;
- editar capacidades exclusivas de ADM;
- administrar o ADM primário.

### Funcionário

Não acessa esta tela por padrão. Alterações do próprio perfil pertencem à Tela 11 — Meu perfil.

## 9. Novo usuário

Ação `Novo usuário` aparece somente quando a sessão pode criar usuários.

Campos conceituais:

- nome de exibição;
- login;
- cargo;
- perfil base;
- credencial inicial conforme política de senha vigente;
- estado inicial ativo, salvo regra futura diferente.

O `user_id` é criado pelo sistema e nunca digitado pelo usuário.

### 9.1 Perfis disponíveis na criação

- ADM pode selecionar perfis que sua capacidade permita, inclusive ADM;
- Gerência nunca recebe ADM como opção de criação;
- Funcionário não cria usuários por padrão.

### 9.2 Senha inicial

A UI não deve hardcodar um mínimo ainda não consolidado.

Ela deve:

- solicitar nova senha + confirmação quando esse for o fluxo vigente;
- apresentar requisitos conforme política devolvida/estabelecida pelo Host;
- nunca mostrar hash;
- nunca registrar senha em log/auditoria;
- nunca oferecer recuperar/revelar senha existente.

Fluxos mais sofisticados de convite ou troca obrigatória no primeiro acesso só entram se forem aprovados depois.

## 10. Edição de usuário

A edição administrativa pode alterar, conforme capacidade:

- nome de exibição;
- login, desde que regras de unicidade/segurança permitam;
- cargo;
- estado ativo/inativo;
- perfil base;
- capacidades específicas delegáveis.

Não altera:

- `user_id`;
- histórico passado;
- `is_primary_admin` por um toggle genérico nesta tela.

A transferência ou redefinição de ADM primário, se algum dia necessária, exige regra específica e não é inventada aqui.

## 11. Preset + capacidades granulares

Direção aprovada:

1. `Perfil base` fornece um conjunto padrão de capacidades;
2. a UI mostra um resumo dessas capacidades;
3. quando autorizado, `Personalizar permissões deste usuário` permite ajustar capacidades individuais;
4. usuário personalizado fica visualmente identificado como tal;
5. `Restaurar padrão do perfil` volta ao conjunto padrão do preset selecionado;
6. o Host valida cada capacidade salva e o teto de delegação do ator.

O preset nunca substitui a verificação granular do Host.

## 12. Teto de delegação

Uma sessão nunca pode conceder autoridade que o Host não permita delegar.

Especialmente:

- Gerência não pode conceder capacidades exclusivas de ADM;
- Gerência não pode produzir indiretamente um “ADM equivalente” marcando várias caixas;
- usuário não pode elevar a própria autoridade;
- capacidades críticas podem exigir ADM mesmo quando outras capacidades do mesmo domínio forem delegáveis.

A UI deve ocultar ou desabilitar capacidades fora do teto de delegação, mas o Host continua sendo a barreira real.

## 13. Grupos de capacidades

A apresentação deve ser agrupada por domínio para permanecer compreensível.

Exemplos de grupos já coerentes com o produto:

- `Processos`;
- `Atendimentos`;
- `Usuários`;
- `Configurações`;
- `Backup / restauração` quando aplicável.

A lista exata de capacidades de `Atendimentos` só será consolidada no Bloco 9.

Parâmetros ainda marcados como `PENDENTE` na autenticação — por exemplo determinadas capacidades de Gerência em configuração da empresa ou Backup — não devem ser convertidos em checkbox definitivo até a decisão correspondente ser fechada.

## 14. Mudança de perfil base

Ao trocar o perfil base de um usuário:

- mostrar claramente o novo perfil;
- recalcular o conjunto padrão proposto;
- se houver personalização, informar que as permissões específicas serão reavaliadas;
- não salvar mudança de autoridade silenciosamente;
- exigir `Salvar alterações` explicitamente.

Elevação para ADM só pode existir para ator autorizado e nunca para Gerência.

## 15. Usuário atual e autoelevação

Na Tela 10, a própria conta do ator não permite alteração administrativa de seu perfil/capacidades de forma que eleve sua autoridade.

Direção aprovada para simplificar e reduzir risco:

- perfil/capacidades da própria conta ficam somente leitura nesta tela;
- dados pessoais da própria conta são alterados pela Tela 11 — Meu perfil;
- mudanças administrativas de autoridade do ator exigem outro usuário autorizado.

## 16. ADM e ADM primário

Proteções obrigatórias:

- pelo menos um ADM ativo deve existir;
- operação que deixaria o sistema sem ADM ativo é rejeitada;
- Gerência nunca administra ADM;
- `is_primary_admin` não é um campo editável comum;
- não criar fluxo genérico de “transferir ADM primário” sem requisito específico.

Quando um ADM não puder ser alterado por regra de proteção, a UI deve explicar de forma curta o motivo.

## 17. Ativar e desativar

A operação normal é:

```text
Ativo → Desativar
Inativo → Reativar
```

Não oferecer `Excluir usuário` como ação normal.

Desativação:

- preserva identidade e histórico;
- impede novos logins;
- revoga sessões pertinentes conforme contrato de autenticação;
- é auditada;
- respeita proteção do último ADM ativo.

A ação deve pedir confirmação explícita.

## 18. Redefinição administrativa de senha

Menu contextual pode oferecer `Redefinir senha` quando autorizado.

Regras já coerentes com a arquitetura:

- nunca revelar a senha antiga;
- definir nova credencial/fluxo conforme política vigente;
- revogar sessões pertinentes;
- registrar o evento administrativo sem registrar senha/hash/token;
- confirmar sucesso apenas após commit no Host.

## 19. Menu contextual da linha

Ações possíveis conforme capacidade e alvo:

- `Editar`;
- `Redefinir senha`;
- `Desativar` ou `Reativar`.

ADM pode receber ações adicionais de autoridade somente quando explicitamente permitidas.

Gerência não vê ações administrativas em contas ADM.

Sem ações em massa na primeira versão.

## 20. Estados da interface

### Carregando

Tabela preserva estrutura e apresenta loading no conteúdo.

### Vazio

`Nenhum usuário disponível.`

### Busca sem resultado

`Nenhum usuário encontrado com os critérios informados.`

### Sem permissão

A sidebar normalmente oculta `Usuários`. Acesso direto retorna estado de permissão negada sem vazar dados.

### Host indisponível

Indicar indisponibilidade sem campos técnicos de IP/porta/path.

### Usuário alterado por outro administrador

Não substituir silenciosamente formulário aberto. Informar atualização concorrente e exigir reconciliação.

## 21. Validações

- login deve ser válido e único conforme regra do Host;
- nome de exibição respeita limites definidos antes da implementação;
- cargo é texto curto;
- senha segue política vigente do Host;
- confirmação de senha deve coincidir;
- perfil/capacidades precisam estar dentro do teto de delegação;
- desativação não pode eliminar o último ADM ativo;
- campos administrativos protegidos não podem ser alterados via payload manipulado.

## 22. Salvamento

Salvar é explícito; não há autosave administrativo inicial.

Fluxo conceitual:

```text
editar usuário
→ Salvar alterações
→ Host valida sessão + alvo + capacidades + regras
→ writer/transaction
→ commit
→ auditoria/evento pós-commit
→ UI confirma sucesso
```

Nenhuma alteração deve aparecer como concluída antes do commit.

## 23. Concorrência

Alterações administrativas de usuário devem usar controle otimista equivalente às demais entidades mutáveis quando necessário.

Em conflito:

- não usar `last write wins` silencioso;
- preservar mudanças locais no formulário;
- informar que o usuário foi alterado por outra sessão administrativa;
- permitir reconsultar o estado atual;
- exigir nova decisão antes de salvar novamente.

## 24. Eventos em tempo real

Eventos pós-commit relevantes podem indicar:

- usuário criado;
- usuário alterado;
- usuário ativado/desativado;
- perfil/capacidades alterados.

Na lista, o Client pode reconsultar preservando busca/filtros.

Se o usuário sendo editado mudar, não substituir o formulário silenciosamente.

Se as capacidades da própria sessão forem revogadas, Shell e rota devem reconciliar imediatamente o acesso.

## 25. Auditoria

Devem ser auditáveis, de forma proporcional:

- criação de usuário;
- alteração de perfil/capacidades;
- ativação/desativação;
- redefinição administrativa de senha;
- mudanças administrativas críticas.

Nunca registrar:

- senha;
- token reutilizável;
- hash de senha sem necessidade explícita.

## 26. Acessibilidade

- busca e filtros com labels;
- tabela semanticamente identificada;
- menu contextual acionável por teclado;
- formulários com labels visíveis;
- checkboxes de capacidade com nomes completos;
- capacidades desabilitadas devem explicar o motivo quando necessário;
- erros ligados aos respectivos campos;
- foco restaurado adequadamente ao fechar modal/painel;
- estado não depende apenas de cor.

## 27. Comportamento em janelas menores

Desktop Windows é o alvo principal.

Em janelas menores suportadas:

- tabela pode ocultar `Cargo` antes de ocultar nome/login/perfil/estado;
- formulário passa de duas colunas para uma quando necessário;
- matriz de permissões continua legível sem scroll horizontal obrigatório;
- não criar experiência mobile/hamburger por antecipação.

## 28. Fora do escopo desta tela

- edição do próprio avatar/senha como fluxo principal — Tela 11;
- identidade da empresa e categorias — Tela 12;
- lifecycle e permissões operacionais finais de Atendimento — Bloco 9;
- política numérica final de senha ainda pendente;
- expiração numérica final de sessão ainda pendente;
- convite por e-mail;
- recuperação de senha por e-mail;
- MFA;
- SSO/Active Directory;
- exclusão física normal de usuários;
- ações administrativas em massa;
- workflow complexo de aprovação de acesso;
- implementação funcional.

## 29. Decisões consolidadas nesta tela

Aprovadas pelo PO:

1. usar lista/tabela compacta com busca por nome/login/cargo e filtros `Perfil` + `Estado`;
2. usar painel/formulário único para criação e edição administrativa;
3. manter `Ativo/Inativo` e desativação em vez de exclusão;
4. usar `Perfil base` como preset e permitir `Permissões específicas` granulares quando autorizado;
5. oferecer `Restaurar padrão do perfil` para remover personalizações;
6. Gerência administra somente usuários não-ADM e nunca vê ADM como opção delegável;
7. impedir que personalização de capacidades burle o teto de delegação;
8. manter perfil/capacidades da própria conta somente leitura nesta tela, deixando dados pessoais para `Meu perfil`;
9. proteger o último ADM ativo e não expor `is_primary_admin` como toggle comum;
10. oferecer `Redefinir senha` administrativamente sem revelar credencial antiga e revogando sessões pertinentes;
11. não oferecer exclusão física nem ações em massa inicialmente;
12. não inventar capacidades operacionais de Atendimento antes do Bloco 9.

## 30. Critérios de aceite

- [x] PO aprovou lista compacta e filtros;
- [x] PO aprovou criação/edição no mesmo formulário;
- [x] PO aprovou preset + capacidades específicas;
- [x] PO aprovou teto de delegação explícito;
- [x] PO aprovou regras de Gerência sobre não-ADM;
- [x] PO aprovou proteção da própria autoridade;
- [x] PO aprovou desativação em vez de exclusão;
- [x] PO aprovou reset administrativo de senha;
- [x] último ADM ativo permanece protegido;
- [x] pendências de autenticação continuam pendentes;
- [x] capacidades do Bloco 9 não foram inventadas;
- [x] Host permanece autoridade final;
- [x] nenhuma implementação funcional foi criada.
