# Resultado — Fase 16: notas metodológicas dinámicas

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 16 — Notas metodológicas y advertencias por bajo volumen de datos |
| Fecha | 2026-07-09 |
| Alcance | `PBI/PBI_Indicadores.Report/definition/` únicamente. No se tocó Power Query, medidas DAX, relaciones, tablas del modelo ni `Data/*.xlsx`. |

## 1. Estado inicial de `git status`

Al iniciar esta ejecución (después del cierre de la corrección QA de `Outputs/28`), `git status` mostraba cambios automáticos de Power BI Desktop pendientes de sincronizar:

```
modified:   .../67eff42d82e1c9c15b84/visuals/home_nav_0[1-6]_hitzone/visual.json
modified:   .../p14_calidad_llamadas/visuals/cl_home_hitzone/visual.json
modified:   .../p14_detalle_call_center/visuals/dc_home_hitzone/visual.json
modified:   .../p14_motivacion_comercial/visuals/mc_home_hitzone/visual.json
modified:   .../p14_notas_metodologicas/visuals/nm_home_hitzone/visual.json
modified:   .../p14_resumen_ejecutivo/visuals/re_home_hitzone/visual.json
modified:   .../p14_satisfaccion_capacitaciones/visuals/sc_home_hitzone/visual.json
modified:   .../pages/pages.json
```

El diff confirmó que estos cambios eran puramente cosméticos: eliminación del salto de línea final en los 12 archivos "hitzone" de navegación (creados en `Outputs/28`) y actualización de `activePageName` en `pages.json` (página abierta al cierre de la última sesión de Desktop). Sin cambios de contenido ni de navegación. Se sincronizaron en un commit separado antes de iniciar el trabajo de esta fase:

`bfea60a chore(report): sincronizar cambios automaticos de Power BI Desktop`

## 2. Paso previo obligatorio — Inventario de etiquetas de datos en gráficos

Se revisaron las 7 páginas del informe (Home, Resumen ejecutivo, Calidad de llamadas, Satisfacción de capacitaciones, Motivación comercial, Detalle por call center, Notas metodológicas) buscando todo visual con tipo `barChart`, `columnChart`, `lineChart`, `clusteredBarChart`, `clusteredColumnChart`, `stackedBarChart`, `stackedColumnChart`, `comboChart` o cualquier otro tipo cuantitativo.

Inventario completo de tipos de visual usados en el reporte: `shape` (71), `textbox` (63), `cardVisual` (42), `slicer` (16), `barChart` (4), `tableEx` (4), `columnChart` (3), `image` (1), `lineChart` (1). No existe ningún otro tipo de gráfico cuantitativo fuera de estos 8.

| Página | Visual | Tipo | Etiquetas activas | Color | Tamaño |
|---|---|---|---|---|---|
| Resumen ejecutivo | `re_chart_callcenter` | columnChart | Sí | `#002733` | 9D |
| Resumen ejecutivo | `re_chart_fecha` | lineChart | Sí | `#002733` | 9D |
| Calidad de llamadas | `cl_chart_callcenter` | barChart | Sí | `#002733` | 9D |
| Satisfacción de capacitaciones | `sc_chart_callcenter` | columnChart | Sí | `#002733` | 9D |
| Satisfacción de capacitaciones | `sc_chart_jornada` | barChart | Sí | `#002733` | 9D |
| Motivación comercial | `mc_chart_callcenter` | columnChart | Sí | `#002733` | 9D |
| Motivación comercial | `mc_chart_jornada` | barChart | Sí | `#002733` | 9D |
| Detalle por call center | `dc_chart_registros` | barChart | Sí | `#002733` | 9D |

Home y Notas metodológicas no contienen gráficos (solo tarjetas, paneles y navegación), por lo que no aplica esta revisión en esas dos páginas.

**Total de gráficos revisados:** 8
**Total de gráficos con etiquetas activas:** 8
**Total de excepciones justificadas:** 0

