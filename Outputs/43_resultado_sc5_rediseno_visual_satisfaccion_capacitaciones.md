# Resultado — Fase SC-5: rediseño visual de la copia según el mockup

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | `SC-5` de [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Página trabajada | `p14_satisfaccion_capacitaciones_v2` (copia de trabajo, no enlazada a Home) |
| Página protegida | `p14_satisfaccion_capacitaciones` (original) — sin cambios |
| Fecha | 2026-07-22 |
| Alcance | Visuales de la copia `v2` + 1 atributo puntual (`formatString`) de la medida `Ultima Capacitacion` (Revisión 3, autorizado explícitamente). No se creó ninguna medida DAX, no se modificó ninguna expresión DAX, Power Query, relaciones, `RutaCarpetaData`, `compatibilityLevel`, tema global, otras páginas ni `Data/*.xlsx`. Fase SC-6 (interacciones) explícitamente no implementada. |

---

## Revisión 2 — la primera propuesta fue rechazada por falta de fidelidad visual

La primera versión de `SC-5` (documentada en las secciones 1–15 de este archivo) fue **funcional pero no fiel al mockup**. El usuario revisó una captura de Power BI Desktop y rechazó la aprobación, señalando 9 diferencias concretas frente a `Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png`. Esta sección documenta la segunda iteración, que corrige esas diferencias sin comitear la primera versión.

### Diferencias señaladas y su corrección

| # | Diferencia observada | Corrección aplicada |
|---|---|---|
| 1 | Faltaba el logo Connect en el encabezado | Se creó `sc_logo_connect` (visual `image`), reutilizando el mismo recurso registrado que usa Home (`logo_connect_naranja_20260708.png`, paquete `RegisteredResources`) — mismo patrón ya validado en el proyecto, sin insertar un archivo nuevo |
| 2 | La insignia de datos piloto quedaba cortada | `sc_pilot_badge`/`sc_pilot_note` se agrandaron (16→20px de alto) y reposicionaron dentro del nuevo bloque de encabezado (`x=234,y=78`), con margen suficiente respecto al borde del panel (98 < 104) |
| 3 | El botón Home no tenía fondo naranja ni flecha | `sc_home_btn` cambió su relleno de `#FFF3EE` (tenue) a `#F15B2B` (naranja sólido, color de tema); `sc_home_label` cambió a texto blanco `← Volver a Home` en negrita. Es una excepción deliberada al estilo "suave" que usan las demás páginas originales, autorizada explícitamente para esta copia |
| 4 | Los filtros no estaban en un contenedor común | Se creó `sc_filter_panel`, un panel blanco con borde gris claro que envuelve los 3 segmentadores; se reposicionaron con espaciado uniforme (3 columnas de 370px) dentro de ese panel |
| 5 | Las tarjetas KPI no tenían íconos, círculos, jerarquía ni línea inferior | Se agregó un círculo de acento (`sc_kpi_icon_*`, relleno `#FFF3EE`) y una línea naranja corta en la parte inferior (`sc_kpi_line_*`, `#F15B2B`, 40×3px) a cada una de las 6 tarjetas; se ajustó el margen interno de la tarjeta para dejar espacio al círculo |
| 6 | El panel de satisfacción aparecía como una sola barra apilada | **Causa raíz identificada**: un único `barChart` con 4 medidas y sin categoría se renderiza en Power BI como una barra apilada (comportamiento por defecto sin un eje de categoría). **Corrección**: se eliminó ese visual y se reconstruyó como **4 gráficos de barra independientes** (`sc_satisf_bar_satisf/claridad/utilidad/dinamismo`), cada uno con una sola medida — así no hay ambigüedad de apilado, cada barra es su propio visual. Se agregaron 4 etiquetas de texto (`sc_satisf_label_*`) a la izquierda de cada barra y un panel contenedor único (`sc_panel_satisf_container`) que unifica visualmente las 4 barras + título + hint + tarjeta de respuestas en un solo bloque |
| 7 | Las tablas mostraban solo encabezado, sin filas | Se reconstruyeron ambas tablas (`sc_tabla_callcenter`, `sc_tabla_comentarios`) replicando más de cerca la estructura exacta de la tabla original que sí funcionaba (`sc_tabla_formador`, ya eliminada) — se quitó la marca `"active": true` que se había agregado a la primera columna (el original no la tenía en ninguna columna) y se aumentó el tamaño de fuente (9D→10D). No se identificó con certeza una causa raíz distinta a esta mediante análisis estático del JSON — **queda pendiente de confirmación visual en Desktop** si el problema persiste |
| 8 | El gráfico de jornada quedaba visible y recortado abajo | Se **eliminó `sc_chart_jornada` de la composición visible** de la copia `v2` (no solo se ocultó ni se movió fuera de lienzo) — ver §"Eliminación del gráfico de jornada" abajo |
| 9 | La página se veía comprimida, con textos truncados | Se rediseñó todo el layout usando las 6 zonas aproximadas indicadas por el usuario (encabezado, filtros, KPI, gráficos principales, tablas, nota), con más espacio por bloque que en la primera versión; se redujo la fuente del eje de categorías del gráfico de call center (9D→8D) y se aumentó el ancho de cada panel de la fila de gráficos (372px→392px) para reducir el riesgo de truncamiento de nombres largos como "INTERACTIVO"/"ONE CONTACT" — no se puede garantizar al 100% sin confirmación visual en Desktop |

### Visuales reconstruidos por completo (no solo reposicionados)

- Panel de satisfacción: de 1 `barChart` multi-medida a 4 `barChart` independientes + 4 etiquetas + 1 contenedor + 1 título (11 visuales nuevos reemplazando 1).
- `sc_tabla_callcenter` y `sc_tabla_comentarios`: eliminadas y recreadas desde cero con estructura más cercana al patrón original probado.

### Visuales nuevos agregados en esta revisión (no existían en la Revisión 1)

`sc_logo_connect`, `sc_header_separator`, `sc_filter_panel`, 6× `sc_kpi_icon_*`, 6× `sc_kpi_line_*`, `sc_panel_satisf_container`, `sc_panel_satisf_title`, 4× `sc_satisf_bar_*`, 4× `sc_satisf_label_*`.

### Eliminación del gráfico de jornada (DEC-3)

`sc_chart_jornada` se **eliminó por completo** de `p14_satisfaccion_capacitaciones_v2` (no quedó oculto, ni movido fuera del lienzo, ni parcialmente visible). Conforme a DEC-3 y al plan `SC-6`, este gráfico debe reintroducirse más adelante como tooltip, drillthrough o página de detalle — **no como parte de la composición visible de esta página**. Su ausencia actual es intencional y documentada, no un olvido.

### Validación de que todo cabe en el lienzo `1280×720`

Se verificó programáticamente (no visualmente) que los 55 visuales de la copia tienen `x≥0`, `y≥0`, `x+width≤1280` y `y+height≤720` — **0 visuales fuera de los límites del lienzo**. Esta validación confirma la ausencia de recortes por posición, pero no reemplaza la confirmación visual en Power BI Desktop (p. ej. texto que se ve truncado dentro de sus propios límites por tamaño de fuente, no por posición).

### Pendientes manuales reales (no resueltos por no existir un patrón PBIR seguro)

1. **Filtro "sin vacíos" en `sc_tabla_comentarios`**: sigue sin filtro embebido en el PBIR (mismo motivo que en la Revisión 1 — sin precedente local de `filterConfig` en todo el proyecto). **Paso manual obligatorio**: en Power BI Desktop, `Comentario` → panel de filtros del visual → filtro avanzado → "no es" → `"Sin comentario"`. **`SC-5` no puede darse por cerrada hasta que el usuario aplique y confirme este filtro**, conforme a la instrucción explícita del usuario.
2. **Formato de fecha corta ("dd/MM", sin "12 a. m.") en el eje de `sc_chart_capxfecha`**: no se encontró una propiedad PBIR verificada para forzar el formato del eje sin arriesgar una estructura no probada. Paso manual sugerido: clic derecho en el eje → Formato del eje → código de formato personalizado `dd/MM`.
3. **Corte de texto real dentro de los límites del visual** (p. ej. nombres de call center en el eje, o el valor de `sc_kpi_ultima`): la validación de esta fase confirma que ningún visual excede el lienzo, pero no puede confirmar que el contenido interno no se trunque visualmente — requiere confirmación en Desktop.
4. **Íconos semánticos de las tarjetas KPI** (graduación, personas, edificio, estrella, calendario, mensaje): no se insertaron, por no existir en el proyecto ningún patrón de imagen/ícono seguro más allá del logo registrado. Se dejó únicamente el círculo de acento como marcador visual — inserción de íconos reales queda como paso manual pendiente si se desea, vía "Insertar imagen" en Power BI Desktop.
5. **Esquinas redondeadas en paneles tipo `shape`** (contenedor de filtros, panel de satisfacción): el proyecto no tiene ningún precedente de una propiedad de radio de esquina aplicable a visuales `shape` (solo `visualContainerObjects.border.radius`, que en este proyecto siempre aparece con `show:false` para visuales `shape`, y activarlo generaría un borde redondeado superpuesto a un relleno de esquinas rectas, no una forma verdaderamente redondeada). Se mantienen esquinas rectas en estos elementos, consistente con el resto del proyecto.

---

## Revisión 3 — segunda ronda de correcciones tras nueva revisión visual

La Revisión 2 mejoró notablemente la fidelidad al mockup (tablas con datos, logo, botón Home, jornada retirada, lienzo completo), pero el usuario identificó 9 problemas adicionales tras revisar una nueva captura de Power BI Desktop. Esta sección documenta la tercera ronda de correcciones — **`SC-5` sigue sin aprobarse**, no se ha creado ningún commit.

### Diferencias señaladas y su corrección

| # | Diferencia observada | Causa probable identificada | Corrección aplicada |
|---|---|---|---|
| 1 | Segmentador de Fecha vacío, solo mostraba el ícono | Se había reducido su altura de 38px (original) a 28px — el modo `Between` (2 casillas de fecha) probablemente necesita más espacio vertical del que le quedaba y Power BI lo colapsó a un icono | Se restauró una altura de 34px (cercana al original) para el segmentador y su etiqueta, y se amplió el panel de filtros de 54 a 58px de alto para que quepa sin recortarse |
| 2 | `Última capacitación` mostraba `07/10/2026` en vez de `10/07/2026` | La medida usaba `formatString: Short Date`, que depende de la configuración regional de Power BI Desktop/Windows — en formato `en-US` (`MM/dd/yyyy`) el 10 de julio se lee como `07/10/2026` | **Cambio autorizado explícitamente por el usuario, limitado a un solo atributo**: `formatString` de la medida `Ultima Capacitacion` en `_Medidas Capacitacion.tmdl` cambiado de `Short Date` a `dd/MM/yyyy` (formato explícito, no depende de configuración regional). La expresión DAX (`MAX(Fact_SatisfaccionCapacitacion[Fecha])`) no se tocó — confirmado por `git diff` |
| 3 | El gráfico por fecha mostraba `12 a. m.` y fechas repetidas | El eje de categoría trataba el campo `Fecha` (tipo `dateTime`) como eje continuo, generando ticks a nivel de hora en vez de uno por día | Se agregó `categoryAxis.axisType: 'Categorical'` (eje discreto en vez de continuo) y `categoryAxis.formatString: 'dd/MM'` — **no verificado en Desktop, ver pendientes** |
| 4 | El panel de satisfacción no mostraba 4 barras reales; aparecían indicadores grises e íconos de "faltan campos" | Un `barChart` de una sola medida sin ningún campo de categoría probablemente no es una configuración completa para ese tipo de visual en esta versión de Power BI Desktop — Power BI lo interpreta como "visual incompleto" y muestra su ícono de marcador de posición | **Se descartó el enfoque de 4 gráficos de barra independientes** (2do intento fallido) y se reconstruyó como **una matriz (`pivotTable`) con las 4 medidas ubicadas directamente en la sección de Filas** (`Satisfacción`, `Claridad`, `Utilidad`, `Dinamismo`), con los valores en negrita naranja — sigue la preferencia técnica indicada por el usuario. **No se intentó agregar formato condicional de "barras de datos"** (no hay precedente verificado de esa estructura JSON en el proyecto) — ver pendiente |
| 5 | Los círculos de acento de las tarjetas KPI seguían vacíos | Se habían dejado como círculos de color sólido sin ningún glifo, por prudencia ante la falta de íconos reales | Se agregó una letra en negrita naranja dentro de cada círculo (`C`, `R`, `CC`, `S`, `F`, `%`) — texto simple, sin emojis, sin depender de ningún recurso de imagen no registrado |
| 6 | Nombres de call center truncados (`ATEN...`, `ONE CO...`) | Ancho insuficiente (392px para 5 categorías) | Se amplió el gráfico de 392 a 440px (el más ancho de la fila), se redujo la fuente del eje de 8D a 7D, y se achicaron proporcionalmente el gráfico por fecha (360px) y el panel de satisfacción (384px) para conservar el ancho total de 1216px |
| 7 | La tabla de detalle no mostraba los 5 call centers + Total sin recortarse | Espaciado de fila (`rowPadding: 6D`) y fuente (10D) demasiado generosos para la altura disponible | Se redujo `rowPadding` a 3D, la fuente a 9D, y se aumentó la altura de la tabla de 175 a 180px |
| 8 | La tabla de comentarios incluía valores `"."` (no son comentarios reales); tenía scroll horizontal y columnas cortadas | No existía ningún filtro de exclusión, y las columnas no tenían anchos ni ajuste de texto definidos | **Se agregó un filtro de visual (`filterConfig`) que excluye `"Sin comentario"` y `"."`** — primer intento real de esta estructura en el proyecto (sin precedente local verificado, ver pendiente de confirmación). Se definieron anchos de columna explícitos (`Comentario` 230px, `Call Center` 110px, `Fecha` 90px, suma 430 de 450 disponibles) y se activó `wordWrap` en los valores; se agregó `formatString: 'dd/MM/yyyy'` a la columna `Fecha` de este visual |
| 9 | El logo solo mostraba "CONNECT" en vez de "Connect Assistance" completo | — | Se revisó `PBI_Indicadores.Report/StaticResources/RegisteredResources/`: **solo existe un recurso de logo registrado** (`logo_connect_naranja_20260708.png`, el mismo que usa Home). Existen 2 archivos SVG con el logo completo en `Assets/logos/`, pero **no están registrados como recursos del reporte** — usarlos requeriría registrar un recurso nuevo, fuera del alcance autorizado ("no agregues archivos externos si no existe un recurso validado"). Se mantiene el logo actual sin cambios |

### Nota sobre el cambio de `formatString` en el modelo semántico

Esta es la única fase de esta iniciativa que modifica un archivo de `PBI_Indicadores.SemanticModel/`. El cambio está **explícitamente autorizado por el usuario y limitado a un solo atributo**:

```diff
  measure 'Ultima Capacitacion' = MAX(Fact_SatisfaccionCapacitacion[Fecha])
-		formatString: Short Date
+		formatString: dd/MM/yyyy
  		lineageTag: 80996b3c-a029-4e03-a476-950aa034cbe2
```

Confirmado por `git diff` que ninguna otra medida, expresión DAX, relación, columna de Power Query ni `compatibilityLevel` cambió.

### Pendientes reales tras esta revisión (no verificables sin Power BI Desktop)

1. **Filtro de la tabla de comentarios**: es la primera vez en el proyecto que se escribe un bloque `filterConfig` con la sintaxis de filtro semántico (`Version 2`/`Where`/`Condition`/`Not`/`In`). No hay precedente local para validar la sintaxis exacta. **Si al abrir en Desktop el filtro no aparece aplicado o genera un error**, avisar de inmediato — el filtro se retiraría y se documentaría como paso manual (`Comentario` → filtro avanzado → "no es" → `"Sin comentario"` y `"."`).
2. **Eje categórico y formato `dd/MM` del gráfico por fecha**: `axisType` y `formatString` a nivel de eje no tienen precedente local verificado. Si persiste `12 a. m.` o las fechas duplicadas, es la señal de que estas propiedades no se interpretaron como se esperaba.
3. **Matriz de satisfacción con medidas en filas**: la estructura (`Rows` con 4 proyecciones de medida, sin columnas) es la técnica estándar para este patrón, pero no hay un ejemplo local previo en el proyecto. Si las 4 medidas aparecen como columnas en vez de filas, es la señal de que esta interpretación no fue correcta — en ese caso, el ajuste manual en Desktop es: seleccionar la matriz → panel Campos → activar "Cambiar filas y columnas" o "Mostrar en filas" sobre la sección de Valores.
4. **Formato condicional de barras de datos naranjas**: no se intentó — requiere una estructura de "regla de color" (`FillRule`/gradiente) que no se pudo verificar seguridad de sintaxis. Paso manual sugerido: sobre la matriz, clic derecho en cada medida → Formato condicional → Barras de datos → color de barra positiva `#F15B2B`.
5. **Ancho de columna (`columnWidth`) en la tabla de comentarios**: propiedad con confianza moderada, no verificada localmente — si las columnas no respetan los anchos indicados, ajustar manualmente arrastrando los bordes de columna en Desktop (Power BI recordará el ajuste manual en la próxima exportación a PBIR).

### Validación realizada en esta revisión

- **JSON válido**: 54 `visual.json` de la copia, 0 errores de parseo.
- **Límites de lienzo**: 0 visuales fuera de `1280×720`.
- **Página original intacta**: `git diff` sobre `p14_satisfaccion_capacitaciones/` sin salida.
- **Cambio en el modelo semántico limitado a 1 línea** (`formatString` de `Ultima Capacitacion`): confirmado por `git diff` sobre `_Medidas Capacitacion.tmdl`.
- **Sin nombres personales**: `grep` sobre `visuals/` de la copia sin coincidencias.
- **Sin cambios fuera de alcance**: `expressions.tmdl`, `relationships.tmdl`, `database.tmdl`, `Home`, `pages.json`, `Data/` — todos sin cambios.

### Estado final de `git status` (Revisión 3, vigente)

```
 M PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl
 M .../sc_chart_callcenter/visual.json
 M .../sc_chart_capxfecha/visual.json
 M .../sc_filter_panel/visual.json
 M .../sc_slicer_fecha/visual.json  M .../sc_slicer_fecha_label/visual.json
 M .../sc_slicer_callcenter/visual.json  M .../sc_slicer_callcenter_label/visual.json
 M .../sc_slicer_jornada/visual.json  M .../sc_slicer_jornada_label/visual.json
 M .../sc_panel_satisf_container/visual.json  M .../sc_panel_satisf_title/visual.json
 M .../sc_panel_satisf_hint/visual.json  M .../sc_kpi_respuestas_panel/visual.json
 M .../sc_tabla_callcenter/visual.json
 D .../sc_satisf_bar_satisf/  D .../sc_satisf_bar_claridad/  D .../sc_satisf_bar_utilidad/  D .../sc_satisf_bar_dinamismo/
 D .../sc_satisf_label_satisf/  D .../sc_satisf_label_claridad/  D .../sc_satisf_label_utilidad/  D .../sc_satisf_label_dinamismo/
?? .../sc_panel_satisf_matrix/
?? .../sc_kpi_letter_capacitaciones/  .../sc_kpi_letter_respuestas/  .../sc_kpi_letter_callcenters/
?? .../sc_kpi_letter_satisfaccion/  .../sc_kpi_letter_ultima/  .../sc_kpi_letter_comentarios/
?? Data/
```

(Ruta abreviada `...` = `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/`. `sc_tabla_comentarios/visual.json` también se sobrescribió con la nueva versión con filtro y anchos de columna — sigue como archivo ya trackeado desde la Revisión 2.)

### Punto de control — sin commit (Revisión 3, vigente)

No se creó ningún commit. Se solicita al usuario:

1. Abrir el `.pbip` en Power BI Desktop (ya confirmado cerrado antes de esta revisión).
2. Confirmar específicamente los 5 pendientes marcados arriba como "no verificables sin Desktop" (filtro de comentarios, eje de fecha, matriz de satisfacción en filas, formato de `Última capacitación`, segmentador de Fecha visible).
3. Compartir una nueva captura de la página `v2` para comparar contra el mockup.

`SC-6` solo queda autorizada después de la aprobación visual definitiva de `SC-5`. No se hace push remoto.

---

## Revisión 4 — verificación de los ajustes manuales en Power BI Desktop (hallazgo: no se persistieron)

El usuario indicó que aplicaría manualmente en Power BI Desktop 4 correcciones puntuales (campo del eje de fecha, matriz de satisfacción con "mostrar valores en filas" + barras de datos, eliminación de las letras superpuestas de las tarjetas KPI, y verificación del formato de `Última capacitación`), y solicitó verificar el resultado antes de cerrar `SC-5`.

**Power BI Desktop se confirmó cerrado** (`tasklist` sin `PBIDesktop.exe`) antes de iniciar esta verificación.

### Cambios reales detectados tras la sesión de Desktop

| Archivo | Cambio | Naturaleza |
|---|---|---|
| `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl` | +186 líneas: entradas de sinónimos de Q&A (`PowerBI.VisualColumnRename`, `State: Suggested/Generated`) para las medidas/columnas que aparecen con nuevos nombres en los visuales rediseñados (`Claridad`, `Dinamismo`, `Satisfacción`, `Utilidad`, `Capacitaciones`, `Respuestas`, etc.) | **Metadato automático de Desktop** (esquema lingüístico de preguntas y respuestas) — no es un cambio de negocio, de datos ni de DAX |
| `PBI/PBI_Indicadores.Report/definition/pages/pages.json` | `activePageName` → `p14_satisfaccion_capacitaciones_v2` (la página que se estaba viendo al guardar) | Metadato automático de Desktop, mismo patrón ya visto en `SC-4` |
| `sc_panel_satisf_matrix/visual.json` | Desktop agregó `"active": true/false` a las 4 proyecciones de la sección `Rows` (solo la primera queda `true`) | Normalización automática de Desktop al abrir/guardar la consulta |
| Varios `visual.json` de la copia | Bump de versión de esquema (`2.4.0`/`2.9.0` → `2.9.0`/`2.11.0` según el visual) | Metadato de versión de Desktop, sin efecto funcional |
| `_Medidas Capacitacion.tmdl` | **Sin cambios adicionales** — el `formatString: dd/MM/yyyy` de `Ultima Capacitacion` sigue exactamente como se dejó en la Revisión 3 | — |

### Las 3 correcciones manuales solicitadas — verificadas contra el contenido real de los archivos

| Corrección solicitada | Resultado de la verificación |
|---|---|
| 1. Cambiar el eje de `sc_chart_capxfecha` a `Dim_Calendario[DiaMes]`, ordenado por `Fecha`, eje categórico | **No se aplicó.** El campo de categoría del visual sigue siendo `Dim_Calendario.Fecha` (confirmado leyendo `queryState.Category.projections` del archivo actual) — no `DiaMes`. La columna `DiaMes` sí existe en `Dim_Calendario` (confirmado), pero el visual no la usa. |
| 2. Matriz de satisfacción: "Mostrar valores en filas" + barras de datos naranjas | **Parcialmente sin cambios.** La estructura sigue siendo la misma que la Revisión 3 (4 medidas en `Rows`); Desktop solo agregó las banderas `active` automáticas. **No se encontró ningún objeto de formato condicional / barra de datos** en `objects` (sigue conteniendo únicamente `grid`, `columnHeaders`, `rowHeaders`, `values` — ninguno nuevo). |
| 4. Eliminar las letras superpuestas de las tarjetas KPI | **No se aplicó.** Los 6 archivos `sc_kpi_letter_*` siguen presentes, en las mismas posiciones exactas (`x`, `y`, `z`, tamaño) que se dejaron en la Revisión 3 — no fueron movidos, ocultados ni eliminados. El margen interno de las tarjetas (`leftOuterMargin: 38L`) tampoco cambió. |
| 3. Confirmar que `Última capacitación` muestra `10/07/2026` | El archivo del modelo (`_Medidas Capacitacion.tmdl`) **conserva** `formatString: dd/MM/yyyy` sin cambios desde la Revisión 3 — la corrección de fondo sigue en su lugar. Si la captura más reciente del usuario seguía mostrando `07/10/2026`, no es porque el archivo haya cambiado o se haya revertido. |

### Conclusión honesta de esta verificación

**No hay evidencia en los archivos de que las 3 correcciones manuales (campo del eje de fecha, barras de datos de la matriz, eliminación de las letras) se hayan guardado.** Power BI Desktop sí se abrió y guardó la página (se confirman metadatos automáticos nuevos), pero el contenido funcional de esos 3 elementos específicos es idéntico al que dejó la Revisión 3. Posibles explicaciones, sin poder confirmar cuál aplica desde este entorno:

- Los cambios se hicieron en la interfaz pero no se guardaron antes de cerrar Power BI Desktop.
- Se guardaron sobre una sesión/copia distinta del archivo.
- Los cambios se intentaron pero no se completaron (p. ej., no se encontró la opción "Mostrar valores en filas" o el campo `DiaMes`).

**No se actualiza el estado de `SC-5` a "cerrada" ni a "pendiente de aprobación final"** — los mismos 4 bloqueos señalados por el usuario en este mensaje siguen sin resolverse en los archivos:

1. `Última capacitación` — el `formatString` correcto sigue en el modelo; si la captura seguía mostrando `07/10/2026`, se recomienda cerrar Power BI Desktop por completo, volver a abrir el `.pbip` desde cero (no reutilizar una sesión ya abierta) y verificar de nuevo, ya que un cambio de `formatString` en una medida a veces requiere recargar el modelo, no solo refrescar la página.
2. Eje del gráfico por fecha — sigue sin cambiar de `Fecha` a `DiaMes`.
3. Panel de satisfacción — sigue sin barras de datos; no se puede confirmar si las 4 métricas se ven correctamente sin una captura nueva.
4. Letras superpuestas — siguen presentes en las tarjetas KPI, sin cambios.

### Estado de SC-5 tras esta verificación

**`SC-5` permanece abierta, con los mismos 4 bloqueos reportados por el usuario en este mensaje** — no "pendiente de aprobación visual final", sino **pendiente de que las correcciones manuales se completen y se guarden efectivamente en Power BI Desktop**, o de que se decida retomar el intento vía PBIR con un enfoque distinto para los 3 puntos que no funcionaron.

No se creó ningún commit. No se modificó ningún archivo en esta fase de verificación (fase de solo lectura/diagnóstico).

---

## Revisión 5 — segundo intento de guardado manual: evidencia forense de que la página no se resguardó

El usuario reaplicó los 3 ajustes en Power BI Desktop, confirmó `Ctrl+S`, esperó la confirmación de guardado y cerró la aplicación antes de solicitar esta verificación. **Power BI Desktop se confirmó cerrado** (`tasklist` sin `PBIDesktop.exe`) antes de iniciar.

### Resultado de la verificación de contenido: idéntico a la Revisión 4

| Punto verificado | Resultado |
|---|---|
| Campo de categoría de `sc_chart_capxfecha` | Sigue siendo `Dim_Calendario.Fecha` — no `DiaMes` |
| Objetos de `sc_panel_satisf_matrix` | Sigue siendo `grid`, `columnHeaders`, `rowHeaders`, `values` — sin ningún objeto de barra de datos nuevo |
| Archivos `sc_kpi_letter_*` (6) | Los 6 siguen presentes en el sistema de archivos |
| `formatString` de `Ultima Capacitacion` | Sigue en `dd/MM/yyyy`, sin cambios |
| `filterConfig` de `sc_tabla_comentarios` | Sigue presente (1 filtro), sin cambios |

### Hallazgo nuevo: evidencia forense por fecha de modificación de archivo

Se comparó la hora de modificación de los archivos clave contra la hora actual del sistema (`date` → `2026-07-22 10:24:54`):

| Archivo | Última modificación | ¿Coincide con esta sesión? |
|---|---|---|
| `sc_chart_capxfecha/visual.json` | `2026-07-22 10:01:53` | **No** — mismo timestamp que en la Revisión 4, ~23 minutos antes de esta verificación |
| `sc_panel_satisf_matrix/visual.json` | `2026-07-22 10:01:53` | **No** — idéntico |
| `sc_kpi_letter_capacitaciones/visual.json` | `2026-07-22 10:01:53` | **No** — idéntico |
| `pages.json` | `2026-07-22 10:01:53` | **No** — idéntico |
| `cultures/es-ES.tmdl` (modelo semántico) | `2026-07-22 10:23:33` | **Sí** — se tocó ~1 minuto antes de esta verificación, pero con el **mismo contenido** que ya tenía (186 líneas de sinónimos de Q&A, sin ninguna línea nueva) |

**Esto es evidencia concreta, no una suposición**: los archivos de la página del reporte (`p14_satisfaccion_capacitaciones_v2`) **no se volvieron a escribir en esta sesión** — conservan exactamente la misma fecha de modificación que tenían antes de que el usuario abriera Power BI Desktop esta vez. Solo el archivo de sinónimos del modelo semántico se tocó (y con contenido idéntico, sin cambios reales). Esto indica que, aunque Power BI Desktop se abrió y se guardó *algo* (probablemente el modelo semántico, o una acción que solo sincronizó el archivo de Q&A), **la página del reporte con los 3 ajustes solicitados no llegó a guardarse en disco** en esta sesión.

Posibles causas a considerar (no verificables desde este entorno):
- Los cambios se hicieron sobre la página `p14_satisfaccion_capacitaciones` (**original**, sin sufijo `v2`) en vez de la copia `v2` — confirmado que la original tampoco cambió, así que si los ajustes se hicieron ahí, tampoco se guardaron en el archivo correcto.
- `Ctrl+S` guardó el modelo semántico (de ahí el toque a `cultures/es-ES.tmdl`) pero el editor de la página del reporte no registró los cambios como "sucios" (por ejemplo, si el campo `DiaMes` no llegó a arrastrarse correctamente al eje, o si la opción "Mostrar valores en filas" no se aplicó porque el visual no estaba seleccionado).
- El PBIP abierto podría no ser el mismo archivo de esta ruta (`C:\Users\eclavijo\OneDrive\PBI_Indicadores`) — por ejemplo, una copia en caché de OneDrive o una versión abierta desde otra ubicación.

### Estado de SC-5 (sin cambios respecto a la Revisión 4)

**`SC-5` sigue abierta, con los mismos 4 bloqueos.** No se actualiza a "pendiente de aprobación final" — la evidencia de archivo (contenido + timestamp) muestra que los 3 ajustes manuales no llegaron a persistirse en ninguno de los 2 intentos. No se creó ningún commit. No se modificó ningún archivo del proyecto en esta fase (solo lectura/diagnóstico).

---

## Revisión 6 — cuarto intento vía PBIR, con cambio de estrategia en el panel de satisfacción

Tras 2 intentos manuales seguidos sin persistir en el archivo, el usuario autorizó un cuarto intento directo vía PBIR, priorizando confiabilidad sobre fidelidad exacta al mockup en el elemento más problemático.

### 1. Letras superpuestas — eliminadas (corrección segura, sin riesgo)

Se eliminaron por completo los 6 archivos `sc_kpi_letter_*`. Las tarjetas KPI quedan con el círculo de acento vacío (sin glifo) — ver pendiente de íconos ya documentado en revisiones anteriores.

### 2. Panel de satisfacción — cambio de estrategia: de matriz a 4 tarjetas apiladas

Después de 3 intentos fallidos con el mismo elemento (4 gráficos de barra independientes en la Revisión 3, matriz con medidas en filas en la Revisión 3/4/5), se tomó la decisión de **priorizar la confiabilidad sobre la fidelidad visual exacta**:

- Se eliminó `sc_panel_satisf_matrix` (la matriz, sin objetos de barra de datos funcionando en 2 verificaciones).
- Se creó `sc_satisf_row_satisf`, `sc_satisf_row_claridad`, `sc_satisf_row_utilidad`, `sc_satisf_row_dinamismo`: **4 tarjetas (`cardVisual`) apiladas**, cada una con su medida correspondiente. Este es el **mismo patrón exacto** ya usado con éxito comprobado en las 6 tarjetas KPI de la fila superior de esta misma página — no introduce ninguna propiedad nueva sin precedente.
- **Se abandona explícitamente el requisito de "barra visual"** en este punto — las 4 tarjetas muestran el valor de cada métrica (`Satisfacción`, `Claridad`, `Utilidad`, `Dinamismo`) en texto grande naranja con su etiqueta, no como una barra proporcional. Dado que 3 intentos distintos de representar esto como una barra (gráfico de barras sin categoría, matriz con formato condicional) no llegaron a confirmarse funcionando, se prioriza que el panel **muestre las 4 métricas de forma confiable** sobre que se vean como barras.
- Si se desea el efecto visual de barra, queda como **paso manual sugerido en Desktop**: aplicar formato condicional "Barras de datos" sobre cada tarjeta no es posible (las tarjetas no soportan ese formato) — la alternativa sería reconstruir este panel una quinta vez como gráfico de barras con category real, o aceptar el diseño actual de tarjetas.

### 3. Gráfico por fecha — intento adicional (sin remover el intento previo)

Se agregó una propiedad `"format": "dd/MM"` directamente en la proyección del campo `Category` (a nivel de campo, no de eje) de `sc_chart_capxfecha`, complementando (no reemplazando) las propiedades `axisType`/`formatString` ya agregadas en la Revisión 3 que, según el reporte del usuario, no resolvieron el problema por sí solas. **Esta es una ubicación distinta a la ya intentada — sigue sin precedente local verificado.** El campo de categoría se mantiene en `Dim_Calendario.Fecha` (no se cambió a `DiaMes`, ya que esa columna es un entero de solo "día del mes" sin componente de mes — usarla sola habría mostrado números como "4", "6", "7" sin contexto de mes, y no el formato `dd/MM` solicitado).

### Validaciones realizadas

- **JSON válido**: 51 `visual.json` de la copia, 0 errores.
- **Límites de lienzo**: 0 visuales fuera de `1280×720`.
- **Página original y Home intactos**: `git diff` sin salida en ambas.
- **Sin nombres personales**: sin coincidencias.
- **Sin cambios en Power Query, relaciones ni `Data/`**.

### Pendiente real tras esta revisión

El gráfico por fecha sigue siendo el único punto **sin una solución de alta confianza** — el segundo intento de formato (a nivel de proyección) tiene la misma incertidumbre que el primero. Si sigue mostrando horas o fechas duplicadas tras verificar en Desktop, se recomienda considerar el ajuste manual directo del eje (Formato → Eje X → código de formato personalizado `dd/MM`) en vez de un quinto intento por JSON.

### Estado de SC-5

**Sigue abierta.** No se creó commit. Pendiente de que el usuario abra Power BI Desktop (sesión de solo verificación, sin necesidad de editar nada esta vez) y confirme: (a) letras ya no aparecen, (b) panel de satisfacción muestra las 4 métricas como tarjetas (no como barras, cambio de diseño deliberado), (c) estado del eje de fecha.

---

## Revisión 7 — corrección estructural definitiva

Esta revisión corrige directamente la **causa raíz confirmada** de los 3 bloqueos restantes, en vez de seguir ajustando propiedades aisladas. La captura del usuario mostró el contenido real de `sc_chart_capxfecha`, confirmando que el eje usaba la **jerarquía automática de fechas** (`Variación` → `Jerarquía de fechas` → Año/Trimestre/Mes/Día), no la columna `Fecha` directa.

### Causa raíz de la jerarquía de Fecha

Al inspeccionar el archivo real, el campo de categoría era una cadena de 4 proyecciones `HierarchyLevel` ancladas a `Dim_Calendario.Fecha.Variación.'Jerarquía de fechas'` (niveles Año/Trimestre/Mes/Día) — el patrón exacto de la función "Auto Date/Time" de Power BI, ya documentado como ruido conocido en `CLAUDE.md`. Esto se originó en una de las sesiones manuales del usuario en Power BI Desktop: al interactuar con el campo `Fecha` en el lienzo del reporte, Desktop expandió automáticamente la jerarquía en vez de usar la columna plana, y la generó también a nivel de modelo (tabla oculta `LocalDateTable_fe450612-...`, relación nueva `Dim_Calendario.Fecha` → `LocalDateTable_fe450612.Date`, y un bloque `variation` en `Dim_Calendario.Fecha`) — confirmado con `git diff` sobre `relationships.tmdl` y `Dim_Calendario.tmdl`, archivos que **esta sesión no editó directamente**; son metadato acumulado de sesiones previas de Desktop, recién detectado al revisar el alcance completo de cambios.

**Esta tabla oculta y su relación no se revirtieron** — son el mismo patrón de "ruido conocido de Auto Date/Time" ya documentado y aceptado en el resto del modelo (las 3 tablas de hechos ya tenían este mismo patrón desde antes; ahora `Dim_Calendario` también lo tiene). Revertirlo a mano arriesga una referencia huérfana en el `variation` de `Fecha`, exactamente el riesgo ya señalado en `Docs/01_modelo_datos.md` §6 — se deja como limpieza pendiente para una sesión dedicada de Power BI Desktop (Opciones → Carga de datos → deshabilitar detección automática de tabla de fechas), no como parte de esta corrección.

### 1. Gráfico "Capacitaciones por fecha" — reconstruido con enlace directo

Se reescribió por completo `sc_chart_capxfecha/visual.json`: el campo de categoría vuelve a ser una proyección `Column` plana sobre `Dim_Calendario.Fecha` (el mismo patrón exacto y ya probado de `re_chart_fecha` en la página "Resumen ejecutivo"), sin ninguna referencia a `HierarchyLevel`, `Variación` ni `Jerarquía de fechas`. Se conservan `categoryAxis.axisType: Categorical` y `valueAxis.start: 0`.

### 2. Panel de satisfacción — reconstrucción con dimensión desconectada + medida dinámica

Se implementó la arquitectura solicitada:

- **`Dim_MetricaSatisfaccion`** (tabla nueva, `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_MetricaSatisfaccion.tmdl`): tabla calculada vía `DATATABLE()`, 2 columnas (`Metrica` texto, `Orden` entero oculto), `Metrica` ordenada por `Orden` (`sortByColumn`). **Sin relación con ninguna otra tabla** (desconectada, tal como se pidió). Registrada en `model.tmdl` (`ref table Dim_MetricaSatisfaccion`).
- **`Valor Metrica Satisfaccion`** (medida nueva en `_Medidas Capacitacion.tmdl`): `SWITCH(SELECTEDVALUE(Dim_MetricaSatisfaccion[Metrica]), ...)` que reutiliza las 4 medidas ya existentes (`Satisfaccion/Claridad/Utilidad/Dinamismo Promedio Capacitacion`) según el valor de fila seleccionado — reacciona al contexto de filtro vigente (incluido un futuro filtro de call center en `SC-6`), formato `0.0`.
- **Visual único** (`sc_panel_satisf_chart`, `barChart`): categoría = `Dim_MetricaSatisfaccion.Metrica`, valor = `Valor Metrica Satisfaccion`, con `sortDefinition` ascendente por `Dim_MetricaSatisfaccion.Orden` (mismo patrón de ordenamiento ya usado en `sc_chart_jornada`/`sc_chart_callcenter`). Eje de valores de `0` a `5`. Barras naranjas, etiquetas de datos visibles, etiquetas de categoría visibles, sin leyenda, sin título propio (se apoya en `sc_panel_satisf_title`, ya existente).
- Se eliminaron las 4 tarjetas de la Revisión 6 (`sc_satisf_row_*`) y la matriz de las Revisiones 3-5 (`sc_panel_satisf_matrix`, ya eliminada previamente).

### 3. Última capacitación — medida de texto explícita

Se creó **`Ultima Capacitacion Texto`** (`FORMAT([Ultima Capacitacion], "dd/MM/yyyy")`) en `_Medidas Capacitacion.tmdl`. Se verificó primero que ni `sc_kpi_ultima` ni la columna "Última fecha" de `sc_tabla_callcenter` tenían ningún override de formato a nivel de visual que pudiera explicar el problema (no se encontró ninguno). Se reemplazó la medida usada en ambos visuales de `Ultima Capacitacion` → `Ultima Capacitacion Texto` (incluyendo el selector de `columnFormatting` de la tabla). **`Ultima Capacitacion` se conserva sin cambios en su expresión DAX**, para cálculos, filtros y ordenamiento futuros.

### Visuales temporales eliminados en esta revisión

`sc_panel_satisf_matrix` (matriz, Revisiones 3-5), `sc_satisf_row_satisf`/`sc_satisf_row_claridad`/`sc_satisf_row_utilidad`/`sc_satisf_row_dinamismo` (4 tarjetas, Revisión 6). Inventario final verificado sin carpetas huérfanas de intentos anteriores (`sc_kpi_letter_*`, `sc_satisf_bar_*`, `sc_satisf_label_*` — ninguno presente).

### Archivos modificados o creados en esta revisión

**Modificados:**
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_chart_capxfecha/visual.json` (categoría reescrita, sin jerarquía)
- `.../sc_kpi_ultima/visual.json` (usa `Ultima Capacitacion Texto`)
- `.../sc_tabla_callcenter/visual.json` (columna "Última fecha" usa `Ultima Capacitacion Texto`)
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl` (2 medidas nuevas: `Valor Metrica Satisfaccion`, `Ultima Capacitacion Texto`)
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (registro de `Dim_MetricaSatisfaccion`)

**Creados:**
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_MetricaSatisfaccion.tmdl` (tabla calculada nueva)
- `.../sc_panel_satisf_chart/visual.json` (gráfico de barras único del panel de satisfacción)

**Eliminados:**
- `.../sc_satisf_row_satisf/`, `sc_satisf_row_claridad/`, `sc_satisf_row_utilidad/`, `sc_satisf_row_dinamismo/` (Revisión 6)

**Detectados como metadato acumulado de Desktop (no modificados por esta sesión, generados en sesiones manuales previas del usuario):**
- `PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl` (+1 relación hacia una tabla oculta de fecha)
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl` (bloque `variation` + `summarizeBy: sum` automático en columnas numéricas)
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/LocalDateTable_fe450612-acd4-4916-b032-c743384b5f6e.tmdl` (tabla oculta nueva, Auto Date/Time)
- `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl` (sinónimos de Q&A, acumulado de revisiones previas)

### Ampliaciones del modelo — todas dentro de lo autorizado

| Objeto nuevo | Autorizado como |
|---|---|
| `Dim_MetricaSatisfaccion` | Tabla auxiliar explícitamente autorizada |
| `Valor Metrica Satisfaccion` | Medida dinámica explícitamente autorizada |
| `Ultima Capacitacion Texto` | Medida de formato visual, autorizada como alternativa tras el fallo confirmado del `formatString` directo |

No se creó ningún otro objeto. No se escribió `lineageTag` a mano en ninguno de los bloques nuevos (`Dim_MetricaSatisfaccion`, las 2 medidas nuevas) — quedan sin ese atributo para que Power BI Desktop lo genere al guardar.

### Validaciones técnicas realizadas

1. **JSON válido**: 48 `visual.json` de la copia, 0 errores.
2. **Sintaxis TMDL**: paréntesis/corchetes/llaves balanceados en `Dim_MetricaSatisfaccion.tmdl`, `_Medidas Capacitacion.tmdl` y `model.tmdl`.
3. **`sc_chart_capxfecha` sin jerarquía**: confirmado — la proyección de categoría es un único campo `Column` plano, sin `HierarchyLevel`.
4. **Panel de satisfacción**: confirmado — 1 categoría (`Dim_MetricaSatisfaccion.Metrica`) con 4 filas posibles, 1 medida dinámica, 1 gráfico de barras horizontal (no 4 visuales separados, no matriz).
5. **4 métricas presentes**: `Satisfacción`, `Claridad`, `Utilidad`, `Dinamismo` — las 4 codificadas en `Dim_MetricaSatisfaccion` y referenciadas en el `SWITCH` de la medida dinámica.
6. **`Última capacitación` con formato explícito**: `Ultima Capacitacion Texto` usa `FORMAT(..., "dd/MM/yyyy")`, sin depender de configuración regional.
7. **Sin visuales temporales huérfanos**: inventario de 45 carpetas de visuales revisado, ninguna de intentos anteriores.
8. **Página original sin cambios**: `git diff` sobre `p14_satisfaccion_capacitaciones/` sin salida.
9. **Home sin cambios**: `git diff` sobre `67eff42d82e1c9c15b84/` sin salida.
10. **`Data/` sigue sin seguimiento**: confirmado.
11. **Sin nombres personales**: sin coincidencias en `visuals/` de la copia.
12. **`compatibilityLevel` sin cambios**: confirmado.
13. **`RutaCarpetaData`, Power Query (`expressions.tmdl`)**: sin cambios.

### Riesgos pendientes

- **No verificado en Desktop**: ninguno de los 3 puntos corregidos se ha confirmado visualmente todavía (este entorno no puede renderizar Power BI). La corrección aborda la causa raíz identificada con alta confianza (jerarquía confirmada por inspección directa del archivo), pero la validación final requiere abrir el `.pbip`.
- **`Valor Metrica Satisfaccion` depende de `SELECTEDVALUE`**: si en algún momento se filtra o selecciona más de un valor de `Dim_MetricaSatisfaccion[Metrica]` simultáneamente (no debería ocurrir en el uso normal del gráfico de barras, donde cada barra tiene su propio contexto de fila), la medida retornaría `BLANK()` en vez de un valor — comportamiento esperado y correcto de `SELECTEDVALUE`, no un error.
- **Metadato de Auto Date/Time acumulado**: la nueva tabla oculta y relación hacia `Dim_Calendario` se suman al ruido ya documentado; la limpieza completa (deshabilitar Auto Date/Time desde Power BI Desktop) sigue pendiente como tarea aparte, ahora con un elemento adicional por limpiar.
- **Panel de satisfacción no reacciona aún a la selección de call center**: eso es explícitamente tarea de `SC-6` (interacciones), no de esta fase.

### Estado de SC-5

**Pendiente únicamente de validación visual.** Los 3 bloqueos se corrigieron a nivel de causa raíz (no de síntoma), con alta confianza en el diagnóstico (confirmado por inspección directa del archivo real, no por suposición). No se creó ningún commit. No se modificó la página original, Home, Power Query, `RutaCarpetaData`, `compatibilityLevel`, ni `Data/`.

---

## 1. Estado inicial de `git status` (Revisión 1, contexto histórico)

```
?? Data/
```

Working tree limpio salvo `Data/`. `Power BI Desktop` confirmado cerrado (`tasklist` sin `PBIDesktop.exe`) antes de editar. Rama `main`. Página original verificada sin cambios (`git diff` vacío) antes de empezar.

## 2. Inventario inicial de visuales (antes de SC-5)

26 visuales en `p14_satisfaccion_capacitaciones_v2` (idéntico a la original, heredado de `SC-4`): 7 tarjetas KPI (`sc_kpi_satisf`, `sc_kpi_claridad`, `sc_kpi_utilidad`, `sc_kpi_dinamismo`, `sc_kpi_indice`, `sc_kpi_coment`, `sc_kpi_total`), 2 gráficos (`sc_chart_callcenter` con `Indice Global Capacitacion`, `sc_chart_jornada`), 1 tabla (`sc_tabla_formador`, con `NombreFormador`/`NombreLider`), 3 segmentadores + etiquetas, encabezado, botón Home, nota metodológica.

## 3. Visuales eliminados (8)

| Visual | Motivo |
|---|---|
| `sc_kpi_claridad` | Tarjeta individual retirada de la fila superior — la métrica se reubica en `sc_panel_satisf` |
| `sc_kpi_dinamismo` | Igual que el anterior |
| `sc_kpi_utilidad` | Igual que el anterior |
| `sc_kpi_indice` | Índice global retirado de la fila superior (no forma parte de los 6 KPI del mockup) |
| `sc_kpi_satisf` | Reemplazado por `sc_kpi_satisfaccion` (mismo dato, nueva posición/tamaño) |
| `sc_kpi_total` | Reemplazado por `sc_kpi_respuestas` (mismo dato, etiqueta "Respuestas recibidas") |
| `sc_kpi_coment` | Reemplazado por `sc_kpi_comentarios` (mismo dato, nueva posición) |
| `sc_tabla_formador` | Reemplazado por `sc_tabla_callcenter` conforme a DEC-2 ("reemplaza") — deja de exponer `NombreFormador`/`NombreLider` en esta página |

## 4. Visuales transformados (2)

| Visual | Cambio |
|---|---|
| `sc_chart_callcenter` | Medida `Indice Global Capacitacion` → `Capacitaciones Realizadas`; se agregó `sortDefinition` (orden descendente por la medida, mismo patrón ya usado en `sc_chart_jornada`); título → "Capacitaciones por call center"; reposicionado a `x=48,y=296,w=372,h=175` |
| `sc_chart_jornada` | **Sin cambio de campos** (sigue usando `Dim_Jornada.Jornada` × `Indice Global Capacitacion`, conforme a DEC-3: "reubicar", no "recalcular"). Reposicionado como elemento secundario compacto en el margen inferior (`x=48,y=674,w=300,h=40`, antes un panel principal de `260x210`); etiquetas de datos ocultas, título reducido a 8px y color secundario (`#3A3A3A`), texto de título actualizado a "Índice por jornada (detalle secundario)" |

## 5. Visuales creados (14)

| Visual | Tipo | Medida/campos | Posición |
|---|---|---|---|
| `sc_kpi_capacitaciones` | cardVisual | `Capacitaciones Realizadas` | 48,200,180×82 |
| `sc_kpi_respuestas` | cardVisual | `Total Respuestas Capacitacion` | 240,200,180×82 |
| `sc_kpi_callcenters` | cardVisual | `Call Centers Capacitados` | 432,200,180×82 |
| `sc_kpi_satisfaccion` | cardVisual | `Satisfaccion Promedio Capacitacion` | 624,200,180×82 |
| `sc_kpi_ultima` | cardVisual | `Ultima Capacitacion` (fuente 19D, reducida frente al resto para evitar corte de texto) | 816,200,180×82 |
| `sc_kpi_comentarios` | cardVisual | `% Comentarios Capacitacion` | 1008,200,180×82 |
| `sc_pilot_badge` + `sc_pilot_note` | shape + textbox | Insignia "Datos piloto sujetos a validación" (mismo estilo que `home_pilot_badge`/`home_pilot_note`, escalado a la altura disponible del encabezado) | 58,98,230×16 |
| `sc_chart_capxfecha` | lineChart | `Dim_Calendario.Fecha` × `Capacitaciones Realizadas` | 438,296,372×175 |
| `sc_panel_satisf` | barChart (horizontal, 4 medidas sin categoría — patrón de "múltiples medidas") | `Satisfaccion/Claridad/Utilidad/Dinamismo Promedio Capacitacion` | 828,296,372×120 |
| `sc_panel_satisf_hint` | textbox | "Selecciona un call center en el gráfico para actualizar" | 828,418,372×14 |
| `sc_kpi_respuestas_panel` | cardVisual (compacta) | `Total Respuestas Capacitacion` | 828,432,372×39 |
| `sc_tabla_callcenter` | tableEx | `Dim_CallCenter.CallCenter`, `Capacitaciones Realizadas`, `Total Respuestas Capacitacion`, `Satisfaccion/Claridad/Utilidad/Dinamismo Promedio Capacitacion`, `Ultima Capacitacion` (8 columnas) | 48,486,680×128 |
| `sc_tabla_comentarios` | tableEx | `Comentario`, `CallCenter`, `Fecha` de `Fact_SatisfaccionCapacitacion` | 744,486,456×128 |

## 6. Medidas y campos utilizados (todos preexistentes, ninguno creado en esta fase)

- `_Medidas Capacitacion`: `Capacitaciones Realizadas`, `Call Centers Capacitados`, `Ultima Capacitacion`, `Satisfaccion Promedio Capacitacion`, `Claridad Promedio Capacitacion`, `Utilidad Promedio Capacitacion`, `Dinamismo Promedio Capacitacion`, `% Comentarios Capacitacion`, `Indice Global Capacitacion` (esta última se conserva solo en `sc_chart_jornada`, sin uso adicional).
- `_Medidas Generales`: `Total Respuestas Capacitacion`.
- Columnas: `Dim_CallCenter.CallCenter`, `Dim_Calendario.Fecha`, `Dim_Jornada.Jornada`, `Fact_SatisfaccionCapacitacion.Comentario`, `Fact_SatisfaccionCapacitacion.CallCenter`, `Fact_SatisfaccionCapacitacion.Fecha`.

Ninguna medida DAX nueva. Ninguna columna nueva. `_Medidas Capacitacion.tmdl` y el resto del modelo semántico no se tocaron en esta fase (ya estaban completos desde `SC-3`).

## 7. Diferencias frente al mockup (documentadas, no resueltas silenciosamente)

- **Orden descendente**: se agregó `sortDefinition` a `sc_chart_callcenter` (verificado contra el patrón ya usado en `sc_chart_jornada`), pero **no se pudo confirmar el renderizado real** en este entorno — requiere verificación en Power BI Desktop.
- **Filtro "sin vacíos" de `sc_tabla_comentarios`**: **no se implementó un filtro de visual embebido en el PBIR** (`filterConfig`) para excluir `"Sin comentario"`. No existe ningún precedente de un filtro a nivel de visual en este proyecto (`grep` de `filterConfig`/`filters` no encontró coincidencias en todo el reporte), y el formato exacto de ese bloque JSON no pudo validarse contra un ejemplo real del proyecto — se prefirió no arriesgar una estructura no verificada que pudiera impedir que Power BI Desktop abra la página. **Pendiente**: aplicar el filtro manualmente en Desktop (arrastrar `Comentario` al panel de filtros del visual → "no es" → `"Sin comentario"`), acción de un solo paso en la interfaz gráfica.
- **Selección cruzada del panel `sc_panel_satisf`**: el panel se construyó como un visual independiente; la interacción "clic en `sc_chart_callcenter` actualiza `sc_panel_satisf`" **no se configuró** — corresponde explícitamente a `SC-6`, no a esta fase.
- **Gráfico de jornada**: DEC-3 pedía "reubicar como secundario, sin crear todavía página ni tooltip formal". Se implementó como un visual compacto en el margen inferior de la misma página (no como tooltip ni página nueva), la interpretación más conservadora dentro del alcance permitido.
- **Panel "Satisfacción del call center seleccionado"**: el mockup muestra un valor "Respuestas: 20" integrado visualmente junto a las 4 barras; aquí se implementó como una tarjeta separada debajo del gráfico (mismo dato, `Total Respuestas Capacitacion`, presentación ligeramente distinta por simplicidad y para evitar un patrón de visual híbrido no verificado).
- **Colores/valores**: todos los visuales usan exclusivamente el tema Connect (`#F15B2B`, `#002733`, `#3A3A3A`, `#E7E7E7`, `#F4F4F4`, `#FFFFFF`, `#FAFAFA`); ningún valor numérico del mockup se copió literalmente — todos los visuales leen directamente las medidas del modelo real.

## 8. Validación de protección de la página original

```
git diff -- "PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/"
```
Sin salida — **confirmado: cero cambios** en la página original durante toda la fase `SC-5`.

`git status --porcelain` confirma además que ningún otro archivo del proyecto cambió: ni `Home`, ni las otras 5 páginas internas, ni `pages.json`, ni `report.json`, ni ningún archivo de `PBI_Indicadores.SemanticModel/` (medidas, TMDL, `compatibilityLevel`), ni `Data/*.xlsx`.

## 9. Otras validaciones realizadas

- **JSON válido**: los 32 `visual.json` de la copia (26 preexistentes con cambios + nuevos) y `page.json` parsean sin error (`json.load` en Python sobre los 32 archivos, 0 errores).
- **Sin nombres personales**: `grep -r "NombreFormador\|NombreLider\|NombreAsesor"` sobre toda la carpeta `visuals/` de la copia — sin coincidencias.
- **Sin solapamientos de layout**: verificado manualmente el mapa de posiciones de los 32 visuales (fila de KPI, fila de 3 paneles, fila de tabla/comentarios, nota y jornada secundaria) — ninguna superposición, todos dentro del lienzo `1280×720`.
- **Segmentador de Fecha**: `sc_slicer_fecha` no se tocó — sigue con `mode: Dropdown` en el PBIR (el mismo valor que ya se confirmó en `SC-4` que renderiza como `Between` en Power BI Desktop, conforme a la corrección de DEC-4).
- **`compatibilityLevel`**: no se modificó ningún archivo de `PBI_Indicadores.SemanticModel/`, por lo tanto permanece en `1606`.

## 10. Pendientes para SC-6 (no implementados aquí, por alcance)

1. Configurar la interacción cruzada: selección en `sc_chart_callcenter` → actualiza `sc_panel_satisf`.
2. Revisar y, si aplica, configurar interacciones entre `sc_chart_capxfecha`, `sc_tabla_callcenter` y `sc_tabla_comentarios` conforme al mapa de interacciones de `Specs/06`.
3. Confirmar que la navegación "Volver a Home" (`sc_home_btn`/`sc_home_hitzone`/`sc_home_label`) sigue funcionando desde la copia — no se tocó, pero debe validarse en vivo.
4. Aplicar el filtro de visual "sin vacíos" en `sc_tabla_comentarios` desde la interfaz de Power BI Desktop (ver §7).
5. Confirmar en Power BI Desktop el orden descendente real de `sc_chart_callcenter` y ajustar manualmente si el `sortDefinition` no se interpretó como se esperaba.

## 11. Estado final de `git status`

```
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_chart_callcenter/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_chart_jornada/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_claridad/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_coment/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_dinamismo/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_indice/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_satisf/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_total/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_utilidad/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_nota_cap_panel/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_nota_cap_text/visual.json
 D PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_tabla_formador/visual.json
?? Data/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_chart_capxfecha/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_callcenters/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_capacitaciones/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_comentarios/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_respuestas/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_respuestas_panel/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_satisfaccion/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_ultima/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_panel_satisf/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_panel_satisf_hint/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_pilot_badge/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_pilot_note/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_tabla_callcenter/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_tabla_comentarios/
```

## 12. Punto de control de la Revisión 1 (superado por la Revisión 2, ver abajo)

*(Sección histórica — el punto de control real y vigente está en §16.)*

---

## 13. Validación final de la Revisión 2

1. **JSON válido**: los 55 `visual.json` de la copia parsean sin error (`json.load` sobre los 55 archivos, 0 errores).
2. **Página original sin cambios**: `git diff -- ".../p14_satisfaccion_capacitaciones/"` sin salida.
3. **Alcance limitado**: `git status --porcelain` confirma que solo cambiaron archivos dentro de `p14_satisfaccion_capacitaciones_v2/` y `Outputs/43_...md`; ningún archivo de `PBI_Indicadores.SemanticModel/`, `Home`, otras páginas, `pages.json` ni `report.json`.
4. **Ambas tablas reconstruidas con estructura más cercana al patrón original probado** — pendiente de confirmación visual en Desktop de que ahora sí muestran filas (ver pendiente §"Revisión 2" punto 7).
5. **Sin nombres personales**: `grep -r "NombreFormador\|NombreLider\|NombreAsesor"` sobre `visuals/` de la copia — sin coincidencias.
6. **Ningún visual fuera del lienzo**: verificado programáticamente que los 55 visuales cumplen `x≥0, y≥0, x+width≤1280, y+height≤720` — 0 fuera de límites.
7. **Panel de satisfacción con 4 barras independientes**: confirmado — `sc_satisf_bar_satisf`, `sc_satisf_bar_claridad`, `sc_satisf_bar_utilidad`, `sc_satisf_bar_dinamismo` son 4 visuales `barChart` separados, cada uno con una sola medida (no un único visual con 4 medidas apiladas).
8. **Logo, íconos, badge y botón Home visibles**: `sc_logo_connect` (imagen del logo registrado), `sc_kpi_icon_*` (6 círculos de acento), `sc_pilot_badge`/`sc_pilot_note` (reposicionados sin recorte), `sc_home_btn` (relleno `#F15B2B` sólido) y `sc_home_label` (`← Volver a Home`, blanco) — todos confirmados presentes en el sistema de archivos con el contenido esperado.

## 14. Confirmación de no modificación fuera de alcance (Revisión 2)

No se modificó: medidas DAX, `relationships.tmdl`, `expressions.tmdl`, `database.tmdl` (`compatibilityLevel` permanece en `1606`), `RutaCarpetaData`, tema global, `Home`, ninguna de las otras 5 páginas internas, `pages.json`, `report.json`, ni ningún archivo `Data/*.xlsx`.

## 15. Estado final de `git status` (Revisión 2, vigente)

```
 M .../sc_chart_callcenter/visual.json
 D .../sc_chart_jornada/visual.json
 M .../sc_header_accent/visual.json
 M .../sc_header_panel/visual.json
 M .../sc_home_btn/visual.json
 M .../sc_home_hitzone/visual.json
 M .../sc_home_label/visual.json
 D .../sc_kpi_claridad/visual.json
 D .../sc_kpi_coment/visual.json
 D .../sc_kpi_dinamismo/visual.json
 D .../sc_kpi_indice/visual.json
 D .../sc_kpi_satisf/visual.json
 D .../sc_kpi_total/visual.json
 D .../sc_kpi_utilidad/visual.json
 M .../sc_nota_cap_panel/visual.json
 M .../sc_nota_cap_text/visual.json
 M .../sc_slicer_callcenter/visual.json
 M .../sc_slicer_callcenter_label/visual.json
 M .../sc_slicer_fecha/visual.json
 M .../sc_slicer_fecha_label/visual.json
 M .../sc_slicer_jornada/visual.json
 M .../sc_slicer_jornada_label/visual.json
 M .../sc_subtitle/visual.json
 D .../sc_tabla_formador/visual.json
 M .../sc_title/visual.json
?? Data/
?? Outputs/43_resultado_sc5_rediseno_visual_satisfaccion_capacitaciones.md
?? .../sc_chart_capxfecha/
?? .../sc_filter_panel/
?? .../sc_header_separator/
?? .../sc_kpi_callcenters/  .../sc_kpi_capacitaciones/  .../sc_kpi_comentarios/
?? .../sc_kpi_icon_callcenters/  .../sc_kpi_icon_capacitaciones/  .../sc_kpi_icon_comentarios/
?? .../sc_kpi_icon_respuestas/  .../sc_kpi_icon_satisfaccion/  .../sc_kpi_icon_ultima/
?? .../sc_kpi_line_callcenters/  .../sc_kpi_line_capacitaciones/  .../sc_kpi_line_comentarios/
?? .../sc_kpi_line_respuestas/  .../sc_kpi_line_satisfaccion/  .../sc_kpi_line_ultima/
?? .../sc_kpi_respuestas/  .../sc_kpi_respuestas_panel/  .../sc_kpi_satisfaccion/  .../sc_kpi_ultima/
?? .../sc_logo_connect/
?? .../sc_panel_satisf_container/  .../sc_panel_satisf_title/  .../sc_panel_satisf_hint/
?? .../sc_pilot_badge/  .../sc_pilot_note/
?? .../sc_satisf_bar_claridad/  .../sc_satisf_bar_dinamismo/  .../sc_satisf_bar_satisf/  .../sc_satisf_bar_utilidad/
?? .../sc_satisf_label_claridad/  .../sc_satisf_label_dinamismo/  .../sc_satisf_label_satisf/  .../sc_satisf_label_utilidad/
?? .../sc_tabla_callcenter/  .../sc_tabla_comentarios/
```

(Rutas abreviadas con `...` = `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/`; ver `git status --porcelain` para las rutas completas.)

## 16. Punto de control — sin commit (vigente)

No se creó ningún commit, conforme al punto de control explícito. Se solicita al usuario:

1. Abrir el `.pbip` en Power BI Desktop.
2. Confirmar que `p14_satisfaccion_capacitaciones_v2` carga sin errores.
3. Confirmar específicamente: (a) el logo y el botón Home naranja se ven, (b) las 6 tarjetas KPI muestran círculo + línea inferior, (c) el panel de satisfacción muestra **4 barras separadas** (no una apilada), (d) **ambas tablas muestran filas de datos reales**, (e) el gráfico de jornada ya no aparece en el lienzo principal, (f) nada se ve cortado.
4. Aplicar manualmente el filtro "sin vacíos" en `sc_tabla_comentarios` (paso obligatorio, ver pendiente §"Revisión 2" punto 1) antes de dar por cerrada esta fase.
5. Compartir una nueva captura de la página `v2` para comparar contra el mockup.

## Revisión 8 — cierre técnico previo a aprobación visual

Ronda de corrección dirigida a los 6 bloqueos señalados sobre la captura previa (panel con 3 métricas visibles + scroll, orden de métricas incorrecto, eje de fecha con "12 a. m.", fechas invertidas (`07/10/2026`), fila `CAPITALS` vacía en la tabla de detalle, nombres truncados en el gráfico de call center). Todos los cambios se hicieron directamente vía TMDL/PBIR; no se delegó ningún ajuste al usuario.

### 1. Columna `Fecha Eje` — causa raíz definitiva del "12 a. m."

La Revisión 7 ya había reescrito `sc_chart_capxfecha` para usar una proyección plana de `Dim_Calendario[Fecha]` en vez de la jerarquía Año/Trimestre/Mes/Día, pero el usuario siguió viendo `12 a. m.` en capturas posteriores. Causa: `Dim_Calendario[Fecha]` conserva a nivel de modelo una `variation` (`isDefault`) que apunta a la jerarquía automática de fechas (`LocalDateTable_fe450612-...`), añadida por una sesión previa de Desktop. Aunque el JSON del visual referenciaba la columna plana, Desktop puede seguir aplicando la variación por defecto al reseleccionar el campo.

**Corrección**: se creó una columna calculada nueva, sin ninguna variación asociada (por ser recién creada):

```dax
column 'Fecha Eje' = DATE(YEAR(Dim_Calendario[Fecha]), MONTH(Dim_Calendario[Fecha]), DAY(Dim_Calendario[Fecha]))
    formatString: dd/MM
    summarizeBy: none
```

`sc_chart_capxfecha` se reenlazó a `Dim_Calendario.'Fecha Eje'` (categoría) con `sortDefinition` ascendente explícito por la misma columna. No se escribió `lineageTag` a mano (lo generará Desktop al guardar).

### 2. Panel de satisfacción — 4 métricas sin scroll, en el orden correcto

Se mantuvo la arquitectura de la Revisión 7 (`Dim_MetricaSatisfaccion` desconectada + medida dinámica `Valor Metrica Satisfaccion` + `barChart` único), que es estructuralmente correcta. Los defectos reportados eran de **layout** y de una propiedad de orden que Desktop había eliminado al reprocesar la tabla:

- Se restauraron en `Dim_MetricaSatisfaccion.tmdl` las propiedades `sortByColumn: Orden` (columna `Metrica`) e `isHidden` (columna `Orden`), que Desktop había descartado silenciosamente en una sesión anterior — sin esto, el orden caía a alfabético/natural en vez de Satisfacción→Claridad→Utilidad→Dinamismo.
- Se confirmó estáticamente que la tabla auxiliar contiene exactamente: `Satisfacción=1, Claridad=2, Utilidad=3, Dinamismo=4`.
- Se eliminó el textbox de ayuda `sc_panel_satisf_hint` para liberar espacio vertical.
- Se amplió el área del gráfico (`sc_panel_satisf_chart`: `y=300, height=112`, antes `y=318, height=90`) y se añadió `categoryAxis.innerPadding: 10D` para reducir el espacio entre barras.
- Se reposicionaron en cascada `sc_panel_satisf_title` (`y=284, height=14`) y `sc_kpi_respuestas_panel` (`y=416, height=26`) para que las 4 barras quepan sin desplazamiento vertical dentro del contenedor (`y=282` a `y=446`).

### 3. Fechas visibles — formato `dd/MM/yyyy` con locale explícito

Se confirmó que `sc_kpi_ultima` y la columna "Última fecha" de `sc_tabla_callcenter` ya estaban correctamente enlazados a la medida de texto `Ultima Capacitacion Texto` (no se habían revertido). El problema remanente era de formato/locale, no de enlace:

```dax
measure 'Ultima Capacitacion Texto' = FORMAT([Ultima Capacitacion], "dd/MM/yyyy", "es-CO")
```

(se agregó el tercer parámetro `"es-CO"`, explícito, para no depender de la configuración regional de Desktop/Windows). No se modificó la expresión original de `[Ultima Capacitacion]`. Adicionalmente, se aplicó `formatString: 'dd/MM/yyyy'` a la columna `Fact_SatisfaccionCapacitacion.Fecha` en `sc_tabla_comentarios`, que no tenía formato explícito antes.

### 4. Fila vacía `CAPITALS` en la tabla de detalle

`Dim_CallCenter` es una dimensión compartida construida por unión de los call centers observados en las 3 encuestas; `CAPITALS` existe en esa unión pero no tiene registros en `Fact_SatisfaccionCapacitacion`, por lo que aparecía como fila con medidas vacías. En vez de agregar un filtro adicional (`filterConfig`), se optó por la alternativa más estable que el propio usuario propuso: cambiar la columna de categoría de `sc_tabla_callcenter` de `Dim_CallCenter[CallCenter]` a `Fact_SatisfaccionCapacitacion[CallCenter]`, que por construcción solo contiene los call centers con respuestas reales de esta encuesta. La relación `Dim_CallCenter → Fact_SatisfaccionCapacitacion` sigue intacta, así que el segmentador de Call Center (que filtra vía `Dim_CallCenter`) sigue funcionando igual sobre esta tabla.

### 5. Nombres truncados en el gráfico por call center

Se aplicó el mismo cambio de categoría a `sc_chart_callcenter` (de `Dim_CallCenter[CallCenter]` a `Fact_SatisfaccionCapacitacion[CallCenter]`), eliminando la categoría fantasma `CAPITALS` (con valor 0, no en blanco, por lo que no era descartada automáticamente por el visual) y dejando únicamente los 5 call centers con datos reales — esto por sí solo libera ~20% más ancho por etiqueta. Adicionalmente: ancho del visual `440px → 452px` (usando el margen disponible antes del siguiente visual, sin invadirlo) y tamaño de fuente del eje de categoría `7D → 6D`. No se modificaron nombres reales ni se crearon abreviaciones en el modelo. Se mantuvo el orden descendente por valor y las etiquetas de dato. No se introdujo ninguna propiedad de rotación/wrap sin precedente verificado en este proyecto (no existe uso previo de tal propiedad en ningún otro gráfico del informe), para no repetir el patrón de "adivinar propiedades aisladas" que falló en revisiones anteriores.

### 6. Metadatos de Auto Date/Time — decisión archivo por archivo

Se verificó con `git diff` contra `HEAD` cuáles artefactos de Auto Date/Time eran nuevos en esta sesión, y con `grep` en todo `PBI/PBI_Indicadores.Report/definition` que ninguna jerarquía (`HierarchyLevel`, `Variación`, `LocalDateTable`, `Jerarquía de fechas`) siguiera referenciada por algún visual, antes de tocar nada:

| Archivo | Diagnóstico | Decisión |
| --- | --- | --- |
| `relationships.tmdl` | Relación `d3b3eefb-...` (`Dim_Calendario.Fecha → LocalDateTable_fe450612....Date`) no existía en `HEAD`; generada solo por la jerarquía automática; sin referencia activa tras el cambio a `Fecha Eje` | **Revertida** (eliminada); el archivo vuelve a coincidir exactamente con `HEAD` |
| `Dim_Calendario.tmdl` | Bloque `variation Variación` + `changedProperty = DataType` + `annotation UnderlyingDateTimeDataType = Date` en la columna `Fecha`: no existía en `HEAD`, generado solo por la jerarquía, sin referencia activa | **Revertido** (eliminado); se conserva la columna `Fecha Eje` (adición intencional de esta revisión) |
| `LocalDateTable_fe450612-....tmdl` | Tabla oculta nueva (no rastreada por git), generada solo por la jerarquía, sin ninguna referencia restante en el modelo ni en el informe | **Eliminado** el archivo |
| `model.tmdl` | Línea `ref table LocalDateTable_fe450612-...` no existía en `HEAD` | **Revertida** (eliminada); se conserva `ref table Dim_MetricaSatisfaccion` (adición intencional previa) |
| `Dim_Calendario.tmdl` — `summarizeBy: sum` + `annotation SummarizationSetBy = Automatic` en columnas numéricas (`Anio`, `MesNumero`, `Trimestre`, `SemanaAnio`, `DiaMes`, `DiaSemanaNumero`) | Comportamiento **distinto** de Desktop (autodetección de agregación en columnas numéricas), no generado por la jerarquía de fechas — no cumple la condición (a) del criterio de reversión | **Se conserva sin cambios** (ruido conocido, ya documentado en `CLAUDE.md`, fuera del alcance de esta corrección) |
| `cultures/es-ES.tmdl` | +237 líneas de sinónimos Q&A (`PowerBI.VisualColumnRename`, `State: Suggested/Generated`); confirmado en revisiones anteriores como metadato benigno de un motor distinto (esquema lingüístico de Q&A), no relacionado con la jerarquía de fechas | **Se conserva sin cambios** (no cumple condición (a); no representa riesgo funcional) |

Otras 4 tablas ocultas de Auto Date/Time (`DateTableTemplate_2973bde6...`, `LocalDateTable_225f0da6...`, `LocalDateTable_082769f1...`, `LocalDateTable_c16eb748...`) **ya existían en `HEAD`** antes de esta sesión — están fuera del alcance de esta corrección y no se tocaron, conforme al criterio explícito del usuario y a la nota ya documentada en `CLAUDE.md` sobre limpiarlas manualmente desde Desktop.

### Validaciones realizadas

- **JSON**: los 47 archivos `visual.json` de `p14_satisfaccion_capacitaciones_v2` parsean sin error (`json.load`).
- **Canvas**: ningún visual excede los límites 1280×720.
- **TMDL**: paréntesis/llaves/corchetes balanceados en `Dim_Calendario.tmdl`, `_Medidas Capacitacion.tmdl`, `Dim_MetricaSatisfaccion.tmdl`, `model.tmdl`, `relationships.tmdl`.
- **Alcance**: `git diff --stat` confirma vacío para `p14_satisfaccion_capacitaciones` (original), la página `Home` (`67eff42d82e1c9c15b84`) y `expressions.tmdl` (Power Query) — ninguno de los tres fue tocado.
- **Privacidad**: sin coincidencias de `NombreFormador`/`NombreLider`/`NombreAsesor` en ningún visual de la página `v2`.
- **`relationships.tmdl`** vuelve a coincidir byte a byte con `HEAD` tras la reversión.

### Objetos DAX nuevos autorizados hasta la fecha

`Dim_MetricaSatisfaccion` (tabla desconectada), `Valor Metrica Satisfaccion` (medida dinámica), `Ultima Capacitacion Texto` (medida de formato), `Fecha Eje` (columna calculada, Revisión 8). Ningún otro objeto DAX nuevo.

**Estado: Pendiente únicamente de aprobación visual final.** No se creó ningún commit. SC-6 no se ha iniciado ni autorizado.

`SC-5` solo se aprueba después de esa comparación. No se hace push remoto.

## Revisión 9 — corrección final con Codex

### Diagnóstico recibido de Claude Code

La Revisión 8 dejó documentado que `SC-5` estaba técnicamente lista y pendiente solo de aprobación visual. Sin embargo, la captura revisada por el usuario confirmó cuatro bloqueos visibles: el eje de `sc_chart_capxfecha` seguía mostrando hora (`12 a. m.`), el panel de satisfacción mostraba solo tres métricas con desplazamiento vertical, `Última capacitación` seguía viéndose como `07/10/2026`, y las etiquetas `INTERACTIVO`/`ONE CONTACT` continuaban truncadas. Por instrucción expresa, no se avanza a `SC-6`.

### Estado real encontrado

- Power BI Desktop no estaba abierto al iniciar la revisión.
- El último commit confirmado fue `5469ded fix(data): restaurar rutas locales de fuentes OneDrive`.
- `sc_chart_capxfecha` ya estaba enlazado a `Dim_Calendario[Fecha Eje]`, sin jerarquía automática ni `HierarchyLevel`, pero `Fecha Eje` seguía devolviendo un valor de fecha.
- `Dim_MetricaSatisfaccion` contiene las cuatro filas y conserva `Metrica sortByColumn: Orden`; `sc_panel_satisf_chart` usa la dimensión desconectada y la medida `Valor Metrica Satisfaccion`.
- `sc_kpi_ultima` y `sc_tabla_callcenter` usan `Ultima Capacitacion Texto`, pero la medida dependía todavía de formateo de fecha. `sc_tabla_comentarios` seguía usando la columna `Fact_SatisfaccionCapacitacion[Fecha]`, que conserva `Short Date` y variación automática.
- `sc_chart_callcenter` ya usa `Fact_SatisfaccionCapacitacion[CallCenter]`, sin `CAPITALS`, pero el ancho útil seguía siendo insuficiente para las etiquetas largas.

### Causa raíz de cada bloqueo

- **Gráfico por fecha**: la columna `Fecha Eje` existía, pero era una fecha calculada (`DATE(...)`). En un `lineChart`, Desktop puede tratarla como eje temporal y renderizar etiquetas con componente horario aunque el campo no use jerarquía.
- **Panel de satisfacción**: el modelo y la medida eran correctos; el problema era de espacio útil del visual. El `barChart` tenía 112 px de alto para cuatro categorías, etiquetas y eje 0-5, provocando scroll y ocultamiento de `Dinamismo`.
- **Última capacitación / fechas**: la tarjeta y la tabla de detalle usaban la medida textual correcta, pero la expresión seguía apoyándose en `FORMAT` de fecha; la tabla de comentarios no usaba la medida y permanecía enlazada a una columna `Short Date`.
- **Etiquetas de call center**: tras quitar `CAPITALS`, el visual seguía limitado a 452 px y fuente de eje 6D; el espacio por categoría aún no alcanzaba para `INTERACTIVO` y `ONE CONTACT`.

### Correcciones aplicadas

- `Dim_Calendario[Fecha Eje]` se reutilizó como etiqueta textual `dd/MM` construida con `DAY`/`MONTH`, con `dataType: string`, `summarizeBy: none` y `sortByColumn: Fecha`.
- `sc_chart_capxfecha` mantiene `Fecha Eje` como categoría, pero ordena ascendentemente por `Dim_Calendario[Fecha]` y queda sin dependencia de jerarquía automática.
- `Ultima Capacitacion Texto` se reforzó para construir `dd/MM/yyyy` manualmente con `DAY`, `MONTH` y `YEAR`; no se modificó `[Ultima Capacitacion]`.
- Se agregó `Fact_SatisfaccionCapacitacion[Fecha Texto]` como columna calculada textual `dd/MM/yyyy` para filas de detalle/comentarios donde una medida no sirve por contexto de fila.
- `sc_tabla_comentarios` se reenlazó de `Fact_SatisfaccionCapacitacion[Fecha]` a `Fact_SatisfaccionCapacitacion[Fecha Texto]`, manteniendo el encabezado visible como `Fecha`.
- `sc_panel_satisf_container` se amplió de 164 a 190 px de alto; `sc_panel_satisf_chart` pasó de 112 a 136 px, con fuente 8D e `innerPadding` 4D; `sc_kpi_respuestas_panel` bajó a `y=436` y 20 px de alto.
- `sc_chart_callcenter` se amplió a 500 px de ancho y 180 px de alto; la fuente del eje de categorías bajó a 5D.
- `sc_chart_capxfecha` se movió a `x=544`, con 304 px de ancho y 180 px de alto para preservar el layout de tres paneles.

### Archivos modificados

- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_chart_callcenter/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_chart_capxfecha/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_panel_satisf_container/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_panel_satisf_title/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_panel_satisf_chart/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_respuestas_panel/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_tabla_comentarios/visual.json`
- `Outputs/43_resultado_sc5_rediseno_visual_satisfaccion_capacitaciones.md`

### Objetos reutilizados

- `Dim_MetricaSatisfaccion`
- `Valor Metrica Satisfaccion`
- `Ultima Capacitacion Texto`
- `Dim_Calendario[Fecha Eje]`
- `sc_chart_capxfecha`
- `sc_panel_satisf_chart`
- `sc_kpi_ultima`
- `sc_tabla_callcenter`
- `sc_tabla_comentarios`

### Objetos eliminados

Ninguno en la Revisión 9.

### Validaciones técnicas

- `git diff -- "PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/"` quedó sin salida: la página original protegida sigue intacta.
- Los `visual.json` de `p14_satisfaccion_capacitaciones_v2` parsean como JSON válido.
- Ningún visual de `p14_satisfaccion_capacitaciones_v2` excede el lienzo `1280x720`.
- No hay referencias a `HierarchyLevel`, `LocalDateTable`, `Jerarquía` o `Variación` en `sc_chart_capxfecha` ni en `sc_tabla_comentarios`.
- `expressions.tmdl`, `relationships.tmdl`, `database.tmdl` y la página Home no fueron modificados por esta revisión.
- `RutaCarpetaData` sigue apuntando a `C:\Users\eclavijo\OneDrive\PBI_Indicadores\Data`.
- `compatibilityLevel` permanece en `1606`.

### Diferencias todavía pendientes frente al mockup

Queda pendiente la validación visual en Power BI Desktop. En particular, confirmar que Desktop no reescriba `Fecha Eje`, que `sc_panel_satisf_chart` muestre las cuatro métricas sin scroll, que la tarjeta y ambas tablas muestren fechas `dd/MM/yyyy`, y que `INTERACTIVO`/`ONE CONTACT` ya no aparezcan truncados. `SC-6` sigue sin iniciarse.

## Revisión 10 — corrección de causa raíz en fechas

### Valores originales encontrados en Excel

Se abrió en modo lectura `Data/Satisfacción capacitación (Responses).xlsx`. La columna real es `Timestamp`; Excel la almacena como número serial, no como texto. El estilo de celda del `Timestamp` es `m/d/yyyy h:mm:ss`.

| Valor original del Excel | Valor actual en el modelo | Valor correcto esperado | Filas | Call center |
| --- | --- | --- | ---: | --- |
| serial `46207.379225..46207.480090` (`07/04/2026` en formato Excel `m/d/yyyy`) | `04/07/2026` | `04/07/2026` | 7 | `ONE CONTACT` |
| serial `46209.673694..46209.678398` (`07/06/2026` en formato Excel `m/d/yyyy`) | `06/07/2026` | `06/07/2026` | 24 | `INTERACTIVO` |
| serial `46241.660440..46241.756134` (`08/07/2026` en formato Excel `m/d/yyyy`) | `07/08/2026` | `08/07/2026` | 16 | `GNP` |
| serial `46272.486748..46272.600417` (`09/07/2026` en formato Excel `m/d/yyyy`) | `07/09/2026` | `09/07/2026` | 18 | `BRM` |
| serial `46302.503623..46302.516331` (`10/07/2026` en formato Excel `m/d/yyyy`) | `07/10/2026` | `10/07/2026` | 19 | `ATENTO` |

### Paso M que causaba la interpretación incorrecta

No se encontró `Date.FromText`, `DateTime.FromText`, cultura `en-US`/`es-CO` ni reconstrucción manual previa en `SatisfaccionCapacitacion_Limpio`. El archivo Excel ya traía las últimas tres fechas como seriales equivalentes a agosto, septiembre y octubre. El paso:

```powerquery
TimestampComoFechaHora = Table.TransformColumnTypes(TextoLimpio, {{"Timestamp", type datetime}}),
ColumnaFechaAgregada = Table.AddColumn(TimestampComoFechaHora, "Fecha", each if [Timestamp] = null then null else Date.From([Timestamp]), type date)
```

preservaba correctamente el serial de Excel, pero ese serial ya representaba una fecha invertida frente al periodo real del piloto.

### Cultura o conversión aplicada

Como el origen no es texto sino serial real de Excel, no se aplicó `Date.FromText(..., [Culture="es-CO"])`. Se agregó un paso acotado `TimestampNormalizado` después de convertir el serial a `datetime` y antes de crear `Fecha`.

La regla corrige solo el patrón confirmado en el Excel de satisfacción: año `2026`, día `7`, meses `8`, `9` o `10`. Esos valores se reconstruyen como julio de 2026 usando el mes original como día real (`08/07`, `09/07`, `10/07`) y se preserva la hora original.

### Fechas anteriores y fechas corregidas

- `07/08/2026` -> `08/07/2026`
- `07/09/2026` -> `09/07/2026`
- `07/10/2026` -> `10/07/2026`

Las fechas ya correctas `04/07/2026` y `06/07/2026` no se modifican.

### Efecto esperado sobre el gráfico por fecha

`Fact_SatisfaccionCapacitacion[Fecha]` vuelve a caer dentro del rango real de julio cubierto por `Dim_Calendario`. Por lo tanto, `sc_chart_capxfecha` ya no debería agrupar las tres capacitaciones en `(En blanco)` y debería mostrar las cinco fechas reales:

- `04/07`
- `06/07`
- `08/07`
- `09/07`
- `10/07`

### Efecto esperado sobre Última capacitación

Como `[Ultima Capacitacion]` usa `MAX(Fact_SatisfaccionCapacitacion[Fecha])`, después del refresco debe devolver `10/07/2026`. `Ultima Capacitacion Texto` se conserva para mostrar explícitamente `dd/MM/yyyy` en tarjeta y tabla de detalle.

### Objetos temporales que se conservan

- `Dim_Calendario[Fecha Eje]`: se conserva porque da la etiqueta `dd/MM` del gráfico y evita hora en el eje.
- `Fact_SatisfaccionCapacitacion[Fecha Texto]`: se conserva temporalmente porque la tabla de comentarios necesita una fecha textual por fila en `dd/MM/yyyy`.
- `[Ultima Capacitacion Texto]`: se conserva porque la tarjeta y tabla de detalle necesitan formato inequívoco `dd/MM/yyyy`.

Estos objetos pueden evaluarse para limpieza posterior en `SC-8`, pero no se eliminan en `SC-5` porque están siendo usados por visuales.

### Archivos modificados

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `Outputs/43_resultado_sc5_rediseno_visual_satisfaccion_capacitaciones.md`

### Validaciones realizadas

- Power BI Desktop estaba cerrado antes de editar.
- `Data/Satisfacción capacitación (Responses).xlsx` se inspeccionó en modo lectura; no se modificó.
- Conteos esperados desde el Excel con corrección aplicada: 5 capacitaciones, 5 call centers, 84 respuestas, última capacitación `10/07/2026`.
- `RutaCarpetaData` permanece igual.
- `compatibilityLevel` permanece en `1606`.
- No se agregaron objetos DAX nuevos.
- `SC-6` sigue sin iniciarse.

## Aprobación visual y cierre de SC-5

La validación visual final de `SC-5` fue aprobada por el usuario después de la corrección de causa raíz en fechas.

### Resultado visual confirmado

- `sc_chart_capxfecha` muestra cinco fechas reales: `04/07`, `06/07`, `08/07`, `09/07` y `10/07`.
- Ya no aparece la categoría `(En blanco)`.
- El eje de fecha no muestra horas.
- `sc_kpi_ultima` muestra `10/07/2026`.
- `sc_panel_satisf_chart` presenta las cuatro métricas: Satisfacción, Claridad, Utilidad y Dinamismo.
- Las cuatro barras son visibles y no presentan desplazamiento vertical.
- Los cinco call centers se muestran completos.
- Las tablas presentan fechas correctas.
- `sc_tabla_comentarios` excluye valores vacíos, `Sin comentario` y `"."`.
- El diseño conserva cercanía suficiente con el mockup aprobado.

### Corrección final de fechas

La causa raíz quedó corregida en Power Query (`SatisfaccionCapacitacion_Limpio`) mediante el paso `TimestampNormalizado`, que normaliza los seriales de Excel del piloto que llegaban como agosto, septiembre y octubre, preservando la hora y dejando `Fact_SatisfaccionCapacitacion[Fecha]` dentro de julio de 2026.

### Ajuste menor de tarjeta de respuestas

Antes del commit se ajustó `sc_kpi_respuestas_panel`: se desplazó de `y=436` a `y=434` y se aumentó de `height=20` a `height=26`, manteniendo la medida `Total Respuestas Capacitacion` y la etiqueta `Respuestas`, para evitar recorte del valor `84`.

### Confirmaciones de alcance

- La página original `p14_satisfaccion_capacitaciones` permanece intacta.
- Home permanece intacto.
- `RutaCarpetaData` permanece sin cambios.
- `compatibilityLevel` permanece en `1606`.
- No se modificaron relaciones.
- No se agregaron archivos de `Data/`, Excel, `.pbi`, cachés ni consultas DAX locales al cierre de la fase.

### Estado final

`SC-5` queda cerrada y aprobada.

Próximo paso autorizado por secuencia: `SC-6 — Configuración de interacciones`.
