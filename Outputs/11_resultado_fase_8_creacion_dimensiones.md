# Resultado — Fase 8: Creación de dimensiones

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 8 — Creación de dimensiones (ver `Specs/02_plan_implementacion_informe_powerbi_connect.md`) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/09...`, `Outputs/10...` |
| Fecha | 2026-07-08 |
| Archivos nuevos | `tables/Dim_Calendario.tmdl`, `tables/Dim_CallCenter.tmdl`, `tables/Dim_Jornada.tmdl` |
| Archivo modificado | `model.tmdl` |

---

## Hallazgo previo importante: Power BI Desktop había modificado el modelo por su cuenta

Antes de tocar nada de esta fase, `git status` mostró el working tree **no limpio**, con cambios que yo no había hecho: Power BI Desktop, al abrir el PBIP corregido (Fase 7 + corrección de `lineageTag`), reescribió varios archivos automáticamente. Se investigó cada cambio antes de continuar (no se sobrescribió nada a ciegas) y se confirmó que todo era metadato/comportamiento estándar, sin alterar ninguna transformación de negocio:

- **`lineageTag` propio** en cada tabla/columna de `Fact_*` — en un formato que sí es aceptado por el parser (a diferencia del que yo había escrito a mano en la Fase 7, que causó el error corregido en `Outputs/10`).
- **`summarizeBy` cambió de `none` a `sum`** en las 8 columnas `Preg_*`, con `annotation SummarizationSetBy = Automatic` — comportamiento por defecto de Power BI Desktop al cargar columnas numéricas sin una marca explícita de "elegido por el usuario". Queda como observación para una fase de medidas futura; no se corrigió en esta fase (fuera de alcance).
- **Función "Auto Date/Time"**: Power BI Desktop creó automáticamente 4 tablas ocultas (`DateTableTemplate_...`, y un `LocalDateTable_...` por cada columna `Fecha` de las 3 `Fact_*`) y 3 relaciones en un nuevo archivo `relationships.tmdl`, con jerarquía de fechas automática. Es el comportamiento estándar de Power BI Desktop ante columnas de fecha sin una tabla de calendario explícita — precisamente lo que esta fase resuelve al crear `Dim_Calendario`. **Quedará pendiente, en una fase futura**, evaluar deshabilitar "Auto Date/Time" y eliminar estas tablas ocultas una vez `Dim_Calendario` tenga relaciones reales con las 3 `Fact_*` (eso es trabajo de la Fase 9, no de esta).
- **`annotation PBI_QueryOrder`** agregada en `model.tmdl` (orden de consultas en el panel de Power Query).

Este estado se comiteó por separado (commit `97be1a4`, `chore(modelo): sincronizar cambios automaticos de Power BI Desktop`) **antes** de empezar el trabajo propio de esta fase, para no mezclar cambios de Power BI Desktop con los de la Fase 8 en un mismo commit.

## Estado inicial de `git status` (para la Fase 8 en sí)

Tras el commit de sincronización anterior, el working tree quedó limpio antes de crear las 3 dimensiones. Se verificó de nuevo justo antes de escribir los archivos nuevos.

## Verificación de prerrequisitos

- `model.tmdl` contiene `ref table Fact_CalidadLlamadas`, `ref table Fact_SatisfaccionCapacitacion` y `ref table Fact_MotivacionActividad` — confirmado.
- Los archivos `tables/Fact_CalidadLlamadas.tmdl`, `tables/Fact_SatisfaccionCapacitacion.tmdl` y `tables/Fact_MotivacionActividad.tmdl` existen — confirmado.

## Dimensiones creadas

| Tabla | Archivo | Columnas | Origen de los datos |
|---|---|---|---|
| `Dim_Calendario` | `tables/Dim_Calendario.tmdl` | 11 | Calculada (`List.Dates`) desde el mínimo de `Fecha` en las 3 `Fact_*` hasta hoy |
| `Dim_CallCenter` | `tables/Dim_CallCenter.tmdl` | 1 | Unión dinámica de `CallCenter` distinto en las 3 `Fact_*` |
| `Dim_Jornada` | `tables/Dim_Jornada.tmdl` | 1 | Unión dinámica de `Jornada` distinto en `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` (sin `Fact_CalidadLlamadas`, que no tiene jornada) |

## Lógica usada para cada dimensión

### `Dim_Calendario`
1. `FechaMinCalidad`/`FechaMinCapacitacion`/`FechaMinMotivacion` — `List.Min` de la columna `Fecha` de cada `Fact_*`.
2. `FechaMinima` — el mínimo de los 3 anteriores.
3. `FechaMaxima` — `Date.From(DateTime.LocalNow())` (**hoy**, no el máximo observado en los datos). **Decisión documentada**: se eligió "hoy" en vez del máximo histórico porque el proyecto está en fase piloto con datos que crecerán en cada actualización; anclar el calendario a "hoy" evita que el calendario quede corto la próxima vez que se actualice el modelo con filas nuevas, sin necesidad de rehacer esta tabla. Efecto: el calendario se auto-extiende un día cada vez que se actualiza el modelo.
4. `List.Dates` genera la lista continua de fechas entre el mínimo y hoy (ambos incluidos).
5. Se agregan las 10 columnas derivadas pedidas (`Anio`, `MesNumero`, `MesNombre`, `AnioMes`, `Trimestre`, `SemanaAnio`, `DiaMes`, `DiaSemanaNumero`, `DiaSemanaNombre`, `EsFinDeSemana`) con funciones nativas de Power Query (`Date.Year`, `Date.Month`, `Date.MonthName`, etc.), usando la cultura `es-CO` para nombres de mes/día en español, consistente con `sourceQueryCulture` del modelo.
6. `DiaSemanaNumero` usa `Day.Monday` como referencia (1 = lunes … 7 = domingo, convención ISO-like). `EsFinDeSemana` es verdadero para sábado y domingo.

**Sobre "marcar como tabla de fechas" (instrucción 19):** no se intentó hacerlo desde TMDL. La propiedad relevante en TOM para esto no quedó confirmada con certeza en la documentación oficial que revisé, y dado el incidente reciente de `lineageTag` (una propiedad válida en general que causó un error de apertura por cómo se escribió a mano), preferí no arriesgar otro error especulando con una propiedad de la que no tengo total certeza de sintaxis/contexto. **Queda pendiente marcarla manualmente en Power BI Desktop:** pestaña Modelado → seleccionar `Dim_Calendario` → "Marcar como tabla de fechas" → columna `Fecha`.

### `Dim_CallCenter`
1. Se seleccina solo la columna `CallCenter` de cada una de las 3 `Fact_*` (`Table.SelectColumns`).
2. Se combinan las 3 en una sola tabla (`Table.Combine`).
3. Se filtran nulos reales (`<> null`) — no se filtran los valores `"Sin dato"` que dejó la Fase 6, porque no son nulos técnicos sino un valor de negocio válido.
4. `Table.Distinct` elimina duplicados.
5. `Table.Sort` ordena alfabéticamente.
6. No se usó ninguna lista fija — la tabla se reconstruye dinámicamente en cada actualización.

### `Dim_Jornada`
Misma lógica que `Dim_CallCenter`, pero combinando solo `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` (`Fact_CalidadLlamadas` no tiene columna `Jornada`, tal como señala el diagnóstico original).

## Columnas creadas por dimensión

**`Dim_Calendario`**: `Fecha` (dateTime, formato fecha corta), `Anio` (int64), `MesNumero` (int64), `MesNombre` (string), `AnioMes` (string), `Trimestre` (int64), `SemanaAnio` (int64), `DiaMes` (int64), `DiaSemanaNumero` (int64), `DiaSemanaNombre` (string), `EsFinDeSemana` (boolean).

**`Dim_CallCenter`**: `CallCenter` (string).

**`Dim_Jornada`**: `Jornada` (string).

## Conteo de filas por dimensión

Recalculado sobre los datos reales actuales (mismo método usado en fases anteriores: releer `Data/` con Python y simular la lógica M, ya que no puedo ejecutar Power BI Desktop directamente):

| Dimensión | Filas esperadas | Detalle |
|---|---|---|
| `Dim_Calendario` | **7** | Fecha mínima real en los 3 `Fact_*`: `2026-07-02`. Fecha máxima (hoy, al momento de esta ejecución): `2026-07-08`. 7 días continuos. *(Este número crecerá automáticamente día a día por el uso de `DateTime.LocalNow()`.)* |
| `Dim_CallCenter` | **5** | `CAPITALS`, `INTERACTIVO`, `MILLENIUM`, `ONE CONTACT`, `Sin dato` |
| `Dim_Jornada` | **3** | `Mañana`, `Sin dato`, `Tarde` |

## Validación de valores esperados

Confirmado exactamente contra lo indicado en la instrucción:

- **`Dim_CallCenter`**: contiene los 4 call centers observados (`ONE CONTACT`, `CAPITALS`, `INTERACTIVO`, `MILLENIUM`) **más** `"Sin dato"` — presente porque la Fase 6 sustituyó el único valor nulo de `CallCenter` en `Fact_SatisfaccionCapacitacion` por ese texto. ✅
- **`Dim_Jornada`**: contiene `Mañana`, `Tarde` **y** `"Sin dato"` — por la misma razón (1 fila con `Jornada` nula en `Fact_SatisfaccionCapacitacion`, sustituida en Fase 6). ✅
- **`Dim_Calendario`**: fechas continuas confirmadas por construcción (`List.Dates` con paso de 1 día, sin huecos posibles).

## Confirmación de ausencia de `lineageTag` y `queryGroup`

- Búsqueda de `lineageTag` en los 3 archivos nuevos: **0 coincidencias**.
- Búsqueda de `queryGroup` en los 3 archivos nuevos: **0 coincidencias**.
- Las 3 particiones tienen `mode: import` (confirmado, 1 ocurrencia cada una).
- Balance de paréntesis/llaves por archivo: `Dim_Calendario.tmdl` (33/33, 6/6), `Dim_CallCenter.tmdl` (8/8, 8/8), `Dim_Jornada.tmdl` (7/7, 7/7).

## Archivos modificados en el PBIP

- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_CallCenter.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Jornada.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (modificado: 3 líneas `ref table` agregadas).

*(Adicionalmente, en un commit previo y separado (`97be1a4`) se incorporaron los cambios automáticos que Power BI Desktop ya había aplicado al modelo — ver sección de hallazgo previo arriba.)*

## Errores encontrados y solución aplicada

- No se encontraron errores durante la construcción de esta fase.
- Se aplicó deliberadamente la lección de la Fase 7: **ninguna línea `lineageTag` ni `queryGroup`** en los archivos nuevos, para evitar repetir alguno de los dos incidentes ya corregidos en este proyecto.
- No fue necesario detenerse por ninguna referencia de columna inválida — se verificaron los nombres exactos (`Fecha`, `CallCenter`, `Jornada`) contra lo que realmente producen las 3 `Fact_*` tras las Fases 4-7.

## Resultado del commit

- Mensaje: `feat(modelo): crear dimensiones compartidas`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_CallCenter.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Jornada.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (modificado), `Outputs/11_resultado_fase_8_creacion_dimensiones.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 9

