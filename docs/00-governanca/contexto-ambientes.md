# Contexto dos Ambientes — StepFlow Pocket

**Status:** CONSOLIDADO  
**Atualização:** 2026-09-01

## Objetivo

Separar desenvolvimento, sandbox de execução e ambiente corporativo real, evitando transformar limitações locais em requisito do produto ou inventar validação que ainda depende da empresa.

## Desenvolvimento atual

O desenvolvimento ocorre em computador pessoal, fora da LAN corporativa.

- checkout local previsto: `C:\dev\StepFlow`;
- acesso ao GitHub pela Internet na sessão Windows normal do PO;
- sem acesso normal à LAN corporativa;
- IPs, hostnames, compartilhamentos e paths corporativos ainda não confirmados são conceituais;
- falha de acesso a recurso corporativo indisponível neste ambiente não constitui defeito do StepFlow.

## Sessão Windows normal do PO versus sandbox Codex

A sessão Windows normal do PO é referência para operações locais que dependam de capacidades reais da máquina, como Internet/credenciais, toolchain autorizada, elevação ou configuração do ambiente de desenvolvimento.

O sandbox Codex pode possuir restrições próprias de identidade, rede, credenciais, diretórios e permissões. Essas restrições **não viram requisito do produto**.

O Codex não deve reparar o próprio sandbox alterando ACL, Schannel, registro, PATH global, políticas de segurança ou reinstalando ferramentas válidas na sessão normal do PO.

## Ambiente-alvo

O StepFlow será implantado na rede interna da empresa.

Experiência operacional aprovada:

```text
pasta StepFlow publicada no servidor Windows
→ usuário acessa o compartilhamento
→ executa StepFlow.exe na raiz
→ Launcher prepara/valida Client local
→ login
→ uso
```

`StepFlow.exe` é o Launcher com nome/ícone amigáveis. A árvore técnica publicada fica encapsulada sob `_internal/`.

Enquanto infraestrutura real não estiver confirmada, usar somente notação conceitual:

```text
\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\
```

Nenhum exemplo histórico de IP, hostname, share ou path é configuração oficial.

## Gates reservados ao ambiente corporativo

Quando houver máquinas/rede representativas, validar conforme necessidade da fase:

- hostname/endereço efetivo do Host;
- caminho/permissões do compartilhamento;
- conectividade Client↔Host;
- execução do `StepFlow.exe` pelo share;
- funcionamento sem Internet;
- versões/edições/arquitetura reais de Windows;
- WebView2 Evergreen e fallback Pocket quando necessário;
- antivírus/EDR/firewall/políticas;
- Word e impressoras para contratos documentais;
- SMB real, incluindo falhas/rename/replace/latência;
- transporte HTTP/HTTPS aplicável;
- multiusuário na LAN;
- mecanismo corporativo disponível para iniciar o Controller central quando necessário.

Esses gates não bloqueiam o encerramento documental da Fase 1. Eles são executados no momento da fase técnica correspondente e podem bloquear a saída da etapa/produção se falharem em ambiente que deva ser suportado.

## Classificação de testes

1. **desenvolvimento local / sessão normal do PO** — toolchain, build e capacidades reais da máquina de desenvolvimento;
2. **sandbox Codex** — execução automatizada limitada às capacidades disponíveis;
3. **simulação local** — reproduz contratos sem afirmar validação da empresa;
4. **ambiente corporativo** — valida rede, máquinas, políticas e implantação reais.

Uma verificação impossível no ambiente disponível deve ser marcada como `NÃO APLICÁVEL NESTE AMBIENTE`, não como PASS nem como falha do produto.

## Regra para tarefas

Tarefas que dependam de rede corporativa, SMB, Host real, paths internos, credenciais, elevação ou configuração global devem declarar o ambiente disponível.

Fora da LAN corporativa:

- omitir teste corporativo impossível;
- simular quando isso produzir evidência útil sem falsear o resultado;
- marcar explicitamente `NÃO APLICÁVEL NESTE AMBIENTE` quando necessário;
- nunca inferir configuração oficial a partir de placeholders.

## Consequência arquitetural

Gates de ambiente real permanecem registrados até o momento correto, mas não reabrem decisões arquiteturais consolidadas sem evidência objetiva de incompatibilidade.
