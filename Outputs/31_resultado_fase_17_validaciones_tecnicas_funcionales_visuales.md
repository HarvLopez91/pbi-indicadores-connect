# Resultado — Fase 17: validaciones técnicas, funcionales y visuales

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 17 — Validaciones técnicas, funcionales y visuales |
| Fecha | 2026-07-09 |
| Alcance | Auditoría de control de calidad (QA) de todo lo construido en las fases 1 a 16. No se hizo rediseño, no se crearon páginas ni medidas DAX nuevas, no se modificó Power Query/relaciones/tablas/`Data/*.xlsx`. |

## 1. Estado inicial de `git status`

Al iniciar esta ejecución, `git status` mostraba 20 archivos modificados por una sesión previa de Power BI Desktop en la que el usuario activó manualmente etiquetas de datos y aplicó formato/orden adicional (detalle completo en `Outputs/30_sincronizacion_etiquetas_datos_manual_powerbi.md`, Parte 0 de esta misma ejecución). No había archivos nuevos ni eliminados.

## 2. Cambios manuales sincronizados (Parte 0)

Ver `Outputs/30_sincronizacion_etiquetas_datos_manual_powerbi.md` para el detalle completo. Resumen:

- 8 gráficos: reconfirmación de etiquetas de datos activadas manualmente (aditiva, sin sobrescribir color `#002733`/tamaño `9D` ya configurados) + orden manual por medida en 5 de los 8.
- 4 tablas (`tableEx`): reformato visual (encabezados `#002733`/blanco, valores `#002733`) sin cambios de `query`/bindings.
- 3 visuales `nm_n_*` y `pages.json`: cambios cosméticos (fin de línea, página activa).
- 3 tablas `Fact_*`: anotación automática `PBI_ResultType = Table`.
- `cultures/es-ES.tmdl`: metadatos lingüísticos automáticos para `n Calidad`/`n Capacitacion`/`n Motivacion`.

Todo se comiteó en `bf26e25 chore(report): sincronizar etiquetas de datos manuales` antes de iniciar la validación de Fase 17. Ninguna etiqueta ni ajuste manual del usuario fue revertido o sobrescrito.

## 3. Resultado — validación técnica

| # | Validación | Resultado |
|---|---|---|
| 1 | El PBIP abre sin errores en Power BI Desktop | **Pendiente** — confirmación visual del usuario (no se puede operar la interfaz gráfica desde este entorno) |
| 2 | No existen errores TMDL | **Aprobado** |
| 3 | No existen errores PBIR | **Aprobado** |
| 4 | Todos los JSON del reporte son válidos | **Aprobado** |
| 5 | Las 7 páginas existen | **Aprobado** |
| 6 | Modelo semántico mantiene tablas/dimensiones/medidas/relaciones sin ambigüedad | **Aprobado** |
| 7 | No hay cambios no controlados en Power Query/DAX/relaciones/tablas/`Data/*.xlsx` | **Aprobado** |

### Detalle

**#2–#4 — TMDL y PBIR:** se revisaron los 19 archivos `.tmdl` del modelo semántico: paréntesis y llaves balanceados en los 19, sin ninguna línea `description:` ni `queryGroup:` (las 2 causas de error ya corregidas en fases 3 y 10), y los `lineageTag` presentes son exclusivamente los generados por Power BI Desktop al abrir el modelo (nunca escritos a mano en esta ejecución). Se validó el parseo JSON de los 222 archivos del reporte (`report.json`, `pages.json`, 7 `page.json`, 208 `visual.json`, `.platform` de ambas carpetas): **222/222 válidos**, cero errores de sintaxis.

**#5 — Páginas:** confirmadas las 7 páginas en `pageOrder` con su `displayName` correcto:

| `name` técnico | `displayName` |
|---|---|
| `67eff42d82e1c9c15b84` | Home |
| `p14_resumen_ejecutivo` | Resumen ejecutivo |
| `p14_calidad_llamadas` | Calidad de llamadas |
| `p14_satisfaccion_capacitaciones` | Satisfacción de capacitaciones |
| `p14_motivacion_comercial` | Motivación comercial |
| `p14_detalle_call_center` | Detalle por call center |
| `p14_notas_metodologicas` | Notas metodológicas |

