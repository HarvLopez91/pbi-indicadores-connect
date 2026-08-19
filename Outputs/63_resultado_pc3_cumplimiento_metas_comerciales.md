# Resultado PC-3 - Cumplimiento de metas comerciales

## Estado

**PASS — PC-3 cerrado. PC-4 no iniciado.**

## Objetivo

Crear las medidas de meta y cumplimiento, y migrar las medidas de clasificación PUSHER para que utilicen la asignación temporal aprobada.

## Commit técnico

- `b1c60c61f94bb1f599c94c565dd15299011e694a`
- Mensaje: `feat(dax): agrega cumplimiento de metas comerciales`

## Medidas creadas

- `Meta_Asignada = SUM(Fact_MetasComerciales[MetaAltas])`, formato `#,0`.
- `Cumplimiento_Meta_Pct = DIVIDE([Altas_Total], [Meta_Asignada], BLANK())`, formato `0.00%`.

La ausencia de meta se conserva como `BLANK`; no se convierte en cero ni se limita el cumplimiento a 100 %.

## Migración de clasificación PUSHER

El inventario detectó 13 referencias directas a `Dim_Aliado[Pusher]` en 10 medidas. Todas las referencias cuya semántica depende de la clasificación comercial se migraron a `Dim_AsignacionPusherPeriodo[PusherPeriodo]`:

- `Altas_Pusher_1`
- `Altas_Pusher_2`
- `Participacion_Pusher_1_Pct`
- `Participacion_Pusher_2_Pct`
- `Contribucion_Cambio_Pusher_1_Pct`
- `Contribucion_Cambio_Pusher_2_Pct`
- `Contribucion_Cambio_Sin_Asignar_Pct`
- `Cobertura_Clasificacion_Pusher_Pct`
- `Altas_Sin_Asignar`
- `Participacion_Sin_Asignar_Pct`

No se crearon excepciones DAX por aliado y no se modificaron medidas derivadas que heredan correctamente la nueva clasificación.

## Matriz de pruebas DAX

### Julio de 2026

| Contexto | Altas | Meta | Cumplimiento | Estado |
|---|---:|---:|---:|---|
| General | 4.518 | 6.317 | 71,52 % | PASS |
| PUSHER 1 | 1.581 | 2.959 | 53,43 % | PASS |
| PUSHER 2 | 2.429 | 3.358 | 72,33 % | PASS |
| ATENTO | 1.395 | 1.400 | 99,64 % | PASS |
| ONE CONTACT | 562 | 636 | 88,36 % | PASS |
| GNP | 341 | 1.082 | 31,52 % | PASS |
| UNO 27 | 224 | 225 | 99,56 % | PASS |

Las diferencias frente a los valores esperados fueron cero en cantidades; los porcentajes coinciden con la precisión matemática esperada.

### Ausencia de meta

| Contexto | Altas julio | Meta | Cumplimiento | Estado |
|---|---:|---:|---:|---|
| CAV BOGOTA PLAZA IMPERIAL | 5 | BLANK | BLANK | PASS |
| CAV BOGOTA PLAZA DE LAS AMERICAS | BLANK | BLANK | BLANK | PASS |
| CAV BOGOTA CENTRO MAYOR | 1 | BLANK | BLANK | PASS |
| Junio de 2026 | 3.700 | BLANK | BLANK | PASS |

Las altas reales permanecen en los agregados aunque el aliado no tenga meta.

### Regresión por clasificación temporal

| Periodo | PUSHER 1 | PUSHER 2 | Sin asignar | Total | Estado |
|---|---:|---:|---:|---:|---|
| Junio 2026 | 1.193 | 1.857 | 650 | 3.700 | PASS |
| Julio 2026 | 1.581 | 2.429 | 508 | 4.518 | PASS |
| Agosto 2026 parcial | 146 | 330 | 83 | 559 | PASS |

- Cobertura de clasificación en julio: 88,76 %.
- UNO 27 permanece `Sin asignar` de enero a junio, cambia a `PUSHER 1` únicamente en julio mediante override temporal y vuelve a `Sin asignar` en agosto.

## Regresión general

- Altas totales: 33.854.
- Junio: 3.700.
- Julio: 4.518.
- Agosto parcial: 559.
- `Meta_Altas_Promedio_Mas_30_Pct`: expresión intacta y resultado de julio igual a 6.235; se conserva como heurística auxiliar, no como modelo predictivo.
- Las 38 medidas de `_Medidas_Altas` quedaron en estado `Ready`, sin referencias rotas.

## Alcance, privacidad y Git

- Único archivo técnico modificado: `_Medidas_Altas.tmdl`.
- No se modificaron hechos, dimensiones, relaciones, configuraciones Power Query, PBIR, slicers, tarjetas, datos ni archivos Excel.
- No se incorporaron rutas locales, usuarios, correos ni datos personales.
- `git diff --check`: PASS.
- Commit técnico publicado mediante fast-forward seguro a `origin/main`.

## Cierre

PC-3 queda cerrado y publicado. PC-4 no fue iniciado.
