# Revisão — F1-B1-T04G MSVC

**Data:** 2026-08-20  
**Status:** CONCLUÍDA / CAUSA CONSOLIDADA

## Resumo

A F1-B1-T04G confirmou que o Visual Studio Community 2026 está instalado, completo, inicializável e em canal estável, mas a instância não satisfaz o requisito de componente `Microsoft.VisualStudio.Component.VC.Tools.x86.x64` consultado por `vswhere`.

Também não foram localizados, por essa instalação, `VsDevCmd.bat`, `cl.exe` ou `link.exe`. Portanto, a falha observada nas provas Rust não é causada por `-prerelease`, pelo Cargo ou pelo toolchain Rust.

## Evidências consolidadas

- sessão Codex: `EARTH\CodexSandboxOffline`, não elevada;
- checkout local: `main`, HEAD `97c5f4a`, somente diário autorizado modificado;
- Rust/Cargo já validados anteriormente: `rustc 1.97.1`, `cargo 1.97.1`, host `x86_64-pc-windows-msvc`;
- `cargo init --vcs none` e `cargo check --offline` já funcionaram no sandbox;
- `cargo build --offline` falhou anteriormente por ausência de `link.exe`;
- `vswhere` encontra Visual Studio Community 2026 `18.8.12021.73` em `C:\Program Files\Microsoft Visual Studio\18\Community`;
- a instalação não é pré-release;
- `vswhere -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64` não seleciona a instalação;
- `VsDevCmd.bat`, `cl.exe` e `link.exe` não foram localizados pela prova;
- nenhuma instalação, elevação ou alteração persistente foi executada.

## Interpretação

As fontes oficiais atuais do Tauri exigem, para desenvolvimento Windows, Microsoft C++ Build Tools com o workload **Desktop development with C++**, além de WebView2 e Rust.

A documentação atual do Visual Studio 2026 mantém o workload `Microsoft.VisualStudio.Workload.NativeDesktop` para Community/Professional e lista `Microsoft.VisualStudio.Component.VC.Tools.x86.x64` como componente MSVC x64/x86. A ausência desse componente na consulta de `vswhere` é compatível com uma instalação do Visual Studio que possui o IDE, mas não o conjunto C++ requerido para builds MSVC.

Consequentemente, não há justificativa para continuar abrindo probes de detecção no sandbox. A próxima ação correta é preparar uma única vez o ambiente de desenvolvimento na sessão Windows normal do PO.

## Decisão operacional

1. encerrar a sequência F1-B1-T04C/D/E/F/G de probes incrementais;
2. não alterar ACLs, Schannel, credenciais ou PATH permanente para acomodar `CodexSandboxOffline`;
3. não abandonar Tauri por causa desse bloqueio, pois ele ocorre no ambiente de build, não no runtime Pocket;
4. atribuir ao PO, em sessão Windows normal, a preparação de dependências globais de desenvolvimento que exigem instalação/admin/rede;
5. atribuir ao Codex trabalho sobre arquivos, análise, validações locais/offline e builds somente depois que o ambiente e caches necessários estiverem preparados;
6. retomar F1-B1-T04 somente após o gate único definido no plano de adaptação.

## Resultado

A causa operacional mais provável está consolidada: o ambiente Windows de desenvolvimento não possui, na instalação Visual Studio 2026 detectada, o workload/componentes C++ necessários ao linker MSVC usado pelo target Rust `x86_64-pc-windows-msvc`.

A F1-B1-T04 permanece pendente, mas deixa de depender de novos diagnósticos incrementais. O próximo passo é seguir o plano de adaptação de ambiente e execução da Fase 1.