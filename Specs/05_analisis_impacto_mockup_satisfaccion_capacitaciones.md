# Análisis de Impacto — Adaptar la página "Satisfacción de capacitaciones" al mockup

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de documento | Análisis de impacto para posible evolución de una página existente |
| Página analizada | `p14_satisfaccion_capacitaciones` ("Satisfacción de capacitaciones") |
| Mockup de referencia | [`Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png`](../Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png) |
| Fecha del análisis | 2026-07-21 |
| Estado | Diagnóstico sin implementación — no se modificó PBIR, TMDL, Power Query, DAX, relaciones, visuales ni `Data/*.xlsx` |

---

## 1. Resumen ejecutivo

El mockup propone una reorganización sustancial de la página "Satisfacción de capacitaciones": introduce el concepto de **"capacitación" como sesión** (18 capacitaciones) distinto de **"respuesta de encuesta"** (84 respuestas), agrega un panel de detalle interactivo por call center seleccionado, un gráfico de tendencia por fecha, una tabla de comentarios destacados, y reemplaza la tabla actual (agrupada por formador/líder, con nombres reales) por una tabla agrupada por call center (sin nombres individuales).

De los 6 indicadores solicitados en la tarea, **3 no son calculables con el modelo TMDL actual sin una decisión de negocio previa**: `Fact_SatisfaccionCapacitacion` tiene grano de **1 fila = 1 respuesta de encuesta**, no de sesión de capacitación, y no existe ninguna columna que identifique de forma única una sesión (no hay `IdCapacitacion`, `TemaCapacitacion` ni similar en el origen). "Capacitaciones realizadas", "Capacitaciones por fecha" y "Capacitaciones por call center" requieren definir qué combinación de columnas constituye "una capacitación única" antes de poder implementarse — aunque sea como medida DAX con `DISTINCTCOUNT` sobre una clave compuesta. Los otros 3 indicadores (`Última capacitación`, `Satisfacción por call center seleccionado`, `Comentarios destacados`) sí son calculables con el modelo actual: 2 requieren medidas DAX nuevas pero técnicamente simples (sin ambigüedad de grano), y 1 no requiere medida nueva, solo un visual con filtro.

Dado el volumen de cambios (nueva IA de la fila de KPI, medida nueva con dependencia de negocio, tabla reestructurada, 3 visuales nuevos, cambio de estilo de segmentador), la recomendación es **prototipar en una copia de la página**, no editar `p14_satisfaccion_capacitaciones` en sitio — ver §6.

## 2. Página actual — inventario completo

Fuente: `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/` (29 objetos visuales), cotejado contra [Docs/03_mapa_reporte_paginas_visuales.md](../Docs/03_mapa_reporte_paginas_visuales.md) §4 y [Docs/02_catalogo_medidas_dax.md](../Docs/02_catalogo_medidas_dax.md).

- **Lienzo:** 1280×720 (`FitToPage`), igual que las demás 6 páginas.
- **KPI (7 tarjetas):** `sc_kpi_satisf` (Satisfacción promedio), `sc_kpi_claridad` (Claridad promedio), `sc_kpi_utilidad` (Utilidad promedio), `sc_kpi_dinamismo` (Dinamismo promedio), `sc_kpi_indice` (Índice global), `sc_kpi_coment` (% con comentarios), `sc_kpi_total` (Total respuestas).
- **Gráficos (2):** `sc_chart_callcenter` (columnas, eje Y = `Indice Global Capacitacion`, por `Dim_CallCenter.CallCenter`), `sc_chart_jornada` (barras, `Indice Global Capacitacion` por `Dim_Jornada.Jornada`).
- **Tabla (1):** `sc_tabla_formador` — columnas `NombreFormador`, `NombreLider`, `Total Respuestas Capacitacion`, `Indice Global Capacitacion`. Expone **nombres reales** (riesgo ya documentado en [Docs/06_publicacion_powerbi.md](../Docs/06_publicacion_powerbi.md) §2).
- **Segmentadores (3):** Fecha (`Dim_Calendario.Fecha`, modo `Dropdown`), Call Center (`Dim_CallCenter.CallCenter`, `Dropdown`), Jornada (`Dim_Jornada.Jornada`, `Dropdown`) — cada uno con su etiqueta como visual de texto independiente.
- **Encabezado/navegación:** panel y acento de encabezado, título, subtítulo, botón "Volver a Home" + etiqueta + zona clicable.
- **Nota metodológica:** panel + texto (`sc_nota_cap_panel`/`sc_nota_cap_text`) con la advertencia de datos piloto y validación de alias.

