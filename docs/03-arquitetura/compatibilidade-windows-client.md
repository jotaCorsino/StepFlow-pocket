# Plataforma Windows e StepFlow Client — Investigação da Fase 1

**Status:** PESQUISA TÉCNICA INICIAL CONCLUÍDA / PROVA EM MÁQUINA-ALVO AINDA PENDENTE

**Data:** 2026-08-19

## Objetivo

Validar a direção tecnológica do StepFlow Client considerando os requisitos reais do produto:

- UI em HTML, CSS e JavaScript modular;
- aplicativo Windows;
- funcionamento sem dependência de Internet durante o uso;
- início extremamente simples para o técnico;
- ícone personalizado;
- manutenção pequena;
- possibilidade de distribuição central pela rede;
- compatibilidade prioritária com Windows 10 e Windows 11.

## Fontes primárias consultadas

- Tauri — Prerequisites: https://v2.tauri.app/start/prerequisites/
- Tauri — Windows Installer: https://v2.tauri.app/distribute/windows-installer/
- Tauri — App Icons: https://v2.tauri.app/develop/icons/
- Tauri — Configuration: https://v2.tauri.app/reference/config/
- Microsoft — WebView2: https://learn.microsoft.com/microsoft-edge/webview2/
- Microsoft — Edge/WebView2 supported operating systems: https://learn.microsoft.com/deployedge/microsoft-edge-supported-operating-systems
- Electron — Introduction: https://www.electronjs.org/docs/latest/
- Electron — Windows 7/8/8.1 deprecation: https://www.electronjs.org/blog/windows-7-to-8-1-deprecation-notice

## 1. Tauri permanece adequado ao StepFlow

A documentação atual do Tauri 2 confirma que:

- o frontend pode continuar baseado em tecnologias web;
- no Windows a renderização usa Microsoft Edge WebView2;
- aplicações Windows podem ser empacotadas como instalador `.msi` ou `-setup.exe`/NSIS;
- builds x64, x86 e ARM podem ser produzidos;
- o projeto fornece suporte direto a ícones customizados, incluindo geração de `icon.ico` para Windows;
- a aplicação pode usar o renderer nativo do sistema em vez de incorporar uma cópia completa do Chromium no Client.

Essa combinação continua alinhada à proposta do StepFlow: Client pequeno, UI web modular e camada nativa apenas onde necessária.

## 2. Comparação resumida com Electron

Electron também atende HTML/CSS/JavaScript, mas incorpora Chromium e Node.js no aplicativo.

Para o StepFlow isso não produz uma vantagem clara neste momento:

- as versões atuais do Electron também não oferecem suporte oficial moderno a Windows 7/8/8.1;
- a aplicação Pocket não exige o ambiente Node completo dentro de cada Client;
- o objetivo é manter o Client enxuto e a lógica central no StepFlow Host.

Portanto, **Electron permanece alternativa de contingência, não a escolha preferencial**.

## 3. Compatibilidade Windows — interpretação correta

Existe uma diferença importante entre a documentação do Tauri e o suporte corrente do renderer da Microsoft.

A documentação do Tauri ainda descreve mecanismos relacionados a Windows 7. Porém, a documentação atual do WebView2 da Microsoft lista como plataformas Windows Client suportadas as linhas Windows 10 e Windows 11, incluindo variantes LTSC específicas.

Consequência arquitetural:

**o StepFlow não deve declarar suporte oficial a Windows 7, Windows 8 ou Windows 8.1 com base apenas na possibilidade técnica de uma combinação legada.**

Usar runtime antigo e sem suporte para obter compatibilidade artificial aumentaria risco de manutenção e segurança sem benefício suficiente para o cenário atual.

## 4. Baseline recomendado

Para a primeira versão interna:

- **Windows 10 x64** — suportado como alvo principal, sujeito a teste nas versões reais utilizadas pela empresa;
- **Windows 11 x64** — suportado como alvo principal;
- Windows 10 LTSC — candidato suportável, sujeito a identificação da versão real e teste;
- Windows x86/32-bit — tecnicamente possível no Tauri, mas não deve ser assumido como necessário antes de inventário das estações;
- Windows 7/8/8.1 — fora da promessa oficial inicial;
- ARM64 — fora da prioridade inicial, embora tecnicamente possível.

A versão mínima exata de Windows 10 ainda deve ser fechada após conhecer as estações reais. Como política de projeto, deve-se preferir uma linha ainda utilizável pelo WebView2 suportado em vez de congelar runtime legado.

## 5. WebView2 e funcionamento offline

Tauri usa WebView2 no Windows.

A documentação do Tauri informa que o runtime já é distribuído nas versões modernas do Windows 10 e no Windows 11. A Microsoft também documenta distribuição Evergreen e Fixed Version.

O Tauri oferece modos de instalação do WebView2:

| Modo | Internet durante instalação | Impacto aproximado informado pelo Tauri | Uso no StepFlow |
|---|---:|---:|---|
| `downloadBootstrapper` | Sim | mínimo | inadequado como única estratégia para ambiente offline |
| `embedBootstrapper` | ainda pode depender de Internet | pequeno | não é a melhor garantia offline |
| `offlineInstaller` | Não | + ~127 MB | opção forte para pacote de instalação offline |
| `fixedVersion` | Não | + ~180 MB | possível quando for necessário congelar/transportar o runtime |
| `skip` | Não | 0 MB | somente se garantirmos que WebView2 já existe; não usar cegamente |

