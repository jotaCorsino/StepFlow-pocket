# Visão Geral do Produto — StepFlow Pocket

**Status:** CONSOLIDADO

## Propósito

O StepFlow é uma aplicação interna para centralizar documentação de processos técnicos e transformar procedimentos estáticos em guias operacionais fáceis de consultar e executar. O objetivo é reduzir atrito para o técnico, não criar um portal burocrático de gestão documental.

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

## Modelo enxuto de processo

Campos principais:

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
- Histórico de alterações.

Não adicionar campos burocráticos sem valor operacional aprovado.

## Etapas como páginas de manual

Cada etapa deve funcionar como uma página navegável e pode conter título, introdução, passos/subpassos, checklist documental, observações, alertas, comandos/blocos copiáveis, navegação anterior/próxima e indicação de progresso.

O controle de cópia deve ser discreto, somente por ícone, com feedback curto.

## Multiusuário

O produto deve aceitar vários usuários simultâneos em computadores diferentes:

- Clients nunca acessam SQLite diretamente;
- escritas são coordenadas pelo Host;
- edições antigas não sobrescrevem silenciosamente alterações recentes;
- mudanças relevantes chegam aos Clients por eventos/reconsulta;
- fila de escrita não substitui revisão otimista.

## Usuários e permissões

Conta possui identificador estável, login, nome de exibição, cargo, hash de senha, avatar, perfil e permissões.

O usuário pode editar nome de exibição, cargo, avatar e senha dentro das regras. Autorização é sempre verificada no Host.

## Exportação e backup

Requisitos obrigatórios:

- PDF;
- DOCX;
- impressão;
- identidade da empresa no documento exportado;
- backup e restauração simples.

A exportação usa modelo próprio de documento, não captura da tela.

## Requisitos não funcionais

### Pocket

- implantação da máquina central por pasta pronta;
- nenhum runtime/toolchain de desenvolvimento no servidor;
- nenhum serviço/processo StepFlow persistente quando o produto está fechado;
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
- MFA complexo e recuperação por email;
- edição colaborativa caractere a caractere;
- chat corporativo;
- workflow burocrático de aprovação em múltiplas instâncias;
- infraestrutura distribuída de grande porte.

## Pendência de produto ainda aberta

O checklist documental é obrigatório, mas o estado das marcações durante uma execução ainda será decidido no Bloco 9 da Fase 1.

## Critério de sucesso

Um técnico deve conseguir localizar e executar um processo no StepFlow com menos atrito do que usando documentos Word/PDF dispersos em pastas de rede.