## 3. El mockup — inventario de lo propuesto

- **Encabezado:** logo, título "Satisfacción de capacitaciones", subtítulo, insignia "Datos piloto sujetos a validación", botón "Volver a Home" — **equivalente a lo ya existente**, sin cambios de fondo.
- **Filtros (3):** Fecha como **rango con dos casillas de fecha** (`02/07/2026` – `15/07/2026`, con ícono de calendario en cada casilla), Call Center (`Todas`), Jornada (`Todas`) — mismos 3 campos que hoy, pero el segmentador de Fecha cambia de estilo.
- **KPI (6 tarjetas, distintas a las actuales):** Capacitaciones realizadas (18), Respuestas recibidas (84), Call centers capacitados (5), Satisfacción promedio 1–5 (4,8), Última capacitación (15/07/2026), % con comentarios (54,8%).
- **Fila de 3 paneles:**
  1. "Capacitaciones por call center" — columnas, conteo de capacitaciones (no de respuestas) por call center.
  2. "Capacitaciones por fecha" — línea, conteo de capacitaciones por fecha.
  3. "Satisfacción del call center seleccionado" — barras horizontales (Satisfacción, Claridad, Utilidad, Dinamismo) + valor de Respuestas, que **reacciona a la selección hecha en el panel 1** ("Selecciona un call center en el gráfico para actualizar").
- **Tabla "Detalle por call center":** columnas Call Center, Capacitaciones, Respuestas, Satisfacción, Claridad, Utilidad, Dinamismo, Última fecha, con fila Total — agrupada por **call center**, no por formador/líder.
- **Tabla "Comentarios destacados (sin vacíos)":** lista de comentarios individuales con call center y fecha, excluyendo los marcados como `"Sin comentario"`.
- **Nota de pie:** mensaje de muestra piloto, equivalente al actual.

## 4. Comparación — qué existe, qué se ajusta, qué falta

