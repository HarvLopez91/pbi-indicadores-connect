# Plan de implementación — Porcentaje de cumplimiento de meta comercial

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Baseline | `b2c8ab334577090e954d8e8e734f5c8494419345` |
| Análisis de impacto | `Specs/10_analisis_impacto_porcentaje_cumplimiento_gestion_comercial.md` |
| Fecha | 2026-08-18 |
| Estado | Plan listo para aprobación; modelo no modificado |

## 1. Objetivo

Incorporar metas comerciales mensuales como datos, calcular `% Cumplimiento = Altas / Meta asignada` y reemplazar únicamente la segunda proyección de la tarjeta `gc_kpi_mes_anterior`, preservando el promedio histórico, el layout y la lógica auxiliar de Meta +30 %.

La solución debe asignar `UNO 27` a PUSHER 1 exclusivamente en julio de 2026, sin reexpresar enero-junio.

## 2. Decisiones congeladas

- Fuente de ventas: `Fact_AltasTeResuelve`.
- Fuente inicial de metas: configuración Power Query versionable y pública.
- Grano de metas: `Periodo + AliadoKey`.
- Alias aprobado: `ABAI → UNO 27`.
- Asignación aprobada: `202607 + UNO 27 → PUSHER 1`.
- Enero-junio y agosto de UNO 27: clasificación vigente, sin nuevas presunciones.
- Metas vacías: ausencia/`BLANK`, nunca cero.
- Numerador agregado: todas las altas reales del contexto, incluso aliados sin meta.
- Denominador: suma de metas asignadas del contexto.
- No fuzzy matching, many-to-many, filtros bidireccionales ni hardcode de metas en DAX.
- `Meta_Altas_Promedio_Mas_30_Pct` se conserva como auxiliar y deja de mostrarse.

## 3. Arquitectura objetivo

```text
Dim_Calendario ──1:*── Fact_AltasTeResuelve
       │                    ▲
       └──1:*── Fact_MetasComerciales

Dim_Aliado ─────1:*── Fact_AltasTeResuelve
       │                    ▲
       └──1:*── Fact_MetasComerciales

Dim_AsignacionPusherPeriodo ──1:*── Fact_AltasTeResuelve
                 └────────────1:*── Fact_MetasComerciales
```

`Dim_AsignacionPusherPeriodo` no tendrá relaciones con `Dim_Calendario` ni `Dim_Aliado`. De esta forma no se crean rutas ambiguas. Año/Mes/Aliado filtran directamente los hechos y PUSHER filtra ambos mediante `PeriodoAliadoKey`.

## 4. PC-1 — Fuente y homologación de metas

**Objetivo:** crear una fuente gobernada con metas canónicas de julio.

**Prerrequisitos:** baseline limpio; homologaciones y BLANK aprobados.

**Archivos previstos:** `expressions.tmdl`, `model.tmdl` si cambia `PBI_QueryOrder`.

**Acciones:**

- Crear `Config_MetasComerciales` con 14 filas `AnioMes`, `AliadoKey`, `MetaAltas`.
- Usar nombres canónicos exclusivamente.
- Crear `Map_AsignacionPusherPeriodo` con el override `202607 | UNO 27 | PUSHER 1`.
- Mantener ambas consultas con carga deshabilitada si solo alimentan tablas finales.
- Validar tipos enteros positivos y unicidad.

**Pruebas automáticas:** 14 metas; suma P1 2.959; suma P2 3.358; total 6.317; cero duplicados; cero claves nulas; cero aliados huérfanos.

**Validación manual:** revisar catálogo y confirmar que no existen datos personales.

**Riesgos:** error de transcripción o alias; doble meta para un aliado-periodo.

**Criterio de cierre:** configuración conciliada exactamente con la tabla aprobada.

**Rollback:** retirar únicamente las expresiones nuevas y restaurar `PBI_QueryOrder`.

**Commit sugerido:** `feat(data): agrega configuracion de metas comerciales`.

**Push:** sí, después del gate.

## 5. PC-2 — Modelo de metas y asignación temporal

**Objetivo:** crear las tablas y relaciones necesarias sin reexpresar enero-junio.

**Prerrequisitos:** PC-1 cerrado.

**Archivos previstos:** `expressions.tmdl`, `model.tmdl`, `relationships.tmdl`, `Fact_AltasTeResuelve.tmdl`, nuevas tablas TMDL.

**Acciones:**

- Crear `Fact_MetasComerciales` con `FechaPeriodo`, `AnioMes`, `AliadoKey`, `PeriodoAliadoKey`, `MetaAltas`.
- Crear `Dim_AsignacionPusherPeriodo` con clave única y clasificación base más overrides.
- Generar la dimensión desde la unión de combinaciones de altas y metas.
- Aplicar el override con precedencia sobre la clasificación base, reemplazando la fila de la misma `PeriodoAliadoKey` y evitando duplicados.
- Añadir `PeriodoAliadoKey` a `Fact_AltasTeResuelve` sin cambiar filas ni `Altas`.
- Crear cuatro relaciones activas, `1:*`, unidireccionales.
- No modificar `Map_PusherAliado`.

