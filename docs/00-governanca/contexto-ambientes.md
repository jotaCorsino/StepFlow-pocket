# Contexto de ambientes — StepFlow

**Estado vigente:** aplicação web estática, single-user e local-first.

## Produção

O ambiente disponível é um servidor gerenciado por cPanel. O StepFlow não depende de executar processos próprios no servidor.

Produção significa publicar somente artefatos estáticos por HTTPS:

```text
HTML + CSS + JavaScript + manifest + service worker + assets
```

Não são requisitos do StepFlow: PHP, Node, Python, Ruby, Java, Rust, banco de dados, cron, daemon ou terminal no servidor.

## Dados

Os dados funcionais pertencem ao navegador/perfil/dispositivo do usuário e são persistidos localmente em IndexedDB. O servidor não é fonte de verdade dos dados.

Mover ou preservar dados entre dispositivos depende de **Exportar dados / Importar dados**.

## Offline

A forma preferencial é PWA após primeiro acesso HTTPS. O service worker mantém os assets do app disponíveis sem rede. O banco local continua no IndexedDB.

Um `StepFlow.html` autocontido pode existir como artefato portátil opcional, mas não deve depender de comportamento não padronizado de armazenamento em `file://` para garantir preservação dos dados.

## Navegadores

Baseline: navegadores modernos Chromium/Firefox compatíveis com ES2022+, IndexedDB, Service Worker, Blob/File e CSS de impressão. Compatibilidade concreta será medida no início da implementação.

## Restrições

- consumo de CPU/memória/rede deve ser baixo;
- nenhuma conexão contínua;
- nenhuma telemetria obrigatória;
- nenhuma CDN obrigatória em produção;
- nenhuma execução de arquivo próprio no servidor;
- nenhuma dependência de banco do provedor;
- nenhum requisito multiusuário/sincronização.
