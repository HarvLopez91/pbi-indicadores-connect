# Catálogo de medidas DAX — `PBI_Indicadores`

Catálogo completo de las **30 medidas** del modelo semántico, leídas directamente de los archivos `.tmdl` en `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas *.tmdl`. Este documento es de solo lectura respecto al modelo: no se creó ni modificó ninguna medida para producirlo.

El uso por página/visual se obtuvo inspeccionando los `visual.json` del reporte (`PBI/PBI_Indicadores.Report/definition/pages/`); si una medida no aparece enlazada a ningún visual, se indica explícitamente.

## `_Medidas Generales`

### `Total Evaluaciones Calidad`
```dax
Total Evaluaciones Calidad = COUNTROWS(Fact_CalidadLlamadas)
```
- **Qué calcula:** número de llamadas evaluadas (conteo base de calidad).
- **Formato:** entero (`0`).
- **Páginas:** Home, Resumen ejecutivo, Calidad de llamadas, Detalle por call center, Notas metodológicas.
- **Visuales:** `home_kpi_total_calidad`, `re_kpi_calidad`, `cl_kpi_total`, `dc_kpi_calidad`, `nm_kpi_calidad`; también como valor del gráfico `cl_chart_callcenter`.
- **Observaciones:** es el `n` de referencia de todo indicador de calidad — debe leerse junto a cualquier promedio/porcentaje de esta familia.

### `Total Respuestas Capacitacion`
```dax
Total Respuestas Capacitacion = COUNTROWS(Fact_SatisfaccionCapacitacion)
```
- **Qué calcula:** número de respuestas a la encuesta de satisfacción de capacitación.
- **Formato:** entero (`0`).
- **Páginas:** Home, Resumen ejecutivo, Satisfacción de capacitaciones (original y v2 - borrador), Detalle por call center, Notas metodológicas.
- **Visuales:** `home_kpi_total_capacitacion`, `re_kpi_cap`, `sc_kpi_total`, `dc_kpi_cap`, `nm_kpi_cap`; también columna de la tabla `sc_tabla_formador` (original). En la página `v2`: `sc_kpi_respuestas`, `sc_kpi_respuestas_panel` (tarjeta "Respuestas" del panel de satisfacción), y columna "Respuestas" de `sc_tabla_callcenter`.

### `Total Respuestas Motivacion`
```dax
Total Respuestas Motivacion = COUNTROWS(Fact_MotivacionActividad)
```
- **Qué calcula:** número de respuestas a la encuesta de motivación de actividades comerciales.
- **Formato:** entero (`0`).
- **Páginas:** Home, Resumen ejecutivo, Motivación comercial, Detalle por call center, Notas metodológicas.
- **Visuales:** `home_kpi_total_motivacion`, `re_kpi_mot`, `mc_kpi_total`, `dc_kpi_mot`, `nm_kpi_mot`; también columna de la tabla `mc_tabla_ambiente`.

### `Total Registros Piloto`
```dax
Total Registros Piloto = [Total Evaluaciones Calidad] + [Total Respuestas Capacitacion] + [Total Respuestas Motivacion]
```
- **Qué calcula:** volumen total de registros del piloto, sumando las 3 fuentes.
- **Formato:** entero (`0`).
- **Páginas:** Home, Resumen ejecutivo, Detalle por call center, Notas metodológicas.
- **Visuales:** `home_kpi_total_registros`, `re_kpi_total`, `dc_kpi_total`, `nm_kpi_total`; valor de los gráficos `re_chart_callcenter`, `re_chart_fecha`, `dc_chart_registros`.
- **Observaciones:** suma registros de grano distinto (llamada, respuesta de encuesta, respuesta de encuesta) — es un indicador de volumen operativo del piloto, no un total analítico homogéneo.

