# Tela 10 — Impressão e Ficha

A geração documental usa HTML/CSS de impressão do próprio navegador.

## Procedimento

Versão para impressão em A4 multipágina, mantendo hierarquia, etapas, passos e identidade da empresa.

## Ficha do Atendimento

Resumo compacto para prestação de contas:

- empresa;
- atendimento;
- responsável;
- equipamento quando houver;
- procedimentos utilizados;
- resumo do trabalho;
- informações essenciais do serviço.

A Ficha deve caber em uma A4. Se o conteúdo exceder o contrato, a UI informa overflow em vez de truncar silenciosamente.

`window.print()` abre a impressão nativa; o usuário pode selecionar impressora ou `Salvar como PDF` no navegador.
