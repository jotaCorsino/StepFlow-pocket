# Fechamento do Bloco 3 — Launcher e Distribuição do Client

**Data:** 2026-08-20  
**Status:** BLOCO 3 ARQUITETURALMENTE CONCLUÍDO

## Decisão

O StepFlow utilizará um launcher portátil e transitório como ponto de entrada do técnico.

Fluxo consolidado:

```text
ponto de entrada interno
        ↓
duplo clique em StepFlowLauncher.exe
        ↓
ler manifesto publicado
        ↓
validar/atualizar cópia local do Client
        ↓
%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\
        ↓
iniciar StepFlow.exe local
        ↓
launcher encerra
```

## Regras consolidadas

- não exigir instalador tradicional na estação;
- launcher não permanece residente;
- Client operacional roda localmente, não continuamente sobre SMB;
- versões ficam lado a lado para evitar sobrescrita de executável em uso;
- nova versão só é ativada após cópia completa e validação SHA-256;
- versão anterior válida é preservada para fallback/rollback controlado;
- atualização concorrente na mesma estação deve possuir lock transitório;
- nenhum serviço, updater permanente, watchdog ou PATH global;
- launcher não acessa SQLite;
- launcher não tenta iniciar remotamente o Host central;
- IP/hostname/share reais permanecem indefinidos até o ambiente corporativo.

## Tecnologia

Direção: launcher x64 pequeno e self-contained em Rust, sem runtime global na estação.

O Client permanece Tauri 2 com frontend web modular.

## Dependência do Bloco 4

O manifesto/configuração de publicação precisará referenciar a configuração necessária para localizar e validar compatibilidade com o Host. O formato exato pertence ao Bloco 4 — Comunicação Client ↔ Host.

## Validação deferida

Os testes reais de SMB, políticas de execução de rede, permissões, antivírus/EDR, desempenho e múltiplas estações são `NÃO APLICÁVEIS NESTE AMBIENTE` e deverão ser executados futuramente na LAN corporativa com caminhos reais.

Essa validação futura não altera a direção arquitetural consolidada neste bloco, salvo evidência corporativa objetiva de incompatibilidade.

## Resultado

Bloco 3 encerrado. Próximo bloco autorizado: **Bloco 4 — Comunicação Client ↔ Host**.