No se requirió activar ninguna etiqueta adicional ni ajustar color/tamaño: el estado ya cumplía el estilo Connect (`#002733`, 9D) desde la corrección de `Outputs/27` y su reconfirmación en `Outputs/28`. No se desactivó ninguna etiqueta existente.

## 3. Página "Notas metodológicas" — ajustes realizados

La página ya contaba con una estructura sólida de 6 paneles (Fuentes de datos, Estado piloto, Encuesta anónima, Calidad provisional, Llamadas con venta, Pendientes de negocio), 4 tarjetas KPI dinámicas y una nota de cierre, construida en la Fase 14. Se revisó contra el checklist de contenido solicitado y se reforzaron dos textos que no cubrían explícitamente todos los puntos pedidos:

| Punto requerido | Estado previo | Acción |
|---|---|---|
| Alcance del informe | Genérico ("Alcance, fuentes, supuestos y limitaciones del piloto") | **Reescrito** (`nm_subtitle`) para nombrar explícitamente las 3 encuestas y los ejes de desglose |
| Fuentes de información | Cubierto (Excel/Google Forms) | Sin cambio de fondo, ver siguiente fila |
| Datos provienen de archivos Excel en `Data` | No mencionaba la carpeta `Data` | **Reescrito** (`nm_fuentes_text`) |
| Datos pueden cambiar con cada actualización | No mencionado en ningún panel | **Agregado** en `nm_fuentes_text` |
| Interpretar considerando el n visible | Cubierto en texto | Sin cambio; reforzado con 3 tarjetas dinámicas nuevas (ver §4) |
| Informe en fase piloto | Cubierto (`nm_piloto_text`) | Sin cambio |
| Encuesta de motivación es anónima | Cubierto (`nm_anonima_text`) | Sin cambio |
| `% Calidad Promedio Provisional` pendiente de rúbrica | Cubierto (`nm_calidad_text`) | Sin cambio |
| `% Llamadas con Venta` con observación pendiente si aparece en blanco | Cubierto (`nm_venta_text`) | Sin cambio |
| Pendientes de negocio (alias de líderes, catálogo oficial) | Cubierto (`nm_pendientes_text`) | Sin cambio |

### Textos modificados

**`nm_subtitle`** (antes → después):
- Antes: *"Alcance, fuentes, supuestos y limitaciones del piloto."*
- Después: *"Seguimiento piloto de calidad de llamadas, satisfacción de capacitaciones y motivación comercial, por call center, jornada y fecha."*

**`nm_fuentes_text`** (antes → después):
- Antes: *"Tres exportaciones Excel de Google Forms: matriz de calidad de llamadas, satisfacción de capacitación y motivación de actividades comerciales."*
- Después: *"Tres exportaciones Excel de Google Forms en la carpeta Data del proyecto: calidad de llamadas, satisfacción de capacitación y motivación comercial. Los datos se actualizan con cada refresco de estos archivos."*

Ambos textos se mantuvieron dentro del espacio disponible de su contenedor (se amplió ligeramente la altura de los cuadros de texto, sin invadir paneles vecinos ni el encabezado) y sin agregar ningún número fijo.

## 4. Advertencias dinámicas agregadas

Se detectó que las medidas de conteo con prefijo "n=" (`n Calidad`, `n Capacitacion`, `n Motivacion`, creadas desde la Fase 10 en `_Medidas Generales`) no estaban vinculadas a ningún visual del reporte — existían en el modelo pero no se usaban. Se agregaron 3 tarjetas nuevas, pequeñas y sin decoración (sin borde, sin fondo, sin título, texto 11D en `#002733`), ubicadas justo debajo de las tarjetas KPI de calidad, capacitación y motivación en la página Notas metodológicas, ocupando el espacio libre ya existente entre esa fila de KPI y los paneles de texto:

| Visual nuevo | Medida enlazada | Posición (x,y,w,h) | z |
|---|---|---|---|
| `nm_n_calidad` | `[n Calidad]` | 248, 240, 184×22 | 110 |
| `nm_n_cap` | `[n Capacitacion]` | 448, 240, 184×22 | 111 |
| `nm_n_mot` | `[n Motivacion]` | 648, 240, 184×22 | 112 |

