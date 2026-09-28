# Resultado R3 — Asignación PUSHER

| Campo | Valor |
|---|---|
| Fase | R3 — Asignación PUSHER |
| Baseline | `5932a6ca0a5048344c299dfef9915c7b2c5e476b` |
| Fuente privada | `BI - NUEVO.xlsx`, hoja `Asignacion_PUSHER` |
| Fuente de ventas | Consolidado oficial de R2 |
| Estado | **PASS** |
| R4 | No iniciado |

## 1. Objetivo y alcance

Incorporar la asignación autorizada de PUSHER sin crear otra dimensión, sin fuzzy matching, sin usar `ESPECIALISTA` como llave y sin modificar metas, DAX, relaciones, PBIR ni páginas.

La página publicada `GestionComercialAltas` conserva su campo `PusherPeriodo` y su clasificación previa. La nueva clasificación queda disponible en `Dim_AsignacionPusherPeriodo[PusherAsignacion]` para las páginas nuevas de R7-R9. Esta separación evita una regresión en el gráfico histórico mientras se construyen y aprueban las nuevas páginas.

## 2. Implementación mínima

- `Map_PusherAliado` permanece como mapa legado para `GestionComercialAltas`.
- `Map_AsignacionPusherFuente` lee únicamente `Asignacion_PUSHER` desde la fuente privada.
- La equivalencia interna se transforma a las etiquetas públicas `PUSHER 1`, `PUSHER 2` y `PUSHER 3` mediante tres anclas operativas no personales: `COS`, `ONE CONTACT` y `CONTACTMASTER BPO`.
- Los textos se normalizan con `Trim`, `Clean` y mayúsculas.
- La prioridad aplicada es regla específica `DESCRIPCION + DESCRIPCION2`, luego regla general `DESCRIPCION`, y finalmente `Sin asignar`.
- Las reglas específicas amplían `AliadoKey` solo cuando es necesario. Esto permite distinguir operaciones específicas dentro de un mismo aliado sin alterar las relaciones 1:* existentes.
- El override temporal aprobado de julio para `UNO 27` se conserva.
- `Fecha_Inicio_Gestion_Pusher_2` se conserva en `01/07/2026` y se agrega únicamente `Fecha_Inicio_Gestion_Pusher_3 = 01/08/2026`.
- No se creó una dimensión PUSHER, una relación ni un hecho adicional.

### Clasificación histórica y gestión atribuible

La asignación de R3 clasifica el portafolio históricamente; no atribuye por sí sola resultados a la gestión. Para `PUSHER 2`, enero-junio de 2026 es línea base y la gestión comienza en julio. Para `PUSHER 3`, enero-julio es línea base y la gestión comienza en agosto. Los periodos previos permanecen bajo su clasificación de portafolio y no se convierten en `Sin asignar`.

R6 deberá reutilizar ambos parámetros para excluir los periodos de línea base de resultados, cumplimiento, crecimiento e impacto atribuibles. No se creó ni se modificó una fecha de inicio para `PUSHER 1` y no se implementaron medidas DAX en R3.

## 3. Calidad de las reglas

| Control | Resultado |
|---|---:|
| Filas de la fuente de asignación | 24 |
| Grupos de PUSHER | 3 |
| Reglas específicas | 2 |
| Reglas generales únicas | 21 |
| Duplicados exactos deduplicados | 1 |
| Colisiones específicas | 0 |
| Colisiones generales | 0 |
| Claves periodo-contexto evaluadas | 866 |
| Claves ambiguas después de ampliar `AliadoKey` | 0 |
| Pérdida o duplicación de ALTAS | 0 |

Sin ampliar la clave, dos reglas específicas producían once colisiones `Periodo + DESCRIPCION`. El cambio de clave resuelve la causa sin many-to-many ni rutas paralelas.

## 4. Conciliación de la nueva clasificación