**Pruebas automáticas:**

- `Fact_AltasTeResuelve`: 26.496 filas y 33.854 altas.
- Julio 4.518; junio 3.700; agosto 559.
- Metas: 14 filas, total 6.317.
- Unicidad de asignación y metas.
- Cero huérfanos en las cuatro relaciones nuevas.
- UNO 27: `Sin asignar` enero-junio, PUSHER 1 julio, `Sin asignar` agosto.
- Cero relaciones many-to-many, bidireccionales o entre hechos.

**Validación manual:** modelo visual y refresh en Desktop.

**Riesgos:** rutas ambiguas, claves mal formadas o asignación incompleta.

**Criterio de cierre:** modelo estrella válido y cifras de altas sin variación.

**Rollback:** eliminar relaciones/tablas nuevas y retirar solo la clave añadida a la fact.

**Commit sugerido:** `feat(model): agrega metas y asignacion pusher por periodo`.

**Push:** sí, después del gate MCP/Desktop.

## 6. PC-3 — Medidas DAX

**Objetivo:** calcular meta y cumplimiento bajo todos los filtros.

**Prerrequisitos:** PC-2 cerrado y datos materializados.

**Archivos previstos:** `_Medidas_Altas.tmdl`.

**Acciones:**

- Crear `Meta_Asignada` con `SUM(Fact_MetasComerciales[MetaAltas])`.
- Crear `Cumplimiento_Meta_Pct` con `DIVIDE([Altas_Total], [Meta_Asignada], BLANK())`.
- Formato `0.00%`.
- Ajustar genéricamente las medidas PUSHER que hoy dependen de `Dim_Aliado[Pusher]` para usar `PusherPeriodo`, conservando nombres y comportamiento salvo la asignación temporal autorizada.
- No modificar la fórmula de `Meta_Altas_Promedio_Mas_30_Pct`.

**Pruebas automáticas:**

- P2 julio: 2.429 / 3.358 = 72,33 %.
- P1 julio: 1.581 / 2.959 = 53,43 %.
- General julio: 4.518 / 6.317 = 71,52 %.
- ATENTO 99,64 %; ONE CONTACT 88,36 %; GNP 31,52 %; UNO 27 99,56 %.
- Aliado sin meta, mes sin meta y meta cero: `BLANK`.
- Meta +30 % julio permanece 6.235.

**Validación manual:** ninguna cifra que pueda demostrarse por MCP/DAX; solo formato visual posterior.

**Riesgos:** contexto ambiguo o inconsistencia entre el slicer y medidas antiguas.

**Criterio de cierre:** matriz de pruebas con diferencias cero en cantidades y tolerancia de redondeo en porcentajes.

**Rollback:** restaurar `_Medidas_Altas.tmdl` y retirar las dos medidas nuevas.

**Commit sugerido:** `feat(dax): agrega cumplimiento de metas comerciales`.

**Push:** sí.

## 7. PC-4 — Sustitución visual mínima

**Objetivo:** mostrar `% Cumplimiento` sin rediseñar la página.

**Prerrequisitos:** PC-3 cerrado.

**Archivos previstos:** `gc_slicer_pusher/visual.json`, `gc_kpi_mes_anterior/visual.json`.

**Acciones:**

- Cambiar el binding del slicer PUSHER a `Dim_AsignacionPusherPeriodo[PusherPeriodo]`, conservando su etiqueta visible.
- Conservar la primera proyección del card: **Promedio histórico previo**.
- Sustituir solo la segunda proyección por `Cumplimiento_Meta_Pct` con etiqueta **% Cumplimiento**.
- Mantener posición, tamaño, estilos, bordes, espaciado y navegación.
- No eliminar la medida Meta +30 %.

**Pruebas automáticas:** JSON válido; bindings existentes; cero referencias rotas; layout sin cambios geométricos; privacidad PASS.

**Validación manual:** julio general, P2, P1, ATENTO, aliado sin meta; promedio visible; Meta +30 % no visible; sin truncamientos.

**Riesgos:** churn PBIR o cambio accidental de layout.

**Criterio de cierre:** gate visual aprobado.

**Rollback:** restaurar los dos JSON desde el commit anterior.

**Commit sugerido:** `feat(report): agrega porcentaje de cumplimiento comercial`.

**Push:** sí, tras gate manual.

## 8. PC-5 — Conciliación técnica

**Objetivo:** demostrar trazabilidad desde configuración hasta DAX.

**Prerrequisitos:** PC-4 técnicamente listo.

**Archivos previstos:** Output de evidencia; sin cambios de modelo.

**Acciones:** construir matrices por contexto general, PUSHER, aliado y periodo; comparar metas configuradas, altas oficiales y DAX.

**Pruebas:** cifras obligatorias de PC-3; metas BLANK; periodo sin meta; agosto; unicidad; huérfanos; suma de metas por PUSHER.

