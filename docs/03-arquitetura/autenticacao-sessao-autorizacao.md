# Autenticação, Sessão e Autorização — StepFlow

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL CONSOLIDADA PARA A FASE 1

## 1. Objetivo

Definir autenticação interna simples, sessões seguras e autorização centralizada no Host, preservando os perfis ADM, Gerência e Funcionário já aprovados.

## 2. Princípios

- autenticação ocorre no Host;
- autorização é validada no Host em toda operação protegida;
- Client nunca é fonte de verdade de permissão;
- senha nunca é armazenada em texto puro;
- sessão inicial é simples e opaca, sem necessidade de JWT;
- nenhuma função de “lembrar-me” persistente na primeira versão;
- fechar o Client encerra a sessão local do usuário;
- dados de autenticação não são gravados em `deployment.json` nem no manifesto do launcher.

## 3. Identidade do usuário

Campos conceituais mínimos:

```text
user_id                 # identificador imutável
login                   # identificador de autenticação único
password_hash
name                     # nome de exibição
job_title                # cargo
avatar_path / avatar_id
role_preset              # ADM | GERENCIA | FUNCIONARIO
is_active
is_primary_admin
created_at
updated_at
```

Relacionamentos históricos/auditoria usam `user_id`, nunca somente nome/login exibido.

O schema físico definitivo será fechado no Bloco 6.

## 4. Senhas

Usar **Argon2id** para novas senhas.

Baseline inicial:

- algoritmo: Argon2id;
- salt aleatório único por senha;
- parâmetros armazenados no próprio formato PHC;
- baseline mínimo inicial: aproximadamente 19 MiB de memória, 2 iterações, paralelismo 1;
- parâmetros poderão ser elevados após medição sem mudar o modelo de dados;
- verificação usa os parâmetros codificados no hash existente;
- possibilidade de rehash futuro após login bem-sucedido quando parâmetros forem atualizados.

Não usar SHA-256/SHA-512 simples, criptografia reversível ou hash caseiro para senhas.

## 5. Regras de senha da primeira versão

Evitar política burocrática excessiva.

Direção inicial:

- tamanho mínimo razoável, inicialmente 10 caracteres;
- aceitar frases-senha e caracteres variados;
- não exigir composição artificial obrigatória do tipo “uma maiúscula + um símbolo + um número” como regra principal;
- não truncar silenciosamente;
- alteração de senha própria exige senha atual, salvo reset administrativo;
- senha não aparece em logs, auditoria ou mensagens de erro.

O PO poderá ajustar o mínimo antes da implementação definitiva.

## 6. Login

Fluxo:

```text
Client
  ↓ login + senha
Host
  ↓ localizar usuário ativo
  ↓ verificar Argon2id
  ↓ validar status/permissões
  ↓ criar sessão opaca
Client
  ↓ recebe token da sessão
```

Mensagens de credencial inválida não devem revelar se o login existe.

## 7. Sessão

Usar token **opaco e aleatório**, não JWT, na primeira versão.

Motivos:

- Host central já mantém estado;
- revogação imediata é simples;
- mudança de permissão/desativação pode invalidar sessão imediatamente;
- evita complexidade desnecessária de assinatura/claims expirados.

Direção:

- token gerado com fonte criptograficamente segura;
- entropia mínima equivalente a 256 bits;
- Client mantém token apenas em memória durante a sessão;
- Host armazena somente representação derivada/hash do token quando tecnicamente adequado, não o token reutilizável em texto puro;
- sessão vinculada a `user_id`, timestamps, validade e estado;
- logout revoga sessão;
- desativação do usuário revoga todas as sessões daquele usuário;
- troca administrativa de permissão relevante pode revogar sessões existentes.

## 8. Expiração

Política inicial:

- expiração por inatividade: proposta inicial de 8 horas, ajustável por configuração;
- validade absoluta máxima: proposta inicial de 24 horas;
- fechar o Client descarta o token local;
- nenhuma renovação infinita silenciosa;
- sessão expirada retorna erro padronizado `SESSION_EXPIRED`.

Os tempos finais poderão ser ajustados antes da implementação se a rotina da empresa exigir.

## 9. Proteção contra tentativas de login

Não criar bloqueio permanente fácil de usar como negação de serviço.

Direção inicial no Host:

- registrar falhas de autenticação de forma segura;
- aplicar atraso/backoff temporário após repetidas falhas por conta/origem;
- não revelar detalhes de existência da conta;
- zerar/relaxar o contador após autenticação válida e janela adequada;
- permitir que ADM reative/reset uma conta se futuramente houver bloqueio administrativo explícito.

Parâmetros exatos serão definidos na implementação após testes locais.

## 10. Perfis como presets

Os perfis `ADM`, `GERENCIA` e `FUNCIONARIO` são **presets de permissões**, não substitutos da autorização por capacidade.

Permissões conceituais iniciais:

```text
process.read
process.create
process.update
process.delete
process.publish
process.export
users.read
users.create
users.update
users.disable
users.assign_admin
company.read
company.update
backup.create
backup.restore
```

A implementação poderá armazenar permissões granulares para permitir ajustes futuros sem reescrever toda a autorização.