### `n Calidad`
```dax
n Calidad = "n=" & [Total Evaluaciones Calidad]
```
- **Qué calcula:** texto dinámico `"n=<conteo>"` para mostrar el tamaño de muestra de calidad sin escribirlo a mano.
- **Formato:** texto (sin `formatString`, es una cadena).
- **Páginas:** Notas metodológicas.
- **Visuales:** `nm_n_calidad`.
- **Observaciones:** agregada en la Fase 16; antes de esa fase existía en el modelo pero no estaba enlazada a ningún visual.

### `n Capacitacion`
```dax
n Capacitacion = "n=" & [Total Respuestas Capacitacion]
```
- **Qué calcula:** texto dinámico `"n=<conteo>"` de capacitación.
- **Formato:** texto.
- **Páginas:** Notas metodológicas.
- **Visuales:** `nm_n_cap`.

### `n Motivacion`
```dax
n Motivacion = "n=" & [Total Respuestas Motivacion]
```
- **Qué calcula:** texto dinámico `"n=<conteo>"` de motivación.
- **Formato:** texto.
- **Páginas:** Notas metodológicas.
- **Visuales:** `nm_n_mot`.

## `_Medidas Calidad`

### `Puntaje Obtenido Calidad`
```dax
Puntaje Obtenido Calidad =
    SUM(Fact_CalidadLlamadas[Preg_TonoSaludo]) +
    SUM(Fact_CalidadLlamadas[Preg_FraseImpacto]) +
    SUM(Fact_CalidadLlamadas[Preg_PreguntasNecesidad]) +
    SUM(Fact_CalidadLlamadas[Preg_ConexionBeneficio]) +
    SUM(Fact_CalidadLlamadas[Preg_ExplicacionProducto]) +
    SUM(Fact_CalidadLlamadas[Preg_ManejoObjeciones]) +
    SUM(Fact_CalidadLlamadas[Preg_CierreComercial]) +
    SUM(Fact_CalidadLlamadas[Preg_ConfirmacionCierre])
```
- **Qué calcula:** suma de los puntajes obtenidos en las 8 preguntas del checklist de calidad (los `"N/A"` ya están como `null` y `SUM` los ignora).
- **Formato:** decimal (`0.0`).
- **Páginas:** Calidad de llamadas.
- **Visuales:** `cl_kpi_puntaje`.

### `Preguntas Aplicables Calidad`
```dax
Preguntas Aplicables Calidad =
    COUNT(Fact_CalidadLlamadas[Preg_TonoSaludo]) +
    COUNT(Fact_CalidadLlamadas[Preg_FraseImpacto]) +
    COUNT(Fact_CalidadLlamadas[Preg_PreguntasNecesidad]) +
    COUNT(Fact_CalidadLlamadas[Preg_ConexionBeneficio]) +
    COUNT(Fact_CalidadLlamadas[Preg_ExplicacionProducto]) +
    COUNT(Fact_CalidadLlamadas[Preg_ManejoObjeciones]) +
    COUNT(Fact_CalidadLlamadas[Preg_CierreComercial]) +
    COUNT(Fact_CalidadLlamadas[Preg_ConfirmacionCierre])
```
- **Qué calcula:** número de respuestas de checklist realmente aplicables (excluye los `"N/A"` convertidos a `null`, que `COUNT` no contabiliza).
- **Formato:** entero (`0`).
- **Páginas:** Calidad de llamadas.
- **Visuales:** `cl_kpi_aplicables`.

### `Promedio Puntaje Calidad`
```dax
Promedio Puntaje Calidad = DIVIDE([Puntaje Obtenido Calidad], [Preguntas Aplicables Calidad])
```
- **Qué calcula:** puntaje promedio por pregunta aplicable (0 a un máximo que varía por pregunta — ver limitación abajo).
- **Formato:** decimal (`0.0`).
- **Páginas:** Calidad de llamadas.
- **Visuales:** `cl_kpi_promedio`; columna de la tabla `cl_tabla_asesor`.
- **Observaciones:** no equivale a un porcentaje de calidad, porque cada pregunta del checklist tiene un puntaje máximo distinto y esa rúbrica aún no está confirmada por negocio (ver `% Calidad Promedio Provisional` abajo).

