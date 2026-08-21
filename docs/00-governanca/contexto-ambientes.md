# Contexto dos Ambientes — StepFlow Pocket

**Status:** CONSOLIDADO  
**Atualização:** 2026-08-21

## Objetivo

Separar explicitamente o ambiente de desenvolvimento, o sandbox/execução do Codex e o ambiente real de implantação, evitando transformar limitações locais em requisito do produto.

## Ambiente de desenvolvimento atual

O desenvolvimento ocorre em computador pessoal, fora da LAN da empresa.

- checkout local previsto: `C:\dev\StepFlow`;
- acesso ao GitHub pela Internet na sessão Windows normal do PO;
- sem acesso normal à LAN corporativa;
- IPs, hostnames, compartilhamentos e paths corporativos ainda não confirmados são apenas conceituais;
- falha de acesso a recurso corporativo indisponível neste ambiente não constitui defeito do StepFlow nem bloqueio arquitetural.

## Sessão Windows normal do PO versus ambiente Codex

A sessão Windows normal do PO é a referência para operações locais que dependam de capacidades reais da máquina, como:

- acesso confiável à Internet/credenciais;
- instalação ou atualização global de toolchain quando autorizada;
- comandos elevados;
- preparação do ambiente de desenvolvimento.

O sandbox/ambiente de execução do Codex pode ter restrições próprias de identidade, rede, credenciais, diretórios e permissões. Essas restrições **não viram requisito do StepFlow**.

O Codex não deve tentar reparar seu sandbox alterando ACL, Schannel, registro, PATH global, políticas de segurança ou reinstalando ferramentas já funcionais na sessão normal do PO. Se a tarefa realmente depender de uma capacidade indisponível no sandbox, deve reportar a limitação e indicar a validação/ação na sessão normal do PO.

Evitar cadeias de microdiagnósticos repetitivos quando a evidência já mostrar que a limitação é do ambiente de execução e não do projeto.

## Ambiente-alvo

O StepFlow será implantado futuramente na rede interna da empresa.

O requisito consolidado é a experiência operacional:

```text
ponto de entrada interno
        ↓
duplo clique
        ↓
login
        ↓
uso do aplicativo
```

Enquanto a infraestrutura real não estiver confirmada, usar somente notação conceitual:

```text
\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\
```

Nenhum IP, hostname, nome de compartilhamento ou path discutido anteriormente deve ser tratado como configuração oficial sem confirmação no ambiente corporativo.

## Validações reservadas ao ambiente corporativo

Quando houver acesso a máquinas/rede representativas, validar:

- hostname/endereço efetivo do Host;
- caminho real de publicação/compartilhamento;
- conectividade Client↔Host;
- SMB e permissões de leitura/execução;
- launcher/ponto de entrada;
- funcionamento sem Internet;
- versões/arquitetura reais de Windows;
- WebView2;
- antivírus/EDR/firewall e políticas locais;
- transporte HTTP/HTTPS aplicável;
- comportamento multiusuário na LAN real;
- mecanismo corporativo disponível para iniciar o Controller central quando necessário.

## Classificação de testes

Todo teste pertence a um destes contextos:

1. **desenvolvimento local / sessão normal do PO** — toolchain, build e capacidades reais da máquina de desenvolvimento;
2. **sandbox Codex** — execução automatizada limitada às capacidades disponíveis, sem reconfigurar o Windows para acomodar o agente;
3. **simulação local** — reproduz contratos sem afirmar validação da empresa;
4. **ambiente corporativo** — valida rede, máquinas, políticas e implantação reais.

Uma verificação impossível por definição no ambiente disponível deve ser marcada como `NÃO APLICÁVEL NESTE AMBIENTE`, não como falha do produto.

## Consequência para a Fase 1

Os Blocos 0–7 foram fechados sem depender da LAN corporativa. A Fase 1 pode continuar com UI/UX, checklist, exportação, backup e estrutura da Fase 2 usando as decisões vigentes.

Validações dependentes da empresa permanecem registradas para o momento correto e não justificam repetir provas dos blocos já encerrados sem nova evidência objetiva.

## Regra para tarefas Codex

Tarefas que dependam de rede corporativa, SMB, Host real, paths internos, credenciais, elevação ou configuração global devem declarar o ambiente disponível.

Fora da LAN corporativa, verificações corporativas devem ser omitidas, simuladas quando apropriado ou marcadas como `NÃO APLICÁVEIS NESTE AMBIENTE`.

Se a infraestrutura real ainda não estiver consolidada, usar placeholders conceituais e nunca inferir configuração a partir de exemplos históricos.
