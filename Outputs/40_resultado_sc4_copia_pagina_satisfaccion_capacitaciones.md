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

## 2. Pendiente registrado antes de duplicar (no bloqueante)

Conforme a lo indicado por el usuario, se deja registrado como **pendiente de validación funcional, no cerrado**:

> `Ultima Capacitacion = 2026-10-07` (calculado manualmente en `SC-3` a partir del Excel origen) puede corresponder a datos de prueba o a una interpretación incorrecta de día/mes. Debe compararse en Power BI Desktop contra los valores visibles en el archivo fuente **antes del rediseño final y la publicación** (`SC-5` en adelante). No se investigó ni se corrigió en esta fase — la instrucción explícita del usuario fue no tocar la medida DAX hasta confirmar si el problema está en los datos de prueba, en el archivo Excel o en la transformación de `Fecha`.

Este pendiente no bloquea `SC-4` (duplicar la página no depende de resolverlo), pero queda anotado para retomarlo antes de `SC-5`.

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

**Pendiente**, sin cambios desde `SC-3` — no se abrió Power BI Desktop en esta fase (no era parte del alcance de `SC-4`). Sigue pendiente en particular la verificación de `Ultima Capacitacion = 2026-10-07` señalada en §2, que el usuario indicó debe resolverse antes de `SC-5`, no antes de `SC-4`.

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
| PBIP abre correctamente | Pendiente (requiere confirmación del usuario en Power BI Desktop) |
| Validación de medidas SC-3 | Pendiente — incluye explícitamente la revisión de `Ultima Capacitacion = 2026-10-07` antes de `SC-5` |
| ¿Se puede avanzar a SC-5? | **No todavía** — recomendado abrir Power BI Desktop primero para (a) confirmar que la copia carga sin error y (b) resolver el pendiente de `Ultima Capacitacion` señalado en §2, antes de iniciar el rediseño visual |

## 14. Commit

```
feat(report): crear copia pagina satisfaccion capacitaciones
```

Archivos incluidos: `pages.json`, la carpeta completa `p14_satisfaccion_capacitaciones_v2/`, y esta bitácora.

No se hizo push remoto.

## 15. Recomendación para continuar

Antes de iniciar `SC-5` (rediseño visual):

1. Abrir el `.pbip` en Power BI Desktop y confirmar que `p14_satisfaccion_capacitaciones_v2` carga sin error, con el mismo contenido visual que la original en este momento.
2. Colocar `Ultima Capacitacion` en una tarjeta de prueba y comparar contra los valores de fecha visibles directamente en `Data/Satisfacción capacitación (Responses).xlsx` (columna `Timestamp`), para descartar que `2026-10-07` sea un error de datos de prueba o una interpretación incorrecta de día/mes antes de construir el KPI "Última capacitación" del mockup.
3. Confirmar en la misma sesión los valores de `Call Centers Capacitados` (esperado `5`) y `Capacitaciones Realizadas` (esperado `5`), dejados como pendiente desde `SC-3`.
4. Solo después de (1)-(3), autorizar `SC-5`.
