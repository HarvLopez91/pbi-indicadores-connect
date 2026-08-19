# Análisis de impacto — Porcentaje de cumplimiento de meta comercial

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Página afectada | `GestionComercialAltas` |
| Baseline | `b2c8ab334577090e954d8e8e734f5c8494419345` |
| Fecha | 2026-08-18 |
| Estado | Análisis cerrado; implementación técnica no iniciada |

## 1. Resumen ejecutivo

La mejora reemplazará visualmente la segunda proyección del `cardVisual` `gc_kpi_mes_anterior`, hoy denominada **Meta +30 %**, por **% Cumplimiento**. Se conservarán la primera proyección (**Promedio histórico previo**), el tamaño, la posición, el diseño y el resto de la página.

La fórmula aprobada es:

```text
Cumplimiento = Altas reales del contexto / Meta asignada del contexto
```

La meta debe modelarse como dato y no mediante condiciones DAX por aliado. La fuente inicial será una configuración Power Query pequeña, versionable y sin datos personales, con grano `Periodo + AliadoKey`. La imagen aportada se usa únicamente como fuente de metas; `Fact_AltasTeResuelve` continúa siendo la fuente oficial de ventas.

El caso obligatorio queda conciliado: PUSHER 2, julio de 2026, registra 2.429 altas, meta 3.358 y cumplimiento 72,33 %.

La asignación `ABAI → UNO 27 → PUSHER 1` está autorizada solo para `202607`. No debe agregarse al mapa estático porque eso reclasificaría enero-junio sin autorización. Se requiere una asignación comercial por periodo.

## 2. Baseline técnico

El preflight confirmó:

- Rama `main`.
- `HEAD = origin/main = b2c8ab334577090e954d8e8e734f5c8494419345`.
- Working tree y staging vacíos.
- Un único worktree.
- Power BI Desktop y Excel cerrados.
- `git diff --check` PASS.
- Conexión Power BI Modeling MCP establecida en modo offline/solo lectura sobre la carpeta TMDL.

El MCP reportó 18 tablas, 66 medidas y 13 relaciones. Los objetos relevantes son:

- `Fact_AltasTeResuelve`: `FechaAlta`, `AliadoKey`, `Altas`; grano de fila agregada de `Insumo2`.
- `Dim_Aliado`: `AliadoKey`, `Descripcion`, `Pusher`, `Estado_Clasificacion`.
- `Dim_Calendario`: fecha, año, mes y atributos de periodo.
- `_Medidas_Altas`: 36 medidas, incluidas `Promedio_Altas_Hasta_Mes_Anterior` y `Meta_Altas_Promedio_Mas_30_Pct`.
- Relaciones unidireccionales `Dim_Calendario → Fact_AltasTeResuelve` y `Dim_Aliado → Fact_AltasTeResuelve`.

## 3. Visual actual

`gc_kpi_mes_anterior` es un único `cardVisual` con dos proyecciones:

1. `Promedio_Altas_Hasta_Mes_Anterior`, etiqueta **Promedio histórico previo**.
2. `Meta_Altas_Promedio_Mas_30_Pct`, etiqueta **Meta +30 %**.

El impacto visual previsto es mínimo: sustituir exclusivamente la segunda proyección por `Cumplimiento_Meta_Pct`, con etiqueta **% Cumplimiento**. No se rediseñará la página.

## 4. Fuente y transcripción aprobada

Periodo de la fuente: `202607`.