## 11. Matriz padrão inicial

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Ler processos | sim | sim | sim |
| Criar/editar processos | sim | sim | não |
| Excluir/publicar processos | sim | sim | não |
| Exportar/imprimir processos | sim | sim | sim |
| Ler lista de usuários | sim | sim | não |
| Criar/editar/desativar não-ADM | sim | sim | não |
| Criar/promover/rebaixar ADM | sim | não | não |
| Alterar ADM principal | regras especiais | não | não |
| Ler configuração da empresa | sim | sim | conforme necessidade de UI |
| Alterar configuração da empresa | sim | não por padrão | não |
| Criar backup | sim | não por padrão | não |
| Restaurar backup | sim | não | não |

A matriz é preset inicial. Permissões específicas podem ser refinadas posteriormente sem violar as restrições de autoridade abaixo.

## 12. Restrições de autoridade

Regras obrigatórias:

- Gerência nunca cria, promove, rebaixa, desativa ou altera permissões de ADM;
- Gerência não altera o ADM principal;
- somente ADM pode conceder capacidade administrativa de ADM;
- o sistema não pode permitir ficar sem pelo menos um ADM ativo;
- `is_primary_admin` identifica a conta administrativa inicial protegida;
- ações sobre contas ADM devem ser auditadas;
- usuário não pode elevar a própria autoridade apenas por edição de perfil.

Regras mais específicas entre múltiplos ADMs poderão ser fechadas antes da implementação do módulo de usuários.

## 13. Edição do próprio perfil

Todo usuário ativo pode, dentro das regras:

- alterar avatar;
- alterar nome de exibição;
- alterar cargo exibido;
- alterar a própria senha.

Não pode alterar por conta própria:

- `user_id`;
- login, se a política futura exigir controle administrativo;
- perfil/preset;
- permissões administrativas;
- `is_primary_admin`;
- estado ativo/inativo.

## 14. Bootstrap do primeiro ADM

Não permitir que “o primeiro Client que chegar na rede” se torne ADM automaticamente.

A primeira conta ADM deve ser criada por fluxo local/controlado na máquina central enquanto o banco ainda está vazio.

Direção:

```text
primeira inicialização controlada do StepFlow Controller
        ↓
Host detecta ausência de usuários
        ↓
fluxo local de bootstrap
        ↓
criação do ADM principal
        ↓
bootstrap é desabilitado permanentemente para aquele banco
```

O formato exato da pequena UI/comando local de bootstrap será definido na implementação do Controller/Host. Não exige serviço, Internet ou instalação adicional.

## 15. Reset de senha administrativo

ADM pode iniciar reset de senha de usuário não-ADM e, conforme regra futura, de outros ADMs autorizados.

Direção inicial segura:

- não recuperar/exibir senha antiga;
- definir nova senha temporária ou fluxo de redefinição;
- opcionalmente exigir troca no próximo login;
- revogar sessões existentes do usuário após reset;
- registrar ação na auditoria sem registrar a senha.

Gerência pode receber capacidade de reset apenas para usuários não-ADM se isso for mantido no preset final.

## 16. Desativação e exclusão

Preferir **desativar** conta em vez de excluir fisicamente quando houver histórico/auditoria relacionados.

Ao desativar:

- novos logins são recusados;
- sessões existentes são revogadas;
- `user_id` e referências históricas permanecem.

A política física de retenção será fechada no Bloco 6.

## 17. Autorização por operação

Exemplo conceitual:

```text
Client mostra botão Editar
        ↓
usuário clica
        ↓
Client envia comando
        ↓
Host verifica sessão
        ↓
Host verifica process.update
        ↓
Host valida recurso/revisão
        ↓
Host executa ou retorna PERMISSION_DENIED
```

Ocultar botão na UI melhora UX, mas nunca substitui validação no Host.

## 18. Auditoria mínima relacionada a segurança

Registrar ao menos:

- login bem-sucedido quando útil ao diagnóstico;
- falhas repetidas de login de forma não sensível;
- logout/revogação administrativa relevante;
- criação/desativação de usuário;
- mudança de perfil/permissão;
- reset de senha;
- alteração de ADM;
- ações administrativas críticas.

Nunca registrar senha ou token de sessão reutilizável.

## 19. Transporte

Credenciais e token trafegam somente pelo canal Client↔Host definido no Bloco 4.

A forma final de proteção do transporte na LAN continua dependente da infraestrutura corporativa real e será validada antes da implantação. Isso não autoriza enviar senha/token por protocolo deliberadamente inseguro em produção.

## 20. Gate do Bloco 5

O Bloco 5 fica arquiteturalmente fechado com:

1. Argon2id para senha;
2. sessão opaca server-side;
3. token somente em memória do Client inicialmente;
4. autorização por capacidade no Host;
5. ADM/Gerência/Funcionário como presets;
6. Gerência impedida de administrar ADM;
7. pelo menos um ADM ativo obrigatório;
8. bootstrap do ADM principal somente por fluxo local/controlado;
9. desativação preservando histórico;
10. ações administrativas relevantes auditáveis.

Próximo bloco: **Bloco 6 — Modelo de dados, schema, migrations, revisão e histórico**.
