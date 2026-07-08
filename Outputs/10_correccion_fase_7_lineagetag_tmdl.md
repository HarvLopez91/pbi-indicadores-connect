# Corrección — Error `lineageTag` no soportado en tablas TMDL (Fase 7)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Origen del error | Reportado por el usuario al abrir `PBI_Indicadores.pbip` en Power BI Desktop, tras la Fase 7 |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/09_resultado_fase_7_creacion_tablas_hechos.md` |
| Fecha | 2026-07-08 |
| Archivos corregidos | `tables/Fact_CalidadLlamadas.tmdl`, `tables/Fact_SatisfaccionCapacitacion.tmdl`, `tables/Fact_MotivacionActividad.tmdl` |

---

## Error encontrado

```
Error de formato TMDL
Tipo de error de análisis: UnknownKeyword
Propiedad no admitida: lineageTag no es una propiedad admitida en el contexto actual
Documento: ./tables/Fact_CalidadLlamadas
Línea: lineageTag: be71d502-6679-4269-a750-bf3d2e4e3485
```

La línea señalada corresponde específicamente al `lineageTag` de la **partición** de `Fact_CalidadLlamadas` (no de una columna ni de la tabla), tal como se había escrito en la Fase 7.

## Causa probable

`lineageTag` es una propiedad válida en el modelo de objetos tabular (TOM) y está documentada de forma general para tablas, columnas y particiones en la referencia de TMDL de Analysis Services/Fabric. Sin embargo, la función "Guardar como Power BI Project usando TMDL" en Power BI Desktop es una **característica en vista previa**, y su lector/analizador de archivos TMDL dentro de la aplicación de escritorio parece ser más estricto o tener un alcance distinto al de la especificación general de TOM/TMDL (que es la que documenta Microsoft Learn y en la que me basé al escribir los archivos en la Fase 7). Lo más probable es que esta versión de Power BI Desktop no acepte `lineageTag` escrito a mano en archivos de tabla editados externamente, al menos no en el contexto de partición donde se detectó el error.

No fue posible determinar con certeza absoluta si el problema se limitaba solo a la línea de la partición o si también habría afectado a las líneas de tabla/columna (Power BI Desktop se detiene en el primer error de parseo, por lo que no sabemos si las demás 45 líneas `lineageTag` habrían fallado igual). Por eso se siguió la instrucción de eliminarlas **todas**, no solo la señalada por el mensaje de error — es la corrección más segura y no requiere adivinar cuáles habrían funcionado y cuáles no.

`lineageTag` es metadato opcional usado por TOM para rastrear la identidad de un objeto a través de refactorizaciones (renombrados, etc.); si se omite, Power BI Desktop genera uno automáticamente la primera vez que guarda el modelo. Eliminarlo por completo no tiene impacto funcional en el modelo.

## Archivos corregidos

- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_CalidadLlamadas.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_MotivacionActividad.tmdl`

## Cambio aplicado

Se eliminaron **todas** las líneas `lineageTag:` de los 3 archivos (tabla, columnas y partición), sin tocar ninguna otra línea:

| Archivo | Líneas `lineageTag:` eliminadas |
|---|---|
| `Fact_CalidadLlamadas.tmdl` | 19 (1 tabla + 17 columnas + 1 partición) |
| `Fact_SatisfaccionCapacitacion.tmdl` | 15 (1 tabla + 13 columnas + 1 partición) |
| `Fact_MotivacionActividad.tmdl` | 12 (1 tabla + 10 columnas + 1 partición) |
| **Total** | **46** |

Confirmado con `git diff --stat`: los 3 archivos muestran únicamente líneas eliminadas (`46 deletions(-)`, `0 insertions(+)`), sin ninguna línea agregada o modificada.

**No se tocó:**
- Ninguna columna (las 17 + 13 + 10 columnas siguen presentes, con `dataType`, `summarizeBy` y `sourceColumn` intactos).
- Ninguna partición (las 3 particiones `mode: import` con su código M siguen presentes).
- El código M de las particiones (verificado, sin cambios).
- `expressions.tmdl` (no fue necesario tocarlo).
- `model.tmdl` (las 3 líneas `ref table Fact_CalidadLlamadas` / `ref table Fact_SatisfaccionCapacitacion` / `ref table Fact_MotivacionActividad` se verificaron intactas, sin relación con este error).

## Validación realizada

- **Búsqueda global** de `lineageTag` dentro de `definition/tables/`: **0 coincidencias** en los 3 archivos.
- **Balance de paréntesis y llaves** tras el cambio: `Fact_CalidadLlamadas.tmdl` (4/4, 36/36), `Fact_SatisfaccionCapacitacion.tmdl` (25/25, 39/39), `Fact_MotivacionActividad.tmdl` (12/12, 26/26) — idéntico al balance de la Fase 7, ya que solo se removieron líneas completas de propiedad, sin afectar ningún paréntesis o llave.
- **Conteo de columnas** tras el cambio: 17, 13, 10 — sin cambios respecto a la Fase 7.
- **`model.tmdl`**: se confirmó que las 3 referencias `ref table` siguen presentes y sin modificar.

## Confirmación de apertura en Power BI Desktop

**No pude abrir Power BI Desktop yo mismo** (limitación ya documentada repetidamente en este proyecto: no tengo control sobre la interfaz gráfica de la aplicación desde este entorno). Las validaciones anteriores son estructurales (texto, sintaxis, balance), no una ejecución real del parser de Power BI Desktop.

**Acción requerida de tu parte:** cerrar y volver a abrir Power BI Desktop (recuerda que las ediciones externas al TMDL requieren reinicio de la aplicación para recargarse) y confirmar si el error de `lineageTag` desaparece. Si aparece un error TMDL distinto, repórtalo literalmente (mensaje completo, documento y línea señalados) para diagnosticarlo puntualmente — tal como pediste, no se intentarán correcciones adicionales a ciegas sin ese reporte.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 8

**No avanzar a la Fase 8.** Antes de continuar:
1. Confirmar en Power BI Desktop que el error de `lineageTag` ya no aparece.
2. Si el PBIP abre limpio, validar además lo que quedó pendiente de la Fase 7: que las 3 tablas `Fact_*` aparecen en el panel de datos con vista previa sin error, con los conteos de fila esperados (3/32/5) y los tipos de columna correctos.
3. Si aparece un error distinto, repórtalo con el texto exacto para corregirlo de forma dirigida, sin intentar múltiples cambios especulativos a la vez.

---

*Documento generado como registro operativo de la corrección de la Fase 7, según la regla documental vigente: los resultados de ejecución/corrección de fases se documentan en `Outputs/`.*
