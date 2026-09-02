# Exportação e Importação de dados

## Objetivo

Permitir portabilidade, preservação manual e movimentação do conjunto local sem backend.

## Formato

Arquivo versionado com metadados mínimos:

- `format`;
- `formatVersion`;
- `exportedAt`;
- `appVersion`;
- contagens;
- payload de entidades;
- assets administrados.

Formato preferido: arquivo único `.stepflow` quando compactação for adotada. JSON permanece formato interno/fallback de desenvolvimento.

## Exportação

Captura snapshot lógico consistente do IndexedDB em transação de leitura e gera arquivo para download.

## Importação

1. ler arquivo;
2. validar magic/formato/versão/limites;
3. validar tipos e referências;
4. mostrar resumo;
5. pedir confirmação;
6. escrever em transação local;
7. somente após sucesso substituir o estado ativo conforme modo aprovado.

Import falho não deixa meio conjunto aplicado.

## Segurança

Importação trata strings como dados. Não executar HTML, JS ou URLs arbitrárias vindas do arquivo.

Limites de tamanho/profundidade/quantidade serão definidos com dados representativos.