| Nombre mostrado | AliadoKey canónico | PUSHER mostrado | MetaAltas | Cierre mostrado |
|---|---|---|---:|---:|
| BRM | BRM | PUSHER 2 | 50 | 50 |
| PLAZA IMPERIAL | CAV BOGOTA PLAZA IMPERIAL | PUSHER 2 | BLANK | 5 |
| CAPITALS | CAPITALS TELECOM BPO | PUSHER 2 | 70 | 12 |
| ONE CONTACT | ONE CONTACT | PUSHER 2 | 636 | 562 |
| INTERACTIVO | INTERACTIVO | PUSHER 2 | 50 | 9 |
| ATENTO | ATENTO | PUSHER 2 | 1.400 | 1.395 |
| PLAZA AMERICAS | CAV BOGOTA PLAZA DE LAS AMERICAS | PUSHER 2 | BLANK | 0 |
| ALMA | ALMAEXPERIENCE SAS | PUSHER 2 | 70 | 55 |
| GNP | GNP | PUSHER 2 | 1.082 | 341 |
| MILLENIUM | MILLENIUM | PUSHER 1 | 100 | 59 |
| VECTOR | VECTOR | PUSHER 1 | 70 | 13 |
| INTELIGENCE BUSSINES REC… | INTELIGENCE BUSSINES RECOVERY COL | PUSHER 1 | 1.026 | 688 |
| ASISTE ING | ASISTE ING | PUSHER 1 | 70 | 8 |
| COS | COS | PUSHER 1 | 1.343 | 490 |
| CAV BOGOTA CENTRO MAYOR | CAV BOGOTA CENTRO MAYOR | PUSHER 1 | BLANK | BLANK |
| AIB | AIB | PUSHER 1 | 125 | 98 |
| ABAI | UNO 27 | PUSHER 1 | 225 | 224 |

Las homologaciones se aplican únicamente mediante el nombre canónico aprobado y coincidencia exacta. No se utilizará fuzzy matching.

Las metas vacías representan ausencia de meta y no cero. Por ello, la tabla física de metas puede contener solo las 14 filas con meta asignada; la ausencia de fila equivale a `BLANK`.

## 5. Conciliación del gate de datos

| Contexto julio 2026 | Altas reales | Meta | Cumplimiento |
|---|---:|---:|---:|
| PUSHER 2 | 2.429 | 3.358 | 72,33 % |
| PUSHER 1 con asignación temporal de UNO 27 | 1.581 | 2.959 | 53,43 % |
| General | 4.518 | 6.317 | 71,52 % |

La imagen muestra 1.580 cierres para PUSHER 1. La fuente oficial contiene además una alta de `CAV BOGOTA CENTRO MAYOR` el 03/07/2026, por lo que el numerador correcto es 1.581. Esa alta no se elimina.

El numerador agregado incluye las ventas reales de aliados sin meta. El denominador suma solo metas asignadas. A nivel individual, un aliado sin meta devuelve `Meta_Asignada = BLANK` y `Cumplimiento_Meta_Pct = BLANK`.

## 6. Identidad y asignación temporal de UNO 27

La fuente `Insumo2` no contiene `ABAI`; contiene `UNO 27`. La investigación determinó:

- `UNO 27` es el único aliado con 224 altas en julio.
- AliadoKey actual: `UNO 27`.
- PUSHER estático actual: `Sin asignar`.
- Enero-junio: no existe autorización para reclasificarlo.
- Julio 2026: asignación aprobada a PUSHER 1.
- Agosto: no se presume asignación; permanece con la clasificación vigente hasta nueva aprobación.

Por ello no debe modificarse `Map_PusherAliado` para añadir una regla estática.

## 7. Arquitectura evaluada

### 7.1 Alternativas descartadas

**Hardcode DAX por aliado.** Descartado porque mezcla datos con lógica de medida, no escala a meses futuros y viola el requerimiento.

**Actualizar `Map_PusherAliado` estático.** Descartado porque movería 1.225 altas de enero-julio de `Sin asignar` a PUSHER 1 y reexpresaría enero-junio sin autorización.

**Duplicar PUSHER únicamente en `Fact_MetasComerciales`.** Descartado porque el filtro PUSHER produciría denominadores y numeradores con universos distintos.

**Relaciones many-to-many o bidireccionales.** Descartadas por riesgo de ambigüedad.

### 7.2 Arquitectura recomendada

Crear una dimensión de asignación temporal con grano único `Periodo + AliadoKey`:

`Dim_AsignacionPusherPeriodo`

