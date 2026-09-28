# Resultado R6 — Medidas DAX

| Campo | Valor |
|---|---|
| Fase | R6 — Medidas DAX |
| Baseline | `588ce52eb8ec95910d097aa55dea1e1637c6fd9b` (cierre R5) |
| Estado | Validación estática y gate Desktop PASS, con observaciones |
| R7 | No iniciado |

## 0. Hotfix de contrato de incentivos (vigente)

Reemplaza lo que se oponga en este documento sobre incentivos (ver Output 74 §0). Contrato **Recibido → Legalizado → Pendiente**:

- `Valor_Recibido` = SUM(`ValorRecibido`), se mantiene.
- `Valor_Legalizado` = SUM(`ValorLegalizado`), nueva.
- `Pendiente_Legalizar`: BLANK si no hay recibido; en otro caso Valor recibido − Valor legalizado, contando el legalizado BLANK como 0 solo en este cálculo. Los negativos no se corrigen: se reportan como QA de la fuente. No usa valor gastado.
- `Fecha_Corte_Incentivos` se mantiene; con la fuente actual es el 26/09/2026.
- `Valor_Gastado` se eliminó porque la columna ya no existe; no tenía consumidores PBIR.

Siguen sin crearse saldo, ejecución, porcentaje de ejecución ni ROI. Valores conciliados: julio 3.300.000 / 3.300.000 / 0; agosto 2.668.800 / 2.668.800 / 0; septiembre 3.800.000 / 1.603.570 / 2.196.430; total 9.768.800 / 7.572.370 / 2.196.430 (recibido / legalizado / pendiente). 0 medidas en error; ALTAS, metas y crecimiento de R2-R6 sin cambios.

## 1. Inventario

**Reutilizadas sin cambios:** `Altas_Total`, `Meta_Asignada`, `Altas_Julio` y todas las medidas históricas de `GestionComercialAltas` (`Altas_Pusher_*`, `Delta_Pusher_*`, `Variacion_Pusher_*`, `Impacto_Observado_Pusher_2_Desde_Julio`, `Altas_Pusher_2_Desde_Gestion`, etc.). Siguen usando `PusherPeriodo` y `Dim_Calendario[Periodo_Gestion]`; no se migraron.

**Modificada:** `Cumplimiento_Meta_Pct`. Único consumidor PBIR: la tarjeta `% Cumplimiento` de `GestionComercialAltas`. El cambio solo altera contextos con meta y sin ventas (antes BLANK, ahora 0 %); el valor de julio total no cambia (71,31 %).

**Nuevas (en `_Medidas_Altas`):**

| Medida | Semántica |
|---|---|
| `Brecha_Meta` | Altas − Meta; BLANK sin meta |
| `Faltante_Meta` | MAX(Meta − Altas, 0); BLANK sin meta |
| `Fecha_Corte_Altas` | Último día con ALTAS del mes seleccionado, sin filtros de PUSHER, aliado ni otros contextos |
| `Altas_Julio_Dia_Comparable` | Julio 2026 hasta el día del corte global (completo si el mes está cerrado) |
| `Crecimiento_Desde_Julio` / `_Pct` | Altas del mes − base comparable de julio |
| `Altas_Gestion_Atribuible` | Altas de contextos con gestión atribuible |
| `Meta_Gestion_Atribuible` | Meta de contextos con gestión atribuible |
| `Cumplimiento_Gestion_Atribuible_Pct` | Misma regla que `Cumplimiento_Meta_Pct` sobre lo atribuible |
| `Crecimiento_Desde_Julio_Atribuible` / `_Pct` | Crecimiento limitado a los PUSHER con gestión atribuible en el mes |
| `Valor_Recibido` | SUM(`ValorRecibido`) |
| `Valor_Gastado` | SUM(`ValorGastado`); los vacíos de la fuente no suman como cero |
| `Fecha_Corte_Incentivos` | Fecha máxima de `Fact_LegalizacionBonos` en el contexto |

**Columna nueva:** `Dim_AsignacionPusherPeriodo[EsGestionAtribuible]`, calculada en Power Query con `Fecha_Inicio_Gestion_Pusher_2` y `Fecha_Inicio_Gestion_Pusher_3`. Es la única vía para que DAX use esos parámetros sin fechas fijas en las medidas. Es aditiva: no cambia claves, `PusherPeriodo` ni `PusherAsignacion`.

