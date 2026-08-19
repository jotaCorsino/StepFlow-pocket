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
iniciar/configurar o mínimo necessário
        ↓
StepFlow disponível
```

O servidor corporativo não deve ser tratado como uma máquina dedicada ao StepFlow. Ele pode executar outros serviços importantes e deve continuar operando normalmente durante implantação, atualização e manutenção do produto.

## 3. Requisito de baixo impacto

A solução deve buscar, nesta ordem de preferência:

1. não exigir instalação tradicional no servidor;
2. não exigir alterações amplas no Windows;
3. não depender de mudanças manuais em registro, políticas, features do sistema ou PATH global;
4. não exigir múltiplos runtimes/componentes externos instalados apenas para executar o StepFlow;
5. não exigir reinicialização do Windows em uma implantação normal;
6. não substituir ou atualizar componentes compartilhados do sistema de forma que possam afetar outros serviços;
7. manter configurações do StepFlow isoladas da configuração global da máquina;
8. permitir remoção/rollback com impacto mínimo e previsível.

Quando alguma exceção for tecnicamente inevitável, ela deve ser explicitamente validada antes de ser aceita na arquitetura.

## 4. Pasta fixa do produto

O cenário desejado é existir uma pasta operacional própria do StepFlow em local ainda a definir no servidor, conceitualmente:

```text
<CAMINHO-FIXO-DO-SERVIDOR>\StepFlow\
```

A localização real não está definida e não deve ser hardcoded antes da validação no ambiente corporativo.

Estrutura conceitual possível:

```text
StepFlow\
├── app\            # binários/arquivos executáveis da versão publicada
├── config\         # configuração específica da implantação
├── data\           # dados persistentes, se aprovado para esse local
├── logs\           # logs operacionais
├── backups\        # ou referência para local de backup aprovado
└── tools\           # utilitários próprios estritamente necessários
```

Essa árvore é conceitual. A estrutura final será definida após a escolha tecnológica do Host e da estratégia de atualização.

## 5. Binários e dependências

A arquitetura deve favorecer artefatos de execução que carreguem consigo o máximo possível das dependências necessárias ao StepFlow.

Isso significa preferir, quando tecnicamente adequado:

- executáveis/binários self-contained ou equivalentes;
- bibliotecas privadas dentro da pasta do produto;
- configuração local à aplicação;
- recursos estáticos empacotados ou mantidos na própria pasta;
- nenhuma dependência em instalação global que possa colidir com outros serviços.

O ambiente de **desenvolvimento** pode possuir Node.js, Rust, compiladores e outras ferramentas. Isso **não significa** que essas ferramentas poderão ser exigidas no servidor de produção.

**Toolchain de desenvolvimento e runtime de produção são contextos distintos.**

## 6. Host do StepFlow

A existência de um StepFlow Host continua sendo requisito da arquitetura lógica para coordenar SQLite, autenticação, concorrência e eventos.

Porém, a tecnologia e o formato do Host devem respeitar o princípio Pocket.

A solução ideal deverá poder ser publicada junto da pasta do StepFlow sem exigir instalação de uma stack completa de desenvolvimento no servidor.

Questões que ainda precisam ser fechadas na Fase 1:

- se o Host será um executável self-contained;
- se deverá rodar como processo normal, serviço Windows ou outro mecanismo;
- como será iniciado automaticamente com alteração mínima do sistema;
- como será parado/atualizado de forma controlada;
- quais privilégios mínimos serão necessários;
- como manter banco, configuração e logs isolados;
- como realizar rollback sem impactar outros serviços.

Transformar o Host em serviço Windows **não está automaticamente aprovado**. Caso essa alternativa seja escolhida, a instalação/remoção do serviço deverá ser simples, previsível e justificada frente ao requisito de baixo impacto.

## 7. Client e ponto de entrada

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

## 8. Atualização e rollback

A atualização deve seguir a mesma filosofia Pocket.

Direção desejada:

```text
nova versão pronta
        ↓
copiar/publicar nova pasta ou conjunto de artefatos
        ↓
validar
        ↓
ativar nova versão
```

A estratégia final deve buscar evitar:

- instaladores complexos;
- alterações irreversíveis no sistema;
- sobrescrita insegura de banco/configuração;
- downtime prolongado;
- dependência de reinicialização do servidor.

Dados persistentes devem ser separados dos binários de aplicação de modo que uma atualização de versão não dependa de sobrescrever o banco ou arquivos de configuração específicos da implantação.

## 9. Critério de aceitação para tecnologias futuras

Toda tecnologia considerada para Client, Host, banco, exportação ou atualização deve responder:

1. O servidor precisa instalar algo globalmente para executar isso?
2. A instalação pode afetar outros serviços?
3. Exige reinicialização?
4. Exige alteração permanente em PATH, registro, políticas ou features do Windows?
5. Pode ser distribuído dentro da própria pasta do StepFlow?
6. Pode ser atualizado e removido sem deixar dependências frágeis no servidor?
7. Funciona offline depois de implantado?

Quanto mais respostas invasivas houver, menor a aderência ao conceito Pocket.

## 10. Consequência para provas técnicas

As provas da Fase 1 não devem validar apenas se uma tecnologia "funciona".

Elas também devem validar se a tecnologia pode produzir artefatos compatíveis com o modelo de implantação Pocket.

Exemplos:

- a prova do Client deve observar portabilidade/runtime necessário;
- a prova do Host deverá observar execução a partir de pasta própria;
- a estratégia de banco deve permanecer local ao Host sem instalação de servidor de banco externo;
- a estratégia de exportação deve evitar dependências globais desnecessárias;
- a atualização deve ser projetada pensando em substituição controlada de artefatos/pastas.

## 11. Regra final

**O servidor existe antes do StepFlow e continuará existindo independentemente dele. O StepFlow deve se adaptar ao ambiente, e não exigir que o ambiente seja remodelado para recebê-lo.**

Esse é um dos significados centrais do nome **StepFlow Pocket**.