### Direção recomendada

Para o uso diário do técnico, não queremos executar um instalador de WebView2 a cada abertura.

O desenho preferido é:

1. verificar em prova real se as estações Windows 10/11 da empresa já possuem WebView2;
2. o launcher/Client deve detectar uma ausência de pré-requisito de forma controlada;
3. se a empresa precisar preparar uma estação sem WebView2 e sem Internet, disponibilizar pacote administrativo/offline apropriado;
4. não vincular o funcionamento normal do StepFlow à disponibilidade da Internet.

O conteúdo HTML/CSS/JS do StepFlow será local; a comunicação operacional será com o StepFlow Host na LAN.

## 6. Ícone customizado

**Validado pela documentação oficial do Tauri.**

O comando `tauri icon` recebe PNG ou SVG de origem e gera os formatos necessários, incluindo `.ico` para Windows.

Portanto, o requisito de ícone próprio do StepFlow não apresenta bloqueio técnico.

Direção para o projeto:

- manter um arquivo-fonte de alta qualidade e transparente no repositório de assets de design;
- gerar os tamanhos de distribuição durante o processo de build;
- não usar o logo corporativo automaticamente como ícone do aplicativo sem aprovação visual específica.

## 7. Instalador tradicional versus Client operacional

Tauri suporta MSI e NSIS, mas isso não obriga o StepFlow a exigir que cada técnico execute um instalador manualmente.

O requisito do produto continua sendo:

`compartilhamento de rede → duplo clique → StepFlow`

A forma exata de combinar isso com o Client Tauri será validada no bloco de launcher.

Possibilidades ainda abertas:

- launcher copia/atualiza binários do Client em `%LOCALAPPDATA%\StepFlow` e executa localmente;
- instalação inicial administrada uma única vez e launcher apenas inicia/atualiza;
- pacote portátil controlado, se a prova demonstrar que o binário e suas dependências funcionam de forma segura nesse modelo.

Não assumir ainda qual dessas alternativas será a final.

## 8. Por que não executar permanentemente o Client diretamente no SMB

Mesmo que um `.exe` possa ser iniciado a partir do compartilhamento, isso não será adotado como solução definitiva sem teste.

A direção de cópia local continua tecnicamente preferível porque pode reduzir:

- bloqueio do arquivo publicado;
- dependência de desempenho do SMB durante toda a sessão;
- dificuldade para substituir a versão em uso;
- problemas de atualização simultânea;
- interferência de políticas de segurança que tratam executáveis de rede de forma diferente.

O Bloco 3 da Fase 1 deve validar isso mecanicamente.

## 9. Build e arquitetura de CPU

Tauri suporta build Windows x64 e também possui caminho para 32-bit e ARM64.

Recomendação inicial:

**padronizar o primeiro protótipo em x64**, porque reduz combinações e corresponde ao cenário mais provável de Windows 10/11 corporativo.

Antes de transformar isso em requisito definitivo, deve ser verificado se existe alguma estação Windows 32-bit que realmente precise rodar o StepFlow.

## 10. Decisão técnica recomendada após a pesquisa

### StepFlow Client

**Recomendação:** adotar **Tauri 2 + HTML + CSS + JavaScript modular** como stack do Client.

Motivos:

- aderência direta à UI web planejada;
- separação natural entre frontend e capacidades nativas;
- executável Windows;
- suporte a ícone e empacotamento;
- uso de WebView2 do sistema;
- menor necessidade de incorporar runtime web completo no Client em comparação com Electron;
- ausência de requisito que justifique Node/Chromium completos em cada estação.

### Status da decisão

A pesquisa documental não encontrou bloqueio técnico para Tauri 2.

A decisão deve ser considerada **APROVADA EM NÍVEL ARQUITETURAL PARA PROSSEGUIR COM PROVAS DA FASE 1**, mas o gate operacional ainda exige teste real em Windows 10/11 e validação do launcher/SMB antes da fundação definitiva da Fase 2.

## 11. Pendências para fechar o bloco

- [ ] identificar a(s) versão(ões) real(is) do Windows usadas pelas estações da empresa;
- [ ] confirmar se são x64;
- [ ] confirmar presença real do WebView2 em pelo menos uma máquina representativa Windows 10 e, se disponível, Windows 11;
- [ ] executar futuramente uma prova mínima Tauri, explicitamente descartável, nas máquinas-alvo;
- [ ] medir comportamento sem Internet;
- [ ] testar inicialização a partir do fluxo de launcher definido no Bloco 3.

## 12. Resultado

**Tauri 2 permanece a escolha recomendada para o StepFlow Client.**

**Windows 10/11 tornam-se a faixa de suporte oficial recomendada para a primeira versão, com versão mínima exata e arquitetura de CPU ainda dependentes da realidade das estações.**

Não há justificativa técnica atual para comprometer o projeto com versões modernas de Electron ou runtimes legados apenas para perseguir suporte oficial a Windows anteriores ao Windows 10.
