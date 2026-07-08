# Cierre - Fase 13: Home landing page implementado

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 13 - Home landing page |
| Fecha | 2026-07-08 |
| Resultado | Home implementado y persistido en PBIR con visuales versionables. |

## Estado inicial de `git status`

Estado observado al iniciar la implementacion:

```text
?? AGENTS.md
```

`AGENTS.md` seguia sin seguimiento y no corresponde a esta fase. Se mantiene fuera del commit.

## Archivos modificados

Archivos y carpetas creados/modificados para el Home:

- `PBI/PBI_Indicadores.Report/definition/report.json`
- `PBI/PBI_Indicadores.Report/StaticResources/RegisteredResources/logo_connect_naranja_20260708.png`
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/`
- `Outputs/21_cierre_fase_13_home_implementado.md`

No se modificaron:

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/*.tmdl`
- `Data/*.xlsx`
- `Assets/`

## Visuales implementados

Se crearon 35 visuales PBIR bajo:

`PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/`

Resumen por tipo:

- `cardVisual`: 6 visuales KPI.
- `image`: 1 visual para el logo Connect.
- `shape`: 17 visuales de fondo, acentos, contenedores y tarjetas de navegacion.
- `textbox`: 11 visuales de titulo, subtitulo, notas y etiquetas.

Elementos persistidos:

- Header con logo Connect.
- Titulo `Dashboard Comercial y Formativo`.
- Subtitulo `Seguimiento de calidad, capacitación y motivación en call centers asociados a Claro`.
- Nota `Datos piloto sujetos a validación`.
- 6 tarjetas KPI.
- 6 tarjetas de navegacion visual.
- Nota metodologica inferior.

## Medidas usadas en KPIs

| KPI | Medida | Tabla de medidas |
|---|---|---|
| `Total Registros Piloto` | `[Total Registros Piloto]` | `_Medidas Generales` |
| `Total Evaluaciones Calidad` | `[Total Evaluaciones Calidad]` | `_Medidas Generales` |
| `Total Respuestas Capacitacion` | `[Total Respuestas Capacitacion]` | `_Medidas Generales` |
| `Total Respuestas Motivacion` | `[Total Respuestas Motivacion]` | `_Medidas Generales` |
| `Indice Global Capacitacion` | `[Indice Global Capacitacion]` | `_Medidas Capacitacion` |
| `Indice Global Motivacion` | `[Indice Global Motivacion]` | `_Medidas Motivacion` |

## Recursos usados desde `Assets`

Logo fuente:

`Assets/logos/6973ca8b4e3df02ed6efdaa7_logo_connect_naranja.png`

Para que el logo quede persistido en PBIR, se copio como recurso registrado:

`PBI/PBI_Indicadores.Report/StaticResources/RegisteredResources/logo_connect_naranja_20260708.png`

Y se agrego al paquete `RegisteredResources` en `PBI/PBI_Indicadores.Report/definition/report.json`.

## Validaciones realizadas

- Se confirmo que existen cambios versionables en archivos del reporte.
- Se confirmo que el Home ya no queda vacio: existen 35 carpetas de visuales con `visual.json`.
- Se valido parseo JSON de `page.json`, `report.json` y todos los `visual.json`.
- Se valido schema oficial `visualContainer/2.9.0` en los 35 `visual.json`.
- Se valido schema oficial `report/3.0.0` en `report.json`.
- Se confirmo que las 6 medidas usadas por los KPIs existen en el modelo.
- Se confirmo que el recurso del logo esta copiado y registrado.
- Se inicio `PBI/PBI_Indicadores.pbip` con Power BI Desktop; el proceso arranco y permanecio en ejecucion sin error inmediato de carga.
- Se cerro Power BI Desktop al finalizar la validacion de arranque.
- Se confirmo que no quedaron procesos `PBIDesktop` abiertos.
- Se confirmo que no hubo diffs en modelo semantico ni `Data/`.

## Errores encontrados y solucion aplicada

Hallazgo: el reporte no tenia paquete `RegisteredResources`, por lo que un logo local no podia quedar referenciado de forma versionable.

Solucion: se creo `StaticResources/RegisteredResources/`, se copio el PNG del logo y se agrego la entrada correspondiente en `report.json`:

```text
name: logo_connect_naranja_20260708.png
type: Image
```

Intento de validacion: `ajv-cli` no resolvio correctamente la URL remota del schema en Windows. Se resolvio usando un runner temporal Node + `ajv` instalado en `%TEMP%`, sin agregar dependencias al repositorio.

Nota: la validacion de apertura fue de arranque del proceso Power BI Desktop; no se tomo captura visual desde este entorno.

## Resultado del commit

Mensaje de commit:

`feat(report): implementar home landing page connect`

Archivos incluidos: solo cambios relacionados con el Home y este documento. No se incluye `AGENTS.md`.

## Estado final de `git status`

Estado esperado tras el commit:

```text
?? AGENTS.md
```

## Recomendacion para avanzar a Fase 14

La Fase 13 ya tiene Home persistido en PBIR. Antes de iniciar Fase 14, se recomienda abrir visualmente el PBIP en Power BI Desktop y confirmar que:

1. El logo renderiza correctamente.
2. Los seis KPIs muestran valores.
3. Los textos no se recortan.
4. Las tarjetas de navegacion se ven como elementos preparados, sin navegacion funcional todavia.

Despues de esa confirmacion visual, el proyecto puede avanzar a Fase 14. No se avanzo a Fase 14 en esta ejecucion.
