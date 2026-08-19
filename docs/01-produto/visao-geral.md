# Visão Geral do Produto — StepFlow Pocket

**Status:** CONSOLIDADO EM NÍVEL INICIAL

## 1. Propósito

O StepFlow Pocket é um aplicativo interno para centralizar documentação de processos técnicos e transformar procedimentos estáticos em guias operacionais fáceis de consultar e executar.

O foco não é criar um sistema burocrático de gestão documental. O foco é permitir que técnicos encontrem uma documentação, entendam rapidamente o contexto e executem cada etapa com clareza.

## 2. Usuários

### ADM

Responsável por controle total do sistema, configurações, usuários, permissões e documentações.

### Gerência

Responsável por manutenção operacional das documentações e, conforme permissão, gestão de usuários não superiores ao nível administrativo principal.

### Funcionário / Técnico

Responsável por consultar e executar documentações. Por padrão não altera o conteúdo oficial.

## 3. Cenário de uso principal

O técnico acessa o compartilhamento interno:

`\\192.168.5.7\Arquivos\StepFlow\`

Encontra o ponto de entrada do StepFlow, dá dois cliques, realiza login e utiliza a aplicação.

Não deve ser necessário instalar, configurar banco, informar endereço de servidor ou executar comandos manualmente em cada estação para o uso normal.

## 4. Proposta de valor

- documentação técnica centralizada;
- leitura rápida e organizada;
- execução passo a passo;
- instruções copiáveis com um clique;
- checklists durante execução;
- versionamento e histórico;
- permissões simples;
- uso offline da Internet e restrito à rede interna;
- manutenção centralizada;
- possibilidade de uso simultâneo por vários técnicos;
- exportação da documentação para formatos compartilháveis.

## 5. Modelo de uma documentação de processo

Campos principais:

- Código;
- Título;
- Área / Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas do processo;
- Histórico de alterações.

O produto deve resistir à inclusão de campos burocráticos sem valor operacional claro.

## 6. Etapas como páginas

Cada processo é percebido como um manual/livro.

Cada etapa funciona como uma página navegável e pode conter:

- título;
- texto introdutório;
- passos numerados;
- detalhes dos passos;
- observações;
- checklist;
- alertas/informações;
- blocos copiáveis para comandos, caminhos, parâmetros ou instruções específicas;
- navegação anterior/próxima;
- indicador de posição dentro do processo.

O bloco copiável usa apenas ícone de cópia, sem botão textual grande.

## 7. Gestão das documentações

Usuários autorizados devem poder:

- criar;
- editar;
- salvar nova revisão/versão conforme a regra definida;
- excluir ou arquivar conforme política futura;
- pesquisar;
- filtrar;
- consultar histórico;
- exportar.

Usuários de leitura podem consultar e interagir com elementos de execução sem alterar a documentação oficial.

## 8. Concorrência

O sistema precisa aceitar vários usuários simultâneos em computadores diferentes.

Requisitos funcionais:

- uma sessão não pode corromper ou bloquear indevidamente outra;
- alterações devem ser coordenadas pelo host;
- edições concorrentes da mesma documentação não podem resultar em sobrescrita silenciosa;
- clientes devem ser informados quando dados relevantes forem atualizados;
- o sistema pode indicar que outro usuário está editando um documento sem necessariamente impedir leitura.

## 9. Usuários e perfis

Conta contém:

- nome/identificador de login;
- nome de exibição;
- cargo;
- senha;
- avatar;
- perfil/permissões.

O próprio usuário pode editar avatar, nome de exibição, cargo e senha dentro das regras definidas.

Perfis padrão:

- ADM;
- Gerência;
- Funcionário.

Permissões devem ser efetivamente verificadas no host.

## 10. Configuração da empresa

O ADM deve poder personalizar informações institucionais relevantes, incluindo logo PNG transparente.

A interface deve informar dimensões, proporção e peso recomendados antes do upload.

O logo é exibido de forma pequena e clássica no topo esquerdo da sidebar.

## 11. Exportação

Objetivo funcional:

- exportar documentação para PDF;
- exportar documentação para DOCX;
- permitir impressão.

O documento exportado deve ter template próprio e poder incorporar identidade da empresa.

## 12. Backup

O sistema deve permitir backup e restauração de maneira simples, adequada a uma aplicação interna pequena.

## 13. Requisitos não funcionais principais

### Usabilidade

- início por duplo clique;
- interface limpa;
- poucos passos para encontrar um processo;
- navegação previsível;
- leitura confortável;
- ações destrutivas claras;
- feedback imediato para copiar, salvar e conflitos.

### Manutenibilidade

- JavaScript modular;
- código dividido por responsabilidade;
- evitar HTML/JS monolítico;
- arquitetura documentada;
- baixo acoplamento;
- tarefas pequenas e revisáveis.

### Disponibilidade interna

- deve funcionar sem Internet;
- depende apenas dos recursos necessários dentro da rede/estações;
- falha do host deve ser apresentada de forma compreensível, sem corromper dados locais do cliente.

### Compatibilidade

A faixa final de versões Windows suportadas ainda precisa de validação técnica antes de ser prometida como requisito definitivo.

### Segurança proporcional

- hash de senha;
- autorização real no host;
- sessão autenticada;
- trilha de alterações relevantes;
- dados reais fora do Git.

## 14. Fora do escopo inicial

A menos que uma decisão futura altere explicitamente:

- acesso público pela Internet;
- autenticação por redes sociais;
- MFA complexo;
- recuperação de senha por email;
- multiempresa/SaaS;
- grande infraestrutura distribuída;
- edição colaborativa caractere a caractere;
- chat corporativo;
- workflow burocrático de aprovação documental com múltiplas instâncias.

## 15. Critério de sucesso do Pocket

Um técnico deve conseguir abrir o StepFlow, localizar uma documentação e executar um processo passo a passo com menos atrito do que teria abrindo arquivos Word/PDF dispersos em pastas de rede.
