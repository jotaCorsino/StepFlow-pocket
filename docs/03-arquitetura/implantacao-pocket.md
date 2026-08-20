# Implantação Pocket — Restrições do Servidor Windows

**Status:** REQUISITO ARQUITETURAL CONSOLIDADO

## 1. Objetivo

Definir o princípio de implantação que justifica a variante **Pocket** do StepFlow: o produto deve ser introduzido no servidor Windows da empresa com o menor impacto operacional possível, preservando os demais serviços que já dependem dessa máquina.

## 2. Princípio central

A experiência ideal de implantação é **copy-deploy / xcopy-like**:

```text
pacote pronto do StepFlow
        ↓
copiar/arrastar uma pasta
        ↓
pasta fixa no servidor
        ↓
configuração mínima
        ↓
StepFlow pronto para iniciar sob demanda
```

O servidor corporativo não deve ser tratado como uma máquina dedicada ao StepFlow. Ele pode executar outros serviços importantes e deve continuar operando normalmente durante implantação, uso, atualização e manutenção do produto.

## 3. Ciclo de vida Pocket obrigatório

O requisito não é apenas evitar instaladores. O StepFlow também não deve permanecer consumindo recursos do servidor quando não estiver sendo utilizado.

Fluxo desejado:

```text
StepFlow fechado
        ↓
nenhum processo StepFlow ativo
        ↓
usuário inicia o StepFlow
        ↓
processos necessários iniciam sob demanda
        ↓
uso normal, inclusive multiusuário
        ↓
uso do StepFlow termina
        ↓
processos transitórios encerram de forma controlada
        ↓
nenhum processo StepFlow residual
```

Consequências obrigatórias:

1. não instalar tradicionalmente o StepFlow no servidor como requisito normal;
2. não registrar Windows Service persistente como solução padrão;
3. não configurar serviço auto-start;
4. não usar Task Scheduler, watchdog, tray agent ou daemon residente apenas para manter o StepFlow disponível;
5. não deixar Host, launcher ou agente StepFlow ativo quando o produto não estiver em uso;
6. quando fechado, o consumo de CPU/memória do StepFlow deve tender a zero;
7. todos os componentes de execução devem partir da pasta controlada do produto ou de artefatos temporários explicitamente justificados;
8. dados persistentes podem permanecer em disco, mas processos não devem permanecer residentes sem necessidade.

Uma exceção futura só poderá ser aceita se o PO alterar explicitamente este requisito.

## 4. Requisito de baixo impacto

A solução deve:

- evitar alterações amplas no Windows;
- não depender de mudanças manuais em registro, políticas, features do sistema ou PATH global;
- não exigir múltiplos runtimes/componentes externos instalados apenas para executar o StepFlow;
- não exigir reinicialização do Windows em implantação/atualização normal;
- não substituir ou atualizar componentes compartilhados do sistema de forma que possam afetar outros serviços;
- manter configurações do StepFlow isoladas da configuração global da máquina;
- permitir remoção/rollback com impacto mínimo e previsível.

## 5. Pasta fixa do produto

O cenário desejado é existir uma pasta operacional própria do StepFlow em local ainda a definir no servidor, conceitualmente:

```text
<CAMINHO-FIXO-DO-SERVIDOR>\StepFlow\
```

A localização real não está definida e não deve ser hardcoded antes da validação no ambiente corporativo.

Estrutura conceitual:

```text
StepFlow\
├── app\            # binários/arquivos executáveis publicados
├── config\         # configuração específica da implantação
├── data\           # dados persistentes
├── logs\           # logs operacionais
├── backups\        # backups
└── tools\           # utilitários próprios estritamente necessários, se houver
```

A estrutura final será definida após o fechamento dos mecanismos de Host, launcher e atualização.

## 6. Binários e dependências

A arquitetura deve favorecer artefatos de execução que carreguem consigo o máximo possível das dependências necessárias ao StepFlow.

Preferir:

