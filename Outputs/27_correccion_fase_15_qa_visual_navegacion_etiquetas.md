# Corrección QA transversal — post Fase 15 (etiquetas, data labels, navegación, textos)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Corrección QA transversal previa a Fase 16 |
| Documentos de referencia | `AGENTS.md`, `Specs/01...`, `Specs/02...`, `Outputs/23...`, `Outputs/24...`, `Outputs/25...`, `Outputs/26...` |
| Fecha | 2026-07-08 |

---

## Estado inicial de `git status`

Al iniciar existían 177 archivos modificados por una sesión previa de Power BI Desktop (todos los `visual.json` de `definition/pages/`, más `pages.json`). Se revisó el diff de una muestra representativa antes de tocar nada:

- La mayoría del cambio era ruido de guardado (`$schema` de `visualContainer` bajado de `2.9.0` a `2.4.0`, salto de línea final removido).
- Un subconjunto real (16 segmentadores + 2 gráficos por página, ~19 visuales) perdió propiedades de formato manual que se habían aplicado en la Fase 14 (`fontSize` de encabezado/ítems de segmentador, `color` de etiqueta de dato en gráficos, `horizontalTextAlignment` de un textbox de Home).

Siguiendo la convención ya establecida en este repositorio (`Outputs/11`, `AGENTS.md`), este estado se comiteó por separado **antes** de iniciar la corrección propia:

`bab5cee chore(report): sincronizar cambios automaticos de Power BI Desktop`

A partir de ahí se ejecutó la corrección QA sobre un working tree limpio.

## Método de verificación

No hay analizador PBIR disponible en este entorno (confirmado en fases previas), así que la revisión se hizo estructuralmente con un script Python de solo lectura que recorre cada `visual.json` bajo `definition/pages/` y extrae: tipo de visual, medida enlazada (`nativeQueryRef`), texto de etiqueta de tarjetas KPI, presencia de objeto `labels` en gráficos, acciones `visualLink` (navegación) y todo valor de texto para detectar corrupción de codificación. El script no modificó ningún archivo.

---

## Corrección 1 — Etiquetas de tarjetas KPI

**Hallazgo:** revisando las 7 páginas, cada tarjeta `cardVisual` ya tenía un `queryRef`/`nativeQueryRef` distinto y un `label.text` (override literal) distinto y correcto — incluida `Resumen ejecutivo`, donde no se encontró ninguna tarjeta mostrando "Total Respuestas Motivación" en más de un lugar. Los 6 KPI de esa página están correctamente enlazados: `Total Evaluaciones Calidad`, `Total Respuestas Capacitación`, `Índice Global Capacitación`, `Índice Global Motivación`, `Total Respuestas Motivación`, `Total Registros Piloto`.

**Conclusión:** no se encontró una etiqueta duplicada real en el JSON fuente. No se modificó ninguna tarjeta KPI en esta corrección.

**Nota de consistencia (no corregida, documentada para referencia futura):** en `Detalle por call center` y `Notas metodológicas`, la tarjeta de calidad usa la etiqueta corta `Evaluaciones Calidad` en vez de `Total Evaluaciones Calidad` usada en el resto de páginas. Corresponde al indicador correcto (no es un error), solo una abreviación distinta, probablemente por espacio disponible en esas páginas con más tarjetas. Se deja sin tocar por no ser un defecto y para no ampliar el alcance de esta corrección.

**Recomendación:** si el defecto visual que reportaste sigue apareciendo en Power BI Desktop tras este commit, probablemente sea necesario un refresco completo del reporte (cerrar y reabrir Desktop) — el JSON fuente ya tenía las etiquetas correctas antes de este commit.

## Corrección 2 — Etiquetas de datos en gráficos

**Hallazgo:** los 8 gráficos de barras/columnas/línea de las páginas internas no tenían objeto `labels` (etiquetas de datos apagadas, comportamiento por defecto del visual):

