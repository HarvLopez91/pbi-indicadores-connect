# Sincronización — etiquetas de datos manuales y cambios automáticos de Power BI Desktop

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo | Sincronización previa a Fase 17 (Parte 0) |
| Fecha | 2026-07-09 |
| Alcance | Revisión y sincronización de cambios generados por una sesión del usuario en Power BI Desktop. No se modificó ningún archivo manualmente en esta tarea; solo se revisó y comiteó lo ya escrito por Desktop. |

## 1. `git status` al iniciar

Antes de comitear, `git status` mostraba 20 archivos modificados (ningún archivo nuevo ni eliminado):

```
modified:   pages/p14_calidad_llamadas/visuals/cl_chart_callcenter/visual.json
modified:   pages/p14_calidad_llamadas/visuals/cl_tabla_asesor/visual.json
modified:   pages/p14_detalle_call_center/visuals/dc_chart_registros/visual.json
modified:   pages/p14_detalle_call_center/visuals/dc_tabla_callcenter/visual.json
modified:   pages/p14_motivacion_comercial/visuals/mc_chart_callcenter/visual.json
modified:   pages/p14_motivacion_comercial/visuals/mc_chart_jornada/visual.json
modified:   pages/p14_motivacion_comercial/visuals/mc_tabla_ambiente/visual.json
modified:   pages/p14_notas_metodologicas/visuals/nm_n_calidad/visual.json
modified:   pages/p14_notas_metodologicas/visuals/nm_n_cap/visual.json
modified:   pages/p14_notas_metodologicas/visuals/nm_n_mot/visual.json
modified:   pages/p14_resumen_ejecutivo/visuals/re_chart_callcenter/visual.json
modified:   pages/p14_resumen_ejecutivo/visuals/re_chart_fecha/visual.json
modified:   pages/p14_satisfaccion_capacitaciones/visuals/sc_chart_callcenter/visual.json
modified:   pages/p14_satisfaccion_capacitaciones/visuals/sc_chart_jornada/visual.json
modified:   pages/p14_satisfaccion_capacitaciones/visuals/sc_tabla_formador/visual.json
modified:   pages/pages.json
modified:   PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl
modified:   PBI_Indicadores.SemanticModel/definition/tables/Fact_CalidadLlamadas.tmdl
modified:   PBI_Indicadores.SemanticModel/definition/tables/Fact_MotivacionActividad.tmdl
modified:   PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl
```

No hubo cambios en `report.json` ni en `Data/*.xlsx`.

## 2. Revisión detallada por grupo de cambios

### 2.1 Etiquetas de datos activadas manualmente (8 gráficos)

Se revisó el diff completo de los 8 visuales de tipo gráfico (`barChart`/`columnChart`/`lineChart`). En los 8 casos, Power BI Desktop agregó una segunda entrada al arreglo `objects.labels` con `show: true` (sin `selector`), adicional a la entrada ya existente (`selector: {id: "default"}`, color `#002733`, tamaño `9D`). Esto corresponde exactamente a la reconfirmación manual de "Etiquetas de datos" que el usuario activó desde el panel de formato de Power BI Desktop. **No se sobrescribió ni se cambió el color/tamaño ya configurado**; la entrada nueva es aditiva.

Además, en 5 de los 8 gráficos el usuario aplicó manualmente un orden (`sortDefinition`) por la medida principal del gráfico, en orden descendente:

| Visual | Página | Medida de orden agregada |
|---|---|---|
| `cl_chart_callcenter` | Calidad de llamadas | `Total Evaluaciones Calidad` (descendente) |
| `dc_chart_registros` | Detalle por call center | `Total Registros Piloto` (descendente) |
| `mc_chart_callcenter` | Motivación comercial | `Indice Global Motivacion` (descendente) |
| `re_chart_callcenter` | Resumen ejecutivo | `Total Registros Piloto` (descendente) |
| `sc_chart_jornada` | Satisfacción de capacitaciones | `Indice Global Capacitacion` (descendente) |

`mc_chart_jornada`, `re_chart_fecha` y `sc_chart_callcenter` solo recibieron la reconfirmación de etiquetas, sin cambio de orden.

