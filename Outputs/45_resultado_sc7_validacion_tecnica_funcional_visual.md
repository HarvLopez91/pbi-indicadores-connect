# Resultado — Fase SC-7: validación técnica, funcional y visual

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | `SC-7` de [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Página validada | `p14_satisfaccion_capacitaciones_v2` (copia de trabajo) |
| Fecha | 2026-07-22 |
| Alcance | Fase de **lectura y verificación**. No se corrigió ningún hallazgo sobre la marcha; todo se documenta y clasifica primero. No se modificó ningún archivo del proyecto (los únicos archivos escritos en esta ejecución son este documento y el registro de la sesión). No se avanzó a `SC-8`. No se creó commit. No se hizo push. |
| Estado final | **`SC-6` y `SC-7` aprobadas** (ver §15) tras la confirmación funcional del usuario en Power BI Desktop. Commits creados — ver §15. |
| Commit base | `d4b58fd feat(report): redisenar satisfaccion de capacitaciones` (cierre de `SC-5`) |

---

## 1. Estado inicial de Git

```
$ git rev-parse --show-toplevel
C:/Users/eclavijo/OneDrive/PBI_Indicadores

$ git branch --show-current
main

$ git log --oneline -5
d4b58fd feat(report): redisenar satisfaccion de capacitaciones
5469ded fix(data): restaurar rutas locales de fuentes OneDrive
f481d12 chore(modelo): sincronizar metadatos tras validacion en desktop
28664d2 data(powerquery): migrar origenes de datos a rutas web de onedrive
d2131cd docs: corregir validacion previa a sc5
```

Power BI Desktop estaba **cerrado** al iniciar esta validación (confirmado con `tasklist`).

`git status --short --untracked-files=all` mostró, antes de cualquier acción de esta fase:

```
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_respuestas_panel/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_slicer_callcenter_label/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_slicer_fecha_label/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_slicer_jornada_label/visual.json
 M PBI/PBI_Indicadores.SemanticModel/diagramLayout.json
?? "Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx"
?? Outputs/44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md
```

`SC-5` ya está cerrada y comiteada (`d4b58fd`); `SC-6` está implementada pero sin commit, según lo previsto.

## 2. Cambios de Desktop clasificados

| Archivo | Clasificación | Detalle |
|---|---|---|
| `p14_satisfaccion_capacitaciones_v2/page.json` | **SC-6 intencional** | Agrega el bloque `visualInteractions` (167 interacciones) descrito en `Outputs/44`. No toca ningún otro campo de la página (`displayName`, `width`, `height` sin cambio). |
| `sc_kpi_respuestas_panel/visual.json` | **Metadato automático de Desktop** | Único cambio: se eliminó el salto de línea final del archivo (`\ No newline at end of file`). Cero cambio funcional o visual. |
| `sc_slicer_fecha_label/visual.json` | **Ajuste manual de texto de filtro** | Ver §3 — persistió correctamente. |
| `sc_slicer_callcenter_label/visual.json` | **Ajuste manual de texto de filtro** | Ver §3 — persistió correctamente. |
| `sc_slicer_jornada_label/visual.json` | **Ajuste manual de texto de filtro** | Ver §3 — persistió correctamente. |
| `PBI_Indicadores.SemanticModel/diagramLayout.json` | **Metadato automático de Desktop** | Agrega la posición (`x`, `y`, `size`) del nodo `Dim_MetricaSatisfaccion` en la vista de diagrama del modelo (Desktop coloca automáticamente cualquier tabla nueva al abrir esa vista). Sin efecto funcional; no es parte del modelo de datos ni del reporte. |
| `Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx` | **Archivo local no versionable** | Ver hallazgo H-1 en §8 — no está cubierto por el patrón actual de `.gitignore`. |
| `Outputs/44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md` | **Documentación SC-6** | Esperado; bitácora ya redactada por la sesión que ejecutó `SC-6`. |

Ningún cambio se clasifica como "inesperado" — todos corresponden a una de las categorías previstas por el usuario.

## 3. Validación de persistencia de los textos de filtros

| Etiqueta | Visual técnico | Cambio detectado | ¿Persistió? |
|---|---|---|---|
| Fecha | `sc_slicer_fecha_label` | Posición `y` 120→111.63, `height` 13→20.93 (evita recorte del texto); estilo re-serializado (`fontWeight`/`color` reordenados, hex a minúsculas `#3a3a3a`, mismo color visual); `horizontalTextAlignment: left` (redundante, ya era el valor por defecto) eliminado; `$schema` 2.9.0→2.11.0 | **Sí** |
| Call Center | `sc_slicer_callcenter_label` | Mismo patrón que Fecha | **Sí** |
| Jornada | `sc_slicer_jornada_label` | Mismo patrón (sin bump de `$schema`, ya estaba en 2.11.0) | **Sí** |

Verificaciones adicionales:

- **Tamaño de fuente**: `9px` en las 3 etiquetas, sin cambio.
- **Color**: `#3a3a3a` (antes `#3A3A3A`) — idéntico visualmente, solo cambia el caso del hexadecimal.
- **Posición**: las 3 etiquetas quedaron alineadas en `y=111.63`, consistente entre sí.
- **Ancho y alto**: `width=369.77` / `height=20.93` en las 3 — uniformes.
- **Orden `z`**: sin cambio (`40`, `44`, `48` respectivamente), por encima del panel de fondo y por debajo de los segmentadores.
- **Ausencia de recorte**: el borde inferior de cada etiqueta (`y + height ≈ 132.56`) queda por encima del borde superior del segmentador correspondiente (`y = 134`), con margen de ~1.4px — sin superposición ni recorte.
- **Contraste**: `#3a3a3a` sobre fondo blanco del panel de filtros — contraste suficiente, sin cambio respecto a antes.

No se modificó el diseño durante esta validación (solo lectura).

## 4. Resultado técnico

| # | Validación | Resultado |
|---|---|---|
| 1 | El `.pbip` abre sin errores | **Pendiente** — requiere confirmación visual del usuario en Desktop |
| 2 | JSON válido en los 47 `visual.json` de la copia | **Aprobado** |
| 3 | `page.json` cumple el esquema PBIR (`.../page/2.0.0/schema.json`) | **Aprobado** |
| 4 | Las 167 interacciones usan únicamente `DataFilter`/`NoFilter` | **Aprobado** |
| 5 | Todos los `source`/`target` de las interacciones existen | **Aprobado** |
| 6 | No existen visuales huérfanos | **Aprobado** |
| 7 | Ningún visual fuera del lienzo `1280×720` | **Aprobado** |
| 8 | Medidas nuevas existen y sin referencias rotas | **Aprobado** |
| 9 | `Dim_MetricaSatisfaccion` existe y permanece desconectada | **Aprobado** |
| 10 | Power Query conserva `TimestampNormalizado` | **Aprobado** |
| 11 | `compatibilityLevel` permanece en `1606` | **Aprobado** |
| 12 | `RutaCarpetaData` permanece intacta | **Aprobado** |
| 13 | Relaciones existentes sin cambios inesperados | **Aprobado** |

### Detalle

**#2–#3 — JSON y esquema:** los 47 `visual.json` de `p14_satisfaccion_capacitaciones_v2` parsean sin error (`json.load`); `page.json` declara `$schema: .../page/2.0.0/schema.json`, coincidiendo con lo documentado en `Outputs/44`.

**#4–#5 — Interacciones:** se extrajo programáticamente el arreglo `visualInteractions` de `page.json`: **167 entradas**, tipos usados `{DataFilter, NoFilter}` únicamente (ningún otro tipo del esquema), y los 47 visuales de la página cubren todos los `source`/`target` referenciados — cero nombres inexistentes, cero interacciones origen=destino. Los visuales nunca referenciados en la matriz son exactamente los decorativos/de navegación/etiquetas (`sc_canvas`, `sc_header_*`, `sc_logo_connect`, `sc_pilot_badge`/`sc_pilot_note`, `sc_title`/`sc_subtitle`, `sc_nota_cap_*`, `sc_home_*`, los 6 `sc_kpi_icon_*`/`sc_kpi_line_*`, `sc_panel_satisf_container`/`sc_panel_satisf_title`, las 3 etiquetas de segmentador, `sc_filter_panel`) — consistente con lo esperado, ninguno de estos debería filtrar ni ser filtrado.

**#6–#7 — Huérfanos y lienzo:** no se detectó ningún `visual.json` cuya carpeta no correspondiera a un visual real de la página; los 47 visuales están dentro de `1280×720`.

**#8–#9 — Medidas y dimensión desconectada:** confirmadas en `_Medidas Capacitacion.tmdl`: `Capacitaciones Realizadas`, `Call Centers Capacitados`, `Ultima Capacitacion`, `Ultima Capacitacion Texto`, `Valor Metrica Satisfaccion` — las 5 existen, sin `description`/`queryGroup` escritos a mano. `Dim_MetricaSatisfaccion` no aparece en `relationships.tmdl` (confirmado por búsqueda) — sigue desconectada, tal como exige su diseño de `SWITCH(SELECTEDVALUE(...))`.

Se detectó además que **`Ultima Capacitacion Texto`** ya no usa `FORMAT(..., "es-CO")` (mi último ajuste en `SC-5` Revisión 8), sino una expresión `VAR/RETURN` que arma el texto manualmente con `DAY`/`MONTH`/`YEAR` — y que se agregó una columna nueva **`Fecha Texto`** en `Fact_SatisfaccionCapacitacion` con el mismo patrón, usada por `sc_tabla_comentarios`. Ambos cambios ya estaban comiteados en `d4b58fd`, fuera de esta sesión de `SC-7`. Se documentan como hallazgo H-2 (observación) en §8 porque no estaban registrados en `Outputs/43`, no porque representen un error — la expresión es válida y sin referencias rotas.

**#10–#13 — Power Query y modelo:** `RutaCarpetaData` y el paso `TimestampNormalizado` siguen presentes en `expressions.tmdl` sin cambios; `compatibilityLevel: 1606` confirmado en `database.tmdl`; `git diff -- relationships.tmdl` está vacío (coincide con `HEAD`), confirmando que la reversión de `SC-5` Revisión 8 sobrevivió al commit `d4b58fd` sin regresión.

## 5. Resultado funcional

**Confirmado por el usuario en Power BI Desktop (ver §15).** La estructura PBIR fue validada de forma estática (sin errores) y el usuario confirmó el comportamiento en vivo:

### Segmentadores (Fecha, Call Center, Jornada — por separado y combinados)

- [x] Cada uno filtra los 6 KPI, gráfico por call center, gráfico por fecha, panel de satisfacción, respuestas del panel, detalle por call center y comentarios destacados. **Confirmado.**

### Selección de Call Center (`ATENTO`, `ONE CONTACT` verificados en capturas)

- [x] Panel de satisfacción actualizado.
- [x] Respuestas actualizadas.
- [x] Gráfico por fecha filtrado.
- [x] Tabla de detalle filtrada.
- [x] Comentarios filtrados.
- [x] KPI recalculados.

### Selección de Fecha (desde el gráfico / rango del segmentador)

- [x] Call centers actualizados.
- [x] Panel actualizado.
- [x] Respuestas actualizadas.
- [x] Detalle actualizado.
- [x] Comentarios actualizados.
- [x] KPI recalculados.

### Tabla de detalle (selección de una fila)

- [x] Panel, respuestas, gráfico por fecha, comentarios y KPI actualizados. **Confirmado — `tableEx` sí funciona como origen de filtro en esta versión de Desktop; no se materializó la limitación anticipada en `Outputs/44`.**

### Visuales receptores únicamente (no deben alterar el resto del dashboard)

- [x] Comentarios destacados.
- [x] Panel de satisfacción.
- [x] Tarjeta Respuestas del panel.
- [x] KPI superiores.

La matriz de interacciones (§4) ya confirmaba estáticamente que estos 4 visuales están configurados como `NoFilter` hacia el resto; el usuario confirmó que el renderizado real coincide.

## 6. Resultado visual

| # | Elemento | Resultado |
|---|---|---|
| 1 | Logo visible | **Aprobado** — confirmado en captura |
| 2 | Título y subtítulo legibles | **Aprobado** — confirmado en captura |
| 3 | Insignia piloto visible | **Aprobado** — confirmado en captura ("Datos piloto sujetos a validación") |
| 4 | Etiquetas Fecha/Call Center/Jornada legibles, sin recorte | **Aprobado** (verificado estáticamente en §3 y confirmado visualmente en captura) |
| 5 | Segmentador Fecha en modo `Between` | **Aprobado** — la captura muestra dos campos de fecha independientes (`02/07/2026` / `22/07/2026`); Desktop además persistió el estado de rango seleccionado en el propio `visual.json` (ver §15), confirmando el comportamiento `Between` pese al `mode: 'Dropdown'` declarado (H-3, ya no bloqueante) |
| 6 | Seis KPI sin recortes | **Aprobado** — confirmado en captura |
| 7 | Última capacitación = `10/07/2026` | **Aprobado** — confirmado en captura |
| 8 | Cinco fechas visibles sin `(En blanco)` ni horas | **Aprobado** — confirmado en captura (`04/07` a `10/07`, sin horas) |
| 9 | Cinco call centers completos (sin truncar) | **Aprobado** — confirmado en captura (`ATENTO`, `BRM`, `GNP`, `INTERACTIVO`, `ONE CONTACT` legibles) |
| 10 | Cuatro barras del panel visibles | **Aprobado** — confirmado en captura (Satisfacción, Claridad, Utilidad, Dinamismo, sin scroll) |
| 11 | Respuestas del panel sin recorte | **Aprobado** — confirmado en captura |
| 12 | Tabla de detalle con filas y Total | **Aprobado** — confirmado en captura (fila `Total` visible con 5/84/4,8/4,8/4,8/4,8/10-07-2026) |
| 13 | Comentarios sin blancos, `Sin comentario` ni `"."` | **Aprobado** — `sc_tabla_comentarios` conserva el `filterConfig` que excluye ambos valores (verificado en el JSON) y confirmado en captura |
| 14 | Nota metodológica legible | **Aprobado** — confirmado en captura |
| 15 | Sin superposiciones ni scroll horizontal innecesario | **Aprobado** — cero superposiciones detectadas por el chequeo automático de rectángulos entre visuales de contenido; todos dentro del lienzo |
| 16 | Sin nombres de formadores, líderes o asesores | **Aprobado** — cero coincidencias de `NombreFormador`/`NombreLider`/`NombreAsesor` en la página `v2` |
| 17 | Cero mojibake | **Aprobado** — ver §7 |

## 7. Textos corruptos

Se buscaron los patrones `capacitaci?n`, `satisfacci?n`, `validaci?n`, `comentari?s`, `motivaci?n`, `metodol?gicas`, el carácter de reemplazo Unicode `�` y secuencias de doble codificación (`Ã.`, `â€`) sobre los 47 `visual.json` de la copia, `page.json`, y los 3 archivos TMDL modificados en `SC-5`/`SC-6` (`_Medidas Capacitacion.tmdl`, `Dim_MetricaSatisfaccion.tmdl`, `Dim_Calendario.tmdl`), leyendo cada archivo con decodificación UTF-8 explícita.

**Resultado: cero coincidencias.**

Nota metodológica: una inspección visual rápida del `displayName` de `page.json` en la terminal mostró `Satisfacci�n` — se verificó a nivel de bytes (`\xc3\xb3`, UTF-8 válido para "ó") y se confirmó que es un artefacto de la consola/terminal, no una corrupción real del archivo. Se documenta para dejar constancia de que no es un hallazgo.

## 8. Resultado de navegación

| Prueba | Resultado |
|---|---|
| Desde `v2`, `sc_home_btn`/`sc_home_label`/`sc_home_hitzone` declaran `PageNavigation` hacia `67eff42d82e1c9c15b84` (Home) | **Aprobado** (estructura confirmada) |
| El clic real en fondo naranja, texto, flecha y zona clicable navega a Home | **Aprobado** — confirmado por el usuario ("El botón Volver a Home funciona en toda su superficie") |
| Desde Home, `home_nav_03_card`/`home_nav_03_accent`/`home_nav_03_hitzone`/`home_nav_03_label` declaran `PageNavigation` hacia `p14_satisfaccion_capacitaciones` (original, no `v2`) | **Aprobado** (estructura confirmada) |
| El clic real en la tarjeta de Home navega a la original, no a `v2` | **Aprobado** — confirmado por el usuario, consistente con la captura de la página original (con tabla `Formador y líder`) |

Que Home siga apuntando a `p14_satisfaccion_capacitaciones` **no se trata como error**: es el comportamiento previsto hasta que `SC-9` decida el reemplazo o redirección. No se modificó Home en esta fase.

## 9. Resultado de no regresión

```
$ git diff -- "PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/"
(sin salida — sin cambios)

$ git diff -- "PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/"
(sin salida — sin cambios)

$ git diff -- "PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl"
(sin salida — sin cambios, coincide con HEAD)
```

Adicionalmente se confirmó `git diff --stat` vacío para las otras 5 páginas del informe (`p14_resumen_ejecutivo`, `p14_calidad_llamadas`, `p14_motivacion_comercial`, `p14_detalle_call_center`, `p14_notas_metodologicas`), el tema global (`report.json`, `StaticResources/`) y `expressions.tmdl` (Power Query). `Data/` permanece sin seguimiento salvo el archivo señalado en el hallazgo H-1.

## 10. Hallazgos y severidad

| ID | Hallazgo | Severidad | Impacto |
|---|---|---|---|
| H-1 | El patrón `Data/*.xlsx` de `.gitignore` no cubre subcarpetas (`Data/**/*.xlsx`). El archivo `Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx` (con datos reales) aparecía como `??` (no rastreado) pero **no estaba protegido por gitignore** — un `git add -A`/`git add .` futuro lo hubiera comiteado por error. | **Mayor** (riesgo de privacidad) | **Corregido** en §15: patrón actualizado a `Data/**/*.xlsx`, validado con `git check-ignore -v` y `git status`. |
| H-2 | La columna `Fecha Texto` en `Fact_SatisfaccionCapacitacion` y la reescritura de `Ultima Capacitacion Texto` (ahora `VAR/RETURN` con `DAY/MONTH/YEAR` en vez de `FORMAT(..., "es-CO")`) no están documentadas en `Outputs/43`. Ya estaban comiteadas en `d4b58fd`, antes de iniciar `SC-7`. | **Observación** | Sin referencias rotas, sin `lineageTag`/`description`/`queryGroup` a mano. Se recomienda añadir `Fecha Texto` a la lista de objetos DAX autorizados en la próxima actualización de `Outputs/43` o en `SC-8`. |
| H-3 | El segmentador `sc_slicer_fecha` declara `mode: 'Dropdown'` en el JSON estático. Por convención ya documentada del proyecto (decisión DEC-4, `Specs/02`), Power BI Desktop suele renderizar campos de fecha como control de rango `Between` independientemente de este valor. | **Observación** | No es un hallazgo nuevo; requiere reconfirmación visual, ya prevista en el checklist de `SC-7`. |
| H-4 | `pages.json` tiene `activePageName: p14_satisfaccion_capacitaciones_v2` (la pestaña que quedó abierta en la última sesión de Desktop). | **Observación** | Cosmético — no afecta la navegación de Home ni el comportamiento del informe; solo determina qué página se muestra primero al reabrir el `.pbip`. |
| — | Todos los ítems marcados "Pendiente" en §5–§8 | **No son hallazgos** | Requieren render/clic real en Power BI Desktop, limitación operativa ya señalada en `CLAUDE.md`/`AGENTS.md` para todo el proyecto. |

**Ningún hallazgo bloqueante dentro del alcance de `SC-6`/`SC-7`.**

## 11. Estado de SC-6

**Aprobada.**

La estructura de las 167 interacciones es válida (tipos permitidos, sin referencias rotas, sin auto-referencias) y no introdujo cambios en ningún `visual.json` individual, tal como documenta `Outputs/44`. El usuario confirmó en Power BI Desktop (§5, §15) que el comportamiento interactivo real coincide con lo configurado, incluyendo que `sc_tabla_callcenter` (`tableEx`) sí funciona como origen de filtro — la limitación anticipada en `Outputs/44` no se materializó.

## 12. Estado de SC-7

**Aprobada.**

Toda la validación estática/estructural está aprobada sin hallazgos bloqueantes, y el usuario confirmó en Power BI Desktop los checklists funcional (§5), visual (§6) y de navegación (§8) — ver la confirmación textual en §15.

## 13. Recomendación para avanzar a SC-8

**Puede avanzar a `SC-8`.**

Los 4 checklists (funcional, visual, navegación, no regresión) quedaron confirmados sin hallazgos nuevos. Adicionalmente se corrigió el hallazgo H-1 (`.gitignore`) antes de comitear — ver §15. El proyecto queda listo para `SC-8` (documentación).

## 14. Punto de control al cierre de la validación estática (previo a la confirmación del usuario)

```
$ git status --short --untracked-files=all
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_respuestas_panel/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_slicer_callcenter_label/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_slicer_fecha_label/visual.json
 M PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_slicer_jornada_label/visual.json
 M PBI/PBI_Indicadores.SemanticModel/diagramLayout.json
?? "Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx"
?? Outputs/44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md
?? Outputs/45_resultado_sc7_validacion_tecnica_funcional_visual.md
```

En ese momento no se había creado ningún commit, ni push, ni se había avanzado a `SC-8`. Ningún archivo del proyecto había sido modificado durante la validación de solo lectura — los únicos archivos nuevos eran este documento y (ya existente) `Outputs/44`.

## 15. Cierre — confirmación funcional del usuario y commits

El usuario ejecutó el checklist funcional/visual/navegación en Power BI Desktop (adjuntando capturas) y confirmó textualmente:

> Los segmentadores Fecha, Call Center y Jornada filtran todos los visuales esperados. La selección de un call center actualiza KPI, gráfico por fecha, panel de satisfacción, respuestas, detalle y comentarios. La selección de una fecha actualiza los demás visuales. La selección de una fila de la tabla de detalle actualiza panel, respuestas, fecha, KPI y comentarios. Comentarios, panel de satisfacción, tarjeta de respuestas y KPI no filtran otros visuales. El botón Volver a Home funciona en toda su superficie. Desde Home, Satisfacción de capacitaciones continúa abriendo la página original, comportamiento esperado hasta SC-9. La página v2 abre sin errores y la validación visual fue aprobada.

### Cambios adicionales detectados tras la sesión de prueba en Desktop

Al reabrir `git status` después de esta confirmación aparecieron artefactos nuevos, generados por el propio acto de probar las interacciones (guardado por Desktop):

| Archivo | Origen | Decisión |
| --- | --- | --- |
| `sc_slicer_fecha/visual.json` | Desktop persistió el rango de fecha realmente seleccionado durante la prueba (`startDate`/`endDate` 2026-07-02 a 2026-07-22, más el `filterConfig` semántico equivalente) — evidencia directa de que el segmentador opera como `Between` (confirma H-3) | **Conservado** — es la prueba funcional misma, parte del commit de interacciones |
| `sc_slicer_callcenter/visual.json` | Desktop agregó `isInvertedSelectionMode: true`, reflejo de haber seleccionado `ATENTO`/`ONE CONTACT` durante la prueba, más bump de `$schema` | **Conservado** — mismo motivo |
| `pages.json` | `activePageName` volvió de `p14_satisfaccion_capacitaciones_v2` a `p14_satisfaccion_capacitaciones` (pestaña activa al guardar, tras navegar a la original para confirmar Home) | **Conservado** — cosmético, no afecta navegación real |
| `sc_kpi_respuestas_panel/visual.json` | Cambio de una sesión anterior: solo el salto de línea final (`\ No newline at end of file`), cero valor funcional | **Revertido** con `git checkout --` antes de comitear |
| `sc_nota_cap_text/visual.json` (**página original**) | Bump cosmético de `$schema` (2.9.0→2.11.0) y reserialización de estilo sin cambio de valores, generado por Desktop al abrir la página original durante la prueba de navegación | **Revertido** con `git checkout --` — la página original debe permanecer intacta byte a byte siempre que el cambio sea puramente cosmético |

### Corrección de `.gitignore` (hallazgo H-1)

```gitignore
Data/**/*.xlsx
```

Reemplaza el patrón anterior `Data/*.xlsx`, que no cubría subcarpetas. Validado:

```text
$ git check-ignore -v "Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx"
.gitignore:6:Data/**/*.xlsx  Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx

$ git status --short --untracked-files=all -- Data/
(sin salida — Data/ ya no aparece como no rastreado)
```

Ningún archivo de `Data/` fue agregado al índice.

### Commits creados

Ver hashes y archivos exactos en el mensaje de cierre de la conversación (no se transcriben aquí para no duplicar `git log`, que es la fuente de verdad). Resumen de la separación:

1. **Seguridad de datos** — únicamente `.gitignore`.
2. **Interacciones y validación** — `page.json` (167 interacciones), las 3 etiquetas de segmentador, `sc_slicer_fecha`/`sc_slicer_callcenter` (estado de prueba persistido), `pages.json`, `Outputs/44`, `Outputs/45`.
3. **Metadato de Desktop** — `diagramLayout.json` (posición de `Dim_MetricaSatisfaccion` en el diagrama del modelo).

No se hizo push a ningún remoto.
