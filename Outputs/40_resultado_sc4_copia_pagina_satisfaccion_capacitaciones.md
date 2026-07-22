# Resultado — Fase SC-4: creación de copia de página (Satisfacción de capacitaciones)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | `SC-4` de [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Fases previas cerradas | `SC-1` (decisiones DEC-1 a DEC-4), `SC-2` (preparación técnica), `SC-3` (medidas DAX, commit `d0c86f8`) |
| Fecha | 2026-07-21 |
| Alcance | Duplicado de una página del reporte (PBIR). No se modificó TMDL, Power Query, relaciones, medidas DAX ni `Data/*.xlsx`. |

---

## 1. Estado inicial de `git status`

```
?? Data/
```

Working tree limpio salvo `Data/`, `git branch --show-current` = `main`, y el commit `d0c86f8 feat(dax): agregar medidas para satisfaccion capacitaciones` confirmado en `git log --oneline -3`. Sin cambios pendientes en `PBI/`.

## 2. Pendiente registrado antes de duplicar (no bloqueante) — resuelto en §10

Conforme a lo indicado por el usuario, se dejó registrado inicialmente como **pendiente de validación funcional, no cerrado**:

> `Ultima Capacitacion = 2026-10-07` (calculado manualmente en `SC-3` a partir del Excel origen) puede corresponder a datos de prueba o a una interpretación incorrecta de día/mes. Debe compararse en Power BI Desktop contra los valores visibles en el archivo fuente **antes del rediseño final y la publicación** (`SC-5` en adelante).

**Resuelto:** la comparación en Power BI Desktop (§10) confirmó que el valor `2026-10-07` reportado en `SC-3` era efectivamente una interpretación incorrecta — producto de la validación externa por XML, no de la medida DAX ni del dato origen. El valor real es `10/07/2026` (10 de julio de 2026), dentro del rango visible del segmentador de Fecha (`02/07/2026`–`15/07/2026`). Ver corrección aplicada en `Outputs/39` ("Corrección posterior").

## 3. Método de duplicación usado

Este entorno no puede operar la interfaz gráfica de Power BI Desktop (limitación ya documentada en `CLAUDE.md`), por lo que la duplicación se hizo **directamente sobre PBIR**, siguiendo el método alternativo autorizado en el prompt:

1. Se inspeccionó la estructura real de la página original (`page.json` + 26 carpetas de visuales = 27 archivos) antes de duplicar.
2. Se copió la carpeta completa `pages/p14_satisfaccion_capacitaciones/` a `pages/p14_satisfaccion_capacitaciones_v2/` (copia exacta, verificada con `diff -rq` sin diferencias antes de editar nada).
3. Se editó únicamente `page.json` de la copia: `name` → `p14_satisfaccion_capacitaciones_v2`, `displayName` → `Satisfacción de capacitaciones (v2 - borrador)`. `displayOption`, `height` (720) y `width` (1280) quedaron iguales a la original.
4. No se escribió ningún `lineageTag`, `description` ni propiedad administrada por Power BI Desktop — los 26 `visual.json` copiados conservan exactamente sus identificadores internos, sin edición.
5. Se registró la página nueva en `pages.json` (`pageOrder`), inmediatamente después de la original; `activePageName` no se tocó (sigue siendo Home, `67eff42d82e1c9c15b84`).

**Verificación posterior en Power BI Desktop:** pendiente — el usuario debe abrir el `.pbip` para confirmar que la copia carga sin error, conforme a la limitación de esta sesión.

## 4. Nombre técnico y visible de la copia

- **Nombre técnico:** `p14_satisfaccion_capacitaciones_v2`
- **Nombre visible:** `Satisfacción de capacitaciones (v2 - borrador)`

## 5. Archivos creados o modificados

**Modificado (1 archivo, tracked):**
- `PBI/PBI_Indicadores.Report/definition/pages/pages.json` — se agregó `"p14_satisfaccion_capacitaciones_v2"` a `pageOrder`, sin cambiar `activePageName` ni ninguna otra entrada.