### `% Llamadas con Venta`
```dax
% Llamadas con Venta = DIVIDE(CALCULATE(COUNTROWS(Fact_CalidadLlamadas), Fact_CalidadLlamadas[TerminoEnVenta] IN {"Sí", "Si"}), COUNTROWS(Fact_CalidadLlamadas))
```
- **Qué calcula:** proporción de llamadas evaluadas que terminaron en venta.
- **Formato:** porcentaje (`0.0%`).
- **Páginas:** Calidad de llamadas.
- **Visuales:** `cl_kpi_venta`; columna de la tabla `cl_tabla_asesor`.
- **Observación pendiente:** en el piloto puede mostrarse en blanco en vez de `0,0%` en ciertos contextos de filtro. No se ajustó en esta fase (pendiente documentado desde la Fase 11/`Outputs/15`). Fórmula alternativa propuesta y no aplicada: usar `SUMX` sobre una columna booleana en vez de `CALCULATE` + `COUNTROWS` con `IN`.

### `Objecion Principal`
```dax
Objecion Principal =
    VAR ResumenObjeciones =
        FILTER(
            ADDCOLUMNS(
                VALUES(Fact_CalidadLlamadas[ObjecionPrincipal]),
                "Conteo", CALCULATE(COUNTROWS(Fact_CalidadLlamadas))
            ),
            NOT ISBLANK(Fact_CalidadLlamadas[ObjecionPrincipal]) && Fact_CalidadLlamadas[ObjecionPrincipal] <> ""
        )
    RETURN
        IF(
            ISEMPTY(ResumenObjeciones),
            BLANK(),
            MAXX(TOPN(1, ResumenObjeciones, [Conteo], DESC), Fact_CalidadLlamadas[ObjecionPrincipal])
        )
```
- **Qué calcula:** la objeción de cliente más frecuente entre las llamadas evaluadas en el contexto de filtro actual.
- **Formato:** texto (sin `formatString`).
- **Páginas:** Calidad de llamadas.
- **Visuales:** `cl_kpi_objecion`.
- **Observaciones:** con el volumen piloto actual, el "top 1" puede cambiar drásticamente con cada nueva respuesta — leer junto al `n` de calidad.

### `% Calidad Promedio Provisional`
```dax
% Calidad Promedio Provisional = BLANK()
```
- **Qué calcula:** nada todavía — es un marcador de posición explícito que siempre retorna en blanco.
- **Formato:** porcentaje (`0.0%`), preconfigurado para cuando se implemente.
- **Páginas:** no está enlazada a ningún visual del reporte. Se documenta como pendiente en los paneles de texto `cl_nota_calidad_text` (Calidad de llamadas) y `nm_calidad_text` (Notas metodológicas).
- **Observaciones:** requiere la rúbrica oficial de puntaje máximo por pregunta (dependencia D3, ver [05_decisiones_limitaciones_pendientes.md](05_decisiones_limitaciones_pendientes.md)) antes de poder implementarse como `DIVIDE([Puntaje Obtenido Calidad], <puntaje máximo aplicable por rúbrica>)`.

## `_Medidas Capacitacion`

### `Satisfaccion Promedio Capacitacion`
```dax
Satisfaccion Promedio Capacitacion = AVERAGE(Fact_SatisfaccionCapacitacion[SatisfaccionGeneral])
```
- **Qué calcula:** promedio Likert (1–5) de satisfacción general con la capacitación.
- **Formato:** decimal (`0.0`).
- **Páginas:** Satisfacción de capacitaciones (original y v2 - borrador).
- **Visuales:** `sc_kpi_satisf` (original). En la página `v2`: `sc_kpi_satisfaccion`, columna "Satisfacción" de `sc_tabla_callcenter`; también es dependencia directa de `Valor Metrica Satisfaccion` (ver más abajo), que la usa en `sc_panel_satisf_chart`.