**Deliberadamente no creadas:**
- Medidas de asesor (meta individual, faltante, avance, estado, conteos): diferidas a R8. `Fact_AltasTeResuelve` no tiene asesor y `MetaAsesor` es una meta contextual por periodo-aliado sin identidad de asesor. Requieren grano de asesor, `Dim_Asesor`/`AsesorKey` solo si es necesario y el gate de privacidad nominal.
- Valor legalizado, pendiente, saldo, ejecución, porcentaje de ejecución, ROI y cualquier diferencia recibido − gastado.
- Brecha y faltante atribuibles: no requeridos para el alcance.

## 2. Reglas

**Cumplimiento (regla funcional aprobada por negocio):** `Cumplimiento = Altas totales del contexto / Meta disponible del contexto`. Las ventas de aliados sin meta forman parte de las altas del PUSHER o contexto y no se filtran a aliados con meta. Un resultado superior a 100 % es válido bajo esta regla; por ejemplo, Sin asignar en julio da 567 / 325 = 174 %, que no es un error DAX y no debe corregirse. Además: meta BLANK → BLANK; meta > 0 y altas BLANK/0 → 0 %; en otro caso altas / meta.

**Corte:** requiere exactamente un mes de `Dim_Calendario`; se quitan todos los filtros y se conserva solo el mes, por lo que un aliado con ventas hasta el día 20 recibe el corte global del 23/09.

**Crecimiento desde julio:** solo para un único mes posterior a julio 2026 con datos. Si el mes está cerrado (`Dim_Calendario[Es_Periodo_Comparable]`, gobernado por `Periodo_Corte_Comercial = 202608`), compara mes completo contra julio completo; si es parcial, compara el mes hasta el corte global contra julio hasta el mismo día. Mantiene los filtros de PUSHER y aliado. Julio, meses anteriores, multimes, ausencia de datos y aliados sin ventas en julio → BLANK. Un aliado con ventas en julio y cero en el mes seleccionado muestra crecimiento negativo.

**Gestión atribuible (por fila periodo-aliado):**
- PUSHER 1: siempre atribuible; no se inventa fecha de inicio.
- PUSHER 2: atribuible desde `202607`; antes, BLANK.
- PUSHER 3: atribuible desde `202608`; antes, BLANK.
- Sin asignar: nunca atribuible.

**PUSHER 3 (regla funcional aprobada por negocio):** la gestión inicia en agosto de 2026 y julio de 2026 es su línea base. Julio no cuenta como resultado de gestión (altas, cumplimiento y crecimiento atribuibles BLANK), pero sí se usa como base de comparación. Agosto es el primer mes atribuible y su incremento se calcula contra julio completo; septiembre se compara desde julio con día comparable. La base no se traslada a agosto.

La selección de periodo debe venir de `Dim_Calendario`; las medidas no reconocen un filtro puesto solo en `Dim_AsignacionPusherPeriodo[AnioMes]`.

## 3. Pruebas DAX (refresh real en Power BI Desktop)

Esperados calculados con `SUM` directo sobre la fact y filtros de fecha explícitos.

| Mes | Altas | Meta | Cumpl. | Brecha | Faltante | Corte | Base julio | Crec. | Crec. % |
|---|---|---|---|---|---|---|---|---|---|
| 2026-06 | 3.700 | — | BLANK | BLANK | BLANK | 30/06 | BLANK | BLANK | BLANK |
| 2026-07 | 4.519 | 6.337 | 71,31 % | −1.818 | 1.818 | 31/07 | BLANK | BLANK | BLANK |
| 2026-08 | 5.715 | 7.155 | 79,87 % | −1.440 | 1.440 | 31/08 | 4.519 (esperado 4.519) | +1.196 | +26,47 % |
| 2026-09 | 5.628 | 11.080 | 50,79 % | −5.452 | 5.452 | 23/09 | 3.098 (esperado 3.098) | +2.530 | +81,67 % |

Atribuible:

