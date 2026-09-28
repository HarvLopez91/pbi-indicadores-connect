# Resultado R2 — Migración del consolidado de Gestión comercial

| Campo | Valor |
| ----- | ----- |
| Fase | R2 — Migración del consolidado |
| Estado | PASS — refresh, conciliación y revisión post-Desktop aprobados |
| Baseline Git | `0ee66bd90426aa617dc824288261c4541346674c` |
| Fuente oficial | `Data/Informe de Altas/Consolidado informe de Altas.xlsx` |
| Objeto Excel | Tabla formal `Insumo2` |
| Regla comercial | Ventas = `SUM(ALTAS)` |
| Periodo de corte | `202608` |

## 1. Objetivo

Migrar la única fuente activa de ventas desde el snapshot de julio hacia el consolidado acumulado, sin anexar archivos anteriores y sin crear un segundo hecho de ventas.

Contrato de referencia:

- [Análisis de impacto R0](../Specs/12_analisis_impacto_mockups_gestion_comercial_septiembre.md)
- [Plan R1–R10](../Specs/13_plan_implementacion_mockups_gestion_comercial_septiembre.md)
- [Baseline R1](70_resultado_r1_proteccion_baseline_gestion_comercial_septiembre.md)

## 2. Cambios aplicados

Se modificó únicamente `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`:

1. `Ruta_Informe_Altas` y la validación interna de `Base_AltasTeResuelve` seleccionan exclusivamente `Consolidado informe de Altas.xlsx`.
2. Se conserva la lectura de la tabla formal `Insumo2`; no se combinan ni anexan snapshots.
3. `Periodo_Corte_Comercial` cambió de `202607` a `202608`.
4. La existencia del periodo de corte se valida a partir de `FECHA_ALTA`, no del campo auxiliar `MES`.
5. `AltasTeResuelve_Limpio` conserva el valor original de `MES` como `MesFuente` y deriva el periodo canónico `Mes` desde `FechaAlta`.
6. `MesCoincideFecha` permanece como control de calidad. Una discrepancia ya no convierte la fila en inválida ni elimina sus altas.
7. Se retiró la transformación opcional de `UnidadNegocio2`, ya que no tiene consumidores en el modelo ni el reporte. No se creó una columna ficticia.

No se modificaron la lógica ni el esquema público de `Fact_AltasTeResuelve`, las relaciones, las medidas DAX, las páginas, los visuales o la navegación. Power BI Desktop asignó únicamente metadata de identidad y tipo a la fact y a las dimensiones actualizadas.

## 3. Conciliación del consolidado

La inspección local en modo de solo lectura produjo estos controles:

| Control | Resultado |
| ------- | --------: |
| Filas | 35.073 |
| `SUM(ALTAS)` | 44.638 |
| Fecha mínima | 01/01/2026 |
| Fecha máxima | 23/09/2026 |
| Filas con discrepancia `MES`/`FECHA_ALTA` | 1 |
| Altas en filas con discrepancia | 1 |
| Altas inválidas | 0 |
| Fechas inválidas | 0 |

| Periodo derivado de `FECHA_ALTA` | `SUM(ALTAS)` |
| ------------------------------- | ------------: |
| Enero 2026 | 6.296 |
| Febrero 2026 | 5.309 |
| Marzo 2026 | 5.110 |
| Abril 2026 | 4.190 |
| Mayo 2026 | 4.171 |
| Junio 2026 | 3.700 |
| Julio 2026 | 4.519 |
| Agosto 2026 | 5.715 |
| Septiembre 2026, parcial | 5.628 |
| **Total** | **44.638** |

La única discrepancia entre `MES` y `FECHA_ALTA` conserva su alta y se asigna al periodo derivado de `FECHA_ALTA`.

## 4. Validaciones técnicas

- Una sola ruta y un solo nombre de archivo activo para ventas.
- Una sola tabla `Fact_AltasTeResuelve` registrada en el modelo.
- `Fact_AltasTeResuelve` continúa calculando su clave mensual desde `FechaAlta`.
- Cero referencias activas a `UnidadNegocio2` en `PBI/`.
- El snapshot anterior permanece únicamente en documentación histórica; no es una fuente activa.
- Agosto queda cerrado por `Periodo_Corte_Comercial = 202608`.
- Septiembre queda `En curso` por ser el máximo periodo posterior al corte.
- El archivo Excel fue inspeccionado sin escritura.
- No se incorporaron nombres personales ni datos fila a fila en esta evidencia.

## 5. Gate manual en Power BI Desktop

El usuario ejecutó el refresh, validó el resultado, guardó y cerró Power BI Desktop.

- Refresh sin errores: PASS.
- Enero a junio: conciliados con los controles R2.
- Julio: 4.519 altas, cambio de +819 y variación de +22,14 %.
- Agosto: 5.715 altas, cambio de +1.196 y variación de +26,47 %; periodo cerrado.
- Septiembre: 5.628 altas; periodo en curso.
- Página `GestionComercialAltas`: render correcto.

## 6. Revisión post-Desktop

El round-trip fue revisado archivo por archivo. Se conservaron la lógica R2 y la metadata generada por Desktop necesaria para estabilizar los objetos actualizados.

Archivos finales de R2:

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Aliado.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_AltasTeResuelve.tmdl`
- `Outputs/71_resultado_r2_migracion_consolidado_gestion_comercial_septiembre.md`

Se restauró el ruido automático de Desktop en PBIR, página activa, selección persistida, cultura Q&A, medidas y tablas no relacionadas. Home continúa como página activa y el segmentador de mes conserva julio. No hay cambios en relaciones ni expresiones DAX.

Revisión post-Desktop: PASS.

## 7. Rollback

Si se detecta una regresión, no se debe continuar a R3. El rollback consiste en revertir el commit atómico de R2 para volver al baseline `0ee66bd90426aa617dc824288261c4541346674c`; el Excel consolidado no se modifica.

## 8. Resultado

La migración mínima quedó conciliada y validada en Power BI Desktop. R2 queda en estado PASS y se versiona mediante un único commit atómico. No se inició R3.