### `Claridad Promedio Capacitacion`
```dax
Claridad Promedio Capacitacion = AVERAGE(Fact_SatisfaccionCapacitacion[Claridad])
```
- **Qué calcula:** promedio Likert de claridad de la capacitación.
- **Formato:** decimal (`0.0`).
- **Páginas:** Satisfacción de capacitaciones (original y v2 - borrador).
- **Visuales:** `sc_kpi_claridad` (original). En la página `v2`: columna "Claridad" de `sc_tabla_callcenter`; también es dependencia directa de `Valor Metrica Satisfaccion`, que la usa en `sc_panel_satisf_chart`.

### `Utilidad Promedio Capacitacion`
```dax
Utilidad Promedio Capacitacion = AVERAGE(Fact_SatisfaccionCapacitacion[Utilidad])
```
- **Qué calcula:** promedio Likert de utilidad percibida.
- **Formato:** decimal (`0.0`).
- **Páginas:** Satisfacción de capacitaciones (original y v2 - borrador).
- **Visuales:** `sc_kpi_utilidad` (original). En la página `v2`: columna "Utilidad" de `sc_tabla_callcenter`; también es dependencia directa de `Valor Metrica Satisfaccion`, que la usa en `sc_panel_satisf_chart`.

### `Dinamismo Promedio Capacitacion`
```dax
Dinamismo Promedio Capacitacion = AVERAGE(Fact_SatisfaccionCapacitacion[Dinamismo])
```
- **Qué calcula:** promedio Likert de dinamismo de la capacitación.
- **Formato:** decimal (`0.0`).
- **Páginas:** Satisfacción de capacitaciones (original y v2 - borrador).
- **Visuales:** `sc_kpi_dinamismo` (original). En la página `v2`: columna "Dinamismo" de `sc_tabla_callcenter`; también es dependencia directa de `Valor Metrica Satisfaccion`, que la usa en `sc_panel_satisf_chart`.

### `Indice Global Capacitacion`
```dax
Indice Global Capacitacion = DIVIDE([Satisfaccion Promedio Capacitacion] + [Claridad Promedio Capacitacion] + [Utilidad Promedio Capacitacion] + [Dinamismo Promedio Capacitacion], 4)
```
- **Qué calcula:** índice compuesto (promedio de los 4 promedios Likert) para lectura ejecutiva de un solo vistazo.
- **Formato:** decimal (`0.0`).
- **Páginas:** Home, Resumen ejecutivo, Satisfacción de capacitaciones, Detalle por call center.
- **Visuales:** `home_kpi_indice_capacitacion`, `re_kpi_ic`, `sc_kpi_indice`, `dc_kpi_ic`; valor de los gráficos `sc_chart_callcenter`, `sc_chart_jornada`; columna de la tabla `sc_tabla_formador`.

### `% Comentarios Capacitacion`
```dax
% Comentarios Capacitacion = DIVIDE(CALCULATE(COUNTROWS(Fact_SatisfaccionCapacitacion), Fact_SatisfaccionCapacitacion[Comentario] <> "Sin comentario"), COUNTROWS(Fact_SatisfaccionCapacitacion))
```
- **Qué calcula:** proporción de respuestas que dejaron un comentario real (distinto de `"Sin comentario"`).
- **Formato:** porcentaje (`0.0%`).
- **Páginas:** Satisfacción de capacitaciones (original y v2 - borrador).
- **Visuales:** `sc_kpi_coment` (original); `sc_kpi_comentarios` (v2 - borrador).

### `Call Centers Capacitados`
```dax
Call Centers Capacitados = DISTINCTCOUNT(Fact_SatisfaccionCapacitacion[CallCenter])
```
- **Qué calcula:** número de call centers distintos con al menos una capacitación registrada.
- **Formato:** entero (`0`).
- **Páginas:** Satisfacción de capacitaciones (v2 - borrador).
- **Visuales:** `sc_kpi_callcenters`.
- **Origen:** agregada en `SC-3` (iniciativa de rediseño de la página, ver [Outputs/39](../Outputs/39_resultado_sc3_medidas_satisfaccion_capacitaciones.md)).

