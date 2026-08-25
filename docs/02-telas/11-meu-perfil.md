# Tela 11 — Meu perfil

## 1. Identificação

- código/nome da tela: Tela 11 — Meu perfil;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX);
- data da consolidação: 2026-08-25.

## 2. Objetivo

Permitir que qualquer usuário autenticado gerencie os dados pessoais que pode alterar e a própria senha, sem confundir perfil pessoal com administração de contas e permissões.

A tela respeita o contrato já consolidado:

- usuário ativo pode alterar avatar;
- usuário ativo pode alterar nome de exibição;
- usuário ativo pode alterar cargo;
- usuário ativo pode alterar a própria senha;
- `user_id` é imutável;
- login não é alterado pelo próprio usuário nesta tela;
- perfil base e capacidades não são editáveis pelo próprio usuário;
- estado ativo/inativo não é editável pelo próprio usuário;
- `is_primary_admin` não é exposto como controle pessoal;
- Host continua sendo autoridade sobre identidade, autenticação, sessão e autorização.

## 3. Atores e acesso

A Tela 11 está disponível para todo usuário autenticado e ativo, independentemente do preset ADM/Gerência/Funcionário.

O usuário acessa apenas a própria conta. Manipulação do Client não permite consultar ou editar o perfil pessoal de outro usuário por esta rota.

## 4. Entrada e navegação

Fluxo consolidado:

```text
sidebar
→ bloco de perfil no rodapé
→ Meu perfil
→ Tela 11
```

O menu de perfil continua oferecendo `Sair`.

Não criar item global `Meu perfil` separado na sidebar.

## 5. Estrutura visual aprovada

```text
Meu perfil                                      [ Salvar alterações ]

PERFIL

        [ avatar ]
        [ Alterar foto ] [ Remover foto* ]

Nome de exibição     [ Maria Souza                 ]
Cargo                [ Analista de TI              ]
Login                maria.souza                      somente leitura
Perfil base          Gerência                         somente leitura
Acesso               Permissões personalizadas       somente leitura, quando aplicável

────────────────────────────────────────────────────────────
SEGURANÇA

Senha
Sua senha não é exibida pelo StepFlow.

[ Alterar senha ]
```

`*` aparece somente quando existe avatar próprio configurado.

A alteração de senha usa fluxo separado:

```text
Alterar senha

Senha atual          [ •••••••••••• ] [ mostrar/ocultar ]
Nova senha           [ •••••••••••• ] [ mostrar/ocultar ]
Confirmar nova senha [ •••••••••••• ] [ mostrar/ocultar ]

Requisitos: conforme política vigente do Host

                         [ Cancelar ] [ Alterar senha ]
```

## 6. Separação entre perfil pessoal e administração

### Tela 11 — o próprio usuário

Pode alterar:

- avatar;
- nome de exibição;
- cargo;
- própria senha.

Pode visualizar de forma informativa:

- login;
- perfil base;
- indicação de permissões personalizadas, quando aplicável.

Não pode alterar:

- login;
- `user_id`;
- perfil base;
- capacidades;
- estado ativo/inativo;
- marcação de ADM primário.

### Tela 10 — administração

Continua responsável, conforme capacidade, por:

- criar contas;
- alterar login;
- ativar/desativar;
- alterar perfil base/capacidades;
- redefinir senha administrativamente;
- demais ações administrativas aprovadas.

## 7. Avatar

### 7.1 Ausência de avatar

Usar placeholder discreto ou iniciais derivadas do nome, sem depender de serviço externo.

### 7.2 Escolher ou substituir

`Alterar foto` abre o seletor local do Client.

Após a escolha:

- mostrar preview local;
- não apresentar a imagem como persistida antes do save;
- persistir somente após `Salvar alterações` e confirmação do Host.

### 7.3 Remover

Quando houver avatar próprio, `Remover foto` marca a remoção para o próximo save. Após commit confirmado, voltar ao placeholder/iniciais.

### 7.4 Segurança do arquivo

- Host valida conteúdo/tipo e não confia apenas na extensão;
- formatos suportados devem ser explícitos;
- limites de bytes e dimensões serão fechados antes da implementação correspondente;
- arquivo não vira caminho arbitrário controlado pelo usuário;
- armazenamento segue a política de arquivos controlados pelo Host;
- avatar não executa conteúdo ativo;
- UI preserva proporção sem distorção.