- `PeriodoAliadoKey`
- `AnioMes`
- `AliadoKey`
- `PusherPeriodo`
- `TipoAsignacion`

La dimensión se construirá con todas las combinaciones reales de periodo y aliado presentes en altas, unidas con las combinaciones presentes en metas. La clasificación base provendrá de `Dim_Aliado[Pusher]`; una tabla de overrides versionable contendrá exclusivamente las excepciones aprobadas, inicialmente:

| AnioMes | AliadoKey | PusherPeriodo | Motivo |
|---:|---|---|---|
| 202607 | UNO 27 | PUSHER 1 | Alias ABAI y asignación comercial aprobada para julio |

El override tendrá precedencia sobre la clasificación base y la sustituirá para la misma clave `PeriodoAliadoKey`; no se anexarán ambas filas. La validación exigirá una sola asignación por periodo y aliado.

`Fact_AltasTeResuelve` y `Fact_MetasComerciales` recibirán `PeriodoAliadoKey`. La dimensión temporal tendrá relaciones `1:*`, activas y unidireccionales hacia ambas tablas de hechos. No se relacionará con `Dim_Calendario` ni `Dim_Aliado`, evitando rutas ambiguas.

Los filtros operarán así:

- Año y Mes: `Dim_Calendario` filtra directamente ambos hechos.
- Aliado: `Dim_Aliado` filtra directamente ambos hechos.
- PUSHER: el segmentador usará `Dim_AsignacionPusherPeriodo[PusherPeriodo]` y filtrará ambos hechos por la clave de asignación.

Esta solución permite que UNO 27 sea PUSHER 1 solo en julio, mantenga `Sin asignar` entre enero-junio y no impone una regla para agosto.

## 8. Tabla de metas propuesta

`Fact_MetasComerciales` tendrá grano único `FechaPeriodo + AliadoKey` y columnas mínimas:

- `FechaPeriodo` (primer día del mes).
- `AnioMes`.
- `AliadoKey`.
- `PeriodoAliadoKey`.
- `MetaAltas`.

No contendrá PUSHER como atributo funcional, porque la asignación temporal se resolverá desde `Dim_AsignacionPusherPeriodo`.

La fuente inicial recomendada es una configuración Power Query versionable con los 14 registros canónicos que tienen meta. Ventajas:

- auditable en Git;
- sin Excel adicional ni ruta local;
- segura para publicación pública;
- extensible mediante nuevas filas mensuales;
- no contiene datos personales.

Riesgo: el mantenimiento requiere editar deliberadamente la configuración por cada nuevo periodo. Debe documentarse un control de unicidad `AnioMes + AliadoKey`.

## 9. Medidas

```DAX
Meta_Asignada =
SUM(Fact_MetasComerciales[MetaAltas])

Cumplimiento_Meta_Pct =
DIVIDE([Altas_Total], [Meta_Asignada], BLANK())
```

`DIVIDE` garantiza `BLANK` cuando la meta no existe o es cero. La medida se formateará como `0.00%` y no se multiplicará por 100.

Las medidas PUSHER existentes deberán evaluarse contra la asignación temporal para mantener coherencia entre el segmentador, el numerador y las metas. El cambio debe ser genérico por `PusherPeriodo`; no debe contener una condición específica para UNO 27.

## 10. Comportamiento esperado

- General: altas reales totales / metas asignadas totales.
- PUSHER: altas reales de aliados asignados al PUSHER en el periodo / metas de esos aliados.
- Aliado: altas reales del aliado / su meta del periodo.
- Periodo: usa exclusivamente las metas del mes seleccionado.
- Aliado sin meta: `BLANK` individual.
- Mes sin metas: `BLANK`.
- Mes parcial con meta: avance acumulado contra meta mensual; no se bloquea por `Es_Periodo_Comparable`.
- Múltiples meses: suma de altas / suma de metas de los periodos seleccionados.

## 11. Preservación de Meta +30 %

