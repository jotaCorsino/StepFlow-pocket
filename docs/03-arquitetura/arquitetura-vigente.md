# Arquitetura vigente — StepFlow

## Visão geral

```text
cPanel / HTTPS
    ↓ arquivos estáticos
Browser / PWA
    ├─ UI HTML/CSS/JS
    ├─ IndexedDB
    ├─ Cache Storage / Service Worker
    └─ Exportar / Importar dados
```

O StepFlow não possui backend obrigatório.

## Runtime

Produção executa somente no navegador. O cPanel entrega arquivos estáticos.

Não fazem parte da arquitetura vigente: processo próprio no servidor, banco de dados remoto, API do StepFlow, WebSocket, autenticação, sessões ou sincronização entre dispositivos.

## Organização de source proposta

```text
src/
├── app/
├── components/
├── features/
│   ├── procedures/
│   ├── attendances/
│   ├── equipments/
│   ├── settings/
│   └── data-transfer/
├── services/
│   ├── storage/
│   ├── export-import/
│   └── print/
├── shared/
└── styles/
```

O source permanece modular. O build pode consolidar tudo em poucos artefatos estáticos.

## Princípios

- single-user;
- local-first;
- baixo consumo;
- dependências mínimas;
- Web Platform nativa antes de bibliotecas;
- nenhuma lógica crítica espalhada em HTML monolítico;
- dados estruturados em vez de HTML arbitrário;
- exportação/importação versionada;
- PWA/offline sem sincronização remota.
