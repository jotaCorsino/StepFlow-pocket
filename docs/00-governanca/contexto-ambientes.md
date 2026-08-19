# Contexto dos Ambientes — StepFlow Pocket

**Status:** CONSOLIDADO

## Objetivo

Separar explicitamente o ambiente usado para desenvolvimento do ambiente real de implantação do StepFlow, evitando interpretar resultados locais como se fossem evidências da rede corporativa.

## Ambiente de desenvolvimento atual

O desenvolvimento está sendo realizado em um computador pessoal, fora da rede da empresa.

Características deste contexto:

- pasta local do projeto: `C:\dev\StepFlow`;
- acesso ao GitHub pela Internet;
- não existe conexão normal com a LAN corporativa durante o desenvolvimento atual;
- o endereço `192.168.5.7` pertence ao cenário interno da empresa e não deve ser tratado como recurso que necessariamente estará acessível a partir da máquina de desenvolvimento;
- falhas de acesso a caminhos SMB corporativos nesta máquina não constituem, por si só, defeito do StepFlow, problema de permissões da futura implantação ou bloqueio arquitetural.

## Ambiente-alvo de implantação

O StepFlow será futuramente implantado na rede interna da empresa.

O cenário operacional esperado inclui:

```text
\\192.168.5.7\Arquivos\StepFlow\
```

ou caminho equivalente que venha a ser consolidado no ambiente real.

Nesse ambiente deverão ser validados, quando houver acesso a uma máquina representativa da empresa:

- conectividade com o Host;
- acesso ao compartilhamento SMB;
- permissões de leitura/execução necessárias;
- comportamento do launcher/ponto de entrada;
- funcionamento sem Internet;
- compatibilidade com as versões reais de Windows das estações;
- WebView2 e demais pré-requisitos nas máquinas-alvo;
- comportamento multiusuário em LAN real.

## Regra de interpretação de testes

Todo teste deve ser classificado como pertencente a um destes contextos:

1. **desenvolvimento local** — valida toolchain, build, comportamento do código e provas que não dependem da infraestrutura corporativa;
2. **simulação local** — usa serviços/paths locais ou mocks para reproduzir contratos sem afirmar que a infraestrutura da empresa foi validada;
3. **ambiente-alvo corporativo** — valida rede, SMB, Host, permissões, distribuição e comportamento real entre estações.

Um resultado negativo em uma capacidade indisponível por definição no ambiente de desenvolvimento, como acesso ao compartilhamento interno da empresa, deve ser registrado como **não aplicável neste ambiente**, e não como bloqueio técnico do produto.

## Consequência para a Fase 1

A Fase 1 pode continuar fechando arquitetura, stack, toolchain e protótipos locais sem acesso à rede da empresa.

Os gates que dependem de infraestrutura corporativa devem permanecer identificados para validação posterior em máquina/rede representativa, sem impedir atividades que não dependam deles.

## Regra para tarefas Codex

Tarefas que incluam teste de rede corporativa, SMB, Host real ou caminhos internos devem declarar previamente se o Codex está executando em ambiente com acesso à LAN da empresa.

Se estiver fora da rede corporativa, esses testes devem ser omitidos, simulados quando apropriado, ou marcados como `NÃO APLICÁVEL NESTE AMBIENTE`, conforme a natureza da tarefa.
