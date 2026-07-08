# Resultado - Fase 14: diseño de páginas internas

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 14 - Diseño profesional de páginas internas |
| Fecha | 2026-07-08 |
| Resultado | Se crearon 6 páginas internas en PBIR con visuales versionables y estilo consistente con el Home. |

## Estado inicial de `git status`

Al iniciar la ejecución existían cambios pendientes no relacionados directamente con Fase 14:

- Reescrituras automáticas de Power BI Desktop en archivos PBIR del Home, principalmente fin de archivo.
- Metadatos lingüísticos automáticos en `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl`.
- `AGENTS.md` sin seguimiento.

`AGENTS.md` no se incluye en ningún commit de esta fase.

## Cambios automáticos sincronizados

Antes de implementar Fase 14 se sincronizaron cambios automáticos seguros de Power BI Desktop en un commit separado:

`8a34864 chore(powerbi): sincronizar metadatos automaticos de desktop`

Incluyó reescrituras PBIR del Home y metadatos lingüísticos generados por `PowerBI.VisualColumnRename`. No incluyó `Data/*.xlsx`.

## Páginas creadas

Se crearon estas páginas:

1. `Resumen ejecutivo` (`p14_resumen_ejecutivo`)
2. `Calidad de llamadas` (`p14_calidad_llamadas`)
3. `Satisfacción de capacitaciones` (`p14_satisfaccion_capacitaciones`)
4. `Motivación comercial` (`p14_motivacion_comercial`)
5. `Detalle por call center` (`p14_detalle_call_center`)
6. `Notas metodológicas` (`p14_notas_metodologicas`)

`PBI/PBI_Indicadores.Report/definition/pages/pages.json` queda con `Home` como `activePageName` y las 6 páginas internas agregadas al `pageOrder`.

## Visuales creados por página

| Página | Visuales PBIR | Composición |
|---|---:|---|
| `Resumen ejecutivo` | 23 | 6 KPI, 3 filtros, 1 columna, 1 línea, textos, formas y nota |
| `Calidad de llamadas` | 21 | 6 KPI, 2 filtros, 1 barra, 1 tabla, textos, formas y nota |
| `Satisfacción de capacitaciones` | 25 | 7 KPI, 3 filtros, 2 gráficos, 1 tabla, textos, formas y nota |
| `Motivación comercial` | 25 | 7 KPI, 3 filtros, 2 gráficos, 1 tabla, textos, formas y nota |
| `Detalle por call center` | 23 | 6 KPI, 3 filtros, 1 tabla comparativa, 1 barra, textos, formas y nota |
| `Notas metodológicas` | 41 | 4 KPI de conteo, paneles metodológicos, textos y formas |

Total del reporte después de la fase: 193 `visual.json` válidos, incluyendo Home.

## Medidas usadas por página

### Resumen ejecutivo

- `[Total Registros Piloto]`
- `[Total Evaluaciones Calidad]`
- `[Total Respuestas Capacitacion]`
- `[Total Respuestas Motivacion]`
- `[Indice Global Capacitacion]`
- `[Indice Global Motivacion]`

### Calidad de llamadas

- `[Total Evaluaciones Calidad]`
- `[Puntaje Obtenido Calidad]`
- `[Preguntas Aplicables Calidad]`
- `[Promedio Puntaje Calidad]`
- `[% Llamadas con Venta]`
- `[Objecion Principal]`

Se documenta en la nota de página que `[% Calidad Promedio Provisional]` queda en blanco hasta confirmar la rúbrica oficial, y que `[% Llamadas con Venta]` no se modifica en esta fase.

### Satisfacción de capacitaciones

- `[Total Respuestas Capacitacion]`
- `[Satisfaccion Promedio Capacitacion]`
- `[Claridad Promedio Capacitacion]`
- `[Utilidad Promedio Capacitacion]`
- `[Dinamismo Promedio Capacitacion]`
- `[Indice Global Capacitacion]`
- `[% Comentarios Capacitacion]`

### Motivación comercial