### `Ultima Capacitacion`
```dax
Ultima Capacitacion = MAX(Fact_SatisfaccionCapacitacion[Fecha])
```
- **Qué calcula:** la fecha más reciente con una capacitación registrada, a partir de `Fact_SatisfaccionCapacitacion[Fecha]` (la fecha ya normalizada por Power Query, ver `TimestampNormalizado` en [04_fuentes_y_actualizacion_datos.md](04_fuentes_y_actualizacion_datos.md)).
- **Formato:** fecha (`dd/MM/yyyy`).
- **Páginas:** no está enlazada directamente a ningún visual — es la base de `Ultima Capacitacion Texto` (ver abajo).
- **Origen:** `SC-3`.
- **Observaciones:** el `formatString` de fecha es sensible a la configuración regional de Power BI Desktop/Windows; para presentación en tarjetas y tablas se usa la versión de texto (`Ultima Capacitacion Texto`), no esta medida directamente.

### `Capacitaciones Realizadas`
```dax
Capacitaciones Realizadas =
    COUNTROWS(
        SUMMARIZE(
            Fact_SatisfaccionCapacitacion,
            Fact_SatisfaccionCapacitacion[Fecha],
            Fact_SatisfaccionCapacitacion[CallCenter],
            Fact_SatisfaccionCapacitacion[NombreFormador]
        )
    )
```
- **Qué calcula:** número de sesiones de capacitación distintas, usando como clave provisional la combinación `Fecha + CallCenter + NombreFormador` (cada combinación distinta cuenta como una sesión).
- **Formato:** entero (`0`).
- **Páginas:** Satisfacción de capacitaciones (v2 - borrador).
- **Visuales:** `sc_kpi_capacitaciones`, `sc_chart_callcenter`, `sc_chart_capxfecha`, `sc_tabla_callcenter`.
- **Origen:** `SC-3`.
- **Limitación importante:** `Fecha + CallCenter + NombreFormador` **no es un identificador oficial de sesión de capacitación** — es una clave provisional. Si existen variantes de escritura del mismo formador (p. ej. con o sin tilde, o con apellido abreviado) sin alias unificado, o si dos sesiones distintas del mismo formador ocurren el mismo día en el mismo call center, la medida puede **sobreestimar o subestimar** el número real de sesiones. No debe presentarse como conteo definitivo de capacitaciones hasta que el origen entregue un identificador de sesión explícito (ver `D9` en [05_decisiones_limitaciones_pendientes.md](05_decisiones_limitaciones_pendientes.md)).

### `Valor Metrica Satisfaccion`
```dax
Valor Metrica Satisfaccion =
    SWITCH(
        SELECTEDVALUE(Dim_MetricaSatisfaccion[Metrica]),
        "Satisfacción", [Satisfaccion Promedio Capacitacion],
        "Claridad", [Claridad Promedio Capacitacion],
        "Utilidad", [Utilidad Promedio Capacitacion],
        "Dinamismo", [Dinamismo Promedio Capacitacion],
        BLANK()
    )
```
- **Qué calcula:** selecciona dinámicamente uno de los 4 promedios Likert de capacitación (`Satisfaccion`/`Claridad`/`Utilidad`/`Dinamismo` Promedio Capacitacion) según el valor de la fila de la tabla desconectada `Dim_MetricaSatisfaccion[Metrica]`.
- **Formato:** decimal (`0.0`).
- **Páginas:** Satisfacción de capacitaciones (v2 - borrador).
- **Visuales:** `sc_panel_satisf_chart` (panel de satisfacción de 4 barras).
- **Origen:** `SC-5` (Revisión 7).
- **Dependencias:** requiere la tabla desconectada `Dim_MetricaSatisfaccion` (ver "Objetos de soporte técnico" abajo) — sin relación con ninguna otra tabla del modelo. Si `Dim_MetricaSatisfaccion` se elimina o pierde sus 4 filas, la medida retorna `BLANK()` para todas las categorías.

