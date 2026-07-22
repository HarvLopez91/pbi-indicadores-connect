# Resultado — cierre del plan de Satisfacción de capacitaciones

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Plan cerrado | [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Fecha | 2026-07-22 |
| Rama | `main` |
| Estado final | `Plan cerrado y listo para push` |

## Síntesis de SC-1 a SC-9

| Fase | Resultado | Evidencia |
|---|---|---|
| `SC-1` | Decisiones `DEC-1` a `DEC-4` confirmadas para ejecutar la adaptación. | [`Docs/05`](../Docs/05_decisiones_limitaciones_pendientes.md) |
| `SC-2` | Entorno Git y protección de la página original validados antes de cambios estructurales. | Bitácoras de la iniciativa |
| `SC-3` | Medidas de capacitación creadas y validadas. | [`Outputs/39`](39_resultado_sc3_medidas_satisfaccion_capacitaciones.md) |
| `SC-4` | Copia `p14_satisfaccion_capacitaciones_v2` creada como página de trabajo. | [`Outputs/40`](40_resultado_sc4_copia_pagina_satisfaccion_capacitaciones.md) |
| `SC-5` | Rediseño visual aprobado tras correcciones de layout, fechas y comentarios. | [`Outputs/43`](43_resultado_sc5_rediseno_visual_satisfaccion_capacitaciones.md) |
| `SC-6` | Interacciones `DataFilter`/`NoFilter` configuradas. | [`Outputs/44`](44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md) |
| `SC-7` | Validación técnica, funcional y visual aprobada en Power BI Desktop. | [`Outputs/45`](45_resultado_sc7_validacion_tecnica_funcional_visual.md) |
| `SC-8` | Documentación técnica actualizada. | [`Outputs/46`](46_resultado_sc8_documentacion_satisfaccion_capacitaciones.md) |
| `SC-9` | Página `v2` convertida en oficial y página original retirada del PBIR activo. | [`Outputs/47`](47_resultado_sc9_reemplazo_satisfaccion_capacitaciones.md) |

## Decisiones DEC-1 a DEC-4

- `DEC-1`: la clave provisional de capacitación única queda como `Fecha + CallCenter + NombreFormador`, válida para el piloto pero dependiente de `D9`.
- `DEC-2`: la tabla nominal de formador/líder se reemplaza por una tabla de detalle por call center.
- `DEC-3`: el gráfico de Jornada se retira del lienzo principal de Satisfacción de capacitaciones.
- `DEC-4`: el segmentador de Fecha conserva el comportamiento visual `Between` en la página de Satisfacción.

## Principales objetos creados

- Medidas en `_Medidas Capacitacion`: `Call Centers Capacitados`, `Ultima Capacitacion`, `Capacitaciones Realizadas`, `Valor Metrica Satisfaccion`, `Ultima Capacitacion Texto`.
- Tabla desconectada `Dim_MetricaSatisfaccion`.
- Columna `Dim_Calendario[Fecha Eje]` para etiquetas `dd/MM` en el eje.
- Columna `Fact_SatisfaccionCapacitacion[Fecha Texto]` para visualización explícita `dd/MM/yyyy`.
- Página `p14_satisfaccion_capacitaciones_v2`, ahora página oficial `Satisfacción de capacitaciones`.

## Corrección de fechas en Power Query

Durante `SC-5` se identificó que tres fechas del piloto habían quedado interpretadas como agosto, septiembre y octubre, aunque correspondían a julio de 2026. La corrección se aplicó en Power Query con el paso `TimestampNormalizado` dentro de `SatisfaccionCapacitacion_Limpio`.

Resultado validado:

- Fechas reales visibles: `04/07`, `06/07`, `08/07`, `09/07`, `10/07`.
- `Última capacitación`: `10/07/2026`.
- Sin categoría `(En blanco)` en el gráfico por fecha.

## Rediseño visual

La página oficial conserva:

- Encabezado y logo Connect.
- Filtros de Fecha, Call Center y Jornada.
- Seis KPI.
- Gráfico de capacitaciones por call center.
- Gráfico de capacitaciones por fecha.
- Panel de satisfacción con Satisfacción, Claridad, Utilidad y Dinamismo.
- Tabla de detalle por call center.
- Tabla de comentarios destacados sin vacíos, `Sin comentario` ni `"."`.
- Nota metodológica.

## Interacciones

`SC-6` configuró interacciones explícitas para que:

- Los segmentadores filtren los visuales de datos.
- Call center, fecha y detalle por call center funcionen como orígenes analíticos.
- Comentarios, panel y KPI sean receptores, no filtros hacia el resto del dashboard.
- El botón `Volver a Home` conserve navegación hacia Home.

La validación funcional fue aprobada en `SC-7` y reconfirmada en `SC-9`.

## Validaciones realizadas

- JSON del reporte válido.
- `pages.json` con 7 páginas.
- Home como página inicial.
- Una sola página oficial llamada `Satisfacción de capacitaciones`.
- Cero referencias activas al ID anterior `p14_satisfaccion_capacitaciones`.
- Cero referencias a `NombreFormador` o `NombreLider` en el reporte activo.
- Modelo semántico, DAX, Power Query, relaciones y `Data/` sin cambios durante el cierre documental.
- Documentación `Docs/00`, `Docs/02`, `Docs/03`, `Docs/05`, `Docs/06`, `Docs/07` y `README.md` actualizada en `SC-8`/`SC-9`.

## Reemplazo de la página anterior

En `SC-9`, `p14_satisfaccion_capacitaciones_v2` reemplazó definitivamente a la página original:

- La página nueva conserva su nombre técnico.
- Su nombre visible es `Satisfacción de capacitaciones`.
- Home apunta a la página nueva.
- La carpeta `p14_satisfaccion_capacitaciones` fue retirada del PBIR activo.
- Git conserva el respaldo histórico.

## Commits principales

- `d0c86f8 feat(dax): agregar medidas para satisfaccion capacitaciones`
- `ba4666e feat(report): crear copia pagina satisfaccion capacitaciones`
- `d4b58fd feat(report): redisenar satisfaccion de capacitaciones`
- `1bf9198 feat(report): configurar y validar interacciones de satisfaccion`
- `149a072 docs: actualizar documentacion satisfaccion capacitaciones`
- `63ea78d feat(report): reemplazar pagina de satisfaccion de capacitaciones`
- `b48c94e docs: cerrar reemplazo de pagina de satisfaccion`

## Estado final del reporte

- Número final de páginas: 7.
- Página inicial: Home (`67eff42d82e1c9c15b84`).
- Página oficial de Satisfacción: `p14_satisfaccion_capacitaciones_v2`.
- La página anterior no forma parte del PBIR activo.
- No hay nombres de formadores/líderes en el reporte activo.

## Riesgos abiertos

- `D9`: falta un identificador oficial de sesión de capacitación; la clave actual es provisional.
- El enlace público sin autenticación sigue implicando riesgo de exposición por `cl_tabla_asesor` (`NombreAsesor`) mientras Calidad de llamadas conserve ese desglose.
- La republicación en Power BI Service sigue pendiente; no se realizó durante esta iniciativa.

## Estado final

`Plan cerrado y listo para push`.