**Antes de avanzar:**
1. Cerrar y volver a abrir Power BI Desktop y confirmar que el PBIP abre sin errores.
2. Confirmar que `Dim_Calendario`, `Dim_CallCenter` y `Dim_Jornada` aparecen en el panel de datos, con los conteos de fila indicados arriba (7 / 5 / 3 — recordando que el de `Dim_Calendario` crecerá con el tiempo).
3. Confirmar visualmente los valores de `Dim_CallCenter` y `Dim_Jornada` contra la tabla de "Validación de valores esperados".
4. Opcional pero recomendado: marcar `Dim_Calendario` como tabla de fechas manualmente (Modelado → Marcar como tabla de fechas → columna `Fecha`), ya que no se hizo desde TMDL en esta fase.

Si la validación es exitosa, el proyecto queda listo para la **Fase 9 — Diseño del modelo de datos y relaciones**, donde además de crear las relaciones `Dim_* → Fact_*` sería razonable revisar si conviene deshabilitar "Auto Date/Time" y limpiar las tablas ocultas `LocalDateTable_*`/`DateTableTemplate_*` que Power BI Desktop generó automáticamente, ya que `Dim_Calendario` las vuelve redundantes. **No se avanzó a la Fase 9 en esta ejecución**, conforme a la restricción indicada.

---

*Documento generado como registro operativo de la Fase 8, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
