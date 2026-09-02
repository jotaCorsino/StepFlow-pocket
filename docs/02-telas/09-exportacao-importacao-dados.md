# Tela 09 — Exportar / Importar dados

Substitui qualquer mecanismo de Backup/Restore.

## Exportar

Ação principal `Exportar dados` gera um arquivo portátil contendo todo o estado funcional local e assets administrados.

A tela informa:

- versão do formato;
- data/hora da exportação;
- contagens principais;
- tamanho estimado/gerado;
- nome sugerido do arquivo.

## Importar

Fluxo:

```text
Selecionar arquivo
→ validar
→ mostrar resumo
→ confirmar
→ aplicar localmente em transação
→ recarregar estado
```

Arquivo incompatível/corrompido não altera o conjunto atual.

Importação com substituição integral exige confirmação clara. Não existe recovery de servidor, journal ou operação remota.
