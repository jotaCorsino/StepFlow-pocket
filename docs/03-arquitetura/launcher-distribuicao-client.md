# Launcher e Distribuição do StepFlow Client

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-20

## Objetivo

Preservar a experiência:

```text
ponto de entrada interno
→ duplo clique
→ Client local atualizado
→ login
```

sem instalador tradicional obrigatório e sem processo de atualização residente.

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

## Cópia local

O Client operacional fica por usuário:

```text
%LOCALAPPDATA%\StepFlow\Client\
└── versions\
    ├── <versao-atual>\
    └── <versao-anterior>\
```

Não exigir privilégio administrativo para criar/atualizar essa pasta.

## Fluxo do launcher

1. ler/validar manifesto e deployment;
2. verificar versão local íntegra;
3. se necessário, copiar nova versão para pasta separada/temporária;
4. validar SHA-256 antes de ativar;
5. preservar versão anterior válida;
6. iniciar `StepFlow.exe` da versão local escolhida;
7. encerrar o launcher.

O launcher será um executável Rust x64 pequeno/self-contained, sem runtime global na estação.

## Atualização e concorrência

- nunca sobrescrever executável Client em uso;
- versões lado a lado;
- ativação somente após cópia completa e hash válido;
- lock transitório para impedir duas atualizações simultâneas no mesmo perfil;
- lock desaparece ao encerrar o launcher;
- retenção deve evitar acúmulo indefinido de versões.

## Fallback

Se a publicação estiver indisponível e existir versão local previamente validada, ela pode ser iniciada somente se ainda for compatível com a configuração/Host. Sem versão local válida, informar indisponibilidade.

Nunca iniciar artefato parcialmente copiado ou com hash divergente.

## Falhas que devem ser distinguíveis

- publicação indisponível;
- manifesto/deployment inválido;
- arquitetura/versão incompatível;
- falha de cópia ou espaço/permissão local;
- hash divergente;
- atualização local já em andamento;
- Client local ausente/corrompido;
- Host/configuração incompatível.

## Relação com o Host

O launcher executado no PC do técnico **não inicia remotamente o Host na máquina central**. O Host segue o ciclo Pocket de `host-pocket.md` e precisa estar ativo para uso operacional.

## O que não será usado

- MSI/NSIS obrigatório para cada técnico;
- serviço/updater/watchdog residente;
- PATH/registro global;
- execução permanente do Client pelo SMB;
- acesso SQLite pelo launcher/Client;
- hardcode de IP/hostname/share de exemplo.

## Validação corporativa pendente

- caminho e permissões SMB reais;
- políticas para execução de launcher de rede;
- antivírus/EDR;
- desempenho de cópia;
- WebView2;
- múltiplas estações reais.

Essas verificações ficam para a LAN corporativa e não reabrem a arquitetura sem evidência objetiva de incompatibilidade.
