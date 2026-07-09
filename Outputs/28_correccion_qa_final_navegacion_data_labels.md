# Corrección QA final — navegación clicable y etiquetas de datos

**Fecha:** 2026-07-09
**Tipo:** Corrección QA (cierre previo a Fase 16, no es una fase nueva del plan)
**Alcance:** `PBI/PBI_Indicadores.Report/definition/` únicamente. No se tocó el modelo semántico, Power Query, medidas DAX, relaciones ni `Data/*.xlsx`.

## 1. Contexto y objetivo

Tras la Fase 15 ([Outputs/26_resultado_fase_15_navegacion_paginas.md](26_resultado_fase_15_navegacion_paginas.md)) y su corrección QA previa ([Outputs/27_correccion_fase_15_qa_visual_navegacion_etiquetas.md](27_correccion_fase_15_qa_visual_navegacion_etiquetas.md)), el usuario reportó que en la práctica seguía percibiendo que debía ubicar el cursor con precisión para que la navegación por clic funcionara, pese a que la auditoría estructural de Outputs/27 no había encontrado tarjetas o botones con solo una parte clicable. Se descartó la hipótesis de "percepción del cursor" y se solicitó una solución técnica concreta: superponer un rectángulo transparente por cada módulo de navegación, con acción `PageNavigation` propia, que cubra el área visual completa del módulo (tarjeta + acento + texto, o botón + etiqueta).

Se pidió además reconfirmar que las etiquetas de datos de los gráficos están activas y que no queda texto corrupto (`capacitaci?n`, `motivaci?n`, etc.) en el reporte.

## 2. Sincronización previa de cambios de Power BI Desktop

Antes de iniciar el trabajo intencional, se detectaron y comitearon por separado cambios automáticos generados por una sesión previa de Power BI Desktop:

- `re_chart_callcenter/visual.json`: Desktop agregó `show:true` a un segundo elemento de `labels`.
- `sc_home_label/visual.json`: Desktop bajó la versión de `$schema` de `visualContainer` de 2.9.0 a 2.4.0 y reordenó propiedades.
- `pages.json`: Desktop cambió `activePageName` a `p14_resumen_ejecutivo`.

Commit: `0a99204 chore(report): sincronizar cambios automaticos de Power BI Desktop`.

## 3. Etiquetas de datos en gráficos (tarea 1)

Se auditaron los 8 gráficos de barras/columnas/líneas del reporte. Los 8 ya tenían `objects.labels` con `show:true` activado desde la corrección de la Fase 15 (Outputs/27), con color `#002733` y tamaño de fuente moderado (9D), consistente con la paleta de marca Connect:

| Visual | Tipo | Etiquetas activas |
|---|---|---|
| `cl_chart_callcenter` | barChart | Sí |
| `dc_chart_registros` | barChart | Sí |
| `mc_chart_callcenter` | columnChart | Sí |
| `mc_chart_jornada` | barChart | Sí |
| `re_chart_callcenter` | columnChart | Sí |
| `re_chart_fecha` | lineChart | Sí |
| `sc_chart_callcenter` | columnChart | Sí |
| `sc_chart_jornada` | barChart | Sí |

**Resultado:** no se requirieron cambios adicionales. Se reconfirma el estado ya corregido en la Fase 15.

## 4. Navegación clicable completa (tarea 2 — cambio principal de esta corrección)

### Diagnóstico

La navegación de la Fase 15 se armaba con 2–3 visuales apilados por módulo (tarjeta/botón + acento + etiqueta de texto), cada uno con su propio `visualLink` idéntico. Aunque estructuralmente no había huecos entre ellos, esta composición depende de que los bordes de cada sub-elemento coincidan exactamente al píxel — cualquier micro-espacio entre tarjeta, acento y texto (o cualquier variación de renderizado) puede sentirse como una zona "muerta" al usuario, incluso si técnicamente no la hay.

### Solución aplicada

