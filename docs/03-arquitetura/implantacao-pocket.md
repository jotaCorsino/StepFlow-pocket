# Implantação Pocket — Máquina Central e Estações

**Status:** REQUISITO ARQUITETURAL CONSOLIDADO  
**Atualização:** 2026-09-01

## Princípio central

A máquina central já existe e pode hospedar outros serviços importantes. O StepFlow deve adaptar-se a ela com o menor impacto possível.

Além disso, uma estação autorizada deve conseguir usar o StepFlow a partir do ponto de entrada publicado na rede **sem instalação individual do aplicativo ou preparação manual de dependências**.

Implantação desejada:

```text
pacote StepFlow pronto
→ copiar/arrastar pasta para o servidor Windows
→ configuração central mínima
→ iniciar Controller/Host sob demanda

estação autorizada
→ acessar compartilhamento
→ executar StepFlow.exe na raiz
→ Launcher prepara/valida o Client local
→ Client abre
→ Launcher encerra
```

`StepFlow.exe` é o Launcher empacotado com nome/ícone amigáveis e é o único ponto de entrada normal apresentado ao usuário.

## Contrato Pocket

Pocket significa, simultaneamente:

### Na máquina central

- pasta pronta/substituível;
- sem instalador tradicional obrigatório;
- sem toolchain de desenvolvimento;
- sem serviço StepFlow persistente;
- ciclo central iniciado/encerrado sob demanda;
- artefatos centrais encapsulados na estrutura técnica da publicação.

### Nas estações

- sem MSI/MSIX/NSIS obrigatório;
- sem Rust/Node/npm/Cargo/Visual Studio;
- sem configuração manual de runtime/dependência;
- sem privilégio administrativo no uso normal;
- sem Internet obrigatória durante o uso normal;
- usuário entra no compartilhamento e executa `StepFlow.exe` na raiz;
- Launcher prepara/atualiza o Client local automaticamente;
- Client operacional roda localmente, não permanentemente do SMB;
- usuário não precisa navegar por executáveis/pastas técnicas para descobrir como iniciar.

Qualquer solução que exija instalação, elevação ou preparação manual por estação não atende ao contrato Pocket e exige redesign ou decisão explícita do PO.

## Ciclo de vida obrigatório

```text
StepFlow fechado
→ nenhum processo StepFlow ativo

Controller iniciado sob demanda
→ Host inicia
→ múltiplos Clients utilizam o sistema
→ encerramento operacional
→ Host e Controller terminam
→ nenhum processo StepFlow residual
```

Regras:

- sem Windows Service StepFlow persistente;
- sem auto-start, Task Scheduler, watchdog, tray agent ou daemon StepFlow;
- sem Node.js, npm, Rust, Cargo, Visual Studio/MSVC ou SQLite Server exigidos em runtime;
- sem alteração global de PATH como requisito;
- sem reboot normal para implantação/atualização;
- consumo de CPU/memória do StepFlow tende a zero quando fechado.

Qualquer exceção futura exige mudança explícita do requisito pelo PO.

## Estrutura controlada da publicação

```text
StepFlow\
├── StepFlow.exe
└── _internal\
    ├── client\
    │   ├── manifest.json
    │   ├── deployment.json
    │   └── releases\
    └── server\
        ├── app\
        ├── config\
        ├── data\
        ├── logs\
        └── backups\
```

Na máquina central, o subdiretório `_internal/server/` é a raiz lógica dos artefatos e estado central:

- `app/` contém artefatos substituíveis;
- `config/`, `data/`, `logs/` e `backups/` são preservados entre atualizações;
- paths reais ainda dependem do ambiente corporativo;
- preferir caminhos relativos à raiz da implantação quando apropriado.

`_internal/` é encapsulamento organizacional. Atributo Hidden/System pode ser aplicado pelo packaging apenas como acabamento visual, nunca como condição de segurança, integridade ou funcionamento.

`.lnk` não é requisito baseline. A entrada portátil é o próprio `StepFlow.exe` da raiz.

## Host Pocket