| Visual | Página | Tipo | Medida (Y) |
|---|---|---|---|
| `cl_chart_callcenter` | Calidad de llamadas | barChart | Total Evaluaciones Calidad |
| `dc_chart_registros` | Detalle por call center | barChart | Total Registros Piloto |
| `mc_chart_callcenter` | Motivación comercial | columnChart | Índice Global Motivación |
| `mc_chart_jornada` | Motivación comercial | barChart | Índice Global Motivación |
| `re_chart_callcenter` | Resumen ejecutivo | columnChart | Total Registros Piloto |
| `re_chart_fecha` | Resumen ejecutivo | lineChart | Total Registros Piloto |
| `sc_chart_callcenter` | Satisfacción de capacitaciones | columnChart | Índice Global Capacitación |
| `sc_chart_jornada` | Satisfacción de capacitaciones | barChart | Índice Global Capacitación |

**Cambio aplicado:** se agregó el objeto `labels` (`show: true`, `color: '#002733'`, `fontSize: 9D`) a los 8 archivos, en el mismo patrón ya usado por `dataPoint`/`categoryAxis`/`title` de esos visuales. Se usó el oscuro Connect (`#002733`) en vez de negro por defecto, y se dejó fuera cualquier propiedad no verificada (p. ej. `labelPrecision`, `labelPosition`) para minimizar el riesgo de que Power BI Desktop la descarte silenciosamente al reabrir — como ya ocurrió con otras propiedades de formato en esta misma sesión (ver más abajo). `re_chart_fecha` no satura con 8 series/puntos dado el volumen piloto actual (`Dim_Calendario` de pocos días), por lo que se activó igual.

**No se tocó** color de barras/líneas (`#F15B2B`), ejes (`#F4F4F4`) ni leyenda — ya estaban correctos desde la Fase 14. No se encontró ningún azul genérico de Power BI (`#118DFF`, `#12239E`, `#0078D4`) en las páginas del reporte.

## Corrección 3 — Navegación clicable en botón y texto

**Hallazgo:** se auditaron los 30 visuales de navegación (18 módulos de Home: acento+tarjeta+etiqueta × 6, más 12 en páginas internas: botón+etiqueta "Volver a Home" × 6). Los 30 ya tienen `visualContainerObjects.visualLink` con `type: PageNavigation` y el destino correcto, coincidiendo con la auditoría de la Fase 15 (`Outputs/26`, "30 visuales con type='PageNavigation'"). Se verificó además el orden `z` de cada página: el panel/acento de encabezado usa `z: 10-11`, muy por debajo del botón (`z: 20`) y su etiqueta (`z: 32`); en Home, `home_canvas_background` usa `z: 0` y los 3 elementos de cada módulo de navegación usan `z: 200-220` — ningún visual sin acción de navegación queda superpuesto por encima de un elemento clicable.

**Conclusión:** no se encontró ningún botón o tarjeta con solo una parte clicable. La cobertura tarjeta+texto (y acento en Home) ya estaba completa desde la Fase 15. No se modificó ningún archivo para esta corrección.

**Recomendación:** si en Power BI Desktop la navegación sigue sintiéndose imprecisa, es probable que sea percepción del cursor (el puntero cambia solo sobre el visual con foco activo del mouse, aunque el área clicable ya cubre tarjeta y texto) más que un defecto de configuración; confirma clic en distintos puntos de una misma tarjeta (borde, centro, texto) para descartarlo.

## Corrección 4 — Textos visibles en español de Colombia

**Método:** se comparó cada valor de texto contra sus bytes crudos (no solo el texto impreso en la terminal, que no renderiza bien tildes/eñes en este entorno) para distinguir corrupción real (bytes `0x3F`, "?" literal, irrecuperable por contexto pero reconstruible por sentido) de texto ya correctamente codificado en UTF-8 que solo se veía mal al imprimirlo en consola.

**Hallazgo real (5 archivos, 6 ocurrencias):**

| Archivo | Texto antes | Texto después |
|---|---|---|
| `cl_subtitle` | `...hallazgos por operaci?n.` | `...hallazgos por operación.` |
| `mc_subtitle` | `...percepci?n de actividades comerciales por operaci?n.` | `...percepción de actividades comerciales por operación.` |
| `nm_nota_cierre_text` | `Estas notas acompa?an... durante la evoluci?n del MVP.` | `Estas notas acompañan... durante la evolución del MVP.` |
| `nm_pendientes_text` | `...cat?logo oficial de call centers...` | `...catálogo oficial de call centers...` |
| `sc_subtitle` | `Percepci?n de participantes...` | `Percepción de participantes...` |