### `Ultima Capacitacion Texto`
```dax
Ultima Capacitacion Texto =
    VAR Fecha = [Ultima Capacitacion]
    RETURN
        IF(
            ISBLANK(Fecha),
            BLANK(),
            FORMAT(DAY(Fecha), "00") & "/" & FORMAT(MONTH(Fecha), "00") & "/" & FORMAT(YEAR(Fecha), "0000")
        )
```
- **Qué calcula:** el mismo valor que `Ultima Capacitacion`, pero como texto armado manualmente con `DAY`/`MONTH`/`YEAR` en formato `dd/MM/yyyy`, para evitar que el formato de fecha se muestre invertido (`MM/dd/yyyy`) según la configuración regional de Power BI Desktop/Windows.
- **Formato:** texto (sin `formatString`, es una cadena).
- **Páginas:** Satisfacción de capacitaciones (v2 - borrador).
- **Visuales:** `sc_kpi_ultima`, columna "Última fecha" de `sc_tabla_callcenter`.
- **Origen:** `SC-5` (Revisión 7–8, iteró desde un `FORMAT(..., "es-CO")` inicial hasta la construcción manual actual, ver [Outputs/43](../Outputs/43_resultado_sc5_rediseno_visual_satisfaccion_capacitaciones.md)).
- **Limitación:** es una **medida de presentación**. Al retornar texto, no puede usarse para ordenar cronológicamente ni en cálculos de fecha (restas, `DATEDIFF`, comparaciones) — para eso debe usarse `Ultima Capacitacion` (la medida de fecha real).

### Objetos de soporte técnico (no son medidas)

Tres objetos del modelo usados por la página `p14_satisfaccion_capacitaciones_v2` (ver [03_mapa_reporte_paginas_visuales.md](03_mapa_reporte_paginas_visuales.md) §8) no son medidas DAX y no se cuentan en el total de 30:

- **`Dim_Calendario[Fecha Eje]`** — columna calculada **textual** (`dataType: string`), construida con `FORMAT(DAY(Dim_Calendario[Fecha]), "00") & "/" & FORMAT(MONTH(Dim_Calendario[Fecha]), "00")` (formato `dd/MM`), ordenada por `Dim_Calendario[Fecha]` (`sortByColumn: Fecha`). Usada como categoría del gráfico `sc_chart_capxfecha` para evitar que el eje muestre horas o la jerarquía automática de fechas (Auto Date/Time) que Power BI asocia por defecto a `Fecha`. No es una columna de tipo fecha ni una medida — es una etiqueta de eje.
- **`Fact_SatisfaccionCapacitacion[Fecha Texto]`** — columna calculada con la misma lógica de texto que `Ultima Capacitacion Texto` (`DAY`/`MONTH`/`YEAR` armado manualmente), a nivel de fila. Usada como columna de presentación en `sc_tabla_comentarios`.
- **`Dim_MetricaSatisfaccion`** — tabla calculada desconectada (`DATATABLE("Metrica", STRING, "Orden", INTEGER, {...})` con 4 filas: Satisfacción=1, Claridad=2, Utilidad=3, Dinamismo=4). No tiene relación con ninguna otra tabla del modelo; su columna `Metrica` se ordena por `Orden` (`sortByColumn`). Existe solo para que `Valor Metrica Satisfaccion` pueda seleccionar dinámicamente entre los 4 promedios Likert de capacitación en un único visual de barras.

## `_Medidas Motivacion`

### `Satisfaccion Promedio Actividad`
```dax
Satisfaccion Promedio Actividad = AVERAGE(Fact_MotivacionActividad[SatisfaccionGeneral])
```
- **Qué calcula:** promedio Likert de satisfacción con la actividad comercial.
- **Formato:** decimal (`0.0`).
- **Páginas:** Motivación comercial.
- **Visuales:** `mc_kpi_satisf`.

