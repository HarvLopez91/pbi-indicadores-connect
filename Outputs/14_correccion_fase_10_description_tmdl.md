# Corrección — Error `description` no soportado en medidas TMDL (Fase 10)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Origen del error | Reportado por el usuario al abrir `PBI_Indicadores.pbip` en Power BI Desktop, tras la Fase 10 |
| Documentos de referencia | `Outputs/13_resultado_fase_10_medidas_dax.md` |
| Fecha | 2026-07-08 |
| Archivos corregidos | `tables/_Medidas Generales.tmdl`, `tables/_Medidas Calidad.tmdl`, `tables/_Medidas Capacitacion.tmdl`, `tables/_Medidas Motivacion.tmdl` |

---

## Error encontrado

```
Error de formato TMDL
Tipo de error de análisis: UnknownKeyword
Propiedad no admitida: description no es una propiedad admitida en el contexto actual
Documento: ./tables/_Medidas Calidad
Línea: description: "Suma de los puntos obtenidos en las 8 preguntas del checklist"
```

## Causa probable

`description` es una propiedad válida y bien documentada del modelo de objetos tabular (TOM) — heredada de la clase base `MetadataObject`, aplicable en general a tablas, columnas y medidas. En la Fase 10 se consideró de bajo riesgo precisamente por ser una propiedad universal de texto (a diferencia de `lineageTag`, una propiedad de identidad/referencia que ya había fallado en la Fase 7). Sin embargo, el resultado muestra que el analizador TMDL en vista previa de esta versión de Power BI Desktop **tampoco acepta `description` en el contexto de una medida** escrita a mano en un archivo `.tmdl` externo, igual que ya ocurrió con `lineageTag`. Esto confirma un patrón: el analizador de esta característica en vista previa es más estricto que la especificación general de TOM/TMDL para varias propiedades de metadatos (no solo `lineageTag`), al menos cuando se editan los archivos fuera de la aplicación.

## Archivos corregidos

- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Generales.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Calidad.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Motivacion.tmdl`

## Cambio aplicado

Se eliminaron **todas** las líneas `description:` de los 4 archivos (una por cada medida), sin tocar ninguna otra línea. No se aplicó una eliminación selectiva (solo la línea señalada por el error) por la misma razón que en el incidente de `lineageTag`: Power BI Desktop se detiene en el primer error de parseo, por lo que no hay forma de saber si las otras 24 líneas `description` también habrían fallado — es más seguro asumir que sí y quitarlas todas.

**No se tocó:**
- Ninguna fórmula DAX (verificado con `git diff --stat`: 0 inserciones, solo eliminaciones).
- Ningún `formatString`.
- Ninguna medida, tabla de medidas, columna placeholder ni partición.
- `model.tmdl` (las 4 referencias `ref table '_Medidas ...'` no se modificaron).
- `relationships.tmdl` (sin cambios, no relacionado con este error).

## Cantidad de líneas `description:` eliminadas

| Archivo | Líneas `description:` eliminadas |
|---|---|
| `_Medidas Generales.tmdl` | 7 |
| `_Medidas Calidad.tmdl` | 6 |
| `_Medidas Capacitacion.tmdl` | 6 |
| `_Medidas Motivacion.tmdl` | 6 |
| **Total** | **25** |

Confirmado con `git diff --stat`: los 4 archivos muestran exactamente `25 deletions(-)` y `0 insertions(+)` en total.

## Validación de que las 25 medidas siguen existiendo

- Búsqueda global de `description:` en los 4 archivos: **0 coincidencias**.
- Conteo de medidas por archivo (patrón `measure '`): `_Medidas Generales` = 7, `_Medidas Calidad` = 6, `_Medidas Capacitacion` = 6, `_Medidas Motivacion` = 6 — **25 en total, sin cambios respecto a la Fase 10**.
- `model.tmdl` conserva las 4 referencias: `ref table '_Medidas Generales'`, `ref table '_Medidas Calidad'`, `ref table '_Medidas Capacitacion'`, `ref table '_Medidas Motivacion'`.
- Balance de paréntesis/llaves por archivo tras el cambio: `_Medidas Generales.tmdl` (4/4, 1/1), `_Medidas Calidad.tmdl` (34/34, 2/2), `_Medidas Capacitacion.tmdl` (10/10, 1/1), `_Medidas Motivacion.tmdl` (13/13, 1/1) — consistente, sin desbalances introducidos por la eliminación de líneas completas.
- `grep` de `lineageTag` y `queryGroup` en los 4 archivos: **0 coincidencias** en ambos casos (no se reintrodujo ninguno de los dos incidentes previos).

## Confirmación de apertura en Power BI Desktop

**No pude abrir Power BI Desktop yo mismo** (limitación ya documentada repetidamente en este proyecto — no controlo su interfaz gráfica desde este entorno). La validación realizada es estructural: se confirmó que el texto exacto señalado por el error (`description:`) ya no existe en ninguno de los 4 archivos, y que la sintaxis restante permanece balanceada.

**Acción requerida de tu parte:** cerrar y volver a abrir Power BI Desktop (recordando que las ediciones externas al TMDL requieren reiniciar la aplicación para recargarse) y confirmar si el error de `description` desaparece. Si aparece un error TMDL distinto, repórtalo con el texto literal completo — tal como se pidió, no se intentarán correcciones adicionales especulativas sin ese reporte.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 11

**No avanzar a la Fase 11.** Antes de continuar:
1. Confirmar en Power BI Desktop que el error de `description` ya no aparece.
2. Si el PBIP abre limpio, retomar la validación que ya quedaba pendiente desde la Fase 10: confirmar que las 25 medidas calculan sin error y comparar sus valores contra los esperados documentados en `Outputs/13_resultado_fase_10_medidas_dax.md`.
3. Si aparece un error distinto, repórtalo con el texto exacto para corregirlo de forma dirigida.

**Nota para futuras fases:** dado que ya van dos propiedades de metadatos (`lineageTag`, `description`) documentadas en TOM/TMDL general pero rechazadas por este analizador en vista previa al escribirse a mano, se recomienda evitar por defecto cualquier propiedad de metadatos "descriptiva" adicional (anotaciones, etc.) en archivos `.tmdl` editados externamente, y limitarse a las propiedades estrictamente estructurales ya validadas como seguras en este proyecto (`dataType`, `formatString`, `summarizeBy`, `sourceColumn`, `isHidden`, `mode`, `source`, `fromColumn`/`toColumn`, `joinOnDateBehavior`), salvo que Power BI Desktop las agregue automáticamente por su cuenta.

---

*Documento generado como registro operativo de la corrección de la Fase 10, según la regla documental vigente: los resultados de ejecución/corrección de fases se documentan en `Outputs/`.*
