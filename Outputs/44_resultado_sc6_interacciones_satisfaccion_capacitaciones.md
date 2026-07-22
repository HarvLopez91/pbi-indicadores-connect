# Resultado SC-6 - Interacciones de Satisfacción de capacitaciones

## Estado inicial de Git

- Raíz detectada: `C:/Users/eclavijo/OneDrive/PBI_Indicadores`.
- Rama: `main`.
- Commit base confirmado: `d4b58fd feat(report): redisenar satisfaccion de capacitaciones`.
- Power BI Desktop cerrado al iniciar la revisión.
- Estado inicial: sin cambios en `PBI/`; solo archivo no rastreado en `Data/Informe de Altas/`.
- Página original `p14_satisfaccion_capacitaciones`: sin diferencias antes de modificar.

## Inventario real de visuales

| Visual | Nombre técnico | Tipo |
| --- | --- | --- |
| Segmentador Fecha | `sc_slicer_fecha` | `slicer` |
| Segmentador Call Center | `sc_slicer_callcenter` | `slicer` |
| Segmentador Jornada | `sc_slicer_jornada` | `slicer` |
| KPI Capacitaciones realizadas | `sc_kpi_capacitaciones` | `cardVisual` |
| KPI Respuestas recibidas | `sc_kpi_respuestas` | `cardVisual` |
| KPI Call centers capacitados | `sc_kpi_callcenters` | `cardVisual` |
| KPI Satisfacción promedio | `sc_kpi_satisfaccion` | `cardVisual` |
| KPI Última capacitación | `sc_kpi_ultima` | `cardVisual` |
| KPI % con comentarios | `sc_kpi_comentarios` | `cardVisual` |
| Gráfico Capacitaciones por call center | `sc_chart_callcenter` | `columnChart` |
| Gráfico Capacitaciones por fecha | `sc_chart_capxfecha` | `lineChart` |
| Panel de satisfacción | `sc_panel_satisf_chart` | `barChart` |
| Tarjeta Respuestas del panel | `sc_kpi_respuestas_panel` | `cardVisual` |
| Tabla Detalle por call center | `sc_tabla_callcenter` | `tableEx` |
| Tabla Comentarios destacados | `sc_tabla_comentarios` | `tableEx` |
| Botón Home | `sc_home_btn` | `shape` |
| Etiqueta Home | `sc_home_label` | `textbox` |
| Hitzone Home | `sc_home_hitzone` | `shape` |

## Estructura PBIR utilizada

No se encontró un precedente local ya persistido de `visualInteractions` en otras páginas del informe.

Se utilizó el bloque soportado por el esquema PBIR de página `2.0.0`:

- Propiedad: `visualInteractions`.
- Campos por interacción: `source`, `target`, `type`.
- Tipos válidos aplicados: `DataFilter` y `NoFilter`.
- Referencia de esquema revisada: `https://developer.microsoft.com/json-schemas/fabric/item/report/definition/page/2.0.0/schema.json`.

No se modificaron visuales individuales, campos, medidas, formato ni layout.

## Matriz de interacciones configurada

| Visual origen | Visual destino | Interacción |
| --- | --- | --- |
| `sc_slicer_fecha` | 6 KPI, gráficos, panel, respuestas del panel, detalle y comentarios | `DataFilter` |
| `sc_slicer_callcenter` | 6 KPI, gráficos, panel, respuestas del panel, detalle y comentarios | `DataFilter` |
| `sc_slicer_jornada` | 6 KPI, gráficos, panel, respuestas del panel, detalle y comentarios | `DataFilter` |
| `sc_chart_callcenter` | 6 KPI, gráfico por fecha, panel, respuestas del panel, detalle y comentarios | `DataFilter` |
| `sc_chart_capxfecha` | 6 KPI, gráfico por call center, panel, respuestas del panel, detalle y comentarios | `DataFilter` |
| `sc_tabla_callcenter` | 6 KPI, gráfico por fecha, panel, respuestas del panel y comentarios | `DataFilter` |
| `sc_tabla_comentarios` | Resto de visuales de datos | `NoFilter` |
| `sc_panel_satisf_chart` | Resto de visuales de datos | `NoFilter` |
| `sc_kpi_respuestas_panel` | Resto de visuales de datos | `NoFilter` |
| KPI superiores | Resto de visuales de datos | `NoFilter` |