Estas 3 tarjetas muestran el texto dinámico `"n=" & [conteo]` (por ejemplo `n=3`, `n=32`, `n=5` con los volúmenes actuales, y cualquier valor futuro automáticamente cuando `Data/*.xlsx` se actualice) sin que ningún número quede escrito de forma fija en el reporte. No se creó ninguna medida DAX nueva: las 7 medidas señaladas por el usuario (`n Calidad`, `n Capacitacion`, `n Motivacion`, `Total Registros Piloto`, `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`) ya existían en el modelo desde la Fase 10 y eran suficientes.

Las 4 tarjetas KPI ya existentes en la página (`Total Registros Piloto`, `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`) se dejaron sin cambios: ya eran dinámicas y no contenían ningún conteo escrito a mano.

### Propuesta de medida DAX no implementada (a valorar en fases futuras)

El documento `Specs/02` (Fase 16 original) sugiere mostrar también una "fecha de corte de los datos" en la página de notas. Actualmente no existe una medida para esto y no se implementó en esta fase porque no fue solicitada explícitamente en el prompt de esta ejecución y las restricciones piden proponer antes de implementar. Se deja documentada como sugerencia para una fase posterior (por ejemplo, Fase 17):

```
Fecha Corte Datos = MAX(Dim_Calendario[Fecha])
```

Formateada como fecha corta y mostrada en una tarjeta pequeña junto a la nota de cierre, reforzaría la advertencia de vigencia sin depender de texto fijo. No se implementó — queda como recomendación, no como pendiente bloqueante.

## 5. Nota transversal en páginas internas

Se revisaron los 5 paneles de nota discreta ya existentes en las páginas internas (creados en la Fase 14). Los 5 ya eran generales y dinámicos (ningún conteo fijo), pero uno no mencionaba la interpretación por volumen/n visible:

| Página | Visual | Estado | Acción |
|---|---|---|---|
| Resumen ejecutivo | `re_nota_piloto_text` | Ya general/dinámica | Pulido de redacción menor, sin cambio de fondo |
| Calidad de llamadas | `cl_nota_calidad_text` | Ya general/dinámica, cubre `% Calidad Promedio Provisional` y `% Llamadas con Venta` | Sin cambio |
| Satisfacción de capacitaciones | `sc_nota_cap_text` | Ya general/dinámica, cubre piloto + alias de líderes | Sin cambio |
| Motivación comercial | `mc_nota_mot_text` | Cubría solo el carácter anónimo, sin referencia a piloto/n visible | **Ampliada** |
| Detalle por call center | `dc_nota_detalle_text` | Ya general/dinámica | Sin cambio |
| Home | `home_method_note_text`, `home_pilot_note` | Ya generales/dinámicas | Sin cambio |

**`re_nota_piloto_text`** (antes → después):
- Antes: *"Muestra piloto: interpretar cada indicador con su base n y evitar conclusiones definitivas hasta aumentar volumen de respuestas."*
- Después: *"Muestra piloto: interpretar cada indicador con base en el n visible y evitar conclusiones definitivas hasta aumentar el volumen de respuestas."*

**`mc_nota_mot_text`** (antes → después):
- Antes: *"Encuesta anónima: no permite análisis por asesor."*
- Después: *"Encuesta anónima: no permite análisis por asesor individual. Interpretar con base en el n visible del piloto."*

`cl_nota_calidad_text`, `sc_nota_cap_text` y `dc_nota_detalle_text` no se modificaron: ya cumplían el criterio de ser generales, dinámicas y sin conteos fijos, y ya aportaban una advertencia específica y relevante para su página.

## 6. Confirmación — sin conteos fijos

Se ejecutó una búsqueda automatizada de patrones tipo `N registros`, `N respuestas`, `N evaluaciones`, `N filas`, `N encuestas` (con `N` numérico) sobre el texto de los 63 visuales `textbox` del reporte completo. **No se encontró ninguna coincidencia.** Todos los conteos visibles en el informe se muestran exclusivamente a través de tarjetas (`cardVisual`) enlazadas a medidas DAX, nunca como texto escrito a mano.