O Host é um papel/processo da arquitetura, não um serviço permanente.

A direção vigente é `_internal/server/app/StepFlowController.exe` iniciar `_internal/server/app/StepFlowHost.exe` como processo-filho na máquina central, aguardar readiness, impedir segunda instância sobre os mesmos dados e coordenar shutdown gracioso.

Detalhes em `host-pocket.md`.

## Limitação aceita da rede

Executar um arquivo armazenado em SMB a partir do PC do técnico executa o programa naquela estação; isso não cria automaticamente um processo na máquina central.

Como nenhum componente StepFlow fica residente, o Controller precisa ser iniciado na máquina central ou por mecanismo remoto corporativo já existente e aprovado. Não instalar silenciosamente um serviço para contornar essa limitação.

## Client e Launcher

Experiência do técnico:

```text
share interno
→ duplo clique em StepFlow.exe
→ Launcher localiza _internal/client/ relativamente à própria pasta publicada
→ valida manifest/deployment
→ valida/copia versão local
→ valida recursos locais necessários
→ inicia Client local
→ Launcher encerra
```

Diretório operacional previsto:

```text
%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\
```

A cópia local é automática e versionada. Ela não é tratada como instalação tradicional.

Detalhes em `launcher-distribuicao-client.md`.

## WebView2 dentro do Pocket

Tauri usa WebView2 no Windows.

- Evergreen compatível existente é preferível;
- disponibilidade deve ser detectada;
- não baixar/instalar runtime silenciosamente da Internet em produção;
- Fixed Version não pode ser executado por localização de rede/UNC;
- fallback autocontido deve ser copiado/preparado localmente;
- esse fallback só entra em produção após PoC provar funcionamento sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo.

Se uma estação que deva ser suportada não puder receber o fallback de modo automático e sem admin, isso é bloqueador técnico do fallback, não autorização para enfraquecer o requisito Pocket.

## Atualização e rollback

### Máquina central

```text
encerrar processos StepFlow
→ backup quando necessário
→ substituir/ativar artefatos de _internal/server/app/
→ preservar _internal/server/data|config|logs|backups
→ iniciar novamente
→ validar readiness
```

### Client

```text
publicar nova versão em _internal/client/releases/
→ Launcher detecta versão
→ copia para nova pasta local
→ valida SHA-256
→ ativa somente após cópia íntegra
→ preserva versão anterior válida para fallback controlado
```

Rollback de binário não pode apagar banco automaticamente. Compatibilidade de schema/migrations deve ser respeitada.

## Critérios de aceitação de tecnologias

Toda escolha deve responder:

1. exige instalação global na máquina central?
2. exige instalação/configuração manual por estação?
3. exige privilégio administrativo no uso normal?
4. pode interferir em outros serviços?
5. exige reboot ou alteração permanente de sistema?
6. pode ser distribuída dentro da pasta StepFlow ou preparada automaticamente no perfil local?
7. funciona sem Internet após implantação?
8. permite atualização/remoção simples?
9. deixa processo StepFlow ativo quando o produto está fechado?

Quanto maior a interferência, menor a aderência ao Pocket.

## Validações de ambiente real

- caminho/permissões SMB;
- política de execução do `StepFlow.exe`/Launcher vindo do share;
- Windows 10/11 reais;
- WebView2 Evergreen;
- PoC de fallback local WebView2 sem elevação/manualidade;
- antivírus/EDR;
- múltiplas estações;
- comportamento sem Internet;
- comportamento opcional de ocultação de `_internal/`, se usado pelo packaging.

Fora da LAN corporativa, registrar `NÃO APLICÁVEL NESTE AMBIENTE` em vez de inventar resultado.

## Regra final

**O servidor existe antes do StepFlow e continuará existindo independentemente dele. O usuário deve poder acessar a pasta publicada, identificar imediatamente `StepFlow.exe` e iniciar o produto sem instalar, procurar executável técnico ou preparar manualmente a estação. O StepFlow adapta-se ao ambiente; não remodela o ambiente para recebê-lo.**
