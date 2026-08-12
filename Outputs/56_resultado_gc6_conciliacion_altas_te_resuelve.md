# Resultado GC-6 — Conciliación técnica de Altas Te Resuelve

## Objetivo y resultado

Se validó la trazabilidad de la fuente vigente desde la tabla formal de Excel hasta Power Query, `Fact_AltasTeResuelve` y las medidas DAX. La regla oficial es `SUM(Fact_AltasTeResuelve[Altas])`.

**Resultado global: PASS.** Las cantidades conciliadas presentan diferencia cero. No se modificaron el modelo, Power Query, DAX, el reporte ni los datos fuente.

## Evidencia congelada de la fuente

| Propiedad | Valor |
| --- | --- |
| Archivo | `Data/Informe de Altas/INFORME ALTAS TE RESUELVE Cierre Julio.xlsx` |
| Tamaño | 4.839.303 bytes |
| Fecha de modificación | 10/08/2026 11:59:10, hora Colombia |
| SHA-256 | `C0D42E46095B47F7EA5ABE2703A6A53AE84ED9A9A33C43C7EBEA4E9B808D4893` |
| Hoja | `Tabla1_2 (2)` |
| Tabla formal | `Insumo2` |
| Rango formal | `A1:R26497` |
| Filas de datos | 26.496 |
| Columnas | 18 |
| Rango de fechas | 01/01/2026–04/08/2026 |
| Periodos | 202601–202608 |

La revisión fue de solo lectura y no abrió ni guardó el libro mediante Excel Desktop.

## Trazabilidad

| Capa | Control | Resultado |
| --- | --- | --- |
| Excel `Insumo2` | 26.496 filas / suma de `ALTAS` | 33.854 |
| `Base_AltasTeResuelve` | lectura por nombre de tabla y validación de columnas | PASS |
| `AltasTeResuelve_Limpio` | conciliación de filas y altas, sin columnas sensibles | PASS |
| `Fact_AltasTeResuelve` | 26.496 filas / `SUM(Altas)` | 33.854 |
| DAX `Altas_Total` | archivo completo | 33.854 |

## Conciliación general

| Alcance | Excel | Power BI/DAX | Diferencia | Estado |
| --- | ---: | ---: | ---: | --- |
| Archivo completo | 33.854 | 33.854 | 0 | PASS |
| Enero–julio | 33.295 | 33.295 | 0 | PASS |
| Agosto parcial | 559 | 559 | 0 | PASS |
| Filas de agosto | 452 | 452 | 0 | PASS |

## Conciliación mensual

| Mes | Altas Excel | Altas Power BI | Diferencia | Estado |
| --- | ---: | ---: | ---: | --- |
| Enero 2026 | 6.297 | 6.297 | 0 | PASS |
| Febrero 2026 | 5.310 | 5.310 | 0 | PASS |
| Marzo 2026 | 5.109 | 5.109 | 0 | PASS |
| Abril 2026 | 4.190 | 4.190 | 0 | PASS |
| Mayo 2026 | 4.171 | 4.171 | 0 | PASS |
| Junio 2026 | 3.700 | 3.700 | 0 | PASS |
| Julio 2026 | 4.518 | 4.518 | 0 | PASS |

## Junio, julio y línea base abril–junio

| Indicador | Esperado | Obtenido | Estado |
| --- | ---: | ---: | --- |
| Diferencia junio–julio | +818 | +818 | PASS |
| Variación junio–julio | 22,1081 % | 22,1081 % | PASS |
| Promedio abril–junio | 4.020,3333 | 4.020,3333 | PASS |
| Julio vs. promedio abril–junio | 12,3787 % | 12,3787 % | PASS |

Las diferencias porcentuales se evaluaron con tolerancia de redondeo; las cantidades exigieron igualdad exacta.

## Conciliación por PUSHER