## 7. Redacción en español de Colombia

Se confirmó el uso correcto de tildes y caracteres especiales en los textos nuevos y modificados: *capacitación, satisfacción, motivación, metodológicas, validación, índice, rúbrica, anónima*. Una búsqueda de patrones de mojibake (`capacitaci?n`, `motivaci?n`, `Satisfacci?n`, `metodol?gicas`, `validaci?n`, y en general `?` seguido de vocal acentuada) sobre los 208 archivos `visual.json` del reporte **no encontró ninguna coincidencia**.

## 8. Validaciones JSON/PBIR

- Los 208 archivos `visual.json` del reporte (205 preexistentes + 3 nuevos: `nm_n_calidad`, `nm_n_cap`, `nm_n_mot`) parsean correctamente sin errores de sintaxis.
- Los 3 visuales nuevos siguen el schema `visualContainer/2.9.0` y no incluyen `lineageTag`, `description` ni `queryGroup` escritos a mano.
- Se verificó que las posiciones de los 3 nuevos visuales (`y=240`, alto 22px) no colisionan con la fila de tarjetas KPI (termina en `y=236`) ni con los paneles de texto inferiores (comienzan en `y=272`).
- `git status --porcelain -- Data/ PBI/PBI_Indicadores.SemanticModel/` no devolvió ninguna línea: confirma que no hubo cambios en el modelo semántico ni en los archivos fuente.

## 9. Confirmación de no modificación del modelo semántico

No se modificaron durante esta fase:

- Power Query (`expressions.tmdl`)
- Medidas DAX (`_Medidas Generales.tmdl`, `_Medidas Calidad.tmdl`, `_Medidas Capacitacion.tmdl`, `_Medidas Motivacion.tmdl`)
- Relaciones (`relationships.tmdl`)
- Tablas del modelo (`tables/*.tmdl`)
- Archivos `Data/*.xlsx`

Los 3 visuales nuevos referencian medidas ya existentes (`n Calidad`, `n Capacitacion`, `n Motivacion`) sin crear, editar ni renombrar ninguna medida.

## 10. Archivos modificados

Modificados:
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_subtitle/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_fuentes_text/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_resumen_ejecutivo/visuals/re_nota_piloto_text/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_motivacion_comercial/visuals/mc_nota_mot_text/visual.json`

Nuevos:
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_n_calidad/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_n_cap/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/visuals/nm_n_mot/visual.json`

Documentación:
- `Outputs/29_resultado_fase_16_notas_metodologicas_dinamicas.md` (este documento)

## 11. Resultado del commit

Commit sugerido por el usuario:

`feat(report): fortalecer notas metodologicas dinamicas`

## 12. Estado final de `git status`

Tras comitear los 7 archivos anteriores más este documento, `git status` queda limpio (`working tree clean`), sin rastros pendientes de `Data/*.xlsx` ni del modelo semántico.

## 13. Recomendación para avanzar a Fase 17

Antes de avanzar a la Fase 17 (validaciones técnicas, funcionales y visuales), se recomienda que el usuario:

1. Abra el PBIP en Power BI Desktop y confirme que la página "Notas metodológicas" renderiza sin recortes de texto, con los 3 nuevos valores `n=` visibles y legibles bajo las tarjetas de calidad, capacitación y motivación.
2. Confirme visualmente que el subtítulo ampliado y el panel de "Fuentes de datos" no se superponen con los segmentadores de fecha/call center del encabezado.
3. Revise la nota ampliada de la página "Motivación comercial" para verificar que el texto más largo no se recorta en el ancho actual del panel.
4. Evalúe la propuesta de medida `Fecha Corte Datos` (sección 4) para decidir si se incorpora en la Fase 17 o se deja para una fase posterior.
5. Ejecute `git status`/`git diff` tras la sesión de Desktop, por si se generan cambios automáticos adicionales, y comitéelos por separado siguiendo el patrón ya establecido.

Con estas confirmaciones, el informe queda listo para iniciar la Fase 17 (control de calidad integral técnico, funcional y visual).
