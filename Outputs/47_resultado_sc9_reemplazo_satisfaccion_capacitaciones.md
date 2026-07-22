# Resultado — SC-9: reemplazo controlado de Satisfacción de capacitaciones

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | `SC-9` de [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Fecha | 2026-07-22 |
| Commit base | `149a072 docs: actualizar documentacion satisfaccion capacitaciones` |
| Estado final | `SC-9 pendiente de validación final en Desktop` |

## Estado inicial de Git

- Raíz detectada: `C:/Users/eclavijo/OneDrive/PBI_Indicadores`.
- Rama: `main`.
- Power BI Desktop cerrado.
- Commit `149a072` presente en `git log`.
- Working tree limpio después de revertir dos metadatos no funcionales generados por Desktop:
  - `PBI/PBI_Indicadores.Report/definition/pages/pages.json` (`activePageName` de sesión previa).
  - `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/sc_kpi_respuestas_panel/visual.json` (salto de línea final).

No se creó commit antes de iniciar `SC-9`.

## Decisión adoptada

Se aplicó el reemplazo definitivo y controlado:

- `p14_satisfaccion_capacitaciones_v2` queda como página oficial.
- Su nombre visible pasa a `Satisfacción de capacitaciones`.
- La página original `p14_satisfaccion_capacitaciones` se retira del PBIR activo.
- El respaldo histórico queda en Git.
- La tabla nominal de formador/líder deja de formar parte del informe activo.
- La clave provisional de capacitaciones sigue dependiendo de `D9`; el reemplazo no la convierte en definición oficial de negocio.

## Navegación anterior y nueva

Antes de `SC-9`, los cuatro elementos del módulo de Home apuntaban a:

- `p14_satisfaccion_capacitaciones`

Después de `SC-9`, los cuatro apuntan a:

- `p14_satisfaccion_capacitaciones_v2`

Elementos actualizados:

- `home_nav_03_card`
- `home_nav_03_accent`
- `home_nav_03_label`
- `home_nav_03_hitzone`

Se conservaron diseño, posición, tamaño, texto, tooltip, acción `PageNavigation` y orden `z`.

## Página convertida en oficial

Archivo:

- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json`

Cambio aplicado:

- `displayName`: `Satisfacción de capacitaciones (v2 - borrador)` -> `Satisfacción de capacitaciones`

Se conservaron:

- Nombre técnico `p14_satisfaccion_capacitaciones_v2`.
- Visuales e IDs.
- Layout.
- Interacciones `visualInteractions`.
- Segmentadores.
- Botón `Volver a Home`.
- Medidas y campos.
- Configuración del lienzo.

## Página original retirada

Se eliminó del PBIR activo la carpeta:

- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/`

También se retiró su entrada de `pages.json`.

`pages.json` queda con 7 páginas y Home como página inicial:

1. `67eff42d82e1c9c15b84`
2. `p14_resumen_ejecutivo`
3. `p14_calidad_llamadas`
4. `p14_satisfaccion_capacitaciones_v2`
5. `p14_motivacion_comercial`
6. `p14_detalle_call_center`
7. `p14_notas_metodologicas`

## Archivos eliminados, modificados y creados

### Eliminados

- Carpeta completa `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/`.

### Modificados

- `PBI/PBI_Indicadores.Report/definition/pages/pages.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_03_card/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_03_accent/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_03_label/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_03_hitzone/visual.json`
- `Docs/00_indice_documentacion.md`
- `Docs/02_catalogo_medidas_dax.md`
- `Docs/03_mapa_reporte_paginas_visuales.md`
- `Docs/05_decisiones_limitaciones_pendientes.md`
- `Docs/06_publicacion_powerbi.md`
- `Docs/07_arquitectura_sistema.md`
- `README.md`

### Creados

- `Outputs/47_resultado_sc9_reemplazo_satisfaccion_capacitaciones.md`

## Documentos actualizados

- `Docs/00`: conteo corregido a 30 medidas, dependencias D1-D9 y referencia a `Docs/05 §5`.
- `Docs/02`: usos de medidas actualizados para reflejar que la página oficial es `p14_satisfaccion_capacitaciones_v2`; se retiraron referencias activas a visuales de la página original.
- `Docs/03`: el informe vuelve a documentarse como 7 páginas oficiales; se elimina la sección separada de `v2 - borrador`; Home queda documentado apuntando a la nueva página oficial.
- `Docs/05`: se agrega la decisión `SC-9`, se elimina el pendiente de reemplazo/coexistencia/descarte y se mantiene `D9` como pendiente.
- `Docs/06`: se retira `sc_tabla_formador` de la lista de visuales activos con nombres personales; se mantiene el riesgo de `cl_tabla_asesor`.
- `Docs/07` y `README`: conteos y referencias de dependencias ajustados a 30 medidas y D1-D9.

## Confirmación de modelo semántico

No se modificó el modelo semántico:

- Sin cambios en DAX.
- Sin cambios en Power Query.
- Sin cambios en relaciones.
- Sin cambios en `compatibilityLevel`.
- Sin cambios en `RutaCarpetaData`.
- Sin cambios en `Data/`.

## Validaciones técnicas

- Todos los JSON del reporte parsean correctamente.
- `pages.json` contiene exactamente 7 páginas.
- Home permanece como página inicial (`activePageName = 67eff42d82e1c9c15b84`).
- Solo existe una página visible llamada `Satisfacción de capacitaciones`.
- La carpeta `p14_satisfaccion_capacitaciones` ya no existe.
- Los cuatro elementos `home_nav_03_*` apuntan a `p14_satisfaccion_capacitaciones_v2`.
- `sc_home_btn`, `sc_home_label` y `sc_home_hitzone` siguen apuntando a Home.
- No quedan referencias exactas al ID anterior `p14_satisfaccion_capacitaciones` como destino o página activa.
- No hay referencias a `NombreFormador` ni `NombreLider` en el reporte activo.
- `Docs/00` indica 30 medidas, D1-D9 y referencia a `Docs/05 §5`.
- `Docs/06` ya no señala `sc_tabla_formador` como visual activo publicado.

## Pruebas pendientes en Power BI Desktop

Abrir `PBI/PBI_Indicadores.pbip` y confirmar:

1. El informe abre sin errores.
2. Home es la página inicial.
3. La tarjeta completa de Satisfacción abre la nueva página oficial.
4. Solo existe una pestaña de Satisfacción.
5. El nombre visible ya no contiene `v2 - borrador`.
6. La página conserva el diseño, filtros e interacciones aprobados.
7. `Volver a Home` funciona en toda su superficie.
8. Las otras cinco tarjetas de Home siguen navegando correctamente.

## Estado de publicación

El PBIP queda preparado para reemplazo en la próxima republicación. No se republicó desde Power BI Service; la republicación sigue siendo manual y debe ocurrir solo después de validar en Power BI Desktop.

## Estado final

`SC-9 pendiente de validación final en Desktop`.

No se creó commit y no se hizo push.

## Validación en Power BI Desktop y cierre de SC-9

La validación visual y funcional final fue aprobada por el usuario en Power BI Desktop.

### Resultado confirmado

- El PBIP abrió correctamente.
- Home quedó como página inicial al cerrar y volver a abrir el proyecto.
- La navegación Home -> Satisfacción funciona desde toda la tarjeta `Satisfacción de capacitaciones`.
- La navegación Satisfacción -> Home funciona desde toda la superficie de `Volver a Home`.
- Solo existe una página oficial de Satisfacción de capacitaciones.
- El nombre visible definitivo es `Satisfacción de capacitaciones`.
- El diseño aprobado se conserva: seis KPI, filtros, dos gráficos, panel de satisfacción, tablas y nota metodológica.
- Las interacciones aprobadas en `SC-6`/`SC-7` se conservaron.
- Las fechas continúan correctas.
- Las cuatro métricas del panel siguen visibles.
- No aparece la tabla nominal de formadores y líderes.
- Las otras cinco rutas de navegación de Home continúan funcionando.

### Limpieza posterior a Desktop

Después de la sesión de validación, se revisaron los cambios generados por Power BI Desktop y se restauraron únicamente metadatos sin valor funcional:

- Cambio exclusivo de salto de línea final en `sc_kpi_respuestas_panel/visual.json`.
- Cambios automáticos en `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl`.
- `activePageName` se dejó explícitamente en Home (`67eff42d82e1c9c15b84`).

### Estado de publicación

El PBIP queda listo para republicación manual en Power BI Service. No se republicó desde esta fase y el enlace público debe validarse después de la republicación.

### Estado final aprobado

`SC-9 aprobada y cerrada`.