**Creados (27 archivos nuevos, untracked):**
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/*/visual.json` (26 carpetas, mismas que la original: `sc_canvas`, `sc_chart_callcenter`, `sc_chart_jornada`, `sc_header_accent`, `sc_header_panel`, `sc_home_btn`, `sc_home_hitzone`, `sc_home_label`, `sc_kpi_claridad`, `sc_kpi_coment`, `sc_kpi_dinamismo`, `sc_kpi_indice`, `sc_kpi_satisf`, `sc_kpi_total`, `sc_kpi_utilidad`, `sc_nota_cap_panel`, `sc_nota_cap_text`, `sc_slicer_callcenter`, `sc_slicer_callcenter_label`, `sc_slicer_fecha`, `sc_slicer_fecha_label`, `sc_slicer_jornada`, `sc_slicer_jornada_label`, `sc_subtitle`, `sc_tabla_formador`, `sc_title`)

## 6. Número de visuales copiados

**26 visuales** (más el `page.json` de la página, total 27 archivos), idéntico al inventario de la original documentado en `Specs/05` §2.

## 7. Confirmación — la página original no cambió

```
git diff -- "PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/"
```
Sin salida (diff vacío) — **confirmado: cero cambios** en la carpeta de la página original. `pages.json` registra `p14_satisfaccion_capacitaciones` una sola vez, en su posición original de la lista.

## 8. Confirmación — Home sigue apuntando a la original

Se verificaron los 4 elementos de navegación del módulo "Satisfacción de capacitaciones" en Home (`home_nav_03_card`, `home_nav_03_label`, `home_nav_03_accent`, `home_nav_03_hitzone`): los 4 referencian exclusivamente `'p14_satisfaccion_capacitaciones'` (la original). **Ninguno referencia `_v2`.**

## 9. Confirmación — la copia no está en navegación

Búsqueda de `satisfaccion_capacitaciones_v2` en todo `PBI/`: solo aparece en 2 archivos — `pages.json` (registro de orden) y el propio `page.json` de la copia (su `name`). Ningún visual de Home ni de otra página enlaza hacia la copia. La copia solo es accesible desde el panel de páginas de Power BI Desktop.

## 10. Estado de la validación de las 3 medidas de SC-3

**Completa.** El usuario abrió el `.pbip` en Power BI Desktop y ejecutó la siguiente consulta en la vista de Consulta DAX:

```DAX
EVALUATE
ROW (
    "Call Centers Capacitados", [Call Centers Capacitados],
    "Capacitaciones Realizadas", [Capacitaciones Realizadas],
    "Ultima Capacitacion", [Ultima Capacitacion]
)
```

Resultado real del modelo:

| Medida | Resultado manual (SC-3, derivado del Excel) | Resultado real en el modelo | Coincide |
|---|---|---|---|
| `Call Centers Capacitados` | 5 | **5** | Sí |
| `Capacitaciones Realizadas` | 5 | **5** | Sí |
| `Ultima Capacitacion` | ~~2026-10-07~~ (valor incorrecto, error de la validación externa por XML) | **10/07/2026** (10 de julio de 2026) — dato mostrado por el modelo en formato regional `DD/MM/AAAA` | Sí, una vez corregida la interpretación de la fecha |

Las 3 medidas calculan exactamente lo esperado según la lógica definida en `SC-3` — no hay error de fórmula ni de referencia.

**El pendiente de `Ultima Capacitacion` queda cerrado.** El modelo confirma que la fórmula está bien implementada y que el dato real es `10/07/2026` — una fecha dentro del rango visible del segmentador de Fecha de la página (`02/07/2026`–`15/07/2026`), consistente con el resto del dataset piloto y sin indicios de datos de prueba ni fechas anómalas. El resultado erróneo (`2026-10-07`) reportado en `SC-3` provino de la validación externa por lectura directa del XML del `.xlsx` (este entorno no tenía `openpyxl`/`pandas` disponibles), no de la medida ni del origen de datos. **Power BI Desktop queda establecido como la fuente de validación definitiva** para este tipo de comparación en fases futuras — no se debe repetir una validación externa por XML como sustituto de la lectura directa del modelo cuando Power BI Desktop esté disponible.

### Hallazgo adicional — tipo real del segmentador de Fecha (DEC-4 corregida)

Se releyó `sc_slicer_fecha/visual.json` en la copia `p14_satisfaccion_capacitaciones_v2` (sin modificarlo): la propiedad `objects.data[0].properties.mode` sigue siendo `'Dropdown'`, **idéntica** a la de la página original (confirmado también por el `diff -rq` sin diferencias de `SC-4`, §3). Sin embargo, Power BI Desktop confirma que el segmentador de Fecha se renderiza como un **rango con 2 casillas de calendario ("Between")**, no como una lista desplegable de fechas individuales.

Esto confirma que la propiedad PBIR `mode: Dropdown` **no determina** si un segmentador sobre una columna de tipo Fecha se renderiza como lista/dropdown o como rango — Power BI Desktop aplica el estilo "Between" por defecto a los segmentadores de campos de fecha continuos, independientemente de ese valor. Como el JSON del segmentador es idéntico entre la copia y la página original (nunca se editó en `SC-4`), este comportamiento **ya existía en la página original**, no fue introducido por la duplicación.

**DEC-4 corregida por el usuario:** se conserva el comportamiento actual del segmentador de Fecha en modo `Between`, únicamente en la página de Satisfacción de capacitaciones. No se requiere modificarlo durante `SC-5` — el diseño ya coincide con el mockup de referencia (`Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png`), que muestra el mismo estilo de rango con 2 casillas. La decisión original registrada en `SC-1` ("mantener Dropdown") se actualiza: la intención de esa respuesta era preservar el comportamiento vigente sin introducir un cambio nuevo, y el comportamiento vigente resultó ser `Between`, no una lista desplegable — el resultado práctico (no tocar el segmentador en `SC-5`) es el mismo.

Esta corrección se limita a esta página. La verificación de si los segmentadores de Fecha de las otras 6 páginas del reporte también renderizan como `Between` (lo cual contradiría `Docs/05_decisiones_limitaciones_pendientes.md` §1.11 de forma más amplia) queda **fuera de alcance de este plan** — se recomienda evaluarlo en una iniciativa aparte antes de tratar `Docs/05` §1.11 como definitivo para segmentadores de Fecha en general.

## 11. Confirmación de no modificación fuera de alcance

No se modificó:

- `_Medidas Capacitacion.tmdl` ni ninguna otra medida DAX.
- `expressions.tmdl` (Power Query) ni `relationships.tmdl`.
- Ningún archivo de `PBI_Indicadores.SemanticModel/` (`git status --porcelain -- "PBI/PBI_Indicadores.SemanticModel/"` sin salida).
- Ningún visual ni `page.json` de `p14_satisfaccion_capacitaciones` (original), `Home`, ni de ninguna otra página.
- Ningún archivo `Data/*.xlsx`.

## 12. Estado final de `git status`

```
 M PBI/PBI_Indicadores.Report/definition/pages/pages.json
?? Data/
?? PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/
```

## 13. Resumen de validación

| Validación | Resultado |
|---|---|
| Página original modificada | No |
| Página nueva creada | Sí — `p14_satisfaccion_capacitaciones_v2` |
| Visuales duplicados | 26 (idéntico al inventario original) |
| Copia enlazada desde Home | No |
| Cambios en medidas/TMDL | No |
| Cambios en Power Query | No |
| Cambios en Data | No |
| PBIP abre correctamente | **Sí** — confirmado por el usuario en Power BI Desktop, `p14_satisfaccion_capacitaciones_v2` carga sin error con el mismo contenido visual que la original |
| Validación de medidas SC-3 | **Completa** — las 3 medidas calculan en el modelo exactamente el valor esperado (ver §10). `Ultima Capacitacion` real = `10/07/2026` (corregido; el valor `2026-10-07` reportado en `SC-3` fue un error de la validación externa por XML, no de la medida) |
| Tipo real del segmentador de Fecha | **"Between" (rango con 2 casillas de calendario)**, confirmado por Power BI Desktop. Comportamiento idéntico y preexistente en la página original (JSON sin cambios desde la duplicación). **DEC-4 corregida**: se conserva este comportamiento en esta página, sin cambios en `SC-5` — coincide con el mockup (ver hallazgo detallado en §10) |
| ¿Se puede avanzar a SC-5? | **Sí** — ambos puntos que quedaban abiertos se resolvieron: `Ultima Capacitacion` confirmada como `10/07/2026` (dato real, dentro del rango del piloto) y DEC-4 corregida a "conservar Between" |

## 14. Commit inicial de SC-4 (ya realizado, sin cambios)

```
feat(report): crear copia pagina satisfaccion capacitaciones
```

Archivos incluidos: `pages.json`, la carpeta completa `p14_satisfaccion_capacitaciones_v2/`, y la versión original de esta bitácora. Este commit ya quedó registrado (`ba4666e`) antes de la sesión de validación en Power BI Desktop y de las correcciones descritas en este documento.

## 15. Revisión de cambios automáticos de Power BI Desktop (post-validación)

Al abrir y guardar el `.pbip` para ejecutar la validación de §10, Power BI Desktop reescribió, como es su comportamiento conocido (`CLAUDE.md`):

| Archivo | Cambio | Acción tomada |
|---|---|---|
| `pages.json` | `activePageName` → `p14_satisfaccion_capacitaciones_v2` (la página vista al guardar); schema `1.0.0`→`1.1.0` | **Revertido** `activePageName` a Home (`67eff42d82e1c9c15b84`) antes de versionar, para conservar Home como apertura predeterminada del reporte. Schema `1.1.0` se conserva (metadato de Desktop, no de negocio) |
| `database.tmdl` | `compatibilityLevel` `1550` → `1600` | Conservado — cambio de motor generado por Desktop, no se revierte |
| `model.tmdl` | Anotación `PBI_ProTooling` ganó `"DaxQueryView_Desktop"` | Conservado — metadato de Desktop |
| `_Medidas Capacitacion.tmdl` | Se generaron `lineageTag` para las 3 medidas nuevas de `SC-3` | Conservado — es exactamente el comportamiento esperado (nunca se escriben a mano, Desktop los genera al guardar) |
| `PBI/PBI_Indicadores.SemanticModel/DAXQueries/` (nueva carpeta) | Contenía `Consulta 1.dax` (la consulta de validación de §10) y `.pbi/daxQueries.json` (metadato local) | `Consulta 1.dax` **eliminado** (no se usará como prueba permanente). `.pbi/daxQueries.json` se deja en disco pero **no se versiona** (metadato local de Desktop) |

Ningún cambio funcional del reporte (visuales, medidas, Power Query, relaciones) se mezcló con estos metadatos.

## 16. Recomendación para continuar

Los 2 puntos que bloqueaban `SC-5` quedaron resueltos en esta sesión (§10, §13): `Ultima Capacitacion` confirmada como `10/07/2026` (dato real) y DEC-4 corregida a "conservar Between" en esta página. **`SC-5` queda desbloqueada.**

Antes de iniciar `SC-5`, se recomienda comitear en 2 pasos separados (ver §17): primero los metadatos legítimos de Desktop (`chore`), luego las correcciones documentales (`docs`) — sin mezclarlos, siguiendo el patrón ya establecido en el proyecto.

## 17. Commits de esta sesión de corrección

**Commit 1 — Metadatos de Desktop** (`chore(pbi): sincronizar metadatos tras validacion en desktop`):

- `PBI/PBI_Indicadores.Report/definition/pages/pages.json` (schema 1.1.0, `activePageName` revertido a Home)
- `PBI/PBI_Indicadores.SemanticModel/definition/database.tmdl` (`compatibilityLevel: 1600`)
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (anotación `PBI_ProTooling`)
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl` (`lineageTag` generados por Desktop para las 3 medidas de `SC-3`)

**Commit 2 — Correcciones documentales** (`docs: corregir validacion previa a sc5`):

- `Outputs/39_resultado_sc3_medidas_satisfaccion_capacitaciones.md` (corrección de `Ultima Capacitacion`)
- `Outputs/40_resultado_sc4_copia_pagina_satisfaccion_capacitaciones.md` (este documento, actualizado)
- `Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md` (DEC-4 corregida)

No se hizo push remoto en ninguno de los 2 commits.
