# Resultado GC-5 — Medidas DAX de altas Te Resuelve

## Objetivo

Implementar y validar el catálogo DAX de Gestión Comercial en una tabla exclusiva de medidas, con guardas para periodos parciales, filtros dinámicos y divisiones seguras.

## Implementación

- Commit técnico: `95875987a2e62ea7988a021e9896b7e96c8607ff`.
- Se creó `_Medidas_Altas` con 28 medidas: 23 principales y 5 auxiliares para drivers.
- `Columna1` permanece oculta, vacía y no participa en ningún cálculo.
- El volumen comercial usa exclusivamente `SUM(Fact_AltasTeResuelve[Altas])`.
- Las cantidades y deltas tienen formato entero; los porcentajes permanecen como proporciones 0–1 con formato porcentual; las etiquetas son medidas de texto.
- Todas las divisiones usan `DIVIDE` con resultado vacío ante denominadores inválidos. No se divide ningún porcentaje nuevamente entre 100.

## Conciliación principal

- Archivo completo: 33.854 altas.
- Enero–julio: 33.295 altas.
- Abril: 4.190; mayo: 4.171; junio: 3.700; julio: 4.518.
- Promedio abril–junio: 4.020,33.
- Julio frente al promedio: 12,38 %.
- Junio–julio: diferencia de 818 y variación de 22,11 %.
- PUSHER 1: 1.193 en junio, 1.357 en julio y delta de 164.
- PUSHER 2: 1.857 en junio, 2.429 en julio y delta de 572.
- Contribución de PUSHER 2 al cambio neto: 69,93 %.
- Cobertura ponderada enero–julio: 86,26 %.

## Periodos, filtros y drivers

- Agosto parcial conserva 452 filas y 559 altas en la medida base.
- Las comparaciones de agosto devuelven vacío por tratarse de un periodo no comparable.
- Se probaron mes, PUSHER, aliado, combinaciones de filtros, `Sin asignar`, primer mes, múltiples meses y denominador cero.
- Las comparaciones mensuales exigen un único periodo comparable.
- Los drivers son dinámicos: ATENTO y ONE CONTACT aparecen como principales contribuciones positivas actuales y GNP como principal caída, sin nombres hardcodeados.

## Regresión y alcance

- `Fact_AltasTeResuelve`: 26.496 filas y 33.854 altas.
- `Fact_CalidadLlamadas`: 3 filas.
- `Fact_SatisfaccionCapacitacion`: 150 filas.
- `Fact_MotivacionActividad`: 5 filas.
- `Dim_Aliado`: 167 claves, sin nulos, duplicados ni huérfanos.
- Relaciones sin cambios y `LocalDateTable` = 3.
- Sin cambios en hechos, dimensiones, relaciones, Power Query, reporte, navegación, `Data` o Satisfacción.
- Estado: **GC-5 cerrado / GC-6 no iniciado**.