| Periodo | Clasificación | Excel | Power BI/DAX | Diferencia | Estado |
| --- | --- | ---: | ---: | ---: | --- |
| Enero–julio | PUSHER 1 | 12.221 | 12.221 | 0 | PASS |
| Enero–julio | PUSHER 2 | 16.499 | 16.499 | 0 | PASS |
| Enero–julio | Sin asignar | 4.575 | 4.575 | 0 | PASS |
| Enero–julio | Total | 33.295 | 33.295 | 0 | PASS |
| Junio | PUSHER 1 | 1.193 | 1.193 | 0 | PASS |
| Junio | PUSHER 2 | 1.857 | 1.857 | 0 | PASS |
| Junio | Sin asignar | 650 | 650 | 0 | PASS |
| Junio | Total | 3.700 | 3.700 | 0 | PASS |
| Julio | PUSHER 1 | 1.357 | 1.357 | 0 | PASS |
| Julio | PUSHER 2 | 2.429 | 2.429 | 0 | PASS |
| Julio | Sin asignar | 732 | 732 | 0 | PASS |
| Julio | Total | 4.518 | 4.518 | 0 | PASS |

Controles adicionales:

- cobertura ponderada enero–julio: 86,2592 %;
- delta PUSHER 1 junio–julio: +164;
- delta PUSHER 2 junio–julio: +572;
- contribución observada de PUSHER 2 al cambio neto: 69,9267 %.

Las cifras anteriores describen asociación temporal y contribución al cambio; no demuestran causalidad.

## Drivers junio–julio

La evaluación dinámica sobre los 167 aliados identificó como principales resultados actuales:

| Aliado | Cambio junio–julio | Estado |
| --- | ---: | --- |
| ATENTO | +381 | PASS |
| ONE CONTACT | +201 | PASS |
| GNP | -70 | PASS |

La suma completa de contribuciones por aliado fue **+818**, igual al cambio neto. Para esta conciliación se trataron como cero los aliados sin volumen en uno de los dos meses. La agregación directa de la medida auxiliar `Delta_Aliado_Mes` produce 809 porque conserva `BLANK` cuando no existe base previa; no altera los drivers mostrados, pero GC-7 no debe utilizar la suma de esa medida como total de contribuciones.

## Conciliación con el dashboard del Excel

La tabla dinámica `TablaDinámica2`, ubicada en `INFORME ALTAS MULTIASISTENCIAS`, usa como origen la tabla formal `Insumo2` y como métrica `Suma de ALTAS`. El periodo visible corresponde a julio de 2026 y el total general almacenado en el pivote es **4.518**, igual al detalle y a Power BI. Los demás filtros almacenados no restringen el universo visible.

El pivote es evidencia de caché del libro y no fue recalculado en Excel Desktop. Esta limitación no bloquea la conciliación porque el total almacenado coincide exactamente con `Insumo2` y DAX. El dashboard se utilizó solo como referencia, no como fuente de hechos.

## Agosto parcial

Agosto permanece cargado con 452 filas y 559 altas. En `Dim_Calendario` se confirmó `Estado_Periodo = En curso` y `Es_Periodo_Comparable = FALSE`. Las medidas mensuales comparativas devuelven `BLANK`; agosto no se presenta como cierre comparable contra julio.

## Diferencia histórica de 25 altas

La versión histórica documentada registraba 29.366 altas hasta el 05/07, mientras la fuente vigente registra 29.341 para el mismo corte. La diferencia neta de 25 altas se conserva como **riesgo de versionado de fuente**; no se corrigió artificialmente. Para GC-6, la única fuente operativa es el archivo vigente, que concilia completamente con Power BI.

## Regresión y alcance

- `Fact_CalidadLlamadas`: 3 filas.
- `Fact_SatisfaccionCapacitacion`: 150 filas.
- `Fact_MotivacionActividad`: 5 filas.
- `Fact_AltasTeResuelve`: 26.496 filas y 33.854 altas.
- `Dim_Aliado`: 167 claves; cero nulos, duplicados y hechos huérfanos.
- Fechas huérfanas: cero.
- Relaciones del modelo: 13, sin cambios; las dos relaciones comerciales siguen activas, 1:* y unidireccionales.
- LocalDateTable: 3, sin cambios.
- No hubo cambios en expresiones, tablas TMDL, medidas, relaciones, páginas, navegación, `Data`, el Excel fuente ni Satisfacción.

## Cierre

**GC-6 cerrado / GC-7 no iniciado.** No existen discrepancias no justificadas entre `Insumo2` vigente y Power BI.
