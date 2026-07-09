# Mapa del reporte — páginas y visuales

Inventario de las **7 páginas** del informe (`PBI/PBI_Indicadores.Report/definition/pages/`), construido leyendo directamente los `page.json` y `visual.json` vigentes. El catálogo de medidas referenciadas está en [`02_catalogo_medidas_dax.md`](02_catalogo_medidas_dax.md).

## 1. Home

- **Nombre técnico de página:** `67eff42d82e1c9c15b84` · **Rol:** landing page / punto de entrada.
- **Objetivo:** orientar al usuario con una vista ejecutiva de 6 KPI y navegación visual hacia las 6 páginas internas — no analiza, solo orienta.
- **Indicadores principales:** Total de registros del piloto, Total de evaluaciones de calidad, Total de respuestas de capacitación, Total de respuestas de motivación, Índice global de capacitación, Índice global de motivación.
- **Medidas usadas:** `Total Registros Piloto`, `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`, `Indice Global Capacitacion`, `Indice Global Motivacion`.
- **Visuales principales:** 6 tarjetas KPI (`home_kpi_*`), logo Connect, título y subtítulo, insignia "piloto" (`home_pilot_badge`), 6 tarjetas de navegación (tarjeta + acento + etiqueta + zona clicable por módulo).
- **Segmentadores:** ninguno (landing page).
- **Notas visibles:** *"Los datos actuales corresponden a una muestra piloto. Interpretar los indicadores considerando el n de respuestas."* (`home_method_note_text`); insignia `home_pilot_note`.
- **Navegación:** 6 módulos, cada uno con tarjeta + acento + etiqueta + zona clicable transparente (hitzone) que cubre el área completa, todos apuntando a su página interna correspondiente (Resumen ejecutivo, Calidad de llamadas, Satisfacción de capacitaciones, Motivación comercial, Detalle por call center, Notas metodológicas).

## 2. Resumen ejecutivo

- **Nombre técnico:** `p14_resumen_ejecutivo`.
- **Objetivo:** vista general del piloto comercial, formativo y de calidad en una sola página, con tendencia en el tiempo y comparación por call center.
- **Indicadores principales:** Total de evaluaciones de calidad, Total de respuestas de capacitación, Total de respuestas de motivación, Total de registros del piloto, Índice global de capacitación, Índice global de motivación.
- **Medidas usadas:** las 6 anteriores; los 2 gráficos usan `Total Registros Piloto`.
- **Visuales principales:** 6 tarjetas KPI (`re_kpi_*`), gráfico de columnas por call center (`re_chart_callcenter`), gráfico de líneas por fecha (`re_chart_fecha`).
- **Segmentadores:** Fecha (`Dim_Calendario.Fecha`), Call Center (`Dim_CallCenter.CallCenter`), Jornada (`Dim_Jornada.Jornada`).
- **Notas visibles:** *"Muestra piloto: interpretar cada indicador con base en el n visible y evitar conclusiones definitivas hasta aumentar el volumen de respuestas."*
- **Navegación:** botón "Volver a Home" (botón + etiqueta + zona clicable) hacia Home.

## 3. Calidad de llamadas

- **Nombre técnico:** `p14_calidad_llamadas`.
- **Objetivo:** seguimiento de calidad comercial de llamadas y hallazgos por operación, a partir del checklist de auditoría.
- **Indicadores principales:** Total de evaluaciones, puntaje obtenido, preguntas aplicables, promedio de puntaje, % de llamadas con venta, objeción principal.
- **Medidas usadas:** `Total Evaluaciones Calidad`, `Puntaje Obtenido Calidad`, `Preguntas Aplicables Calidad`, `Promedio Puntaje Calidad`, `% Llamadas con Venta`, `Objecion Principal`.
- **Visuales principales:** 6 tarjetas KPI (`cl_kpi_*`), gráfico de barras de evaluaciones por call center (`cl_chart_callcenter`), tabla `cl_tabla_asesor` (asesor, total de evaluaciones, promedio de puntaje, % llamadas con venta).
- **Segmentadores:** Fecha, Call Center. **No incluye Jornada** — la fuente de calidad no captura esa pregunta (ver [Docs/01_modelo_datos.md](01_modelo_datos.md) §1).
- **Notas visibles:** *"% Calidad Promedio Provisional queda en blanco hasta confirmar rúbrica oficial. % Llamadas con Venta puede mostrarse en blanco en el piloto; no se ajusta la medida en esta fase."*
- **Navegación:** botón "Volver a Home" hacia Home.

