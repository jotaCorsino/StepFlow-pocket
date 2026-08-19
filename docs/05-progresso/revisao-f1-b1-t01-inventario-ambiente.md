# Revisão — F1-B1-T01 Inventário do Ambiente Windows

**Data:** 2026-08-19  
**Status:** CONCLUÍDA

## Objetivo

Revisar o relatório produzido pelo Codex na tarefa `F1-B1-T01 — Inventário do Ambiente Windows e Pré-requisitos` e fechar as pendências locais necessárias antes da preparação da prova Tauri.

## Contexto correto do ambiente

A estação atualmente usada para desenvolver o StepFlow é um computador pessoal, fora da rede da empresa.

Consequentemente, qualquer tentativa de acesso a um caminho SMB interno da empresa nesta estação não representa uma validação do ambiente corporativo.

Além disso, o caminho anteriormente utilizado como `\\192.168.5.7\Arquivos\StepFlow\` foi apenas um **exemplo ilustrativo**. O endereço IP, nome do compartilhamento e subpasta reais ainda não foram definidos/confirmados e não devem ser tratados como configuração oficial.

O contexto de ambientes está documentado em `docs/00-governanca/contexto-ambientes.md`.

## Evidências do inventário

- Git `2.55.0.windows.4`;
- Node.js `v24.14.0`;
- npm `11.9.0`;
- Rust/rustup/cargo não encontrados;
- Visual Studio Community 2026 `18.8.12021.73` detectado;
- MSVC `14.51.36231` detectado;
- WebView2 detectado localmente nas versões `151.0.4129.86` e `151.0.4129.93`;
- arquitetura AMD64/x64;
- CPU AMD Ryzen 5 5600X;
- tentativa histórica de acesso ao caminho SMB ilustrativo retornou `Acesso negado`;
- nenhuma escrita foi realizada;
- consulta WMI/CIM apresentou restrições de permissão, mas as informações relevantes foram confirmadas por outras fontes locais.

## Fechamento da identidade do Windows

A tarefa `F1-B1-T02 — Confirmar identidade e versão real do Windows` levantou:

- `WindowsProductName`: `Windows 10 Pro`;
- `EditionID`: `Professional`;
- `DisplayVersion`: `25H2`;
- `CurrentBuild`: `26200`;
- `UBR`: `9168`;
- arquitetura AMD64/x64.

A validação externa em fontes oficiais da Microsoft confirmou que:

- Windows 11 versão 25H2 corresponde à linha de OS build `26200`;
- a atualização de 11 de agosto de 2026, KB5121003, produz OS build `26200.9168` para Windows 11 versão 25H2.

Fontes:

- Microsoft Learn — Windows 11 release information: https://learn.microsoft.com/windows/release-health/windows11-release-information
- Microsoft Support — KB5121003: https://support.microsoft.com/en-us/servicing/os/windows-11/2026/08/kb5121003-windows-11-24h2-25h2-security-update

Portanto, para fins do projeto, esta estação de desenvolvimento é registrada como:

**Windows 11 Pro, versão 25H2, OS build 26200.9168, x64.**

O valor literal `Windows 10 Pro` retornado por `ProductName`/`WindowsProductName` deve ser preservado como evidência de uma fonte local divergente, mas não deve prevalecer sobre a correspondência oficial entre versão/build e a linha do produto publicada pela Microsoft.

O valor `WindowsVersion = 2009` também é tratado apenas como identificador legado retornado pela API local, não como versão comercial atual.

## Avaliação dos pré-requisitos para futura prova Tauri

O ambiente de desenvolvimento já possui:

- Git;
- Node.js/npm;
- WebView2;
- toolchain Microsoft C++/MSVC;
- Windows x64 compatível com a direção arquitetural atual.

O requisito ausente identificado é:

- Rust/rustup/cargo.

A ausência de Rust não é erro do ambiente; ela apenas define o próximo passo de preparação controlada antes de uma prova mínima Tauri.

## Compartilhamento SMB

O resultado histórico `Acesso negado` **não é bloqueio do projeto e não gera tarefa de diagnóstico SMB nesta estação**.

Motivos:

- a estação de desenvolvimento está fora da LAN da empresa;
- o caminho usado foi apenas ilustrativo;
- o endereço real do compartilhamento corporativo ainda será definido/confirmado;
- conectividade, permissões e comportamento do SMB só poderão ser validados em ambiente realmente conectado à infraestrutura da empresa.

Classificação do teste anterior:

**NÃO APLICÁVEL AO AMBIENTE DE DESENVOLVIMENTO ATUAL.**

## Resultado da revisão

A tarefa `F1-B1-T01` está encerrada.

A tarefa `F1-B1-T02` resolveu a única pendência técnica local remanescente do inventário.

Não existe bloqueio local conhecido para preparar o toolchain Rust e, posteriormente, executar uma prova Tauri descartável.

## Próximo passo

Preparar, em tarefa separada e delimitada, o toolchain Rust necessário ao desenvolvimento Tauri, sem criar ainda scaffold ou código do StepFlow.