**Decisión: se conservan tal cual.** No se revirtió ni se sobrescribió ninguna etiqueta ni el orden manual aplicado por el usuario.

### 2.2 Reformato de tablas (4 visuales `tableEx`)

`cl_tabla_asesor`, `dc_tabla_callcenter`, `mc_tabla_ambiente` y `sc_tabla_formador` muestran diffs grandes (de 235 a 306 líneas), pero corresponden a una reorganización de propiedades de formato de tabla hecha desde Power BI Desktop: encabezados de columna con fondo `#002733` y texto blanco en negrita, valores con texto `#002733`, cuadrícula horizontal `#F4F4F4`, alineados a la paleta Connect. Se verificó explícitamente que **ninguna línea de la sección `query`/`projections` (bindings de columnas y medidas) cambió** en los 4 archivos — solo se reorganizaron propiedades visuales entre `objects` (grid, columnHeaders, values) y `visualContainerObjects` (border, title, padding, visualHeader), sin alterar ningún dato ni campo mostrado.

**Decisión: se conservan.** Es una mejora de formato manual del usuario, alineada con la marca Connect, sin riesgo funcional.

### 2.3 Cambios cosméticos menores (`nm_n_*`, `pages.json`)

- `nm_n_calidad`, `nm_n_cap`, `nm_n_mot`: Desktop eliminó el salto de línea final (mismo patrón ya documentado en `Outputs/28` y `Outputs/29`).
- `pages.json`: actualizó `activePageName` a `p14_resumen_ejecutivo` (última página abierta en la sesión de Desktop).

**Decisión: se conservan.** Sin impacto de contenido.

### 2.4 Metadatos automáticos del modelo semántico

- **`Fact_CalidadLlamadas.tmdl`, `Fact_MotivacionActividad.tmdl`, `Fact_SatisfaccionCapacitacion.tmdl`**: Desktop agregó la anotación `annotation PBI_ResultType = Table` al final de cada tabla (metadato estándar de tipo de resultado de partición, generado automáticamente al abrir/guardar). No se tocó ninguna expresión de Power Query, ninguna columna ni ningún tipo de dato.
- **`cultures/es-ES.tmdl`**: Desktop agregó 3 bloques de metadatos lingüísticos (`PowerBI.VisualColumnRename`, `State: Suggested`) para las medidas `n Calidad`, `n Capacitacion` y `n Motivacion` — generados automáticamente porque esas 3 medidas se usaron por primera vez en visuales durante la Fase 16 (`Outputs/29`). Es el mismo mecanismo ya documentado en `Outputs/23` (Fase 14) para otras columnas.

**Decisión: se conservan.** Son metadatos generados automáticamente por Power BI Desktop al abrir el modelo, no ediciones de Power Query, DAX, relaciones ni tablas en el sentido de lógica de negocio — coherente con la regla ya establecida en `AGENTS.md` y `CLAUDE.md` de no escribir `lineageTag`/`description`/`queryGroup` a mano, pero sí aceptar y sincronizar lo que Desktop genera.

## 3. Cambios NO relacionados con etiquetas/formato

No se encontró ningún cambio en `report.json`, en las consultas Power Query (`expressions.tmdl`), en `relationships.tmdl`, ni en `Data/*.xlsx`.

## 4. Confirmación de restricciones

- No se modificó Power Query.
- No se modificó ninguna fórmula DAX (solo metadatos lingüísticos autogenerados, sin tocar la definición de las medidas).
- No se modificaron relaciones.
- No se modificó la estructura de ninguna tabla del modelo (solo se agregó una anotación estándar de Desktop).
- No se tocó `Data/*.xlsx`.
- No se revirtió ni se sobrescribió ninguna etiqueta de datos ni orden aplicado manualmente por el usuario.
- No se hizo push.

## 5. Resultado del commit

Se comitearon los 20 archivos revisados en un solo commit:

`chore(report): sincronizar etiquetas de datos manuales`

## 6. Estado final de `git status`

Tras el commit, `git status` queda limpio (`working tree clean`), listo para iniciar la Fase 17.
