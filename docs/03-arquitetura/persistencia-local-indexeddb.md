# Persistência local — IndexedDB

IndexedDB é a fonte de verdade do estado funcional local.

## Motivos

- assíncrono;
- transações;
- índices;
- objetos e Blob;
- adequado a dados maiores que preferências simples.

`localStorage` pode guardar apenas preferências pequenas e não essenciais.

## Wrapper

`dexie/Dexie.js` é o candidato preferencial para reduzir complexidade de schema, migrations e transações. Sua adoção depende de medição do bundle e revisão de licença/versão.

## Versões

A persistência possui `schemaVersion` local. Migrations são forward-only e executadas antes de liberar o app para mutação.

Falha de migration bloqueia mutações e preserva o banco anterior quando a transação permitir rollback.

## Dados principais

- company;
- categories;
- responsible_people;
- procedures;
- procedure_revisions;
- procedure_stages;
- attendances;
- attendance_procedures;
- attendance_stage_state;
- equipments;
- managed_assets;
- app_meta.

A forma concreta das stores será fechada no passo de implementação da persistência.