Não incluir cropper/editor avançado na primeira versão sem requisito posterior.

## 8. Nome de exibição e cargo

Ambos são editáveis pelo próprio usuário.

Regras:

- textos curtos;
- limites numéricos definidos antes da implementação;
- normalização/validação pelo Host;
- alteração só é oficial após commit;
- cargo não representa autoridade e nunca modifica capacidades;
- nome/cargo confirmados atualizam o resumo do rodapé da sidebar via reconciliação/refetch.

## 9. Login

Login é exibido somente para referência e fica **somente leitura**.

Não oferecer `Alterar meu login` na primeira versão. Mudança de login permanece ação administrativa da Tela 10.

## 10. Perfil base e capacidades

Mostrar somente de forma informativa:

- perfil base em apresentação amigável;
- indicação discreta de permissões personalizadas, quando houver override efetivo.

Não mostrar matriz completa de capacidades por padrão.

Mensagem auxiliar possível:

`Seu acesso é administrado pela empresa.`

A Tela 11 não oferece:

- troca de perfil;
- solicitação de elevação;
- checkboxes de capacidade;
- restaurar preset;
- controle de `is_primary_admin`.

## 11. Salvamento dos dados pessoais

Avatar, nome e cargo usam salvamento explícito e conjunto.

```text
editar perfil
→ Salvar alterações
→ Host valida sessão + identidade do próprio usuário + campos/arquivo
→ writer/transaction quando aplicável
→ commit
→ evento/refetch
→ UI confirma sucesso
```

Regras:

- sem autosave inicial;
- botão de save habilitado somente quando houver mudança local válida;
- sair com mudanças não salvas exige confirmação;
- não criar rascunho local persistente.

## 12. Alteração da própria senha

A senha é alterada em operação separada do save do perfil.

Campos obrigatórios:

- senha atual;
- nova senha;
- confirmação da nova senha.

Regras:

- Host verifica a senha atual;
- nova senha segue a política vigente do Host;
- confirmação deve coincidir;
- UI não hardcoda parâmetros numéricos ainda pendentes;
- frases-senha permanecem aceitas conforme contrato;
- nunca revelar senha existente;
- nunca registrar senha em logs/auditoria;
- campos de senha começam vazios e são limpos após sucesso/fechamento.

Se o usuário não souber a senha atual, usa o fluxo administrativo de redefinição da Tela 10 quando houver responsável autorizado. Não criar recuperação por e-mail nesta versão.

## 13. Sessões após troca da própria senha

Decisão consolidada para a primeira versão:

```text
Host confirma a troca da senha
→ sessão corrente permanece válida
→ demais sessões ativas da mesma conta são revogadas
→ Client atual confirma sucesso sem forçar novo login
```

Uma política futura que revogue também a sessão corrente exige decisão explícita; não pode ser inferida durante implementação.

## 14. Feedback de senha

Sucesso:

`Senha alterada.`

Senha atual inválida:

`A senha atual informada não está correta.`

Nova senha fora da política:

- informar critérios permitidos pela política vigente;
- não expor Argon2id, hash ou salt.

Falha genérica:

`Não foi possível alterar a senha.`

## 15. Estados da interface

### Carregando

Manter estrutura estável enquanto os dados do próprio usuário são consultados.

### Host indisponível

Informar indisponibilidade sem IP/porta/path e bloquear save dependente do Host.

### Sessão expirada ou revogada

Retornar ao Login conforme contrato global.

### Conta desativada durante a sessão

Após revogação, retornar ao Login e não manter formulário utilizável como se a conta estivesse ativa.

### Avatar indisponível

Usar placeholder/iniciais sem quebrar a tela.

### Alterações não salvas

Avisar antes de descartar avatar/nome/cargo modificados localmente.

## 16. Concorrência

A conta pode ser alterada administrativamente pela Tela 10 enquanto o próprio usuário edita a Tela 11.

Regras:

- sem `last write wins` silencioso;
- evento remoto não substitui campos locais em edição;
- informar quando houver versão mais recente;
- usar controle otimista equivalente quando necessário;
- em conflito, preservar alterações locais visíveis e oferecer reconsulta.

Mudanças administrativas de perfil/capacidades reconciliam imediatamente o acesso da sessão, independentemente da Tela 11 estar aberta.

