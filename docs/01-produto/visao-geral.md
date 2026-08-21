# Visão Geral do Produto — StepFlow Pocket

**Status:** CONSOLIDADO

## Propósito

O StepFlow é uma aplicação interna para centralizar documentação de processos técnicos, transformar procedimentos estáticos em guias operacionais fáceis de consultar e executar e, quando necessário, registrar ocorrências reais de atendimento/execução.

O objetivo é reduzir atrito para o técnico, não criar um portal burocrático de gestão documental.

O produto não é restrito à manutenção de computadores. Deve acomodar procedimentos de manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias técnicos e outros procedimentos internos.

## Usuários

- **ADM:** controle total, configurações, usuários, permissões e documentações;
- **Gerência:** manutenção das documentações e gestão delegada de usuários não-ADM;
- **Funcionário/Técnico:** consulta e execução; por padrão não altera conteúdo oficial.

## Experiência principal

```text
ponto de entrada interno do StepFlow
        ↓
duplo clique
        ↓
Client local preparado/atualizado
        ↓
login
        ↓
consulta e execução
```

O técnico não deve instalar dependências, informar banco/servidor manualmente nem executar comandos no uso normal.

O endereço real do ponto de entrada corporativo ainda não está definido. Usar apenas placeholders como:

`\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\`

## Procedimentos e categorização

O procedimento é o **modelo reutilizável oficial** que explica como executar determinada atividade.

Campos principais:

- Código;
- Título;
- Área/Departamento;
- Categorias;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas;
- Histórico de alterações.

Categorias são configuráveis pela empresa e um procedimento pode pertencer a uma ou mais categorias. Exemplos como Manutenção, TI, Service Desk, Help Desk, Infraestrutura, Redes e Guias não são hardcoded.

Não adicionar campos burocráticos sem valor operacional aprovado.

## Etapas como páginas de manual

Cada etapa deve funcionar como uma página navegável e pode conter título, introdução, passos/subpassos, checklist documental, observações, alertas, comandos/blocos copiáveis, navegação anterior/próxima e indicação de progresso.

O controle de cópia deve ser discreto, somente por ícone, com feedback curto.

## Atendimento/execução

Quando houver necessidade de registrar um serviço real, o StepFlow deve separar:

- **Procedimento:** como fazer;
- **Atendimento/Execução:** o que foi feito em uma ocorrência concreta;
- **Equipamento:** ativo físico opcional relacionado ao atendimento.

Um atendimento pode usar um ou mais procedimentos e deve preservar qual revisão do procedimento foi efetivamente utilizada.

Procedimentos gerais podem existir e ser consultados sem atendimento formal. Atendimentos podem existir sem equipamento quando o contexto não envolver um ativo físico.

Detalhes em `categorizacao-atendimentos-equipamentos.md`.

## Equipamentos

Para cenários como manutenção de computadores/notebooks, deve existir ficha opcional com informações como:

- código interno estável;
- nome do equipamento;
- cliente/solicitante/responsável;
- processador;
- RAM;
- armazenamento;
- sistema operacional e versão;
- número de série/patrimônio quando disponíveis;
- MAC(s) quando úteis;
- saúde da bateria quando aplicável;
- observações.

MAC, serial ou patrimônio podem ajudar na busca, mas não substituem o identificador interno estável do StepFlow.

## Ficha/relatório compacto

Atendimentos com equipamento devem poder gerar saída compacta própria para impressão e anexação física ao equipamento, contendo identificação, características principais, resumo do serviço/procedimentos realizados e observações.

Essa saída é documento próprio, não captura de tela. Formato físico e estratégia técnica serão fechados no Bloco 10.

## Busca operacional

Além de buscar procedimentos por código/título/termo/categoria, o sistema deve permitir localizar registros operacionais por informações úteis como:

- código de atendimento;
- ordem de serviço/referência externa;
- código/nome do equipamento;
- cliente/solicitante;
- serial/patrimônio;
- MAC normalizado quando disponível.

## Multiusuário

O produto deve aceitar vários usuários simultâneos em computadores diferentes:

- Clients nunca acessam SQLite diretamente;
- escritas são coordenadas pelo Host;
- edições antigas não sobrescrevem silenciosamente alterações recentes;
- mudanças relevantes chegam aos Clients por eventos/reconsulta;
- fila de escrita não substitui revisão otimista.

Essas regras também se aplicam aos novos registros de equipamento/atendimento quando houver risco de alteração concorrente.

## Usuários e permissões

Conta possui identificador estável, login, nome de exibição, cargo, hash de senha, avatar, perfil e permissões.

O usuário pode editar nome de exibição, cargo, avatar e senha dentro das regras. Autorização é sempre verificada no Host.

A matriz operacional para categorias, equipamentos e atendimentos será fechada antes da implementação correspondente.

## Exportação e backup

Requisitos obrigatórios para documentação:

- PDF;
- DOCX;
- impressão;
- identidade da empresa no documento exportado;
- backup e restauração simples.

A exportação usa modelo próprio de documento, não captura da tela.

O Bloco 10 também deve fechar a saída compacta imprimível de atendimento/equipamento.

## Requisitos não funcionais

### Pocket

- implantação da máquina central por pasta pronta;
- nenhum runtime/toolchain de desenvolvimento no servidor;
- nenhum serviço/processo StepFlow persistente após encerramento do ciclo central;
- Client distribuído sem instalador tradicional obrigatório;
- funcionamento sem dependência da Internet durante o uso normal.

### Compatibilidade

Baseline inicial: **Windows 10/11 x64**. Tauri 2 usa WebView2. Versões reais das estações e presença do runtime ainda serão verificadas no ambiente corporativo.

### Manutenibilidade

- frontend modular em HTML/CSS/JavaScript com ES Modules;
- baixo acoplamento;
- código organizado por responsabilidade/domínio;
- evitar monólitos e superengenharia.

### Segurança proporcional

- Argon2id para senhas;
- sessão autenticada e opaca;
- autorização no Host;
- auditoria de ações relevantes;
- nenhum dado real/segredo no Git.

## Fora do escopo inicial

- acesso público pela Internet;
- SaaS/multiempresa;
- CRM completo/faturamento;
- estoque de peças;
- inventário corporativo completo/RMM;
- sistema completo de chamados/SLA;
- MFA complexo e recuperação por email;
- edição colaborativa caractere a caractere;
- chat corporativo;
- workflow burocrático de aprovação em múltiplas instâncias;
- infraestrutura distribuída de grande porte.

## Pendências de produto ainda abertas

O Bloco 9 deve fechar o comportamento operacional do atendimento/execução, incluindo marcações de checklist, lifecycle e regras de conclusão/reabertura quando necessárias.

Também permanecem pendentes os detalhes exatos de permissões operacionais e do formato físico da ficha compacta.

## Critério de sucesso

Um técnico deve conseguir localizar o procedimento adequado, executá-lo com baixo atrito e, quando o trabalho exigir rastreabilidade, registrar o atendimento/equipamento e produzir um resumo físico útil sem recorrer a controles paralelos dispersos.