- `[Total Respuestas Motivacion]`
- `[Satisfaccion Promedio Actividad]`
- `[Claridad Utilidad Promedio Actividad]`
- `[Motivacion Promedio Actividad]`
- `[Indice Global Motivacion]`
- `[% Ambiente Motivado]`
- `[% Comentarios Motivacion]`

### Detalle por call center

- `[Total Registros Piloto]`
- `[Total Evaluaciones Calidad]`
- `[Total Respuestas Capacitacion]`
- `[Total Respuestas Motivacion]`
- `[Indice Global Capacitacion]`
- `[Indice Global Motivacion]`

### Notas metodológicas

- `[Total Registros Piloto]`
- `[Total Evaluaciones Calidad]`
- `[Total Respuestas Capacitacion]`
- `[Total Respuestas Motivacion]`

## Filtros incluidos

Se agregaron slicers PBIR versionables:

- `Dim_Calendario[Fecha]`
- `Dim_CallCenter[CallCenter]`
- `Dim_Jornada[Jornada]` en páginas donde aplica capacitación/motivación o comparativo operativo.

La página `Calidad de llamadas` no incluye `Jornada` porque la fuente de calidad no captura esa columna.

## Navegación

Se crearon botones visuales de retorno a Home en encabezados de páginas internas, pero sin acción funcional.

Decisión: no se configuró navegación funcional por PBIR en esta fase porque la Fase 15 está dedicada explícitamente a navegación y acciones de página. Esto evita introducir propiedades de acción no confirmadas y mantiene la Fase 14 centrada en diseño de páginas.

## Validaciones JSON/PBIR realizadas

- `report.json` y 201 JSON bajo `definition/pages/` parsean correctamente.
- 193 archivos `visual.json` validan contra schema oficial PBIR `visualContainer/2.9.0`.
- `pages.json` valida contra schema oficial `pagesMetadata/1.0.0`.
- No se encontraron patrones conocidos de texto corrupto por reemplazo de tildes o mojibake en los JSON del reporte.
- Se detectó un proceso Power BI Desktop abierto con ventana `PBI_Indicadores`; no se cerró ni manipuló desde esta ejecución.

## Confirmación de textos en español

Los textos visibles de las nuevas páginas usan español de Colombia con tildes y caracteres especiales, incluyendo:

- `Satisfacción`
- `Motivación`
- `Capacitación`
- `Notas metodológicas`
- `Índice`
- `validación`
- `rúbrica`
- `anónima`

## Confirmación de no modificación del modelo semántico

Durante la implementación de Fase 14 no se modificaron:

- Power Query (`expressions.tmdl`)
- Medidas DAX
- Relaciones (`relationships.tmdl`)
- Tablas del modelo (`tables/*.tmdl`)
- Archivos `Data/*.xlsx`
- `Assets/`

## Archivos modificados

- `PBI/PBI_Indicadores.Report/definition/pages/pages.json`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_resumen_ejecutivo/`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_calidad_llamadas/`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_motivacion_comercial/`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_detalle_call_center/`
- `PBI/PBI_Indicadores.Report/definition/pages/p14_notas_metodologicas/`
- `Outputs/23_resultado_fase_14_diseno_paginas_internas.md`

## Resultado del commit

Mensaje de commit:

`feat(report): crear paginas internas del informe connect`

El commit incluye solo páginas internas, `pages.json` y este documento. No incluye `AGENTS.md` ni `Data/*.xlsx`.

## Estado final de `git status`

Estado final validado tras el commit:

```text
?? AGENTS.md
```

## Recomendación para avanzar a Fase 15

Antes de Fase 15, validar visualmente en Power BI Desktop:

1. Que las 6 páginas aparecen en el orden esperado.
2. Que cada página renderiza sin visuales en blanco por error de campo.
3. Que los slicers cargan valores.
4. Que las etiquetas con tildes se muestran correctamente.
5. Que el diseño no recorta textos en tarjetas y paneles.

Luego avanzar a Fase 15 para configurar navegación funcional entre Home y páginas internas.
