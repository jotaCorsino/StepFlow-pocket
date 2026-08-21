# Tela 01 — Login

**Status:** CONSOLIDADO FUNCIONALMENTE / IDENTIDADE VISUAL DETALHADA COMPARTILHADA COM O SHELL  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo da tela

Permitir que um usuário existente e ativo se autentique no StepFlow com o mínimo de atrito, sem expor configuração técnica da implantação.

A tela de Login não cria usuários, não configura Host/banco e não executa bootstrap de ADM pela rede.

## 2. Atores e permissões

A tela é comum a:

- ADM;
- Gerência;
- Funcionário.

A diferença de permissões só passa a valer depois da autenticação. Usuário inativo não obtém sessão.

## 3. Como o usuário chega à tela

```text
ponto de entrada interno
→ launcher prepara/inicia Client local
→ Client lê deployment.json
→ Client valida compatibilidade/alcance do Host
→ Login
```

Se o Host estiver indisponível ou Client/Host forem incompatíveis, o usuário deve receber esse estado antes de uma tentativa normal de autenticação sempre que tecnicamente possível.

## 4. Layout e hierarquia visual

### Consolidado

- visual corporativo, limpo e discreto;
- uma única área central de autenticação;
- foco imediato nos campos;
- identificação discreta do StepFlow/empresa;
- sem painel decorativo grande ou composição dupla;
- nenhum campo técnico de servidor, banco, porta ou path;
- nenhuma propaganda, feed ou conteúdo não relacionado ao acesso.

Composição funcional aprovada:

```text
[ identidade StepFlow / empresa ]

Entrar no StepFlow

Login
[________________________]

Senha
[____________________] [mostrar/ocultar]

[ Entrar ]

mensagem contextual, quando necessária
```

Cores, tipografia, dimensões, espaçamento, bordas e sombras devem usar o mesmo sistema visual definido para o Shell principal. Não criar um estilo visual independente só para o Login.

## 5. Elementos fixos

Necessários:

- identificação discreta do StepFlow/empresa;
- campo `Login`;
- campo `Senha`;
- botão discreto para mostrar/ocultar senha;
- ação primária `Entrar`;
- área de feedback contextual.

Não incluir na primeira versão:

- “lembrar-me”;
- recuperação por e-mail;
- cadastro livre;
- MFA complexo;
- seleção manual de servidor;
- configuração de rede/porta;
- seleção de perfil/role no login.

## 6. Componentes

### Login

- texto simples;
- obrigatório;
- aceita o identificador de login definido para o usuário;
- não expõe indicação prévia de existência da conta.

### Senha

- obrigatória;
- mascarada por padrão;
- ação mostrar/ocultar é temporária e discreta;
- nunca persistir a senha no Client.

### Entrar

- ação primária;
- durante autenticação, evita envio duplicado;
- não indica sucesso antes da resposta válida do Host.

## 7. Interações

1. usuário informa login;
2. usuário informa senha;
3. aciona `Entrar` ou pressiona Enter quando apropriado;
4. Client envia credenciais ao Host;
5. Host valida usuário e senha;
6. em sucesso, Client recebe sessão opaca;
7. o usuário segue para `Início/Dashboard`;
8. em falha, permanece no Login e recebe feedback adequado.

A navegação só ocorre após sessão válida.

## 8. Navegação

**Destino aprovado após login:** `Início/Dashboard`.

Logout futuro revoga a sessão e retorna ao Login.

## 9. Estados

### Inicial

- campos vazios;
- foco inicial no campo Login;
- botão Entrar segue a regra de validação local.

### Autenticando

- `Entrar` fica temporariamente indisponível contra duplicação;
- feedback curto de atividade;
- não criar requisições concorrentes a partir dos campos.

### Credenciais inválidas

Mensagem genérica, sem revelar qual parte estava correta.

Direção textual:

`Login ou senha inválidos.`

### Usuário inativo

Informar que o acesso não está disponível e orientar contato interno, sem detalhes administrativos desnecessários.

### Host indisponível

Distinguir claramente de credencial inválida. Não oferecer edição de IP/porta.

### Client/Host incompatíveis

Bloquear autenticação e orientar reinício pelo ponto de entrada/launcher quando essa for a recuperação aplicável.

### Erro interno

Mensagem simples; detalhes técnicos seguros permanecem no diagnóstico do Host.

## 10. Validações

No Client:

- Login obrigatório;
- Senha obrigatória;
- normalização do login apenas conforme contrato do Host;
- não alterar silenciosamente a senha.

No Host:

- usuário existe e está ativo;
- hash Argon2id confere;
- política de autenticação/sessão vigente;
- autorização posterior sempre Host-side.

Parâmetros ainda pendentes em `autenticacao-sessao-autorizacao.md` continuam pendentes.

## 11. Feedback e mensagens

- curtos e operacionais;
- sem stack trace, SQL, path interno ou segredo;
- diferenciar autenticação inválida de indisponibilidade do sistema;
- não facilitar enumeração de usuários;
- erro comum de credencial aparece contextualizado na própria área do Login, sem modal.

