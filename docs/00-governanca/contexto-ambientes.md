# Contexto dos Ambientes — StepFlow Pocket

**Status:** CONSOLIDADO  
**Atualização:** 2026-08-20

## Objetivo

Separar explicitamente o ambiente de desenvolvimento do ambiente real de implantação, evitando interpretar resultados locais como evidência da rede corporativa ou transformar exemplos de infraestrutura em configuração definitiva.

## Ambiente de desenvolvimento atual

O desenvolvimento ocorre em computador pessoal, fora da LAN da empresa.

- checkout local previsto: `C:\dev\StepFlow`;
- acesso ao GitHub pela Internet;
- sem acesso normal à LAN corporativa;
- IPs, hostnames, compartilhamentos e paths corporativos ainda não confirmados são apenas conceituais;
- falha de acesso a recurso corporativo indisponível neste ambiente não constitui defeito do StepFlow nem bloqueio arquitetural.

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

1. **desenvolvimento local** — toolchain, build e comportamento independente da infraestrutura corporativa;
2. **simulação local** — reproduz contratos sem afirmar validação da empresa;
3. **ambiente corporativo** — valida rede, máquinas, políticas e implantação reais.

Uma verificação impossível por definição no ambiente atual deve ser marcada como `NÃO APLICÁVEL NESTE AMBIENTE`, não como falha do produto.

## Consequência para a Fase 1

Os Blocos 0–7 foram fechados sem depender da LAN corporativa. A Fase 1 pode continuar com UI/UX, checklist, exportação, backup e estrutura da Fase 2 usando as decisões vigentes.

Validações dependentes da empresa permanecem registradas para o momento correto e não justificam repetir provas dos blocos já encerrados sem nova evidência objetiva.

## Regra para tarefas Codex

Tarefas que dependam de rede corporativa, SMB, Host real ou paths internos devem declarar o ambiente disponível.

Fora da LAN corporativa, essas verificações devem ser omitidas, simuladas quando apropriado ou marcadas como `NÃO APLICÁVEIS NESTE AMBIENTE`.

Se a infraestrutura real ainda não estiver consolidada, usar placeholders conceituais e nunca inferir configuração a partir de exemplos históricos.