`Promedio_Altas_Hasta_Mes_Anterior` y `Meta_Altas_Promedio_Mas_30_Pct` se conservarán. La segunda dejará de mostrarse en la página, pero permanecerá como referencia histórica heurística para una futura iniciativa. No se describe como modelo predictivo.

## 12. Impacto por componente

### Power Query

- Configuración canónica de metas.
- Configuración de overrides de asignación por periodo.
- Construcción de `Dim_AsignacionPusherPeriodo`.
- Construcción de `Fact_MetasComerciales`.
- Adición de `PeriodoAliadoKey` a `Fact_AltasTeResuelve`.

### Modelo semántico

- Dos tablas nuevas.
- Una columna técnica nueva en la fact de altas.
- Cuatro relaciones nuevas: calendario y aliado hacia metas; asignación temporal hacia altas y metas.
- Sin cambios en el mapa PUSHER estático.

### DAX

- `Meta_Asignada`.
- `Cumplimiento_Meta_Pct`.
- Ajuste genérico de medidas por PUSHER para utilizar la asignación temporal, sujeto a regresión completa.

### PBIR

- Cambio del campo del segmentador PUSHER a `PusherPeriodo`.
- Sustitución exclusiva de la segunda proyección de `gc_kpi_mes_anterior`.
- Sin cambios de layout.

## 13. Riesgos y regresiones potenciales

- Una asignación temporal incompleta podría dejar ventas fuera de un PUSHER.
- Duplicados `Periodo + AliadoKey` producirían doble meta o una relación inválida.
- Usar simultáneamente el PUSHER estático y el temporal generaría cifras inconsistentes.
- Los periodos futuros requieren metas y overrides explícitos; no deben reutilizar julio.
- El KPI general incluye altas de aliados sin meta por la semántica aprobada.
- Power BI Desktop puede introducir churn TMDL/PBIR que debe separarse antes de commit.
- La tabla de metas será pública y expone metas agregadas por aliado; no contiene información personal.

## 14. Estrategia de pruebas

- Unicidad de `AnioMes + AliadoKey` en metas y asignación.
- Cero huérfanos de metas y altas frente a la asignación temporal.
- Cero huérfanos frente a calendario y aliado.
- Julio PUSHER 2: 2.429 / 3.358 = 72,33 %.
- Julio PUSHER 1: 1.581 / 2.959 = 53,43 %.
- Julio general: 4.518 / 6.317 = 71,52 %.
- ATENTO: 1.395 / 1.400 = 99,64 %.
- ONE CONTACT: 562 / 636 = 88,36 %.
- GNP: 341 / 1.082 = 31,52 %.
- UNO 27: 224 / 225 = 99,56 %.
- Aliado sin meta y mes sin meta: `BLANK`.
- Agosto sin meta configurada: `BLANK`.
- Regresión de altas, cambio, variación, promedio, meta auxiliar, drivers y navegación.

## 15. Privacidad y publicación

Las metas son información agregada de negocio y se incorporarán conscientemente al modelo distribuido mediante **Publicar en la Web**. No se incluirán personas, cargos sensibles, correos, comentarios, rutas ni identificadores personales. La imagen no se publicará ni se incorporará como recurso del reporte.

## 16. Archivos probables

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_AltasTeResuelve.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_MetasComerciales.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_AsignacionPusherPeriodo.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas_Altas.tmdl`
- `PBI/PBI_Indicadores.Report/definition/pages/GestionComercialAltas/visuals/gc_slicer_pusher/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/GestionComercialAltas/visuals/gc_kpi_mes_anterior/visual.json`
- `Docs/GESTION_COMERCIAL_ALTAS_TE_RESUELVE.md`
- Specs y Outputs de esta iniciativa.

## 17. Conclusión

La mejora es viable sin reclasificar enero-junio de UNO 27. La condición de diseño es usar una asignación comercial temporal y migrar el filtro PUSHER de la página a esa dimensión. La implementación del modelo solo puede comenzar después de aprobar el plan técnico y confirmar que esta arquitectura temporal es aceptada.