## 12. Dados exibidos

- identidade StepFlow/empresa;
- Login/Senha;
- estado de autenticação ou erro;
- incompatibilidade/indisponibilidade quando aplicável.

## 13. Dados enviados/alterados

Enviados ao Host:

- login;
- senha;
- metadados técnicos mínimos previstos pelo contrato, se necessários.

O Login não altera dados de negócio, exceto sessão e registros de segurança/auditoria aplicáveis.

## 14. Regras de negócio

- somente usuário ativo autentica;
- login não escolhe perfil;
- perfil/permissões vêm do Host;
- primeiro ADM não é criado automaticamente pelo primeiro Client da rede;
- sem cadastro livre;
- sem “lembrar-me” na primeira versão.

## 15. Autorização

Não há capacidade especial necessária para tentar autenticação.

Após autenticar, o Host define as capacidades efetivas; o Client adapta a UI, mas nunca concede autoridade por conta própria.

## 16. Persistência

- consulta `users`/equivalente pelo Host;
- validação de `password_hash`;
- criação de sessão server-side;
- atualização de atividade/auditoria quando aplicável.

Nenhuma credencial reutilizável deve ser persistida em texto puro.

## 17. Contratos Client ↔ Host necessários

Sem fixar rotas finais nesta etapa, a tela depende de:

1. compatibilidade Client↔Host antes do login;
2. autenticação login+senha;
3. retorno de sessão opaca;
4. erros distinguíveis para credencial inválida, usuário inativo, incompatibilidade, indisponibilidade e erro interno;
5. logout/revogação para o fluxo posterior.

## 18. Eventos em tempo real

Nenhum WebSocket é necessário para o login inicial. O canal autenticado é estabelecido após sessão válida.

## 19. Concorrência

Login simultâneo de usuários diferentes é permitido.

A política sobre múltiplas sessões do mesmo usuário não é definida por esta tela e não pode ser inventada na implementação.

## 20. Acessibilidade e teclado

- labels associados aos campos;
- ordem de Tab previsível;
- Enter pode acionar login quando aplicável;
- foco visível;
- erros não dependem apenas de cor;
- mostrar/ocultar senha é acionável por teclado e possui nome acessível.

## 21. Tamanhos de janela

A tela deve continuar utilizável nos tamanhos suportados pelo Client sem esconder os campos essenciais. Dimensões mínimas exatas serão fechadas junto do sistema visual do Shell.

## 22. Decisões visuais preservadas

Aprovado:

- linguagem corporativa limpa/discreta;
- composição central única;
- marca discreta;
- sem painel decorativo grande;
- botão mostrar/ocultar senha;
- nenhuma configuração técnica exposta;
- destino pós-login `Início/Dashboard`.

Compartilhado com o contrato visual do Shell:

- paleta;
- tipografia;
- escala de espaçamento;
- raios/bordas/sombras;
- dimensões mínimas de janela;
- estilo final dos componentes.

## 23. Divergências anteriores

Nenhuma divergência de requisito. As três propostas funcionais originalmente abertas foram aprovadas pelo PO em 2026-08-21.

## 24. Decisões consolidadas nesta análise

1. Login usa composição central simples em uma única área;
2. senha é mascarada por padrão com ação mostrar/ocultar;
3. login bem-sucedido navega para `Início/Dashboard`;
4. identidade visual detalhada deve ser compartilhada com o Shell, não definida isoladamente.

## 25. Pendências

- política de múltiplas sessões do mesmo usuário somente se necessária antes da implementação;
- parâmetros operacionais ainda pendentes em autenticação;
- tokens visuais finais dependem da consolidação do Shell.

Nenhuma dessas pendências reabre as decisões funcionais desta tela.

## 26. Fora do escopo

- bootstrap do primeiro ADM;
- criação/gestão de usuários;
- redefinição administrativa de senha;
- recuperação por e-mail;
- MFA;
- “lembrar-me”;
- parâmetros Argon2id/senha/sessão pendentes;
- implementação Tauri/código de produção.

## 27. Critérios de aceite da especificação

- [x] fluxo alinhado ao launcher/Host;
- [x] autenticação/autorização Host-side;
- [x] nenhuma configuração técnica exposta;
- [x] estados principais diferenciados;
- [x] composição central aprovada;
- [x] destino `Início/Dashboard` aprovado;
- [x] mostrar/ocultar senha aprovado;
- [x] detalhes visuais remetidos ao mesmo sistema do Shell;
- [x] nenhuma UI de produção criada.

**Tela 01 — Login: contrato funcional consolidado para a Fase 1.**

## 28. Casos de teste futuros

1. login válido de usuário ativo;
2. senha inválida;
3. login inexistente sem enumeração explícita;
4. usuário inativo;
5. Host indisponível;
6. Client/Host incompatíveis;
7. Enter envia uma única tentativa;
8. clique repetido não duplica autenticação;
9. senha permanece mascarada por padrão;
10. mostrar/ocultar funciona por mouse/teclado;
11. sessão válida leva a `Início/Dashboard`;
12. nenhuma senha/token reutilizável aparece em logs do Client.