## 17. Eventos em tempo real

Eventos relevantes podem sinalizar:

- dados pessoais alterados;
- avatar alterado/removido;
- perfil/capacidades alterados administrativamente;
- conta desativada;
- sessão revogada.

Sem edição local conflitante, Tela 11 e rodapé da sidebar podem refazer consulta e refletir o estado atual.

## 18. Auditoria e privacidade

Pode ser auditado de forma proporcional:

- evento de mudança da própria senha, sem conteúdo da senha;
- alteração de dados pessoais quando a política exigir.

Nunca registrar ou exibir:

- senha atual;
- nova senha;
- confirmação de senha;
- token reutilizável;
- hash de senha sem necessidade explícita;
- detalhes sensíveis de sessão.

## 19. Validações

### Perfil

- nome/cargo respeitam limites e normalização do Host;
- avatar respeita tipo/tamanho/dimensões autorizados;
- payload não altera campos protegidos;
- alvo da operação deriva da sessão, não de `user_id` livre enviado pelo Client.

### Senha

- senha atual obrigatória;
- nova senha segue política vigente;
- confirmação coincide;
- senha tratada apenas durante a operação;
- parâmetros numéricos ainda pendentes continuam pendentes.

## 20. Acessibilidade

- avatar com nome/texto acessível;
- ações de foto acionáveis por teclado;
- labels visíveis para nome, cargo e senha;
- mostrar/ocultar senha com nome acessível e estado anunciado;
- foco visível;
- erros ligados ao campo correspondente;
- feedback não depende somente de cor;
- foco retorna corretamente após fechar diálogo de senha.

## 21. Janelas menores

Desktop Windows permanece alvo principal.

Em janela menor suportada:

- campos passam para uma coluna;
- avatar permanece compacto;
- ações não exigem scroll horizontal;
- não criar experiência mobile/hamburger antecipadamente.

## 22. Fora do escopo

- administrar outros usuários;
- alterar o próprio login;
- alterar o próprio perfil/capacidades;
- ativar/desativar a própria conta;
- transferir ADM primário;
- solicitar permissões/workflow de aprovação;
- recuperação ou convite por e-mail;
- MFA;
- SSO/Active Directory;
- tela detalhada de sessões/dispositivos;
- editor avançado de avatar;
- política numérica final de senha;
- parâmetros Argon2id finais;
- duração final de sessão;
- implementação funcional.

## 23. Relação com outras telas

### Tela 02 — Shell

Avatar/nome/cargo confirmados alimentam o resumo do perfil no rodapé.

### Tela 10 — Usuários/Permissões

Administração da conta permanece separada do perfil pessoal.

### Login

Expiração ou revogação de sessão pode redirecionar ao Login conforme contrato global.

## 24. Decisões consolidadas pelo PO

1. página única simples com seções `Perfil` e `Segurança`;
2. avatar, nome e cargo editáveis com um único `Salvar alterações`;
3. login somente leitura;
4. perfil base e indicação de personalização somente informativos;
5. nenhuma alteração de autoridade pela Tela 11;
6. avatar pode ser escolhido, substituído e removido, com preview antes do save e placeholder/iniciais quando ausente;
7. própria senha alterada em fluxo separado com senha atual + nova + confirmação;
8. após troca de senha, sessão atual permanece e demais sessões da conta são revogadas;
9. sem recuperação por e-mail; esquecimento usa redefinição administrativa autorizada;
10. sem autosave do perfil;
11. conflitos concorrentes preservam alterações locais;
12. parâmetros numéricos pendentes de autenticação continuam pendentes.

## 25. Critérios de aceite

- [x] estrutura `Perfil` + `Segurança` aprovada;
- [x] avatar/nome/cargo editáveis aprovados;
- [x] login somente leitura aprovado;
- [x] perfil/capacidades somente informativos aprovados;
- [x] fluxo de avatar aprovado;
- [x] fluxo separado de alteração de senha aprovado;
- [x] senha atual obrigatória aprovada;
- [x] manter sessão atual e revogar demais após troca de senha aprovado;
- [x] usuário não consegue elevar a própria autoridade;
- [x] parâmetros pendentes de autenticação continuam pendentes;
- [x] Host permanece autoridade final;
- [x] nenhuma implementação funcional foi criada.