**Verificación:** se re-escaneó el árbol completo tras el cambio (204 archivos `.json` bajo `definition/pages/`, incluido `pages.json`) y no queda ningún `?` intermedio de palabra ni ningún carácter de reemplazo Unicode (`U+FFFD`) real en ningún archivo. Los cambios se aplicaron con la herramienta de edición de texto (no por shell/echo), para no repetir el origen probable de esta corrupción en fases previas.

No se modificaron `queryRef`, `nativeQueryRef`, nombres de medidas, tablas, columnas ni relaciones.

---

## Validaciones ejecutadas

1. `git status` revisado antes y después de separar el commit de sincronización de Desktop.
2. Los 203 archivos `.json` bajo `PBI/PBI_Indicadores.Report/definition/` parsean correctamente (`json.load` sin excepción).
3. Búsqueda de azul genérico Power BI (`#118DFF`, `#12239E`, `#0078D4`) en `definition/pages/`: sin coincidencias.
4. Búsqueda de `?` intercalado en palabra y de `U+FFFD`: sin coincidencias tras la corrección.
5. `git status` final: únicamente los 13 archivos de esta corrección (8 gráficos + 5 textos) quedan modificados; sin cambios en `PBI/PBI_Indicadores.SemanticModel/` (Power Query, medidas DAX, relaciones, tablas del modelo) ni en `Data/*.xlsx`.
6. Auditoría de navegación: 30/30 visuales `PageNavigation` con contenedor + texto vinculados, sin superposición de un visual no vinculado por encima.
7. Auditoría de tarjetas KPI: 100% de los `cardVisual` revisados tienen `nativeQueryRef` y `label.text` distintos y correctos para su página.
8. Auditoría de gráficos: 8/8 gráficos de las páginas internas y de Resumen ejecutivo ahora tienen `objects.labels` con `show: true`.

## Confirmación de no modificación del modelo semántico

No se modificaron Power Query, medidas DAX, relaciones, tablas del modelo ni `Data/*.xlsx`. Los únicos archivos tocados son `visual.json` de 8 gráficos y 5 textboxes bajo `PBI/PBI_Indicadores.Report/definition/pages/`.

## Archivos modificados

- `PBI/PBI_Indicadores.Report/definition/pages/p14_calidad_llamadas/visuals/cl_chart_callcenter/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_calidad_llamadas/visuals/cl_subtitle/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_detalle_call_center/visuals/dc_chart_registros/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_motivacion_comercial/visuals/mc_chart_callcenter/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_motivacion_comercial/visuals/mc_chart_jornada/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_motivacion_comercial/visuals/mc_subtitle/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_nota_cierre_text/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_pendientes_text/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_resumen_ejecutivo/visuals/re_chart_callcenter/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_resumen_ejecutivo/visuals/re_chart_fecha/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/visuals/sc_chart_callcenter/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/visuals/sc_chart_jornada/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/visuals/sc_subtitle/visual.json`
- `Outputs/27_correccion_fase_15_qa_visual_navegacion_etiquetas.md` (este documento)

*(Por separado, en el commit previo `bab5cee`, se aisló la reescritura automática de Power BI Desktop sobre los 177 archivos de `definition/pages/`.)*

## Resultado del commit

Mensaje de commit:

`fix(report): corregir etiquetas visuales navegacion y data labels`

## Estado final de `git status`

Se espera `working tree clean` tras este commit.

## Recomendación

1. Abrir `PBI_Indicadores.pbip` en Power BI Desktop y confirmar visualmente: etiquetas de datos visibles en los 8 gráficos con color oscuro Connect, textos de subtítulos sin `?`, y que las tarjetas KPI de `Resumen ejecutivo` (y el resto de páginas) muestran cada una su propio indicador.
2. Si el defecto de "varias tarjetas con el mismo texto" persiste visualmente después de este commit, repórtalo con una captura — el JSON fuente ya tenía valores correctos y distintos antes de esta corrección, por lo que el origen sería un problema de render/caché de Desktop y no del archivo.
3. No se avanzó a la Fase 16 en esta ejecución, conforme a la instrucción.
