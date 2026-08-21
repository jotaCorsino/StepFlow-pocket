# Tela 01 — Login

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
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

Fluxo esperado:

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
- sem aparência de portal burocrático;
- foco imediato nos campos de autenticação;
- nenhum campo técnico de servidor, banco, porta ou path;
- nenhuma propaganda, feed ou conteúdo não relacionado ao acesso.

### Proposta para aprovação do PO

Composição simples em uma única área central:

```text
[ identidade StepFlow / empresa ]

Entrar no StepFlow

Login
[________________________]

Senha
[____________________] [mostrar/ocultar]

[ Entrar ]

mensagem de estado/erro, quando necessária
```

A proposta evita painel duplo, ilustração decorativa grande e excesso de informação. Cores, dimensões, tipografia exata, raio de borda, sombras e posição final da marca permanecem para aprovação visual.

## 5. Elementos fixos

Necessários:

- identificação discreta do StepFlow/empresa;
- campo `Login`;
- campo `Senha`;
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

## 6. Componentes e blocos

### Campo Login

- texto simples;
- obrigatório;
- não expõe indicação de existência da conta antes do envio;
- aceita o identificador de login definido para o usuário.

### Campo Senha

- obrigatório;
- mascarado por padrão;
- proposta: botão discreto para mostrar/ocultar temporariamente;
- nunca persistir a senha no Client.

### Botão Entrar

- ação primária da tela;
- durante a requisição, evita envios duplicados;
- não deve transmitir impressão de sucesso antes da resposta do Host.

## 7. Interações do usuário

1. usuário informa login;
2. usuário informa senha;
3. usuário aciona `Entrar` ou pressiona Enter quando apropriado;
4. Client envia credenciais ao Host pelo contrato autenticador;
5. Host valida usuário e senha;
6. em sucesso, Client recebe sessão opaca e segue para a área autenticada;
7. em falha, a tela permanece no Login e apresenta mensagem adequada.

A navegação para a área autenticada só ocorre após sessão válida.

## 8. Navegação e destinos

### Proposta

Após login bem-sucedido, abrir `Início/Dashboard` como destino inicial padrão.

Essa escolha é **PROPOSTA DE UX** até aprovação explícita do PO. Nenhum destino alternativo deve ser inventado pelo executor.

Logout futuro retorna para a tela de Login após revogar a sessão.

## 9. Estados da interface

### Inicial

- campos vazios;
- foco inicial proposto no campo Login;
- botão Entrar disponível conforme regra de validação da UI.

### Autenticando

- ação Entrar fica ocupada/bloqueada contra duplicação;
- campos não devem iniciar requisições concorrentes;
- feedback curto, sem travar a aplicação indefinidamente.

### Credenciais inválidas

Mensagem genérica, sem revelar se login ou senha individualmente estava correto.

Exemplo de direção textual:

`Login ou senha inválidos.`

Texto final será tratado como microcopy da especificação visual.

### Usuário inativo

Informar que o acesso não está disponível e orientar contato interno, sem revelar detalhes administrativos desnecessários.

### Host indisponível

Distinguir de “senha errada”. O usuário deve entender que o StepFlow não conseguiu alcançar o serviço central.

Não oferecer edição manual de IP/porta na tela.

### Client/Host incompatíveis

Bloquear login e orientar o usuário a reiniciar pelo ponto de entrada/launcher quando essa for a recuperação aplicável.

### Erro interno

Mensagem simples ao usuário; detalhes técnicos e `request_id` seguros ficam disponíveis para diagnóstico conforme arquitetura.

## 10. Validações

No Client:

- Login obrigatório;
- Senha obrigatória;
- trim/normalização do login somente conforme contrato definido pelo Host;
- não alterar silenciosamente conteúdo de senha.

No Host:

- usuário existe e está ativo;
- hash Argon2id confere;
- política de autenticação e sessão vigente;
- autorização posterior sempre Host-side.

Parâmetros ainda marcados como pendentes em `autenticacao-sessao-autorizacao.md` não são decididos por esta tela.

## 11. Mensagens e feedbacks

Princípios:

- curtas e operacionais;
- sem stack trace, SQL, path interno ou segredo;
- diferenciar problema de autenticação de indisponibilidade do sistema;
- não expor informação útil para enumeração de usuários;
- evitar modal para erro comum de credencial; preferir mensagem contextual na própria área de Login.

## 12. Dados exibidos

- identidade visual do StepFlow/empresa, quando disponível;
- labels e campos de Login/Senha;
- status de autenticação/erro;
- informação de incompatibilidade/indisponibilidade quando aplicável.

## 13. Dados enviados/alterados

Enviados ao Host durante login:

- login;
- senha;
- metadados técnicos mínimos necessários ao contrato, se definidos.

Nenhum dado de negócio é alterado pelo Login, exceto criação/atualização de sessão e registros de auditoria/segurança quando aplicável.

## 14. Regras de negócio

