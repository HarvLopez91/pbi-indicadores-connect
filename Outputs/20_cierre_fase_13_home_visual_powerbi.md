# Cierre complementario - Fase 13: validacion visual del Home

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Cierre complementario de Fase 13 - Home landing page |
| Fecha | 2026-07-08 |
| Objetivo | Validar si el Home construido manualmente en Power BI Desktop quedo persistido en el PBIP. |
| Resultado | Validacion no satisfactoria para visuales persistidos: no hay cambios versionables del Home despues del guardado reportado. |

## Estado inicial de `git status`

Estado observado al iniciar esta validacion:

```text
?? AGENTS.md
```

[AGENTS.md](../AGENTS.md) sigue sin seguimiento y no esta relacionado con esta fase. Se mantiene fuera del commit.

## Archivos modificados por Power BI Desktop

No se detectaron archivos versionables modificados por Power BI Desktop despues de la construccion manual reportada.

Verificaciones realizadas:

- `git diff --name-status`: sin cambios versionables.
- `git status --short --ignored`: solo muestra [AGENTS.md](../AGENTS.md) sin seguimiento y carpetas/archivos ignorados esperados (`Data/`, `.pbi/`, `cache.abf`, `localSettings.json`, `desktop.ini`).
- `git ls-files --others --exclude-standard PBI/PBI_Indicadores.Report Assets`: sin recursos nuevos versionables.
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/page.json` conserva solo metadatos de pagina.

## Confirmacion del Home construido visualmente

No se puede confirmar por evidencia de archivos que el Home visual haya quedado persistido.

La pagina existe como `Home`, segun:

`PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/page.json`

Pero el archivo contiene solo estas propiedades:

```text
$schema, name, displayName, displayOption, height, width
```

No contiene contenedores visuales, carpetas de visuales, imagenes, tarjetas KPI, textos, botones ni recursos registrados asociados al Home.

## Elementos encontrados en el Home

Revision contra los elementos esperados del wireframe:

| Elemento esperado | Estado en archivos PBIP |
|---|---|
| Logo Connect | No encontrado persistido |
| Titulo `Dashboard Comercial y Formativo` | No encontrado persistido |
| Subtitulo | No encontrado persistido |
| Nota `Datos piloto sujetos a validacion` | No encontrada persistida |
| KPIs principales | No encontrados persistidos |
| Tarjetas de navegacion | No encontradas persistidas |
| Nota metodologica inferior | No encontrada persistida |

Conclusion: aunque el usuario haya construido visualmente el Home en la sesion de Power BI Desktop, el guardado no produjo cambios versionables en el PBIP bajo `definition/pages/`, `report.json`, `StaticResources/` ni `Assets/`.

## Diffs revisados

Rutas revisadas:

- `PBI/PBI_Indicadores.Report/definition/pages/`
- `PBI/PBI_Indicadores.Report/definition/report.json`
- `PBI/PBI_Indicadores.Report/StaticResources/`
- `Assets/`

Resultado: sin diffs versionables.

## Validacion de no afectacion del modelo

Confirmado por ausencia de diffs:

- No se modifico Power Query (`expressions.tmdl`).
- No se modificaron medidas DAX (`tables/_Medidas *.tmdl`).
- No se modificaron relaciones (`relationships.tmdl`).
- No se modificaron tablas del modelo (`tables/*.tmdl`).
- No se modificaron archivos `Data/*.xlsx`.

## Confirmacion sobre apertura del PBIP

El PBIP deberia abrir sin errores porque no se detectaron cambios nuevos en la definicion versionable del reporte ni del modelo durante esta validacion.

Pendiente de confirmar en Power BI Desktop: que la vista visual del Home siga presente al cerrar y reabrir el proyecto. Si al reabrir aparece solo la pagina `Home` vacia, entonces el Home no quedo guardado en la definicion PBIP.

## Resultado del commit

Se documenta esta validacion como cierre complementario con resultado pendiente/no satisfactorio para persistencia visual.

Mensaje de commit utilizado:

`docs(report): registrar validacion pendiente home visual`

Archivos incluidos:

- `Outputs/20_cierre_fase_13_home_visual_powerbi.md`

No se incluye [AGENTS.md](../AGENTS.md), `Data/*.xlsx` ni cambios ajenos al cierre de Fase 13.

## Estado final de `git status`

Estado esperado tras el commit:

```text
?? AGENTS.md
```

## Recomendacion para avanzar a Fase 14

No avanzar todavia a Fase 14.

Antes de iniciar paginas internas:

1. Reabrir `PBI/PBI_Indicadores.pbip` en Power BI Desktop.
2. Confirmar visualmente si el Home construido sigue presente.
3. Si esta vacio, reconstruir el Home y guardar de nuevo.
4. Revisar inmediatamente `git status`; deben aparecer cambios en `PBI/PBI_Indicadores.Report/definition/pages/` y posiblemente en recursos estaticos.
5. Solo cuando existan diffs versionables con logo, textos, KPIs y tarjetas del Home, crear un commit de persistencia visual del Home.

La Fase 14 debe quedar bloqueada hasta que el Home visual este persistido y versionado.
