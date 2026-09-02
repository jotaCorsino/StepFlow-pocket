# Visão geral — StepFlow

StepFlow é uma aplicação web estática e local-first para documentar procedimentos técnicos e registrar sua execução operacional com o mínimo de infraestrutura e consumo.

## Domínio

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

### Procedimento

Documento operacional versionado. Campos principais:

- Código
- Título
- Área / Departamento
- Responsável
- Status
- Versão
- Objetivo
- Observações
- Pré-requisitos
- Categorias
- Etapas do processo
- Histórico de alterações

Cada Etapa funciona visualmente como uma página de manual/livro e pode conter passos, observações, checklist de referência e blocos de instrução/comando com copiar por ícone.

### Atendimento / Execução

Registro de um trabalho real. Pode usar um ou mais Procedimentos e opcionalmente Equipamentos. Guarda checklist operacional, observações por Etapa, resumo do trabalho e revisão exata consultada.

### Equipamento

Cadastro opcional/reutilizável de ativo atendido. Pode conter patrimônio, serial, MAC e outros dados pertinentes sem transformar esses campos em identidade canônica.

## Modelo de uso

O baseline é **single-user**. Não existem contas, login, perfis de permissão, sessão ou sincronização entre dispositivos.

`Responsável` é dado de negócio e pode apontar para uma lista local de responsáveis configuráveis; isso não cria usuários do sistema.

## Persistência

Dados ficam no navegador via IndexedDB. O servidor, quando usado, hospeda somente os arquivos estáticos do aplicativo.

## Portabilidade de dados

Exportar dados cria um arquivo portátil versionado. Importar dados valida e carrega esse conjunto localmente. Esse mecanismo substitui qualquer conceito de Backup/Restore no produto.

## Offline

A aplicação hospedada pode ser instalada como PWA e continuar operando sem rede após o primeiro carregamento. Também pode existir uma variante `StepFlow.html` autocontida para uso portátil.

## Limites do produto

StepFlow não é CRM, ERP, financeiro, estoque, RMM, help desk/SLA completo, plataforma de identidade ou sistema de sincronização em nuvem.
