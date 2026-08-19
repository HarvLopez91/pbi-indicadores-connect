# Resultado PC-1 — Fuente y homologación de metas comerciales

## Estado

**PASS.** PC-1 quedó implementado y conciliado. PC-2 no fue iniciado.

## Preflight

- Repositorio local del proyecto.
- Rama: `main`.
- Baseline local y remoto: `81cda6187ce939a4544609b85c76065d20ee90a4`.
- Working tree y staging iniciales: limpios.
- Remote: `HarvLopez91/pbi-indicadores-connect`.
- Power BI Desktop y Excel: cerrados.
- Worktrees: únicamente el principal.

## Implementación

Archivos técnicos modificados:

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`.
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl`.

Objetos creados mediante Power BI Modeling MCP:

- `Config_MetasComerciales`: expresión M sin carga, con grano único `AnioMes + AliadoKey` y columnas `AnioMes`, `AliadoKey`, `MetaAltas`.
- `Map_AsignacionPusherPeriodo`: expresión M sin carga, con el único override `202607 | UNO 27 | PUSHER 1`.

`model.tmdl` solo incorpora ambos objetos al orden de consultas. No se modificó `Map_PusherAliado`.

## Conciliación

| Control | Resultado | Estado |
|---|---:|---|
| Metas con valor | 14 | PASS |
| Meta PUSHER 1 | 2.959 | PASS |
| Meta PUSHER 2 | 3.358 | PASS |
| Meta general | 6.317 | PASS |
| Duplicados `AnioMes + AliadoKey` | 0 | PASS |
| `AliadoKey` nulos | 0 | PASS |
| Metas no positivas | 0 | PASS |
| Aliados fuera del catálogo canónico aprobado | 0 | PASS |
| Overrides configurados | 1 | PASS |

La igualdad `2.959 + 3.358 = 6.317` se cumple exactamente. Las metas vacías de Plaza Imperial, Plaza de las Américas y Centro Mayor permanecen como ausencia de fila y, por tanto, como ausencia de meta; no se crearon valores cero.

Los tipos declarados son `Int64.Type`, `text` e `Int64.Type` para `AnioMes`, `AliadoKey` y `MetaAltas`. La configuración no contiene personas, cargos sensibles, correos, rutas locales ni otros datos confidenciales.

## Validación técnica y alcance

- El TMDL fue recargado correctamente por el MCP: 18 tablas, 66 medidas y 13 relaciones.
- `Config_MetasComerciales` y `Map_AsignacionPusherPeriodo` fueron recuperadas correctamente después de la recarga.
- `Fact_MetasComerciales`, `Dim_AsignacionPusherPeriodo`, nuevas relaciones, claves, medidas y cambios PBIR: inexistentes.
- `Meta_Altas_Promedio_Mas_30_Pct`: sin cambios.
- `Map_PusherAliado`: sin cambios.
- Enero-junio de UNO 27: sin reclasificación.
- Data y archivos Excel: sin cambios y fuera del commit.
- `git diff --check`: PASS antes del commit técnico.

## Git

- Commit técnico: `91c0794d89583b31f2751a9fdf410bb07b976e06`.
- Mensaje: `feat(data): agrega configuracion de metas comerciales`.
- Push: publicado en `origin/main`.
- Archivos incluidos: exclusivamente `expressions.tmdl` y `model.tmdl`.

## Cierre

PC-1 cerrado con estado **PASS**. PC-2 no iniciado.
