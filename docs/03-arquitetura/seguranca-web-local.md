# Segurança — aplicação web local-first

## Escopo

O StepFlow não autentica usuários no baseline. A proteção de acesso físico e do perfil do navegador pertence ao dispositivo/ambiente operacional.

## Código hospedado

- HTTPS;
- CSP restritiva;
- dependências locais/bundled;
- sem `eval`;
- sem scripts de terceiros em runtime;
- Subresource Integrity somente se algum recurso externo for excepcionalmente necessário.

## Conteúdo

Entradas do usuário são dados. Evitar `innerHTML` com conteúdo não confiável. Preferir `textContent` e componentes estruturados.

## Importação

Arquivo importado é não confiável até validação completa. Aplicar limites de tamanho, tipos, contagens e estrutura.

## Privacidade

Sem telemetria obrigatória e sem envio automático dos dados locais. Exportação acontece somente por ação explícita do usuário.