## 4. Satisfacción de capacitaciones

- **Nombre técnico:** `p14_satisfaccion_capacitaciones`.
- **Objetivo:** percepción de los participantes sobre claridad, utilidad y dinamismo de las capacitaciones recibidas.
- **Indicadores principales:** satisfacción, claridad, utilidad y dinamismo promedio, índice global de capacitación, % de respuestas con comentario, total de respuestas.
- **Medidas usadas:** `Satisfaccion Promedio Capacitacion`, `Claridad Promedio Capacitacion`, `Utilidad Promedio Capacitacion`, `Dinamismo Promedio Capacitacion`, `Indice Global Capacitacion`, `% Comentarios Capacitacion`, `Total Respuestas Capacitacion`.
- **Visuales principales:** 7 tarjetas KPI (`sc_kpi_*`), gráfico de columnas del índice global por call center (`sc_chart_callcenter`), gráfico de barras del índice global por jornada (`sc_chart_jornada`), tabla `sc_tabla_formador` (formador, líder, total de respuestas, índice global).
- **Segmentadores:** Fecha, Call Center, Jornada.
- **Notas visibles:** *"Resultados sobre datos piloto. Usar junto con n de respuestas y validar alias de líderes/formadores antes de conclusiones nominales."*
- **Navegación:** botón "Volver a Home" hacia Home.

## 5. Motivación comercial

- **Nombre técnico:** `p14_motivacion_comercial`.
- **Objetivo:** motivación y percepción de las actividades comerciales por operación — **sin desglose por asesor**, porque la encuesta es anónima.
- **Indicadores principales:** satisfacción, claridad/utilidad y motivación promedio, índice global de motivación, % de ambiente motivado, % de respuestas con comentario, total de respuestas.
- **Medidas usadas:** `Satisfaccion Promedio Actividad`, `Claridad Utilidad Promedio Actividad`, `Motivacion Promedio Actividad`, `Indice Global Motivacion`, `% Ambiente Motivado`, `% Comentarios Motivacion`, `Total Respuestas Motivacion`.
- **Visuales principales:** 7 tarjetas KPI (`mc_kpi_*`), gráfico de columnas del índice global por call center (`mc_chart_callcenter`), gráfico de barras del índice global por jornada (`mc_chart_jornada`), tabla `mc_tabla_ambiente` (ambiente de equipo, total de respuestas, % ambiente motivado).
- **Segmentadores:** Fecha, Call Center, Jornada.
- **Notas visibles:** *"Encuesta anónima: no permite análisis por asesor individual. Interpretar con base en el n visible del piloto."*
- **Navegación:** botón "Volver a Home" hacia Home.

## 6. Detalle por call center

- **Nombre técnico:** `p14_detalle_call_center`.
- **Objetivo:** comparativo operativo por call center, cruzando las 3 fuentes en una sola tabla y un gráfico de volumen.
- **Indicadores principales:** total de evaluaciones de calidad, total de respuestas de capacitación, total de respuestas de motivación, total de registros del piloto, índice global de capacitación, índice global de motivación.
- **Medidas usadas:** las 6 anteriores.
- **Visuales principales:** 6 tarjetas KPI (`dc_kpi_*`), gráfico de barras de registros totales por call center (`dc_chart_registros`), tabla comparativa `dc_tabla_callcenter` (call center, total de evaluaciones de calidad, total de respuestas de capacitación, total de respuestas de motivación, índice global de capacitación, índice global de motivación).
- **Segmentadores:** Fecha, Call Center, Jornada.
- **Notas visibles:** *"Comparativo operativo de referencia. Evitar conclusiones fuertes por el bajo volumen actual del piloto."*
- **Navegación:** botón "Volver a Home" hacia Home.