| Periodo | PUSHER 1 | PUSHER 2 | PUSHER 3 | Sin asignar | Total | Cobertura anterior | Cobertura R3 |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 202601 | 2.226 | 3.053 | 194 | 823 | 6.296 | 85,63 % | 86,93 % |
| 202602 | 2.279 | 2.339 | 205 | 486 | 5.309 | 88,15 % | 90,85 % |
| 202603 | 1.925 | 2.568 | 193 | 424 | 5.110 | 89,51 % | 91,70 % |
| 202604 | 1.418 | 2.191 | 167 | 414 | 4.190 | 88,28 % | 90,12 % |
| 202605 | 1.490 | 1.933 | 233 | 515 | 4.171 | 84,85 % | 87,65 % |
| 202606 | 1.160 | 1.850 | 145 | 545 | 3.700 | 82,43 % | 85,27 % |
| 202607 | 1.523 | 2.427 | 2 | 567 | 4.519 | 88,76 % | 87,45 % |
| 202608 | 1.019 | 3.641 | 635 | 420 | 5.715 | 83,34 % | 92,65 % |
| 202609 | 1.432 | 3.187 | 658 | 351 | 5.628 | 82,11 % | 93,76 % |

Total conciliado: 35.073 filas y 44.638 ALTAS. La cobertura R3 acumulada es 40.093 / 44.638 = 89,82 %.

## 5. Comparación julio y agosto

| Periodo | Clasificación | PUSHER 1 | PUSHER 2 | PUSHER 3 | Sin asignar | Total |
|---:|---|---:|---:|---:|---:|---:|
| 202607 | Anterior | 1.582 | 2.429 | 0 | 508 | 4.519 |
| 202607 | R3 | 1.523 | 2.427 | 2 | 567 | 4.519 |
| 202607 | Diferencia | -59 | -2 | +2 | +59 | 0 |
| 202608 | Anterior | 1.115 | 3.648 | 0 | 952 | 5.715 |
| 202608 | R3 | 1.019 | 3.641 | 635 | 420 | 5.715 |
| 202608 | Diferencia | -96 | -7 | +635 | -532 | 0 |

Los cambios obedecen a la fuente autorizada, a nuevas operaciones y a las dos reglas específicas. La reducción de PUSHER 1 corresponde a operaciones que ya no tienen una regla exacta en la nueva fuente. No se fuerza la clasificación anterior. Los drivers agregados de julio mantienen los controles funcionales de R2 porque las ALTAS y el grano por aliado no cambiaron.

## 6. Sin asignar y homologaciones

Hay 153 descripciones sin coincidencia exacta. Principales volúmenes agregados:

| Descripción operativa | ALTAS | Periodos |
|---|---:|---|
| TRESUELVE HOGAR | 1.544 | 202601-202609 |
| MILLENIUM | 522 | 202601-202608 |
| TEAM COMUNICACIONES | 132 | 202601-202609 |
| INVERSIONES ARAUJO | 101 | 202601, 202602, 202604-202608 |
| CAV CUCUTA AV GRAN COLOMBIA | 99 | 202601-202609 |
| CAV PEREIRA ESTACION CENTRAL | 76 | 202601-202609 |
| CAV CUCUTA CENTRO | 69 | 202601-202609 |
| CAV BUCARAMANGA CABECERA | 67 | 202601-202609 |
| RHANDOM SAS | 60 | 202601-202607 |
| CAV YOPAL | 57 | 202601-202605, 202607-202609 |

No existe una correspondencia exacta respaldada por `Asignacion_PUSHER` para estas descripciones. No se aplicó ninguna homologación inferida. Cualquier equivalencia futura requiere aprobación funcional explícita.

## 7. Integridad, dependencias y privacidad

