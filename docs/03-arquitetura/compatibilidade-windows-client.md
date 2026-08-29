# Compatibilidade Windows e StepFlow Client

**Status:** CONSOLIDADO PARA A FASE 1 / VALIDAÇÃO CORPORATIVA PENDENTE  
**Atualização:** 2026-08-29

## Decisão

O StepFlow Client utilizará **Tauri 2 + HTML/CSS/JavaScript modular**.

Baseline inicial:

- Windows 10 x64;
- Windows 11 x64;
- WebView2 como renderer/runtime do Client;
- Windows 7/8/8.1 fora da promessa oficial inicial;
- x86/ARM64 somente se inventário real demonstrar necessidade.

A versão mínima exata de Windows 10 será definida após inventário das estações corporativas.

## Critério Pocket superior

Compatibilidade não é apenas “o executável abre no Windows”. Para uma estação ser considerada suportada, o StepFlow deve preservar o contrato Pocket:

```text
acessar pasta publicada no servidor
→ executar StepFlowLauncher.exe
→ preparação local automática
→ Client abre
```

Sem instalador tradicional, sem preparação manual por estação, sem elevação administrativa no fluxo normal, sem toolchain e sem Internet obrigatória para uso.

Qualquer dependência que exija ação manual por computador é incompatibilidade com o modelo Pocket até que exista mecanismo automático/autocontido aprovado.

## Evidência já obtida

A PoC descartável da Fase 1 confirmou no computador de desenvolvimento:

- criação de Client Tauri 2 Vanilla JavaScript/npm;
- build release `--no-bundle` bem-sucedido;
- executável isolado de aproximadamente 8,5 MiB;
- execução fora da árvore de build;
- execução sem Node.js, npm, Rust ou Cargo no `PATH` de runtime;
- nenhuma necessidade de MSI/NSIS para o smoke test.

Conclusão: toolchain pesado fica no ambiente de desenvolvimento; o runtime do Client não exige transportar Node/Rust/Visual Studio para a estação.

## WebView2

Tauri usa WebView2 no Windows.

A primeira versão deve:

- detectar ausência/incompatibilidade de forma compreensível;
- não depender de Internet durante o uso normal;
- não instalar WebView2 repetidamente a cada abertura;
- preferir o Evergreen Runtime já presente e gerenciado pelo Windows quando compatível;
- possuir estratégia Pocket para uma estação suportada que não disponha de runtime utilizável;
- nunca exigir que o usuário faça instalação/configuração manual para conseguir abrir o StepFlow.

### Evergreen

O Evergreen Runtime já é amplamente distribuído com Windows modernos e é a opção preferível quando disponível, inclusive porque recebe servicing/atualizações de segurança pelo ecossistema Microsoft.

Entretanto, a arquitetura não deve assumir cegamente que toda instalação de Windows 10/11 ou edição corporativa possui uma cópia compatível. O Launcher/Client deve detectar disponibilidade real.

### Fixed Version como fallback possível

A Microsoft permite distribuir uma versão fixa do WebView2 junto da aplicação, mas há condições relevantes para o StepFlow:

- o Fixed Version deve ser mantido/atualizado pelo próprio distribuidor da aplicação;
- não pode ser executado de localização de rede ou caminho UNC;
- em Windows 10, versões modernas do Fixed Runtime em aplicativos Win32 unpackaged possuem requisitos adicionais de ACL/AppContainer.

Consequência arquitetural:

```text
share/UNC
→ Launcher
→ cópia local validada do Client
→ fallback Fixed local, se necessário e tecnicamente aprovado
→ Client local
```

Nunca:

```text
share/UNC
→ executar Fixed Runtime diretamente da rede
```

O fallback Fixed só será consolidado como mecanismo de produção após PoC descartável provar que pode ser preparado em `%LOCALAPPDATA%` **sem elevação e sem intervenção manual**, inclusive no Windows 10 alvo. Se isso não for possível em uma estação que deva ser suportada, o resultado é bloqueador técnico do fallback e deve voltar à arquitetura; não autoriza introduzir instalador obrigatório.

## Distribuição vigente

O Client não será executado permanentemente sobre SMB e não terá instalador tradicional obrigatório.

A arquitetura aprovada é:

```text
ponto de entrada interno
→ launcher transitório no compartilhamento
→ cópia validada/versionada em %LOCALAPPDATA%\StepFlow\Client
→ StepFlow.exe local
```

A cópia/preparação local automática é parte do modelo Pocket, não uma instalação convencional.

Detalhes em `launcher-distribuicao-client.md`.

## Impressão e WebView2 nativo

A Etapa 11 do Bloco 10 confirmou a viabilidade do fluxo Windows por WebView2 nativo + `ShowPrintUI(System)`, com pequeno adaptador isolado e família Tauri/Wry/WebView2 pinada/testada.

O lifecycle exato de PDF local durante impressão e o comportamento de drivers continuam dependentes de teste Windows/corporativo real.

## Ícone e metadados

Tauri suporta ícone Windows customizado. O arquivo-fonte do ícone será aprovado visualmente e mantido nos assets do projeto; não assumir automaticamente que o logo corporativo é o ícone do aplicativo.

## Alternativas

Electron permanece somente contingência. Não reabrir comparação sem evidência objetiva de que Tauri falha em requisito real do produto, especialmente o contrato Pocket.

## Validações ainda dependentes da empresa

- versões/edições reais de Windows;
- arquitetura x64 das estações relevantes;
- Evergreen WebView2 instalado e funcional;
- execução sem Internet;
- políticas de execução/antivírus/EDR;
- comportamento real do launcher a partir do compartilhamento corporativo;
- PoC de fallback WebView2 autocontido sem instalação/elevação nas estações suportadas;
- impressão/driver WebView2 real.

Esses itens são validação de ambiente, não motivo para reabrir a stack enquanto não houver incompatibilidade concreta. **Zero instalação e zero preparação manual por estação permanecem requisito de compatibilidade, não conveniência opcional.**