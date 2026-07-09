# Corrección - Fase 13: textos del Home en español de Colombia

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Corrección complementaria de Fase 13 - Home landing page |
| Fecha | 2026-07-08 |
| Objetivo | Corregir textos visibles del Home para mostrar español de Colombia con tildes y caracteres especiales. |

## Estado inicial de `git status`

Al iniciar la corrección, el repositorio no estaba limpio:

- [AGENTS.md](../AGENTS.md) estaba sin seguimiento.
- Archivos PBIR del Home aparecían modificados, principalmente por cambios mecánicos de fin de archivo generados al abrir/guardar en Power BI Desktop.
- `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl` aparecía modificado con metadatos lingüísticos generados por Power BI Desktop.

[AGENTS.md](../AGENTS.md) y `cultures/es-ES.tmdl` no se incluyen en este commit.

## Archivos revisados

Se revisaron textos visibles y metadatos del reporte en:

- `PBI/PBI_Indicadores.Report/definition/pages/`
- `PBI/PBI_Indicadores.Report/definition/report.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/*/visual.json`

También se consultaron:

- [Outputs/21_cierre_fase_13_home_implementado.md](21_cierre_fase_13_home_implementado.md)
- [AGENTS.md](../AGENTS.md)
- [Specs/01_analisis_de_impacto_informe_powerbi_connect.md](../Specs/01_analisis_de_impacto_informe_powerbi_connect.md)
- [Specs/02_plan_implementacion_informe_powerbi_connect.md](../Specs/02_plan_implementacion_informe_powerbi_connect.md)

## Textos corregidos

| Archivo visual | Antes | Después |
|---|---|---|
| `home_header_subtitle` | `Seguimiento de calidad, capacitaci?n y motivaci?n en call centers asociados a Claro` | `Seguimiento de calidad, capacitación y motivación en call centers asociados a Claro` |
| `home_pilot_note` | `Datos piloto sujetos a validaci?n` | `Datos piloto sujetos a validación` |
| `home_nav_03_label` | `Satisfacci?n de capacitaciones` | `Satisfacción de capacitaciones` |
| `home_nav_04_label` | `Motivaci?n comercial` | `Motivación comercial` |
| `home_nav_06_label` | `Notas metodol?gicas` | `Notas metodológicas` |
| `home_kpi_total_capacitacion` | `Total Respuestas Capacitacion` | `Total Respuestas Capacitación` |
| `home_kpi_total_motivacion` | `Total Respuestas Motivacion` | `Total Respuestas Motivación` |
| `home_kpi_indice_capacitacion` | `Indice Global Capacitacion` | `Índice Global Capacitación` |
| `home_kpi_indice_motivacion` | `Indice Global Motivacion` | `Índice Global Motivación` |

En las tarjetas KPI solo se corrigieron `displayName` y la etiqueta visible `label.text`. No se modificaron `Measure.Property`, `queryRef`, `nativeQueryRef`, nombres de tablas, medidas ni carpetas.

## Codificación UTF-8

Los `visual.json` corregidos se escribieron y validaron como UTF-8. La lectura posterior con Node.js mostró correctamente los caracteres `á`, `ó` e `Í` en los textos visibles.

## Validaciones JSON/PBIR

Validaciones realizadas:

- Parseo JSON correcto de `report.json` y de todos los JSON bajo `definition/pages/`.
- Schema PBIR oficial `visualContainer/2.9.0` válido para los 35 `visual.json` del Home.
- Schema oficial `report/3.0.0` válido para `report.json`.
- Búsqueda sin resultados para patrones corruptos: `capacitaci?n`, `motivaci?n`, `Satisfacci?n`, `metodol?gicas`, `validaci?n`.
- Extracción programática de textos visibles confirmando tildes correctas en subtítulo, nota, navegación y KPIs.

No se abrió visualmente Power BI Desktop en esta corrección; la validación fue estructural PBIR/JSON.

## Modelo semántico

No se modificaron nombres técnicos, medidas DAX, Power Query, relaciones ni tablas del modelo.

Nota de control: `cultures/es-ES.tmdl` ya estaba modificado al inicio por metadatos lingüísticos generados por Power BI Desktop. Ese cambio no se tocó y no se incluye en este commit.

## Archivos modificados por esta corrección

- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_header_subtitle/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_pilot_note/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_03_label/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_04_label/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_nav_06_label/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_kpi_total_capacitacion/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_kpi_total_motivacion/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_kpi_indice_capacitacion/visual.json`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_kpi_indice_motivacion/visual.json`
- `Outputs/22_correccion_fase_13_textos_espanol_colombia.md`

## Resultado del commit

Mensaje de commit:

`fix(report): corregir textos del home en español`

El commit incluye únicamente la corrección de textos visibles del Home y este documento. No incluye [AGENTS.md](../AGENTS.md), `Data/*.xlsx` ni cambios del modelo semántico.

## Estado final de `git status`

Estado esperado tras el commit:

- [AGENTS.md](../AGENTS.md) sigue sin seguimiento.
- Persisten cambios no incluidos generados previamente por Power BI Desktop en archivos PBIR sin contenido funcional nuevo y en `cultures/es-ES.tmdl`.

## Recomendación para avanzar a Fase 14

Antes de avanzar a Fase 14, abrir el PBIP en Power BI Desktop y confirmar visualmente que el Home muestra correctamente:

- `capacitación`
- `motivación`
- `Satisfacción`
- `metodológicas`
- `validación`
- `Índice`

No se avanzó a Fase 14 en esta ejecución.