Notas:

- No se configuraron interacciones hacia objetos decorativos, encabezados, contenedores, notas ni navegación.
- No se configuró al gráfico de call center como destino de la tabla de detalle para evitar que una selección de fila de detalle deforme el gráfico categórico principal.
- No se configuraron interacciones de origen hacia el mismo visual.

## Navegación Home verificada

Los visuales `sc_home_btn`, `sc_home_label` y `sc_home_hitzone` conservan:

- Acción `PageNavigation`.
- Destino `67eff42d82e1c9c15b84`, correspondiente a Home.
- Tooltip `Volver a Home`.
- `sc_home_hitzone` mantiene la misma posición y tamaño del botón: `x=1084`, `y=30`, `width=150`, `height=36`.
- `sc_home_hitzone` mantiene `z=40`, por encima del botón y debajo del contenido no interactivo posterior.

No se modificó Home ni se enlazó la página `v2` desde Home.

## Archivos modificados

- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json`
- `Outputs/44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md`

## Validaciones estáticas

- `page.json` de `p14_satisfaccion_capacitaciones_v2` parsea como JSON válido.
- Las interacciones usan únicamente tipos permitidos por el esquema revisado: `DataFilter` y `NoFilter`.
- Todos los `source` y `target` referencian visuales existentes en la página `v2`.
- No se modificaron `visual.json`; por tanto, no hubo cambios de diseño, tamaño, posición, colores, títulos, campos ni medidas.
- Página original `p14_satisfaccion_capacitaciones`: sin diferencias.
- Home: sin diferencias.
- Modelo semántico, Power Query y relaciones: sin diferencias.
- `Data/` continúa sin seguimiento.
- No se agregaron medidas, columnas, tablas, bookmarks, tooltips, drillthrough ni páginas adicionales.

## Riesgos pendientes

- Aunque la estructura PBIR es válida, Power BI Desktop debe confirmar el comportamiento renderizado de cada interacción.
- Los slicers suelen operar como filtros aun sin `visualInteractions`; la configuración explícita se dejó para documentar el comportamiento esperado.
- La interacción de selección en `tableEx` debe validarse visualmente porque Desktop puede tratar algunas tablas como receptoras robustas pero origen limitado según versión.

## Estado final de Git

- Cambios rastreados:
  - `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json`
- Archivos nuevos no rastreados:
  - `Outputs/44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md`
  - `Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx`
- `git diff --stat` reporta cambios solo en `page.json`; la bitácora aparece en `git status` por ser archivo nuevo no rastreado.
- No se creó commit.

## Instrucciones de validación en Power BI Desktop

Abrir `PBI/PBI_Indicadores.pbip` y validar en la página `p14_satisfaccion_capacitaciones_v2`:

1. Los tres segmentadores filtran todos los visuales de datos.
2. Seleccionar un call center actualiza panel, respuestas, fecha, detalle, comentarios y KPI.
3. Seleccionar una fecha actualiza call center, panel, respuestas, detalle, comentarios y KPI.
4. Seleccionar una fila del detalle actualiza panel, respuestas, fecha, comentarios y KPI.
5. Seleccionar un comentario no altera el dashboard.
6. Seleccionar una métrica del panel no altera el dashboard.
7. Las tarjetas KPI no filtran otros visuales.
8. El botón `Volver a Home` funciona en toda su superficie.

Estado de fase: `SC-6 pendiente de validación funcional en Power BI Desktop`.