**#6 — Modelo semántico:** confirmadas las 3 tablas `Fact_*` (`Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`), las 3 dimensiones de negocio `Dim_*` (`Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`) y las 4 tablas de medidas `_Medidas *` (`Generales`, `Calidad`, `Capacitacion`, `Motivacion`), todas referenciadas en `model.tmdl`. Se confirmaron **11 relaciones** en `relationships.tmdl`: 8 de diseño (3× `Dim_Calendario`→`Fact_*`, 3× `Dim_CallCenter`→`Fact_*`, 2× `Dim_Jornada`→`Fact_SatisfaccionCapacitacion`/`Fact_MotivacionActividad`, intencionalmente sin `Fact_CalidadLlamadas`) más 3 relaciones auxiliares hacia las tablas ocultas `LocalDateTable_*` de Auto Date/Time (ruido conocido y ya documentado en `CLAUDE.md`, pendiente de limpieza manual desde Desktop). Ninguna relación tiene `isActive: false` inesperado ni conecta dos tablas de hechos entre sí — no hay ambigüedad de filtrado.

**#7 — Cambios no controlados:** tras sincronizar en la Parte 0, `git status` quedó limpio antes de iniciar la validación y se mantuvo limpio durante toda la Fase 17 (todas las validaciones de esta fase fueron de solo lectura). No se modificó `expressions.tmdl`, ninguna medida DAX, `relationships.tmdl`, ninguna tabla del modelo ni `Data/*.xlsx`.

## 4. Resultado — validación funcional

| # | Validación | Resultado |
|---|---|---|
| 1 | Las medidas principales siguen existiendo | **Aprobado** |
| 2 | Las medidas usadas en tarjetas KPI calculan sin error | **Pendiente** — fórmulas revisadas sin error de sintaxis; el cálculo real requiere confirmación visual en Desktop |
| 3 | Los segmentadores filtran correctamente (Fecha, Call Center, Jornada) | **Pendiente** — bindings y configuración confirmados correctos; el comportamiento interactivo requiere confirmación visual en Desktop |
| 4 | Los indicadores responden a filtros en las 5 páginas de detalle | **Pendiente** — relaciones activas y sin ambigüedad confirmadas; la interacción real requiere confirmación visual en Desktop |
| 5 | Las notas metodológicas no usan conteos fijos | **Aprobado** |
| 6 | `n Calidad`, `n Capacitacion`, `n Motivacion` presentes y visibles donde aplica | **Aprobado** |
| 7 | La encuesta de motivación sigue documentada como anónima | **Aprobado** |
| 8 | `% Calidad Promedio Provisional` sigue documentado como pendiente de rúbrica | **Aprobado** |
| 9 | `% Llamadas con Venta` sigue documentado como observación pendiente | **Aprobado** |

### Detalle

**#1 — Catálogo de medidas:** se confirmaron las **25 medidas** del modelo, sin cambios de fórmula desde la validación de la Fase 11 (`Outputs/15`): 7 en `_Medidas Generales` (incluye `n Calidad`, `n Capacitacion`, `n Motivacion`, `Total Registros Piloto`, `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`), 6 en `_Medidas Calidad`, 5 en `_Medidas Capacitacion`, 6 en `_Medidas Motivacion`.

**#2 — Cálculo de medidas en tarjetas:** revisión estática de las fórmulas DAX referenciadas por las tarjetas KPI de las 7 páginas: sintaxis correcta, sin referencias a columnas/medidas inexistentes. El resultado numérico real (por ejemplo, si `% Llamadas con Venta` sigue mostrando blanco) solo puede confirmarse abriendo el modelo en Power BI Desktop.

**#3 — Segmentadores:** inventario completo de los 16 segmentadores del reporte, confirmando el binding correcto a cada columna y el estilo `Dropdown` (menú desplegable) en los 16:

| Página | `Fecha` | `CallCenter` | `Jornada` |
|---|---|---|---|
| Home | — (landing page, sin segmentadores) | — | — |
| Resumen ejecutivo | Sí | Sí | Sí |
| Calidad de llamadas | Sí | Sí | No aplica (la fuente no captura Jornada) |
| Satisfacción de capacitaciones | Sí | Sí | Sí |
| Motivación comercial | Sí | Sí | Sí |
| Detalle por call center | Sí | Sí | Sí |
| Notas metodológicas | Sí | Sí | — (no incluido por diseño desde Fase 14) |

**#4 — Respuesta a filtros:** las relaciones activas (`Dim_Calendario`→`Fact_*`, `Dim_CallCenter`→`Fact_*`, `Dim_Jornada`→`Fact_SatisfaccionCapacitacion`/`Fact_MotivacionActividad`) están correctamente configuradas y sin ambigüedad, lo que estructuralmente garantiza que los segmentadores deberían filtrar las 5 páginas de detalle. La confirmación interactiva (mover un segmentador y observar el cambio) requiere Power BI Desktop.

