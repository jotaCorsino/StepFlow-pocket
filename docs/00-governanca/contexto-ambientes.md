# Contexto dos Ambientes — StepFlow Pocket

**Status:** CONSOLIDADO

## Objetivo

Separar explicitamente o ambiente usado para desenvolvimento do ambiente real de implantação do StepFlow, evitando interpretar resultados locais como se fossem evidências da rede corporativa e evitando transformar exemplos de infraestrutura em configurações definitivas.

## Ambiente de desenvolvimento atual

O desenvolvimento está sendo realizado em um computador pessoal, fora da rede da empresa.

Características deste contexto:

- pasta local do projeto: `C:\dev\StepFlow`;
- acesso ao GitHub pela Internet;
- não existe conexão normal com a LAN corporativa durante o desenvolvimento atual;
- endereços IP, nomes de servidor, compartilhamentos SMB e paths corporativos usados durante o planejamento são apenas ilustrativos enquanto não forem confirmados no ambiente real;
- falhas de acesso a caminhos SMB corporativos nesta máquina não constituem, por si só, defeito do StepFlow, problema de permissões da futura implantação ou bloqueio arquitetural.

## Ambiente-alvo de implantação

O StepFlow será futuramente implantado na rede interna da empresa.

O requisito consolidado é a **experiência operacional**, não um endereço específico:

```text
compartilhamento/ponto de entrada interno
        ↓
duplo clique no StepFlow
        ↓
login
        ↓
uso do aplicativo
```

Enquanto a infraestrutura real não estiver confirmada, usar somente notação conceitual:

```text
\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\
```

O endereço `\\192.168.5.7\Arquivos\StepFlow\` apareceu nas conversas e documentos iniciais apenas como **exemplo ilustrativo**. Ele não é um requisito, não está aprovado como caminho real e não deve ser embutido em código, configuração ou teste definitivo.

No ambiente corporativo real deverão ser validados, quando houver acesso a uma máquina representativa da empresa:

- endereço/hostname efetivo do Host;
- caminho real do compartilhamento, caso essa estratégia seja mantida;
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

Um resultado negativo em uma capacidade indisponível por definição no ambiente de desenvolvimento, como acesso ao compartilhamento interno da empresa, deve ser registrado como **NÃO APLICÁVEL NESTE AMBIENTE**, e não como bloqueio técnico do produto.

## Consequência para a Fase 1

A Fase 1 pode continuar fechando arquitetura, stack, toolchain e protótipos locais sem acesso à rede da empresa.

Os gates que dependem de infraestrutura corporativa devem permanecer identificados para validação posterior em máquina/rede representativa, sem impedir atividades que não dependam deles.

Nenhum IP, hostname ou caminho SMB deve ser congelado na arquitetura antes de ser confirmado no ambiente de implantação.

## Regra para tarefas Codex

Tarefas que incluam teste de rede corporativa, SMB, Host real ou caminhos internos devem declarar previamente se o Codex está executando em ambiente com acesso à LAN da empresa e se o endereço utilizado já está consolidado.

Se estiver fora da rede corporativa, esses testes devem ser omitidos, simulados quando apropriado, ou marcados como `NÃO APLICÁVEL NESTE AMBIENTE`.

Se o endereço ainda não estiver consolidado, usar placeholders conceituais e nunca inferir que um exemplo anterior é configuração oficial.