Se agregó **un visual `shape` adicional por módulo de navegación**, tipo "hitzone": un rectángulo totalmente transparente (`fill.transparency: 100D`, `outline.show:false`, sin fondo de contenedor ni borde visibles) del **tamaño exacto** de la tarjeta/botón completo, colocado con el **z-index más alto** de su módulo, y con el mismo `visualLink` (`navigationSection` y `tooltip`) que ya usaban los elementos visibles. Así, el clic en cualquier punto del área — sin importar si cae sobre la tarjeta, el acento o el texto — es capturado por una única superficie continua.

### Archivos nuevos (12)

**Página Home (`67eff42d82e1c9c15b84`) — 6 hitzones hacia páginas internas:**

| Visual | Posición (x,y,w,h) | z | Destino (`navigationSection`) |
|---|---|---|---|
| `home_nav_01_hitzone` | 48, 386, 352×84 | 230 | `p14_resumen_ejecutivo` |
| `home_nav_02_hitzone` | 448, 386, 352×84 | 231 | `p14_calidad_llamadas` |
| `home_nav_03_hitzone` | 848, 386, 352×84 | 232 | `p14_satisfaccion_capacitaciones` |
| `home_nav_04_hitzone` | 48, 498, 352×84 | 233 | `p14_motivacion_comercial` |
| `home_nav_05_hitzone` | 448, 498, 352×84 | 234 | `p14_detalle_call_center` |
| `home_nav_06_hitzone` | 848, 498, 352×84 | 235 | `p14_notas_metodologicas` |

**Páginas internas — 6 hitzones sobre el botón "Volver a Home":**

| Página | Visual | Posición (x,y,w,h) | z | Destino |
|---|---|---|---|---|
| `p14_resumen_ejecutivo` | `re_home_hitzone` | 1086, 48, 130×34 | 40 | `67eff42d82e1c9c15b84` (Home) |
| `p14_calidad_llamadas` | `cl_home_hitzone` | 1086, 48, 130×34 | 40 | `67eff42d82e1c9c15b84` (Home) |
| `p14_satisfaccion_capacitaciones` | `sc_home_hitzone` | 1086, 48, 130×34 | 40 | `67eff42d82e1c9c15b84` (Home) |
| `p14_motivacion_comercial` | `mc_home_hitzone` | 1086, 48, 130×34 | 40 | `67eff42d82e1c9c15b84` (Home) |
| `p14_detalle_call_center` | `dc_home_hitzone` | 1086, 48, 130×34 | 40 | `67eff42d82e1c9c15b84` (Home) |
| `p14_notas_metodologicas` | `nm_home_hitzone` | 1086, 48, 130×34 | 40 | `67eff42d82e1c9c15b84` (Home) |

Cada posición replica exactamente la de la tarjeta (`_card`) o botón (`_btn`) original de su módulo, y cada `tooltip` reutiliza el texto ya usado por el visual visible correspondiente (p. ej. `'Ir a Resumen ejecutivo'`, `'Volver a Home'`).

### Validaciones ejecutadas sobre la navegación

1. **Validez JSON**: los 205 archivos `visual.json` del reporte (incluidos los 12 nuevos) parsean correctamente sin errores de sintaxis.
2. **Posición y destino**: se confirmó, leyendo de vuelta cada archivo, que la posición de cada hitzone coincide con la de su tarjeta/botón, y que `navigationSection`/`tooltip` coinciden exactamente con los del visual visible del mismo módulo.
3. **Orden Z dentro de cada módulo**: se verificó que el hitzone queda por encima de todos los demás elementos de su propio módulo (tarjeta z=200-205 < acento z=210-215 < etiqueta z=220-225 < **hitzone z=230-235** en Home; botón z=20 < etiqueta z=32 < **hitzone z=40** en páginas internas).
4. **Ausencia de colisión con visuales interactivos**: se comparó el rectángulo de cada hitzone contra la posición de **todos** los demás visuales de su página. Los únicos solapamientos detectados son con `*_canvas_background` (z=0, fondo decorativo de página completa) y `*_header_panel` (z=10, contenedor decorativo del encabezado) — ambos sin `visualLink` propio ni interactividad, y con z-index inferior al del hitzone, por lo que quedar "debajo" es el comportamiento esperado y correcto. No se encontró ningún segmentador, botón u otro visual interactivo ajeno a la navegación en el área de los 12 hitzones.

