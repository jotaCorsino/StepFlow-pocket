# Visão Geral do Produto — StepFlow Pocket

**Status:** CONSOLIDADO

## Propósito

O StepFlow é uma aplicação interna para centralizar documentação de procedimentos técnicos, transformar documentos estáticos em guias operacionais fáceis de consultar/executar e, quando o trabalho exigir rastreabilidade, registrar as informações reais do serviço realizado.

O produto não é restrito à manutenção de computadores. Deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias técnicos e outros procedimentos internos.

O objetivo é reduzir atrito para o técnico, não criar um portal burocrático.

## Usuários

- **ADM:** controle total, configurações, usuários, permissões e documentações;
- **Gerência:** manutenção das documentações e gestão delegada de usuários não-ADM;
- **Funcionário/Técnico:** consulta e execução; por padrão não altera conteúdo oficial.

## Experiência principal

```text
ponto de entrada interno
→ duplo clique
→ Client local preparado/atualizado
→ login
→ consulta/execução
```

O técnico não instala dependências nem informa banco/servidor manualmente no uso normal.

Infraestrutura corporativa real ainda é pendente; exemplos de path continuam apenas conceituais.

## Procedimentos

Campos principais consolidados:

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas;
- Histórico.

Novo requisito confirmado: **categorização de procedimentos**. Cardinalidade, hierarquia e modelo final das categorias ainda estão em aprovação.

Não adicionar campos burocráticos sem valor operacional aprovado.

## Etapas como páginas de manual

Cada etapa funciona como página navegável e pode conter título, introdução, passos/subpassos, checklist documental, observações, alertas, comandos/blocos copiáveis, navegação anterior/próxima e indicação de progresso.

Controle de cópia é discreto, somente por ícone, com feedback curto.

## Registro do serviço/equipamento — novo requisito

Para cenários como manutenção de computadores/notebooks, o sistema deve permitir registrar, quando aplicável:

- nome do equipamento;
- processador;
- RAM;
- armazenamento;
- sistema operacional/versão;
- MAC ou outro identificador útil;
- saúde da bateria;
- observações;
- cliente e/ou ordem de serviço/referência para facilitar busca;
- resumo do que foi feito/procedimentos realizados.

O sistema deve permitir extrair/gerar uma ficha compacta imprimível para anexação física ao equipamento.

A modelagem recomendada — separar `Procedimento`, `Atendimento/Execução` e `Equipamento` — está documentada como **PROPOSTA** em `categorizacao-atendimentos-equipamentos.md` e ainda exige aprovação do PO.

## Busca operacional

Requisito confirmado: facilitar a localização por informações disponíveis do serviço/equipamento, como cliente, OS/referência, nome e identificadores úteis.

Quais campos serão chaves, filtros ou índices depende da modelagem aprovada.

## Multiusuário

- Clients nunca acessam SQLite diretamente;
- escritas são coordenadas pelo Host;
- edições antigas não sobrescrevem alterações recentes;
- mudanças relevantes chegam por eventos/reconsulta;
- fila de escrita não substitui revisão otimista.

Novos registros operacionais aprovados no futuro seguirão essas mesmas regras.

## Usuários e permissões

Conta possui identificador estável, login, nome de exibição, cargo, hash de senha, avatar, perfil e permissões.

Usuário pode editar nome, cargo, avatar e senha dentro das regras. Autorização é sempre Host-side.

Permissões para categorias e registros de serviço/equipamento ainda serão fechadas antes da implementação correspondente.

## Exportação e backup

Documentação exige:

- PDF;
- DOCX;
- impressão;
- identidade da empresa;
- backup/restauração simples.

Exportação usa documento próprio, não screenshot.

Novo requisito: ficha compacta imprimível de serviço/equipamento. Formato físico, PDF e tecnologia serão fechados no Bloco 10.

## Requisitos não funcionais

### Pocket

- implantação central por pasta pronta;
- nenhuma toolchain de desenvolvimento no servidor;
- nenhum processo StepFlow após encerramento do ciclo central;
- Client sem instalador tradicional obrigatório;
- uso normal sem dependência da Internet.

### Compatibilidade

Baseline inicial: Windows 10/11 x64 com WebView2. Ambiente corporativo real ainda será validado.

### Manutenibilidade

- frontend modular HTML/CSS/JavaScript + ES Modules;
- baixo acoplamento;
- organização por responsabilidade/domínio;
- evitar monólitos e superengenharia.

### Segurança proporcional

- Argon2id;
- sessão opaca;
- autorização Host-side;
- auditoria relevante;
- nenhum segredo/dado real no Git.

## Fora do escopo inicial

- acesso público pela Internet;
- SaaS/multiempresa;
- CRM completo/faturamento;
- estoque de peças;
- RMM/inventário automatizado;
- help desk completo com SLA;
- MFA complexo/recuperação por email;
- edição colaborativa caractere a caractere;
- chat corporativo;
- workflow burocrático complexo;
- infraestrutura distribuída de grande porte.

## Pendências atuais

- aprovar modelagem da categorização;
- aprovar ou ajustar separação Procedimento × Atendimento/Execução × Equipamento;
- fechar identidade/busca dos registros;
- fechar lifecycle/checklist/permissões no Bloco 9;
- fechar ficha compacta no Bloco 10.

## Critério de sucesso

Um técnico deve conseguir localizar o procedimento adequado, executar o trabalho com baixo atrito e, quando o cenário exigir registro do serviço/equipamento, produzir um resumo útil e imprimível sem depender de controles paralelos dispersos.