- somente usuário ativo autentica;
- login não escolhe perfil;
- perfil/permissões vêm do usuário autorizado pelo Host;
- primeiro ADM não é criado automaticamente pelo primeiro Client da rede;
- tela não oferece cadastro livre;
- sessão não é persistida como “lembrar-me” na primeira versão.

## 15. Regras de autorização

Não há capacidade especial necessária para tentar login.

Após autenticação, o Host define as capacidades efetivas da sessão; o Client adapta a UI, mas nunca concede autoridade por conta própria.

## 16. Impacto em persistência

- consulta `users`/equivalente pelo Host;
- validação de `password_hash`;
- criação de sessão server-side;
- possível atualização de atividade/registro de auditoria conforme arquitetura final.

Nenhuma credencial reutilizável deve ser persistida em texto puro.

## 17. Contratos Client ↔ Host necessários

Sem fixar nomes finais de rotas nesta etapa, a UI depende de:

1. verificação de compatibilidade Client↔Host antes do login;
2. comando de autenticação com login+senha;
3. retorno de sessão opaca em sucesso;
4. erros distinguíveis para credencial inválida, usuário inativo, incompatibilidade, indisponibilidade e erro interno;
5. logout/revogação para o fluxo posterior.

## 18. Eventos em tempo real relevantes

Nenhum WebSocket é necessário para executar o login inicial.

O canal autenticado de eventos é estabelecido após sessão válida, conforme arquitetura de comunicação.

## 19. Comportamento de concorrência

Login simultâneo de usuários diferentes é permitido.

A política futura sobre múltiplas sessões do mesmo usuário não está definida nesta tela e não deve ser inventada pelo executor.

## 20. Acessibilidade e teclado

Direção obrigatória:

- labels reais associados aos campos;
- ordem de Tab previsível;
- Enter pode acionar login quando os campos necessários estiverem preenchidos;
- foco visível;
- estados de erro não dependem apenas de cor;
- controles de mostrar/ocultar senha devem ser acessíveis por teclado e possuir nome acessível.

## 21. Tamanhos de janela suportados

A tela deve continuar utilizável em janelas menores suportadas pelo Client sem esconder campos essenciais ou exigir resolução incomum.

Dimensões mínimas e comportamento responsivo exatos serão fechados no contrato visual do Bloco 8.

## 22. Preservação visual / decisões aprovadas

Já vigentes:

- linguagem corporativa limpa/discreta;
- nenhuma complexidade técnica exposta ao técnico;
- foco na tarefa principal;
- identidade da empresa preservada sem deformar logo.

Ainda não aprovados:

- composição central exata;
- posição/tamanho da marca no Login;
- tipografia;
- cores;
- espaçamentos;
- bordas/sombras;
- ícones;
- microcopy final.

## 23. Divergências com documentação anterior

Nenhuma divergência de requisito foi identificada.

Esta especificação apenas transforma as decisões de autenticação e experiência operacional em contrato de tela. Parâmetros técnicos ainda pendentes permanecem pendentes.

## 24. Decisões consolidadas nesta análise

Por enquanto, nenhuma nova decisão visual é considerada aprovada apenas pela criação deste documento.

Os comportamentos que já eram requisitos permanecem consolidados; os itens marcados como proposta aguardam decisão do PO.

## 25. Pendências para aprovação

1. aprovar ou ajustar a composição visual central proposta;
2. aprovar `Início/Dashboard` como destino após login;
3. aprovar botão mostrar/ocultar senha;
4. definir posteriormente identidade visual detalhada (cores/tipografia/espaçamento) junto do contrato visual do shell;
5. decidir política de múltiplas sessões do mesmo usuário somente se isso se tornar requisito antes da implementação.

## 26. Fora do escopo

- bootstrap do primeiro ADM;
- tela de criação/gestão de usuário;
- redefinição administrativa de senha;
- recuperação por e-mail;
- MFA;
- persistência de sessão “lembrar-me”;
- escolha de parâmetros Argon2id/senha/sessão ainda pendentes;
- implementação Tauri/código de produção.

## 27. Critérios de aceite da especificação

- [x] fluxo de entrada alinhado ao launcher/Host;
- [x] autenticação e autorização continuam Host-side;
- [x] nenhuma configuração técnica exposta ao usuário;
- [x] estados principais de erro diferenciados;
- [x] parâmetros pendentes não transformados em requisitos;
- [x] nenhuma UI de produção criada;
- [ ] composição visual aprovada pelo PO;
- [ ] destino pós-login aprovado pelo PO;
- [ ] mostrar/ocultar senha aprovado pelo PO;

## 28. Casos de teste/smoke sugeridos para implementação futura

1. login válido de usuário ativo;
2. senha inválida;
3. login inexistente sem enumeração explícita;
4. usuário inativo;
5. Host indisponível;
6. Client/Host incompatíveis;
7. Enter envia uma única tentativa;
8. clique repetido não duplica autenticação;
9. senha permanece mascarada por padrão;
10. navegação por teclado funciona;
11. sessão válida leva ao destino aprovado;
12. nenhuma senha/token reutilizável aparece em logs do Client.