| Mes | Altas atrib. | Meta atrib. | Cumpl. atrib. | Crec. atrib. |
|---|---|---|---|---|
| 2026-07 | 3.950 | 6.012 | 65,70 % | BLANK |
| 2026-08 | 5.295 | 6.655 | 79,56 % | +1.343 |
| 2026-09 | 5.277 | 9.710 | 54,35 % | +2.640 |

Por PUSHER (`PusherAsignacion`):

| Mes | PUSHER | Altas | Base julio | Crec. | Atribuible |
|---|---|---|---|---|---|
| 2026-06 | PUSHER 1 | 1.160 | — | — | 1.160 (no bloqueado) |
| 2026-06 | PUSHER 2 | 1.850 | — | — | BLANK (línea base) |
| 2026-07 | PUSHER 3 | 2 | — | — | BLANK (línea base) |
| 2026-07 | Sin asignar | 567 | — | — | BLANK |
| 2026-08 | PUSHER 1 / 2 / 3 | 1.019 / 3.641 / 635 | 1.523 / 2.427 / 2 | −504 / +1.214 / +633 | igual |
| 2026-08 | Sin asignar | 420 | 567 | −147 | BLANK |
| 2026-09 | PUSHER 1 / 2 / 3 | 1.432 / 3.187 / 658 | 1.055 / 1.581 / 1 | +377 / +1.606 / +657 | igual |
| 2026-09 | Sin asignar | 351 | 461 | −110 | BLANK |

Bases de julio por PUSHER coinciden con el esperado independiente. `EsGestionAtribuible`: PUSHER 2 falso 202601-202606 y verdadero 202607-202609; PUSHER 3 falso 202601-202607 y verdadero 202608-202609; PUSHER 1 siempre verdadero; Sin asignar siempre falso.

Otros casos:
- Multimes julio+agosto, agosto+septiembre y sin filtro de mes: corte, crecimiento y crecimiento atribuible BLANK.
- Mes sin datos (2025-12): corte, crecimiento y cumplimiento BLANK.
- Aliados en septiembre (agregado): 0 aliados con corte distinto del global; 62 con último día propio anterior al 23/09 reciben el global; 0 diferencias entre base de julio y esperado.
- Meta con cero ventas (6 aliados en septiembre): cumplimiento 0 %, faltante = meta.
- Sin meta y con ventas (75 aliados): cumplimiento BLANK.
- Aliados activos sin ventas en julio (32): crecimiento BLANK.

Incentivos: recibido 8.698.800 (3.300.000 / 2.398.800 / 3.000.000); gastado 7.089.800 (3.300.000 / 2.668.800 / 1.121.000); corte 23/09/2026.

## 4. Integridad

ALTAS 44.638 (4.519 / 5.715 / 5.628); metas 6.337 / 7.155 / 11.080 (49 filas); incentivos 29 filas. Relaciones 18, many-to-many 0; sin nuevas relaciones, dimensiones PUSHER ni facts. Sin cambios PBIR. Medidas sin errores tras refresh.

## 5. Observaciones

1. Cumplimiento por encima de 100 % en contextos con aliados sin meta: comportamiento esperado según la regla aprobada de la sección 2.
2. La base de julio de PUSHER 3 es mínima (2 en agosto, 1 en septiembre), por lo que su crecimiento porcentual es extremo. Las páginas deben mostrar el valor absoluto y el `n` base.
3. Las medidas de incentivos no responden a `PusherAsignacion`; en R9 el filtro PUSHER debe usar `Fact_LegalizacionBonos[Pusher]`.
4. La próxima sesión de Desktop que guarde añadirá `lineageTag` a los objetos nuevos y a los de R5, y actualizará `PBI_QueryOrder`; es ruido esperado.

## 6. Documentación

`Specs/13` §9 (R6) y §12 (R9) actualizados al contrato R5 y al diferimiento de asesor a R8.

## 7. Privacidad

Pruebas solo con agregados y etiquetas públicas. Sin nombres, aliados individuales ni datos personales en el Output.

## 8. Rollback

Revertir `_Medidas_Altas.tmdl` y `Dim_AsignacionPusherPeriodo.tmdl` contra `588ce52` y retirar las líneas de R6 de `Specs/13`. No revertir R2-R5.