**Validación manual:** no requerida si MCP/DAX demuestra los resultados.

**Riesgos:** diferencia por filtro o asignación temporal.

**Criterio de cierre:** cero discrepancias no explicadas.

**Rollback:** no aplica; fase de solo lectura.

**Commit sugerido:** `test: concilia cumplimiento de metas comerciales`.

**Push:** sí.

## 9. PC-6 — Validación funcional y visual

**Objetivo:** validar Desktop, filtros, visual y regresión.

**Prerrequisitos:** PC-5 PASS.

**Archivos previstos:** solo churn necesario aprobado; Output de validación.

**Acciones:** abrir PBIP, refresh, probar filtros, revisar visuales existentes y clasificar el diff posterior a Desktop.

**Pruebas manuales mínimas:**

- Julio, Todos: cumplimiento 71,52 %.
- Julio, PUSHER 2: 72,33 %.
- Julio, PUSHER 1: 53,43 %.
- Julio, ATENTO: 99,64 %.
- Aliado sin meta: tarjeta en blanco.
- Agosto sin meta: tarjeta en blanco.
- Promedio histórico visible; Meta +30 % ausente visualmente; layout intacto.

**Riesgos:** serialización automática o diferencia entre PBIR y render real.

**Criterio de cierre:** gate del usuario aprobado y regresión cero.

**Rollback:** restauración selectiva de churn, nunca reset masivo.

**Commit sugerido:** solo si Desktop genera serialización funcional necesaria.

**Push:** según cambios reales.

## 10. PC-7 — Documentación

**Objetivo:** documentar uso, semántica y mantenimiento mensual.

**Prerrequisitos:** PC-6 aprobado.

**Archivos previstos:** `Docs/GESTION_COMERCIAL_ALTAS_TE_RESUELVE.md`, Specs/Outputs.

**Acciones:**

- Explicar `Altas / Meta asignada` y las interpretaciones de 100 %, menor y mayor a 100 %.
- Documentar general, PUSHER, aliado, periodo, mes parcial y BLANK.
- Registrar alias canónicos y asignación temporal de UNO 27.
- Documentar mantenimiento de nuevas metas y asignaciones por periodo.
- Registrar Meta +30 % como auxiliar heurística, no predictiva.
- Reiterar que las metas agregadas se publican públicamente.

**Pruebas:** consistencia de nombres, cifras y archivos; sin información personal.

**Criterio de cierre:** documentación funcional y técnica completa.

**Rollback:** revertir únicamente documentos de la fase.

**Commit sugerido:** `docs: documenta cumplimiento de metas comerciales`.

**Push:** sí.

## 11. PC-8 — Cierre

**Objetivo:** cerrar la iniciativa y preparar publicación manual.

**Prerrequisitos:** PC-1 a PC-7 cerrados.

**Archivos previstos:** Spec/Output final.

**Acciones:** validación estática final, SHAs, working tree limpio, documentación de publicación manual.

**Pruebas:** JSON/TMDL/DAX, referencias, relaciones, privacidad, Git, regresión y gate manual.

**Criterio de cierre:** todos los criterios de aceptación cumplidos; `main = origin/main`; staging y working tree vacíos.

**Rollback:** revert atómico del último commit si el cierre descubre un defecto bloqueante.

**Commit sugerido:** `docs: registra cierre cumplimiento de metas comerciales`.

**Push:** sí.

## 12. Gates manuales

- M-1: confirmar las 14 metas y el override temporal.
- M-2: validar refresh, tablas, relaciones y ausencia de huérfanos en Desktop.
- M-3: aprobar el render del card con `% Cumplimiento`.
- M-4: aprobar filtros General/PUSHER/Aliado y casos BLANK.
- M-5: republicar manualmente en Power BI Service/Publicar en la Web.
- M-6: validar el artefacto público en navegación privada y confirmar privacidad.

Codex ejecutará las validaciones estáticas, MCP, DAX, Git y documentación que no requieran interacción visual.

## 13. Criterios de aceptación globales

- Metas P1 2.959, P2 3.358 y general 6.317.
- P2 julio 72,33 % obligatorio.
- P1 julio usa 1.581 altas, sin eliminar Centro Mayor.
- UNO 27 es PUSHER 1 solo en julio; enero-junio no se reexpresa.
- Año/Mes/PUSHER/Aliado filtran numerador y denominador de forma coherente.
- Aliado o periodo sin meta devuelve `BLANK`.
- Meta +30 % permanece como medida auxiliar.
- `Altas_Total`, drivers, navegación y layout no presentan regresiones.
- Publicación pública sin datos personales.
- Commits atómicos, SHAs remotos confirmados y repositorio limpio.

## 14. Gate antes de PC-1

La implementación técnica debe detenerse si la arquitectura temporal descrita no es aceptada. No se autoriza sustituirla por una reclasificación estática de UNO 27 ni por excepciones hardcodeadas en DAX.