- Graphify señaló la cadena `PeriodoAliadoKey → Dim_AsignacionPusherPeriodo → medidas/visuales`; la dependencia fue confirmada contra TMDL, relaciones y PBIR.
- Las relaciones permanecen sin cambios y conservan cardinalidad 1:*.
- `PusherPeriodo` sigue siendo el consumidor de `GestionComercialAltas`; `PusherAsignacion` es aditiva y no tiene consumidores PBIR en R3.
- No se modificaron medidas DAX, metas, páginas, navegación ni interacciones.
- La fuente privada no se versiona.
- La consulta descarta el identificador nominal de origen y expone únicamente las tres etiquetas públicas.
- No se añadieron asesores, especialistas, jefes, nombres personales ni datos fila a fila al modelo público o a este Output.

## 8. Validaciones automáticas

- `SUM(ALTAS)` antes y después: 44.638.
- Filas antes y después: 35.073.
- Una clasificación R3 por venta/contexto: PASS.
- Tres etiquetas públicas: PASS.
- Colisiones: 0.
- Many-to-many nuevas: 0.
- Relaciones nuevas o ambiguas: 0.
- Diff en DAX, relaciones y PBIR: 0.
- Privacidad del diff: PASS.
- `git diff --check`: PASS.

## 9. Gate manual y revisión post-Desktop

El gate manual se completó con resultado PASS:

- Refresh en Power BI Desktop: PASS, sin errores.
- Consulta DAX por `AnioMes` y `PusherAsignacion`: PASS.
- Julio: PUSHER 1 = 1.523; PUSHER 2 = 2.427; PUSHER 3 = 2; Sin asignar = 567; total = 4.519.
- Agosto: PUSHER 1 = 1.019; PUSHER 2 = 3.641; PUSHER 3 = 635; Sin asignar = 420; total = 5.715.
- Septiembre: PUSHER 1 = 1.432; PUSHER 2 = 3.187; PUSHER 3 = 658; Sin asignar = 351; total = 5.628.
- Render y comportamiento de `GestionComercialAltas`: PASS, sin cambios funcionales.
- Revisión del round-trip de Desktop: PASS.

La consulta validada fue:

```DAX
EVALUATE
CALCULATETABLE (
    SUMMARIZECOLUMNS (
        Dim_AsignacionPusherPeriodo[AnioMes],
        Dim_AsignacionPusherPeriodo[PusherAsignacion],
        "ALTAS", [Altas_Total]
    ),
    TREATAS (
        { 202607, 202608, 202609 },
        Dim_AsignacionPusherPeriodo[AnioMes]
    )
)
ORDER BY
    Dim_AsignacionPusherPeriodo[AnioMes],
    Dim_AsignacionPusherPeriodo[PusherAsignacion]
```

Los resultados validan clasificación de portafolio, no atribución de gestión anterior a las fechas de inicio.

### Archivos finales de R3

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Aliado.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_AsignacionPusherPeriodo.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_AltasTeResuelve.tmdl`
- `Specs/12_analisis_impacto_mockups_gestion_comercial_septiembre.md`
- `Specs/13_plan_implementacion_mockups_gestion_comercial_septiembre.md`
- `Outputs/72_resultado_r3_asignacion_pusher_gestion_comercial_septiembre.md`

### Ruido de Desktop restaurado

Se restauraron al baseline la página activa, una serialización sin salto final de un KPI ajeno, la cultura Q&A y líneas finales añadidas a `Fact_MetasComerciales` y `_Medidas_Altas`. También se retiraron los archivos locales de DAX Query View. `model.tmdl` se conserva porque Desktop registró allí las expresiones nuevas de R3 en `PBI_QueryOrder`.

La separación entre baseline y gestión se mantiene: PUSHER 2 inicia gestión en `202607`; PUSHER 3 en `202608`; los periodos anteriores permanecen clasificados como línea base y R6 deberá excluirlos de resultados, cumplimiento, crecimiento e impacto atribuibles.

## 10. Rollback

Restaurar exclusivamente los archivos de R3 contra `5932a6ca0a5048344c299dfef9915c7b2c5e476b`. No revertir R2 ni el hotfix documental y no usar restauración masiva del working tree.