| Elemento del mockup | Estado | Detalle |
|---|---|---|
| Encabezado, título, subtítulo, insignia, botón Home | **Ya existe** | Sin cambio de fondo; el mockup usa el mismo patrón visual del reporte actual. |
| Segmentador Call Center (`Todas`) | **Ya existe** | `sc_slicer_callcenter`, mismo campo, mismo modo `Dropdown`. |
| Segmentador Jornada (`Todas`) | **Ya existe** | `sc_slicer_jornada`, mismo campo, mismo modo `Dropdown`. |
| Segmentador Fecha (rango con 2 casillas) | **Debe ajustarse** | Hoy es `sc_slicer_fecha` en modo `Dropdown` (lista de fechas); el mockup usa estilo "Between" con 2 casillas de calendario — cambio de modo del slicer, no de campo. |
| KPI "Satisfacción promedio" | **Debe ajustarse** | Ya existe como `sc_kpi_satisf`; se conserva el valor pero cambia de posición/tamaño dentro de una fila de 6 en vez de 7. |
| KPI "Claridad", "Utilidad", "Dinamismo", "Índice global" (tarjetas independientes) | **Debe ajustarse** | El mockup **no las muestra como tarjetas de la fila superior**; su información se reubica en el panel "Satisfacción del call center seleccionado" y en la tabla de detalle. Implica remover 4 tarjetas de la fila de KPI actual. |
| KPI "Total respuestas" | **Debe ajustarse** | Se renombra/reencuadra como "Respuestas recibidas", concepto igual (`Total Respuestas Capacitacion`), solo cambia texto/ícono. |
| KPI "% con comentarios" | **Ya existe** | `sc_kpi_coment` usa `% Comentarios Capacitacion`, mismo valor solicitado. |
| KPI "Capacitaciones realizadas" | **Falta — bloqueado** | No existe ninguna medida de conteo de sesiones; requiere definición de negocio antes de poder crearse (§5, §7). |
| KPI "Call centers capacitados" | **Falta** | No existe una medida `DISTINCTCOUNT` de call centers; técnicamente simple, sin ambigüedad de grano. |
| KPI "Última capacitación" | **Falta** | No existe una medida de fecha máxima; técnicamente simple (`MAX(Fecha)`), sin ambigüedad de grano. |
| Gráfico "Capacitaciones por call center" (conteo) | **Falta — bloqueado** | `sc_chart_callcenter` existe pero grafica el **índice** (promedio Likert), no un **conteo de sesiones**; es un dato distinto, no un ajuste cosmético. Bloqueado por la misma dependencia de grano que el KPI de capacitaciones. |
| Gráfico "Capacitaciones por fecha" | **Falta — bloqueado** | No existe ningún gráfico de tendencia en esta página hoy. Misma dependencia de grano. |
| Panel "Satisfacción del call center seleccionado" | **Falta (viable)** | No existe hoy; es visual nuevo, pero las 4 medidas que necesita (Satisfacción/Claridad/Utilidad/Dinamismo promedio) **ya existen** — solo requiere el visual y la interacción cruzada estándar de Power BI. |
| Gráfico "Índice global por jornada" (`sc_chart_jornada`, actual) | **No aparece en el mockup** | El mockup no incluye ningún desglose por jornada más allá del segmentador — decisión pendiente: ¿se elimina, se reubica, o el mockup es una vista parcial? Ver §7. |
| Tabla "Detalle por call center" | **Falta (parcialmente viable)** | Reemplaza a `sc_tabla_formador` (agrupada por formador/líder) con una tabla agrupada por call center. Las columnas Respuestas/Satisfacción/Claridad/Utilidad/Dinamismo ya existen como medidas; las columnas Capacitaciones y Última fecha dependen de las medidas nuevas de §5. |
| Tabla "Comentarios destacados (sin vacíos)" | **Falta (viable)** | No existe hoy. No requiere medida nueva — es una tabla/lista sobre columnas ya existentes (`Comentario`, `CallCenter`, `Fecha`) con un filtro de visual que excluye `"Sin comentario"`. |
| Nota de pie de muestra piloto | **Ya existe** | Mismo mensaje conceptual que `sc_nota_cap_text`. |

## 5. Validación de los 6 indicadores solicitados contra el modelo actual

`Fact_SatisfaccionCapacitacion` (ver `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl`) tiene las columnas: `FechaHora`, `CallCenter`, `Jornada`, `NombreAsesor`, `NombreLider`, `NombreFormador`, `SatisfaccionGeneral`, `Claridad`, `Utilidad`, `Dinamismo`, `Duracion`, `Comentario`, `Fecha`. **No existe ninguna columna que identifique una sesión de capacitación como entidad propia** (sin `IdCapacitacion`, `TemaCapacitacion`, `CodigoSesion` ni equivalente) — el grano de la tabla es 1 fila = 1 respuesta de encuesta, confirmado en [Docs/01_modelo_datos.md](../Docs/01_modelo_datos.md).