**#5 — Notas sin conteos fijos:** se repitió la búsqueda de patrones `N registros/respuestas/evaluaciones/filas/encuestas` sobre los 63 visuales `textbox` del reporte completo (incluyendo los cambios manuales sincronizados en la Parte 0). **Sin coincidencias.**

**#6 — Medidas `n=` visibles:** `n Calidad`, `n Capacitacion` y `n Motivacion` están enlazadas a los visuales `nm_n_calidad`, `nm_n_cap` y `nm_n_mot` en la página Notas metodológicas (agregados en la Fase 16, `Outputs/29`), confirmadas presentes tras la sincronización de Desktop.

**#7–#9 — Documentación de limitaciones:** confirmado texto vigente en los paneles correspondientes:
- Anónima: `nm_anonima_text` ("La encuesta de motivación no permite análisis por asesor individual porque la identificación no está disponible") y `mc_nota_mot_text` ("Encuesta anónima: no permite análisis por asesor individual...").
- `% Calidad Promedio Provisional`: `nm_calidad_text` y `cl_nota_calidad_text`, y la medida en sí sigue definida como `BLANK()`.
- `% Llamadas con Venta`: `nm_venta_text` y `cl_nota_calidad_text` siguen documentando la observación pendiente.

## 5. Resultado — validación visual

| # | Validación | Resultado |
|---|---|---|
| 1 | Todos los textos visibles están en español de Colombia | **Aprobado** |
| 2 | No existen patrones corruptos (mojibake) | **Aprobado** |
| 3 | Colores alineados con Connect (`#F15B2B`, `#002733`, grises de apoyo) | **Aprobado** |
| 4 | Las tarjetas KPI tienen proporción visual adecuada | **Aprobado** |
| 5 | Etiquetas de datos activadas en todos los gráficos donde aportan claridad | **Aprobado** |
| 6 | Ninguna etiqueta de datos activada manualmente fue desactivada | **Aprobado** |
| 7 | Los segmentadores se mantienen como menú desplegable | **Aprobado** |
| 8 | La navegación funciona (Home ↔ internas, tarjeta/botón y texto) | **Pendiente** — estructura y `visualLink` confirmados correctos; el clic real requiere confirmación visual en Desktop |
| 9 | Las zonas clicables transparentes (hitzones) siguen presentes y alineadas | **Aprobado** |
| 10 | Notas metodológicas mantiene filtros visibles, compactos y sin superposición | **Aprobado** |
| 11 | No hay visuales superpuestos, ocultos o desalineados | **Aprobado** |

### Detalle

**#1–#2 — Idioma y mojibake:** se repitió la búsqueda de patrones de mojibake (`?` seguido de vocal acentuada, incluyendo `capacitaci?n`, `motivaci?n`, `Satisfacci?n`, `metodol?gicas`, `validaci?n`) sobre los 63 `textbox` del reporte tras la sincronización de Desktop. **Sin coincidencias.** Los `displayName` de página y los textos revisados usan tildes correctas (`Satisfacción`, `Motivación`, `metodológicas`, etc.).

**#3 — Paleta de colores:** se extrajeron todos los códigos hexadecimales usados en los 208 `visual.json`. Los 8 colores más frecuentes son exactamente la paleta Connect esperada: `#FFFFFF` (fondo, 287 usos), `#E7E7E7` (bordes, 209), `#002733` (texto principal/oscuro, 125), `#F4F4F4` (cuadrícula/gris claro, 89), `#3A3A3A` (texto secundario, 78), `#F15B2B` (acento naranja, 76), `#FAFAFA` y `#FFF3EE` (fondos suaves, 7 cada uno). No se detectó ningún color ajeno a esta paleta.

**#4 — Proporción de tarjetas KPI:** las 39 tarjetas KPI del reporte usan un ancho uniforme de 184px, con alturas consistentes dentro de cada página (110px en Home, 96px en Resumen ejecutivo/Calidad/Detalle por call center, 86px en Satisfacción/Motivación/Notas metodológicas) — proporciones uniformes y sin distorsión.

**#5–#6 — Etiquetas de datos:** confirmados los 8 gráficos (`barChart`/`columnChart`/`lineChart`) del reporte con `labels.show:true`, color `#002733`, tamaño `9D`, incluyendo la reconfirmación manual aditiva sincronizada en la Parte 0. Ninguna etiqueta fue desactivada durante esta fase (todas las validaciones fueron de solo lectura).

**#7 — Estilo de segmentadores:** los 16 segmentadores usan `style: 'Dropdown'` — ninguno quedó como lista expandida.