### Resultado

Cada módulo de navegación (6 tarjetas en Home, 6 botones "Volver a Home") ahora tiene una única superficie clicable continua que cubre el 100% de su área visual, sin depender de la alineación exacta entre sub-elementos.

## 5. Textos visibles (tarea 3)

Se buscaron patrones de mojibake (`capacitaci?n`, `motivaci?n`, `Satisfacci?n`, `metodol?gicas`, `validaci?n`, y en general `?` seguido de letra minúscula acentuada) en todo `PBI_Indicadores.Report/definition/`. **No se encontró ninguna coincidencia.** Esta corrección ya había sido aplicada y confirmada en la Fase 15 (Outputs/27). No se requirieron cambios.

## 6. Confirmación de alcance — modelo semántico intacto

`git status` final muestra únicamente los 12 archivos nuevos bajo `PBI_Indicadores.Report/definition/pages/.../visuals/*_hitzone/`, todos como `Untracked files`. No aparece ningún archivo bajo `PBI_Indicadores.SemanticModel/` ni bajo `Data/`. Se confirma que no se modificó Power Query, medidas DAX, relaciones ni tablas del modelo.

```
On branch master
Untracked files:
  PBI/.../67eff42d82e1c9c15b84/visuals/home_nav_01_hitzone/
  PBI/.../67eff42d82e1c9c15b84/visuals/home_nav_02_hitzone/
  PBI/.../67eff42d82e1c9c15b84/visuals/home_nav_03_hitzone/
  PBI/.../67eff42d82e1c9c15b84/visuals/home_nav_04_hitzone/
  PBI/.../67eff42d82e1c9c15b84/visuals/home_nav_05_hitzone/
  PBI/.../67eff42d82e1c9c15b84/visuals/home_nav_06_hitzone/
  PBI/.../p14_calidad_llamadas/visuals/cl_home_hitzone/
  PBI/.../p14_detalle_call_center/visuals/dc_home_hitzone/
  PBI/.../p14_motivacion_comercial/visuals/mc_home_hitzone/
  PBI/.../p14_notas_metodologicas/visuals/nm_home_hitzone/
  PBI/.../p14_resumen_ejecutivo/visuals/re_home_hitzone/
  PBI/.../p14_satisfaccion_capacitaciones/visuals/sc_home_hitzone/
```

## 7. Qué debe verificar el usuario en Power BI Desktop

Como se indica en [CLAUDE.md](../CLAUDE.md)/[AGENTS.md](../AGENTS.md), este entorno no puede abrir ni interactuar con la interfaz gráfica de Power BI Desktop, por lo que el cambio debe validarse manualmente:

1. Abrir/reabrir `PBI/PBI_Indicadores.pbip` en Power BI Desktop.
2. En Home, hacer clic en distintas zonas de cada una de las 6 tarjetas (esquinas, borde del acento naranja, sobre el texto, en espacio "vacío" dentro del área blanca) y confirmar que **todas** navegan a su página destino.
3. En cada una de las 6 páginas internas, hacer clic en distintas zonas del botón "Volver a Home" (incluyendo el borde) y confirmar que siempre regresa a Home.
4. Confirmar visualmente que ningún hitzone quedó desalineado respecto a su tarjeta/botón (no debe verse un "salto" ni una zona clicable que sobresalga del diseño visible).
5. Confirmar que los 8 gráficos siguen mostrando sus etiquetas de datos con el estilo esperado.
6. Ejecutar `git status`/`git diff` después de la sesión de Desktop, por si se generan cambios automáticos adicionales (lineageTags, `$schema`, etc.) — comitearlos por separado si aparecen, siguiendo el patrón ya establecido.

## 8. Recomendación

Esta corrección cierra el pendiente de QA visual de navegación reportado tras la Fase 15. **No se avanza a la Fase 16 en esta ejecución**, conforme a lo solicitado. Se recomienda que el usuario confirme visualmente los puntos del apartado 7 antes de iniciar la Fase 16.