| # | Indicador solicitado | ¿Calculable hoy? | Detalle |
|---|---|---|---|
| 1 | **Capacitaciones realizadas** | **No, sin decisión previa** | Requiere contar sesiones distintas, no respuestas. Sin un identificador de sesión en el origen, la única vía es una clave compuesta (p. ej. `Fecha` + `CallCenter` + `NombreFormador`) como proxy de "una capacitación" — una **suposición de negocio**, no un hecho capturado en el formulario. Ver dependencia nueva en §7. |
| 2 | **Capacitaciones por fecha** | **No, sin decisión previa** | Misma dependencia de grano que el punto 1; el conteo de "capacitaciones" por fecha depende de la misma clave compuesta. |
| 3 | **Capacitaciones por call center** | **No, sin decisión previa** | Misma dependencia de grano. El gráfico `sc_chart_callcenter` actual sí puede mostrar `CallCenter`, pero solo con medidas ya existentes (que son promedios, no conteos de sesión). |
| 4 | **Satisfacción por call center seleccionado** | **Sí** | Las 4 medidas necesarias (`Satisfaccion Promedio Capacitacion`, `Claridad Promedio Capacitacion`, `Utilidad Promedio Capacitacion`, `Dinamismo Promedio Capacitacion`) ya existen y responden correctamente a un filtro de `CallCenter`. La interacción "seleccionar en un gráfico y actualizar un panel" es un comportamiento de cruce de Power BI (`Highlight`/interacción de visuales), no requiere DAX nuevo. |
| 5 | **Última capacitación** | **Parcialmente — falta la medida** | Es calculable sin ambigüedad de grano: `MAX(Fact_SatisfaccionCapacitacion[Fecha])` sobre el contexto de filtro. No existe hoy como medida (`Docs/02` no la lista); es una medida nueva pero técnicamente simple, no depende de resolver el grano de "capacitación". |
| 6 | **Comentarios destacados** | **Sí** | No requiere medida DAX nueva. Es una tabla/lista visual sobre las columnas ya existentes `Comentario`, `CallCenter`, `Fecha`, con un filtro de visual `Comentario <> "Sin comentario"` (la limpieza a `"Sin comentario"` para valores vacíos/genéricos ya ocurre en Power Query, ver `Fact_SatisfaccionCapacitacion.tmdl` líneas 127–128). |

**Conclusión de validación:** 2 de 6 indicadores (`4`, `6`) son implementables hoy sin ninguna medida nueva. 1 (`5`) requiere una medida nueva simple, sin dependencia de negocio. 3 (`1`, `2`, `3`) están bloqueados por la misma causa raíz: ausencia de un identificador de sesión de capacitación en el origen de datos.

## 6. Medidas existentes vs. medidas nuevas requeridas

### Reutilizables sin cambio

| Medida | Tabla | Uso en el mockup |
|---|---|---|
| `Satisfaccion Promedio Capacitacion` | `_Medidas Capacitacion` | KPI "Satisfacción promedio", panel de call center seleccionado, columna "Satisfacción" de la tabla |
| `Claridad Promedio Capacitacion` | `_Medidas Capacitacion` | Panel de call center seleccionado, columna "Claridad" de la tabla |
| `Utilidad Promedio Capacitacion` | `_Medidas Capacitacion` | Panel de call center seleccionado, columna "Utilidad" de la tabla |
| `Dinamismo Promedio Capacitacion` | `_Medidas Capacitacion` | Panel de call center seleccionado, columna "Dinamismo" de la tabla |
| `Total Respuestas Capacitacion` | `_Medidas Generales` | KPI "Respuestas recibidas", panel de call center seleccionado ("Respuestas"), columna "Respuestas" de la tabla |
| `% Comentarios Capacitacion` | `_Medidas Capacitacion` | KPI "% con comentarios" |
| `Indice Global Capacitacion` | `_Medidas Capacitacion` | No aparece explícitamente en el mockup, pero se conserva disponible si se decide mantenerlo en la tabla o en otra vista |

### Nuevas requeridas (no creadas en este análisis — requieren autorización explícita)

| Medida propuesta (nombre tentativo) | Complejidad | Bloqueada por decisión de negocio |
|---|---|---|
| `Capacitaciones Realizadas` | Media-alta — `DISTINCTCOUNTX` sobre clave compuesta | **Sí** — depende de qué campos definen "una capacitación única" (§7, dependencia nueva) |
| `Capacitaciones por Fecha` (o reutilizar la anterior con `Fecha` en el eje) | Media-alta — misma lógica que la anterior | **Sí** — misma dependencia |
| `Capacitaciones por Call Center` (o reutilizar `Capacitaciones Realizadas` con `CallCenter` en el eje) | Media-alta — misma lógica que la anterior | **Sí** — misma dependencia |
| `Call Centers Capacitados` | Baja — `DISTINCTCOUNT(Fact_SatisfaccionCapacitacion[CallCenter])` | No |
| `Ultima Capacitacion` | Baja — `MAX(Fact_SatisfaccionCapacitacion[Fecha])` | No |

