# Launcher e Distribuição do StepFlow Client

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-29

## Objetivo

Preservar a experiência Pocket:

```text
pasta StepFlow publicada no servidor Windows
→ usuário acessa o compartilhamento
→ duplo clique em StepFlowLauncher.exe
→ preparação local automática, quando necessária
→ Client local atualizado
→ login
```

Sem instalador tradicional obrigatório, sem preparação manual da estação, sem privilégios administrativos no uso normal e sem processo de atualização residente.

## Contrato Pocket obrigatório

Para o StepFlow, **Pocket** significa que uma pasta pronta pode ser copiada/movida para o servidor Windows e usada pelas estações autorizadas sem instalação individual do aplicativo.

O usuário não deve precisar:

- executar MSI/MSIX/NSIS ou assistente de setup;
- instalar Rust, Node.js, npm, Cargo, Visual Studio ou outra toolchain;
- instalar Office, LibreOffice, Adobe Reader ou browser externo para o StepFlow funcionar;
- alterar `PATH`, registro, políticas globais ou ACLs manualmente;
- possuir privilégio administrativo para o fluxo normal;
- depender de Internet durante o uso normal;
- configurar manualmente runtime/dependência em cada estação.

Se uma solução técnica exigir instalação, elevação ou preparação manual por computador, ela **não atende ao contrato Pocket** e deve ser substituída ou tratada como bloqueador técnico.

## Publicação na rede

Estrutura conceitual:

```text
<PASTA-PUBLICADA>\client\
├── StepFlowLauncher.exe
├── manifest.json
├── deployment.json
└── releases\
    └── <versao>\
        └── artefatos do Client
```

O caminho real será definido no ambiente corporativo.

`manifest.json` contém versão, arquitetura, artefatos, tamanhos quando úteis e SHA-256. `deployment.json` contém configuração não sensível necessária para localizar o Host/contrato.

A pasta publicada é o ponto de entrada e distribuição. O Client operacional não deve depender de execução permanente diretamente do UNC/SMB.

## Cópia local

O Client operacional fica por usuário:

```text
%LOCALAPPDATA%\StepFlow\Client\
└── versions\
    ├── <versao-atual>\
    └── <versao-anterior>\
```

Não exigir privilégio administrativo para criar/atualizar essa pasta.

Essa preparação local é parte automática do comportamento Pocket; não é uma instalação tradicional do aplicativo.

## Fluxo do launcher

1. ler/validar manifesto e deployment;
2. verificar versão local íntegra;
3. se necessário, copiar nova versão para pasta separada/temporária;
4. validar SHA-256 antes de ativar;
5. preservar versão anterior válida;
6. verificar os pré-requisitos técnicos estritamente necessários ao Client;
7. preparar automaticamente recursos locais autocontidos aprovados, quando necessários e quando isso puder ocorrer sem elevação/intervenção manual;
8. iniciar `StepFlow.exe` da versão local escolhida;
9. encerrar o launcher.

O launcher será um executável Rust x64 pequeno/self-contained, sem runtime global próprio na estação.

## WebView2 dentro do contrato Pocket

Tauri usa WebView2 no Windows. Isso não autoriza transformar WebView2 em instalação manual obrigatória.

Direção técnica:

1. preferir o Evergreen Runtime já presente/gerenciado pelo Windows quando compatível;
2. detectar de forma objetiva ausência/incompatibilidade antes de abrir o Client;
3. não baixar nem instalar runtime silenciosamente pela Internet em produção;
4. manter uma estratégia offline/autocontida para estações suportadas sem runtime utilizável;
5. qualquer fallback autocontido deve ser preparado **localmente**, nunca executado do UNC/SMB.

A Microsoft documenta que o WebView2 Fixed Version não pode ser executado de localização de rede/UNC. Além disso, Fixed Version moderno em aplicativo Win32 unpackaged no Windows 10 possui requisitos adicionais de ACL/AppContainer.

Portanto, o uso de Fixed Version como fallback Pocket permanece condicionado a PoC descartável que prove, no Windows 10/11 alvo:

- cópia automática para `%LOCALAPPDATA%` junto da versão do Client;
- seleção automática do runtime local pelo Client;
- preparação de ACL necessária sem elevação e sem ação manual do usuário;
- funcionamento sem Internet;
- atualização/substituição do runtime junto da versão publicada;
- ausência de execução do runtime a partir do compartilhamento.

Se essa PoC demonstrar que uma estação suportada exige elevação, instalador ou configuração manual para o fallback, o resultado é **BLOQUEADOR da estratégia de fallback**, e não permissão para enfraquecer o contrato Pocket.

## Atualização e concorrência

- nunca sobrescrever executável Client em uso;
- versões lado a lado;
- ativação somente após cópia completa e hash válido;
- lock transitório para impedir duas atualizações simultâneas no mesmo perfil;
- lock desaparece ao encerrar o launcher;
- retenção deve evitar acúmulo indefinido de versões;
- runtime autocontido, quando utilizado, acompanha o mesmo modelo de versionamento/validação e não vira instalação global.

## Fallback

Se a publicação estiver indisponível e existir versão local previamente validada, ela pode ser iniciada somente se ainda for compatível com a configuração/Host e possuir os recursos locais necessários.

Sem versão local válida, informar indisponibilidade. Nunca iniciar artefato parcialmente copiado ou com hash divergente.

## Falhas que devem ser distinguíveis

- publicação indisponível;
- manifesto/deployment inválido;
- arquitetura/versão incompatível;
- falha de cópia ou espaço/permissão local;
- hash divergente;
- atualização local já em andamento;
- Client local ausente/corrompido;
- runtime WebView2 ausente/incompatível sem fallback Pocket validado;
- preparação automática de fallback bloqueada por política/permissão;
- Host/configuração incompatível.

## Relação com o Host

O launcher executado no PC do técnico **não inicia remotamente o Host na máquina central**. O Host segue o ciclo Pocket de `host-pocket.md` e precisa estar ativo para uso operacional.

Mover/copiar a pasta publicada para o servidor não substitui esse princípio: o Controller/Host é iniciado na máquina central quando o ciclo StepFlow será utilizado; as estações apenas iniciam seus Clients locais pelo ponto de entrada compartilhado.

## O que não será usado

- MSI/MSIX/NSIS obrigatório para cada técnico;
- serviço/updater/watchdog residente;
- PATH/registro global;
- preparação manual de dependências em cada estação;
- elevação administrativa como requisito normal;
- execução permanente do Client pelo SMB;
- execução de WebView2 Fixed Runtime pelo UNC/SMB;
- acesso SQLite pelo launcher/Client;
- hardcode de IP/hostname/share de exemplo.

## Validação corporativa pendente

- caminho e permissões SMB reais;
- políticas para execução do launcher de rede;
- antivírus/EDR;
- desempenho de cópia;
- WebView2 Evergreen presente/compatível nas estações;
- PoC do fallback autocontido sem instalação/elevação em Windows 10/11 suportados;
- múltiplas estações reais.

Essas verificações ficam para a LAN corporativa e não reabrem a arquitetura sem evidência objetiva de incompatibilidade. O contrato Pocket de **zero instalação e zero preparação manual por estação** permanece o gate superior da distribuição.