> Nota de alcance: el plan original (`Specs/01`, `Specs/02`) proponía además una página separada "Detalle por asesor/líder". En la implementación final ese desglose por asesor quedó cubierto dentro de las tablas `cl_tabla_asesor` (Calidad de llamadas) y `sc_tabla_formador` (Satisfacción de capacitaciones), sin una página dedicada adicional — el informe final tiene 7 páginas, no 8.

## 7. Notas metodológicas

- **Nombre técnico:** `p14_notas_metodologicas`.
- **Objetivo:** documentar de forma centralizada el alcance, las fuentes, las limitaciones y los pendientes de negocio del piloto, para que ningún indicador se lea como definitivo.
- **Indicadores principales:** total de evaluaciones de calidad, total de respuestas de capacitación, total de respuestas de motivación, total de registros del piloto, más 3 leyendas dinámicas `n=` de calidad/capacitación/motivación.
- **Medidas usadas:** `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`, `Total Registros Piloto`, `n Calidad`, `n Capacitacion`, `n Motivacion`.
- **Visuales principales:** 4 tarjetas KPI + 3 leyendas `n=` dinámicas, 6 paneles de texto (Fuentes de datos, Estado piloto, Encuesta anónima, Calidad provisional, Llamadas con venta, Pendientes de negocio), nota de cierre.
- **Segmentadores:** Fecha, Call Center. No incluye Jornada (decisión de diseño desde la Fase 14 — página informativa, no de desglose operativo).
- **Notas visibles (resumen; texto completo de cada panel en [Docs/05_decisiones_limitaciones_pendientes.md](05_decisiones_limitaciones_pendientes.md)):** alcance del piloto, fuentes en `Data/` y su actualización dinámica, estado piloto e interpretación por `n`, encuesta de motivación anónima, `% Calidad Promedio Provisional` pendiente de rúbrica, `% Llamadas con Venta` con observación pendiente, pendientes de negocio (rúbrica, catálogo oficial, alias de líderes, colores de marca).
- **Navegación:** botón "Volver a Home" hacia Home.

## Resumen de navegación global

| Desde | Hacia | Mecanismo |
|---|---|---|
| Home → cualquier página interna | 6 destinos | Tarjeta + acento + etiqueta + zona clicable transparente, mismo `visualLink` en los 4 elementos del módulo |
| Cualquier página interna → Home | 1 destino | Botón "Volver a Home" + etiqueta + zona clicable transparente |

Total de visuales con acción de navegación (`PageNavigation`) en el reporte: **42** (24 en Home — 6 módulos × 4 elementos — y 18 en las 6 páginas internas — 3 elementos × 6 páginas). Detalle técnico de la corrección de áreas clicables en [Outputs/28_correccion_qa_final_navegacion_data_labels.md](../Outputs/28_correccion_qa_final_navegacion_data_labels.md).

## Resumen de segmentadores por página

| Página | Fecha | Call Center | Jornada |
|---|---|---|---|
| Home | — | — | — |
| Resumen ejecutivo | Sí | Sí | Sí |
| Calidad de llamadas | Sí | Sí | No (fuente sin esa columna) |
| Satisfacción de capacitaciones | Sí | Sí | Sí |
| Motivación comercial | Sí | Sí | Sí |
| Detalle por call center | Sí | Sí | Sí |
| Notas metodológicas | Sí | Sí | — (no incluida por diseño) |

Los 16 segmentadores del reporte usan estilo de menú desplegable (`Dropdown`), no lista expandida.
