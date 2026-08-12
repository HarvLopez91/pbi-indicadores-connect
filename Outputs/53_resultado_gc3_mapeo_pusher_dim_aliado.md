# Resultado GC-3 — Mapeo PUSHER y Dim_Aliado

## Objetivo

Crear un catálogo gobernado de aliados comerciales, clasificarlos mediante coincidencia exacta y disponer de un control agregado de valores pendientes, sin crear todavía hechos, relaciones ni medidas.

## Implementación

- **Commit técnico:** `cf9797309a16edcb7c7358b9cdf2119a74f51d03`.
- `Map_PusherAliado`: catálogo auxiliar con carga deshabilitada.
- `Dim_Aliado`: dimensión con `AliadoKey`, `Descripcion`, `Pusher` y `Estado_Clasificacion`.
- `Control_Aliados_Sin_Clasificar`: control agregado con carga deshabilitada, sin información personal.
- `PBI_QueryOrder` y la referencia de `Dim_Aliado` fueron actualizados en `model.tmdl`.

## Catálogo y calidad

- PUSHER 1: **7 aliados**.
- PUSHER 2: **9 aliados**.
- Aliados distintos en la fuente: **167**.
- Aliados clasificados: **16**.
- Aliados `Sin asignar`: **151**.
- Duplicados en el mapa: **0**.
- Duplicados o nulos en `AliadoKey`: **0**.
- Coincidencias aproximadas: **0**.
- Todo valor sin coincidencia exacta permanece `Sin asignar` y `Pendiente`.

## Controles agregados enero-julio

| Clasificación | Altas |
| --- | ---: |
| PUSHER 1 | 12.221 |
| PUSHER 2 | 16.499 |
| Sin asignar | 4.575 |
| **Total** | **33.295** |

La cobertura ponderada de clasificación es **86,26 %**.

La conciliación mensual confirmó:

- junio: **3.700** altas;
- julio: **4.518** altas.

La distribución por clasificación concilia con el total del archivo vigente. `Control_Aliados_Sin_Clasificar[CantidadAltas]` usa todos los periodos disponibles, incluido agosto parcial; por ello no debe compararse directamente con cifras limitadas a enero-julio ni ajustarse artificialmente.

## Gate manual

Power BI Desktop confirmó 16 filas en el mapa, 167 filas en la dimensión y 151 filas en el control. También confirmó los estados `Clasificado` y `Pendiente`, las cargas auxiliares deshabilitadas y la ausencia de alias o coincidencias aproximadas.

## Exclusiones confirmadas

No se modificaron `Dim_CallCenter`, `Dim_Calendario`, relaciones, DAX, páginas, navegación, Data ni Satisfacción. No se crearon `Fact_AltasTeResuelve` ni `_Medidas_Altas`. No se reprodujeron registros personales ni el catálogo completo de aliados pendientes.

## Estado final

**GC-3 cerrado. GC-4 no iniciado.**
