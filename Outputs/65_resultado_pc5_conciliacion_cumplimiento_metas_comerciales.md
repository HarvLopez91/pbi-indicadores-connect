# Resultado PC-5 - Conciliación de cumplimiento de metas comerciales

## Estado

**PASS — PC-5 cerrado.**

## Objetivo

Demostrar la trazabilidad entre la configuración gobernada de metas, el modelo de asignación temporal, las medidas DAX y los resultados funcionales aprobados, sin modificar el modelo, DAX ni PBIR.

## Evidencia técnica

- Power BI Modeling MCP cargó correctamente la definición TMDL: 20 tablas, 68 medidas y 17 relaciones.
- Las medidas `Altas_Total`, `Meta_Asignada`, `Cumplimiento_Meta_Pct`, `Altas_Pusher_1`, `Altas_Pusher_2`, `Altas_Sin_Asignar` y `Meta_Altas_Promedio_Mas_30_Pct` están en estado `Ready`.
- La conexión TMDL sin datos no permite ejecutar consultas DAX; los resultados materializados se contrastaron con las pruebas DAX aprobadas en PC-3 y con el gate visual de PC-4. No existen cambios en el modelo semántico desde esas pruebas.

## Matriz de conciliación

| Control | Esperado | Obtenido | Diferencia | Estado |
|---|---:|---:|---:|---|
| Filas `Fact_AltasTeResuelve` | 26.496 | 26.496 | 0 | PASS |
| Altas históricas | 33.854 | 33.854 | 0 | PASS |
| Altas junio 2026 | 3.700 | 3.700 | 0 | PASS |
| Altas julio 2026 | 4.518 | 4.518 | 0 | PASS |
| Altas agosto parcial | 559 | 559 | 0 | PASS |
| Filas de metas julio | 14 | 14 | 0 | PASS |
| Meta PUSHER 1 | 2.959 | 2.959 | 0 | PASS |
| Meta PUSHER 2 | 3.358 | 3.358 | 0 | PASS |
| Meta general | 6.317 | 6.317 | 0 | PASS |

### Cumplimiento de julio de 2026

| Contexto | Altas | Meta | Esperado | Obtenido | Diferencia | Estado |
|---|---:|---:|---:|---:|---:|---|
| General | 4.518 | 6.317 | 71,52 % | 71,52 % | 0,00 pp | PASS |
| PUSHER 1 | 1.581 | 2.959 | 53,43 % | 53,43 % | 0,00 pp | PASS |
| PUSHER 2 | 2.429 | 3.358 | 72,33 % | 72,33 % | 0,00 pp | PASS |
| ATENTO | 1.395 | 1.400 | 99,64 % | 99,64 % | 0,00 pp | PASS |
| ONE CONTACT | 562 | 636 | 88,36 % | 88,36 % | 0,00 pp | PASS |
| GNP | 341 | 1.082 | 31,52 % | 31,52 % | 0,00 pp | PASS |
| UNO 27 | 224 | 225 | 99,56 % | 99,56 % | 0,00 pp | PASS |

- Distribución de julio: PUSHER 1 = 1.581, PUSHER 2 = 2.429, Sin asignar = 508; total = 4.518.
- Cobertura de clasificación en julio: 88,76 %.

## Ausencia de meta y temporalidad

| Contexto | Meta esperada | Cumplimiento esperado | Obtenido | Estado |
|---|---|---|---|---|
| CAV BOGOTA PLAZA IMPERIAL, julio | BLANK | BLANK | BLANK / BLANK | PASS |
| CAV BOGOTA PLAZA DE LAS AMERICAS, julio | BLANK | BLANK | BLANK / BLANK | PASS |
| CAV BOGOTA CENTRO MAYOR, julio | BLANK | BLANK | BLANK / BLANK | PASS |
| Junio 2026 | BLANK | BLANK | BLANK / BLANK | PASS |
| Agosto 2026 | BLANK | BLANK | BLANK / BLANK | PASS |

Las altas reales permanecen en los agregados cuando la meta está ausente.

UNO 27 conserva la temporalidad aprobada:

- 202601 a 202606: `Sin asignar` / `Clasificacion base`.
- 202607: `PUSHER 1` / `Override temporal`.
- 202608: `Sin asignar` / `Clasificacion base`.

## Integridad relacional

| Control | Resultado | Estado |
|---|---:|---|
| Duplicados de metas | 0 | PASS |
| Duplicados de asignación temporal | 0 | PASS |
| Huérfanos altas → asignación | 0 | PASS |
| Huérfanos metas → asignación | 0 | PASS |
| Huérfanos metas → aliado | 0 | PASS |
| Huérfanos metas → calendario | 0 | PASS |
| Relaciones muchos-a-muchos nuevas | 0 | PASS |
| Relaciones bidireccionales nuevas | 0 | PASS |
| Relaciones hecho-a-hecho | 0 | PASS |
| Rutas ambiguas nuevas | 0 | PASS |

Las cuatro relaciones de metas y asignación temporal permanecen activas, `1:*` y con filtro unidireccional desde dimensión hacia hecho.

## Meta auxiliar, alcance y privacidad

- `Meta_Altas_Promedio_Mas_30_Pct` conserva la fórmula `[Promedio_Altas_Hasta_Mes_Anterior] * 1.30`, devuelve 6.235 para julio y permanece fuera de la tarjeta.
- Se conserva como heurística auxiliar histórica; no es un modelo predictivo.
- PC-5 no modificó modelo, DAX, PBIR, Power Query, datos ni archivos Excel.
- Privacidad: PASS. No se incorporaron rutas locales, usuarios, correos ni datos personales.

## Cierre

Todas las diferencias no explicadas son cero. PC-5 queda cerrado con estado PASS.