### `Claridad Utilidad Promedio Actividad`
```dax
Claridad Utilidad Promedio Actividad = AVERAGE(Fact_MotivacionActividad[ClaridadUtilidad])
```
- **Qué calcula:** promedio Likert de claridad/utilidad de la actividad.
- **Formato:** decimal (`0.0`).
- **Páginas:** Motivación comercial.
- **Visuales:** `mc_kpi_clarutil`.

### `Motivacion Promedio Actividad`
```dax
Motivacion Promedio Actividad = AVERAGE(Fact_MotivacionActividad[MotivacionPostActividad])
```
- **Qué calcula:** promedio Likert de motivación posterior a la actividad.
- **Formato:** decimal (`0.0`).
- **Páginas:** Motivación comercial.
- **Visuales:** `mc_kpi_motiv`.

### `Indice Global Motivacion`
```dax
Indice Global Motivacion = DIVIDE([Satisfaccion Promedio Actividad] + [Claridad Utilidad Promedio Actividad] + [Motivacion Promedio Actividad], 3)
```
- **Qué calcula:** índice compuesto (promedio de los 3 promedios Likert de motivación) para lectura ejecutiva.
- **Formato:** decimal (`0.0`).
- **Páginas:** Home, Resumen ejecutivo, Motivación comercial, Detalle por call center.
- **Visuales:** `home_kpi_indice_motivacion`, `re_kpi_im`, `mc_kpi_indice`, `dc_kpi_im`; valor de los gráficos `mc_chart_callcenter`, `mc_chart_jornada`.

### `% Ambiente Motivado`
```dax
% Ambiente Motivado = DIVIDE(CALCULATE(COUNTROWS(Fact_MotivacionActividad), Fact_MotivacionActividad[AmbienteEquipo] = "Motivado"), COUNTROWS(Fact_MotivacionActividad))
```
- **Qué calcula:** proporción de respuestas que describen el ambiente de equipo como `"Motivado"`.
- **Formato:** porcentaje (`0.0%`).
- **Páginas:** Motivación comercial.
- **Visuales:** `mc_kpi_ambiente`; columna de la tabla `mc_tabla_ambiente`.

### `% Comentarios Motivacion`
```dax
% Comentarios Motivacion = DIVIDE(CALCULATE(COUNTROWS(Fact_MotivacionActividad), Fact_MotivacionActividad[Comentario] <> "Sin comentario"), COUNTROWS(Fact_MotivacionActividad))
```
- **Qué calcula:** proporción de respuestas con comentario real.
- **Formato:** porcentaje (`0.0%`).
- **Páginas:** Motivación comercial.
- **Visuales:** `mc_kpi_coment`.

## Resumen por familia

| Tabla de medidas | Cantidad | Enlazadas directamente | Sin enlace directo |
|---|---:|---:|---:|
| `_Medidas Generales` | 7 | 7 | 0 |
| `_Medidas Calidad` | 6 | 5 | 1 |
| `_Medidas Capacitacion` | 11 | 10 | 1 |
| `_Medidas Motivacion` | 6 | 6 | 0 |
| **Total** | **30** | **28** | **2** |

Las 2 medidas sin enlace directo a ningún visual:

- **`% Calidad Promedio Provisional`** (`_Medidas Calidad`) — documentada únicamente en paneles de texto (`cl_nota_calidad_text`, `nm_calidad_text`); siempre retorna `BLANK()`.
- **`Ultima Capacitacion`** (`_Medidas Capacitacion`) — no está enlazada a ningún visual; es la base DAX de `Ultima Capacitacion Texto` (que sí está enlazada a `sc_kpi_ultima` y `sc_tabla_callcenter`).

> El conteo de 30 medidas corresponde al estado del modelo verificado durante `SC-8`. El número puede cambiar en el futuro si el negocio autoriza nuevas medidas — no lo tome como un valor fijo, verifique siempre contra los archivos `_Medidas *.tmdl` vigentes en `PBI/PBI_Indicadores.SemanticModel/definition/tables/`.