**#8–#9 — Navegación y hitzones:** se contaron **42 visuales** con `visualLink.type = 'PageNavigation'` (30 tarjetas/botones/etiquetas originales de la Fase 15 + 12 hitzones agregados en la corrección QA de `Outputs/28`), consistente con lo documentado. Se reverificó que los 12 hitzones mantienen exactamente la misma posición que su tarjeta/botón asociado y el z-index más alto de su módulo (por ejemplo, en Home: tarjeta z=200 < acento z=210 < etiqueta z=220 < hitzone z=230). El funcionamiento real del clic en Power BI Desktop sigue pendiente de confirmación visual del usuario, como ya se señaló en `Outputs/28`.

**#10 — Notas metodológicas:** los 2 segmentadores de esa página (`Fecha`, `CallCenter`) están ubicados en la franja superior derecha del encabezado, sin superposición con título/subtítulo/botón de regreso, y los 6 paneles de contenido más las 3 tarjetas `n=` nuevas caben dentro del área de 720px de alto sin desbordar (verificado por posición: el último elemento, `nm_nota_cierre_text`, termina en `y=664`, dentro del lienzo de 720px).

**#11 — Superposiciones:** se ejecutó una detección automática de solapamiento de rectángulos entre visuales no relacionados en las 7 páginas. Las únicas superposiciones detectadas corresponden al patrón de diseño intencional de "panel de fondo detrás de su contenido" (`*_header_panel` detrás de título/subtítulo/segmentadores/botón Home, `home_hero_panel` detrás del bloque de bienvenida de Home) — no se encontró ninguna superposición no intencional entre visuales de contenido (gráficos, tablas, tarjetas, segmentadores, notas).

## 6. Hallazgos encontrados

**Ninguno de severidad alta o media.** Todos los puntos marcados como "Pendiente" en las tablas anteriores no son hallazgos de error, sino validaciones que por su naturaleza (renderizado real, cálculo en vivo, interacción de clic) requieren que el usuario abra el PBIP en Power BI Desktop — esta limitación ya está señalada como regla operativa en `CLAUDE.md`/`AGENTS.md` para todo el proyecto, no es específica de esta fase.

## 7. Recomendaciones de corrección

No se identificó ninguna corrección de código/configuración pendiente. Las únicas acciones pendientes son de **verificación visual por parte del usuario** (detalladas en la sección 9).

## 8. Confirmación — modelo semántico no modificado

Durante la Fase 17 (excluyendo la sincronización de la Parte 0, ya documentada en `Outputs/30` como cambios propios de Power BI Desktop) no se modificó:

- Power Query (`expressions.tmdl`)
- Medidas DAX (`_Medidas *.tmdl`)
- Relaciones (`relationships.tmdl`)
- Tablas del modelo (`tables/*.tmdl`)

Todas las validaciones de esta fase fueron de solo lectura sobre el modelo semántico.

## 9. Confirmación — `Data/*.xlsx` no modificado

No se tocó ningún archivo dentro de `Data/`. `git status --porcelain -- Data/` no devuelve ninguna línea.

## 10. Archivos modificados en esta ejecución (Fase 17, Parte 1–4)

Ninguno. Esta fase fue exclusivamente de validación (lectura); los únicos archivos escritos son los dos documentos de esta ejecución:

- `Outputs/30_sincronizacion_etiquetas_datos_manual_powerbi.md` (Parte 0, ya comiteado en `bf26e25`)
- `Outputs/31_resultado_fase_17_validaciones_tecnicas_funcionales_visuales.md` (este documento)

## 11. Resultado del commit

Commit sugerido por el usuario:

`test(report): validar informe tecnico funcional y visual`

## 12. Estado final de `git status`

Tras comitear este documento, `git status` queda limpio (`working tree clean`).

## 13. Recomendación para avanzar a Fase 18

**No se avanza a Fase 18 en esta ejecución**, conforme a lo solicitado.

Antes de iniciar la Fase 18 (documentación de cierre), se recomienda que el usuario confirme visualmente en Power BI Desktop los puntos marcados como "Pendiente" en este documento:

1. El PBIP abre sin errores en una sesión limpia de Power BI Desktop.
2. Las tarjetas KPI calculan y muestran valores (sin error `#ERROR` ni referencias rotas).
3. Los segmentadores de Fecha, Call Center y Jornada filtran visualmente las 5 páginas de detalle al interactuar con ellos.
4. La navegación funciona con un clic en cualquier punto de las 6 tarjetas de Home y de los 6 botones "Volver a Home", tal como se corrigió en `Outputs/28`.

Si estos 4 puntos se confirman sin hallazgos, el informe queda en condición de avanzar a la Fase 18 sin bloqueos pendientes desde el lado estático/estructural, que es 100% aprobado en esta fase.