- executáveis/binários self-contained ou equivalentes;
- bibliotecas privadas dentro da pasta do produto;
- configuração local à aplicação;
- recursos estáticos empacotados ou mantidos na própria pasta;
- nenhuma dependência em instalação global que possa colidir com outros serviços.

O ambiente de **desenvolvimento** pode possuir Node.js, Rust, compiladores e outras ferramentas. Isso **não significa** que essas ferramentas poderão ser exigidas no servidor de produção.

**Toolchain de desenvolvimento e runtime de produção são contextos distintos.**

## 7. Host do StepFlow

A existência lógica de um StepFlow Host continua necessária para coordenar SQLite, autenticação, concorrência, auditoria e eventos.

Porém, "Host" significa um papel/processo da arquitetura, **não um Windows Service permanente**.

O Host deve:

- executar junto aos dados centrais;
- ser iniciado somente quando necessário;
- suportar múltiplos Clients simultâneos enquanto estiver ativo;
- encerrar de forma segura quando o StepFlow não estiver mais em uso;
- não deixar processo residual depois do encerramento;
- não exigir runtime/toolchain instalado no servidor.

A Fase 1 ainda precisa resolver como o primeiro Client dispara o Host na máquina central e como o último uso permite encerrá-lo sem quebrar sessões concorrentes.

Importante: iniciar um `.exe` armazenado em SMB a partir de uma estação normalmente executa esse programa na estação, não no servidor. Portanto, o bootstrap remoto do Host precisa ser resolvido explicitamente.

## 8. Client e ponto de entrada

A experiência do técnico permanece simples:

```text
ponto de entrada interno
        ↓
duplo clique
        ↓
StepFlow
```

A implementação poderá manter artefatos do Client na pasta publicada do servidor/compartilhamento e usar launcher/cópia local, desde que:

- o técnico não precise instalar dependências;
- o técnico não precise executar comandos;
- a estação não precise de configuração manual complexa;
- atualização central continue simples;
- o servidor não precise sofrer alterações invasivas para disponibilizar a nova versão.

## 9. Atualização e rollback

A atualização deve seguir a mesma filosofia Pocket:

```text
nova versão pronta
        ↓
copiar/publicar nova pasta ou conjunto de artefatos
        ↓
validar
        ↓
ativar nova versão
```

A estratégia final deve evitar:

- instaladores complexos;
- alterações irreversíveis no sistema;
- sobrescrita insegura de banco/configuração;
- downtime prolongado;
- dependência de reinicialização do servidor.

Dados persistentes devem ser separados dos binários de aplicação de modo que uma atualização de versão não dependa de sobrescrever o banco ou arquivos de configuração específicos da implantação.

## 10. Critério de aceitação para tecnologias futuras

Toda tecnologia considerada para Client, Host, banco, exportação ou atualização deve responder:

1. O servidor precisa instalar algo globalmente para executar isso?
2. A instalação pode afetar outros serviços?
3. Exige reinicialização?
4. Exige alteração permanente em PATH, registro, políticas ou features do Windows?
5. Pode ser distribuído dentro da própria pasta do StepFlow?
6. Pode ser atualizado e removido sem deixar dependências frágeis no servidor?
7. Funciona offline depois de implantado?
8. Deixa algum processo StepFlow ativo quando o produto está fechado?

Quanto mais respostas invasivas houver, menor a aderência ao conceito Pocket.

## 11. Consequência para provas técnicas

As provas da Fase 1 não devem validar apenas se uma tecnologia "funciona".

Elas também devem validar se a tecnologia pode produzir artefatos compatíveis com o modelo de implantação Pocket, inclusive o ciclo de vida sob demanda e ausência de processos residuais.

## 12. Regra final

**O servidor existe antes do StepFlow e continuará existindo independentemente dele. O StepFlow deve se adaptar ao ambiente, e não exigir que o ambiente seja remodelado para recebê-lo.**

Esse é um dos significados centrais do nome **StepFlow Pocket**.