Si se resuelve la definición de "capacitación única", las 3 primeras filas podrían implementarse con **una sola medida base** (`Capacitaciones Realizadas`) reutilizada en 3 visuales distintos (tarjeta, por fecha, por call center) mediante el contexto de filtro de cada eje — no se necesitarían 3 medidas separadas, solo 1.

## 7. Riesgos y dependencias

### Riesgos

- **Riesgo de grano no resuelto**: sin una definición de negocio de "capacitación única", cualquier implementación de las 3 medidas bloqueadas sería una suposición no verificable contra el origen — mismo patrón de riesgo ya documentado para `% Calidad Promedio Provisional` (dependencia D3) y el catálogo oficial de call centers (D4) en [Docs/05_decisiones_limitaciones_pendientes.md](../Docs/05_decisiones_limitaciones_pendientes.md).
- **Inconsistencia de estilo de segmentador**: el modo `Dropdown` en los 16 segmentadores del reporte es una decisión de diseño **definitiva** ya registrada (`Docs/05` §1.11). Cambiar el segmentador de Fecha a un estilo de rango en esta página, sin tocar las otras 6, introduce una inconsistencia visual entre páginas que debe confirmarse como intencional.
- **Pérdida de visibilidad de "Índice global por jornada"**: el mockup no incluye ningún visual equivalente a `sc_chart_jornada`; si se adapta la página literalmente, ese análisis desaparece de esta vista aunque el segmentador de Jornada se mantenga.
- **Cambio de grano en la tabla de detalle**: pasar de agrupar por formador/líder a agrupar por call center es, en la práctica, **quitar una tabla y agregar otra**, no un ajuste — se pierde el desglose nominal por formador que hoy existe en `sc_tabla_formador` (aunque esto también **reduce la exposición de nombres reales** en una página del informe público, lo cual es una mejora de gobierno de datos frente al riesgo ya señalado en [Docs/06_publicacion_powerbi.md](../Docs/06_publicacion_powerbi.md) §2 — a confirmar si es intencional).
- **Volumen piloto**: los valores del mockup (18 capacitaciones, 84 respuestas, 5 call centers) son ilustrativos del diseño, no necesariamente representativos del volumen real actual en `Data/`; cualquier implementación debe seguir comunicando el `n` visible, conforme a la restricción de primer orden ya establecida en `CLAUDE.md`.
- **Interacción cruzada del panel "Satisfacción del call center seleccionado"**: depende de que el gráfico de origen tenga selección única habilitada; sin esa configuración, una multi-selección dejaría el panel en un estado ambiguo (Power BI usa el contexto de filtro combinado, no necesariamente "el último clic").

### Dependencias

- **Dependencia nueva (candidata a D9, no formalizada aún en `Docs/05`)**: definición oficial de negocio de qué constituye "una capacitación" única — combinación de campos (p. ej. `Fecha` + `CallCenter` + `NombreFormador`) o, idealmente, un identificador de sesión capturado en el origen de Google Forms. Bloquea 3 de los 6 indicadores solicitados y 2 columnas de la tabla de detalle.
- **Autorización explícita para crear medidas DAX nuevas**: esta tarea de análisis se ejecutó bajo la restricción "no crear medidas DAX"; cualquier implementación futura requiere que el usuario autorice la creación de las medidas de §6 en una fase separada.
- **Confirmación sobre el segmentador de Fecha**: si el cambio de estilo aplica solo a esta página o se extiende a las 7 páginas por consistencia.
- **Confirmación sobre `sc_chart_jornada`**: si se elimina, se reubica, o el mockup es una vista parcial que no representa la intención completa de la página.
- **Confirmación sobre el reemplazo de `sc_tabla_formador`**: si la tabla por call center reemplaza definitivamente el desglose por formador/líder, o si ambas vistas deben coexistir (p. ej. una tabla adicional, no un reemplazo).

## 8. ¿Adaptar la página existente o crear una copia para rediseño?

**Recomendación: crear una copia de la página para prototipar el rediseño**, no editar `p14_satisfaccion_capacitaciones` en sitio. Razones:

