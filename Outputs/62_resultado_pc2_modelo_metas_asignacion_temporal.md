# Resultado PC-2 - Modelo de metas y asignación temporal

## Estado

**PASS — PC-2 cerrado. PC-3 no iniciado.**

## Objetivo

Incorporar al modelo semántico las metas comerciales y la asignación de PUSHER por periodo, conservando el histórico de altas y aplicando únicamente el override temporal aprobado.

## Commit técnico

- `8648400b5006977572d7048ceb43abd00be5b27c`
- Mensaje: `feat(model): agrega metas y asignacion pusher por periodo`

## Objetos implementados

- `Fact_MetasComerciales`, con grano único `AnioMes + AliadoKey` y 14 metas de julio de 2026.
- `Dim_AsignacionPusherPeriodo`, con grano único `PeriodoAliadoKey` y 720 combinaciones periodo-aliado.
- `Fact_AltasTeResuelve[PeriodoAliadoKey]`, clave técnica añadida sin alterar el contenido funcional de altas.

Relaciones nuevas, activas, `1:*` y con filtro unidireccional desde dimensión hacia hecho:

- `Dim_Calendario[Fecha]` → `Fact_MetasComerciales[FechaPeriodo]`.
- `Dim_Aliado[AliadoKey]` → `Fact_MetasComerciales[AliadoKey]`.
- `Dim_AsignacionPusherPeriodo[PeriodoAliadoKey]` → `Fact_AltasTeResuelve[PeriodoAliadoKey]`.
- `Dim_AsignacionPusherPeriodo[PeriodoAliadoKey]` → `Fact_MetasComerciales[PeriodoAliadoKey]`.

## Conciliación materializada

| Control | Resultado | Estado |
|---|---:|---|
| Filas de altas | 26.496 | PASS |
| Altas totales | 33.854 | PASS |
| Altas junio 2026 | 3.700 | PASS |
| Altas julio 2026 | 4.518 | PASS |
| Altas agosto 2026 | 559 | PASS |
| Filas de metas | 14 | PASS |
| Meta PUSHER 1 | 2.959 | PASS |
| Meta PUSHER 2 | 3.358 | PASS |
| Meta total | 6.317 | PASS |
| Duplicados de asignación | 0 | PASS |
| Duplicados de metas | 0 | PASS |
| Huérfanos altas → asignación | 0 | PASS |
| Huérfanos metas → asignación | 0 | PASS |
| Huérfanos metas → aliado | 0 | PASS |
| Huérfanos metas → calendario | 0 | PASS |

## Temporalidad de UNO 27

- Enero a junio de 2026: `Sin asignar` / `Clasificacion base`.
- Julio de 2026: `PUSHER 1` / `Override temporal`.
- Agosto de 2026: `Sin asignar` / `Clasificacion base`.

No se modificó `Map_PusherAliado` ni se reexpresó el histórico de enero a junio.

## Gates y regresión

- Gate automático: aprobado.
- Gate manual en Power BI Desktop: aprobado; refresh sin errores y páginas existentes funcionales.
- Gate MCP sobre el modelo materializado: aprobado.
- Relaciones muchos-a-muchos nuevas: 0.
- Relaciones bidireccionales nuevas: 0.
- Relaciones entre hechos: 0.
- Rutas ambiguas nuevas: 0.
- `Meta_Altas_Promedio_Mas_30_Pct`: intacta y en estado `Ready`.
- PBIR, visuales, Power Query de PC-1, datos fuente y Excel: sin cambios de PC-2.
- `Meta_Asignada` y `Cumplimiento_Meta_Pct`: no creadas.

## Privacidad y Git

- Las tablas nuevas contienen únicamente información comercial agregada y claves no personales.
- No se registraron rutas locales, usuarios, correos ni campos personales.
- `git diff --check`: PASS antes del commit técnico.
- Commit técnico publicado mediante fast-forward seguro a `origin/main`.

## Cierre

PC-2 queda cerrado y publicado. PC-3 no fue iniciado.