1. **Alcance del cambio**: no es un ajuste cosmético — cambia la fila de KPI (7→6 tarjetas, con 3 conceptos nuevos), reemplaza un gráfico existente por uno con datos distintos, agrega 3 visuales nuevos, reestructura la tabla principal (de formador/líder a call center), y depende de una decisión de negocio aún no tomada. Editar en sitio deja la página en un estado intermedio no funcional mientras esa decisión se resuelve.
2. **La página actual está validada y publicada**: `p14_satisfaccion_capacitaciones` pasó la validación de la Fase 17 (`Outputs/31`) y está en el enlace público activo (`Docs/06`). Modificarla directamente arriesga el informe en producción mientras se itera un diseño con dependencias abiertas.
3. **Precedente del propio proyecto**: `Specs/03` §14 punto 4 ya establece que "cualquier trabajo futuro... se trate como una nueva iniciativa versionada, no como una modificación silenciosa de este cierre" — una copia de página para rediseño sigue ese mismo principio.
4. **Reversibilidad**: Power BI Desktop permite duplicar una página fácilmente; una copia sin enlazar aún a la navegación de Home no afecta la experiencia actual del usuario final ni el enlace publicado, hasta que se decida reemplazar.

**No se recomienda** editar la página en sitio hasta que: (a) se resuelva la dependencia de grano de "capacitación" (§7), y (b) se autorice explícitamente la creación de las medidas nuevas de §6.

## 9. Plan sugerido (fases futuras — no ejecutadas en este análisis)

1. **Fase A — Decisiones de negocio**: confirmar la definición de "capacitación única" (dependencia candidata D9), confirmar si `sc_tabla_formador` se reemplaza o coexiste con la tabla por call center, confirmar el alcance de `sc_chart_jornada` en el rediseño, confirmar si el cambio de estilo del segmentador de Fecha se limita a esta página.
2. **Fase B — Autorización de medidas nuevas**: autorizar explícitamente la creación de las medidas identificadas en §6 (`Capacitaciones Realizadas` y, si se resuelve el grano, sus variantes por fecha/call center; `Call Centers Capacitados`; `Ultima Capacitacion`) en `_Medidas Capacitacion`, sin tocar las demás familias de medidas.
3. **Fase C — Prototipo en copia de página**: duplicar `p14_satisfaccion_capacitaciones` en Power BI Desktop (p. ej. como página de borrador, sin enlazar a la navegación de Home) e iterar el layout contra el mockup.
4. **Fase D — Validación**: validar en Power BI Desktop (cálculo en vivo de las medidas nuevas, interacción cruzada del panel de selección, comportamiento del segmentador de Fecha, tabla de comentarios sin vacíos), siguiendo el mismo patrón de las Fases 11 y 17 ya aplicado en el proyecto.
5. **Fase E — Reemplazo y documentación**: decidir si se reemplaza la página original o se repunta la navegación de Home hacia la nueva página; actualizar `Docs/02` (medidas nuevas), `Docs/03` (mapa de página), `Docs/05` (formalizar la dependencia D9 y su estado), y `Docs/06` si cambia la exposición de datos personales; documentar como nuevo `Outputs/NN_resultado_...md`.

## 10. Criterios para continuar

**Continuar si:**
- Negocio confirma la definición de "capacitación única" (dependencia candidata D9).
- Se autoriza explícitamente la creación de las medidas DAX nuevas identificadas en §6.
- Se confirma si la tabla por call center reemplaza o coexiste con `sc_tabla_formador`.

**No continuar (o ajustar el alcance) si:**
- No es posible definir de forma confiable qué constituye "una capacitación" con la estructura actual del origen — en ese caso, el mockup debería ajustarse para mostrar únicamente "Respuestas" (grano real de la fuente) en vez de "Capacitaciones", evitando presentar un conteo de sesiones no verificable.
- No se autoriza la creación de medidas nuevas — en ese caso, solo los indicadores 4 y 6 (§5) podrían implementarse, y la tarjeta "Última capacitación" tampoco sería viable.

## 11. Cierre

Este análisis no modificó `PBI/`, PBIR, TMDL, Power Query, medidas DAX, relaciones, visuales, tema visual ni archivos Excel de `Data/`. Solo se evaluó el impacto de adaptar la página "Satisfacción de capacitaciones" al mockup de referencia ya reubicado en `Assets/mockups/`.
