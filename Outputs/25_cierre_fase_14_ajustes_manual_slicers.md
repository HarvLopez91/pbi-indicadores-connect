# Cierre Fase 14 - Ajustes manuales de segmentadores

## Estado inicial de git

Al iniciar esta ejecucion, el repositorio tenia cambios pendientes en archivos PBIR del reporte, principalmente visuales de paginas internas y `PBI/PBI_Indicadores.Report/definition/pages/pages.json`.

Tambien se confirmaron dos sincronizaciones previas ya versionadas:

- `14a613f docs: agregar guia operativa del repositorio`
- `b09c5cb chore(powerbi): sincronizar metadatos automaticos de desktop`

[AGENTS.md](../AGENTS.md) ya esta versionado como guia operativa del repositorio y no quedo como archivo sin seguimiento.

## Cambios revisados de Power BI Desktop

Se revisaron cambios generados por Power BI Desktop en:

- `PBI/PBI_Indicadores.Report/definition/pages/pages.json`
- Visuales PBIR bajo `PBI/PBI_Indicadores.Report/definition/pages/p14_*`
- Cambio automatico menor en `Home` sobre `home_pilot_note`

No se detectaron cambios pendientes en `Data/*.xlsx`.

## Segmentadores ajustados

Se confirmaron 16 segmentadores en paginas internas con modo `Dropdown`:

- Fecha
- Call Center
- Jornada, donde aplica

Se normalizaron propiedades de fuente de segmentadores para mantener lectura consistente y compacta.

## Ajuste en Notas metodologicas

En la pagina `Notas metodológicas` se ajusto el filtro de fecha:

- `nm_slicer_fecha`: posicion `x=710`, `y=60`, ancho `170`, alto `38`
- `nm_slicer_fecha_label`: posicion `x=710`, `y=38`, ancho `170`, alto `16`
- `nm_slicer_callcenter`: posicion `x=900`, `y=60`, ancho `170`, alto `38`
- `nm_slicer_callcenter_label`: posicion `x=900`, `y=38`, ancho `170`, alto `16`

El subtitulo de la pagina se redujo a ancho `630` para evitar cruce visual con la zona de filtros.

## Tarjetas KPI

Se corrigieron etiquetas visibles de tarjetas KPI que Power BI Desktop habia dejado repetidas o inconsistentes.

Ajustes aplicados:

- Valor principal: `26D`
- Etiqueta: `10D`
- Numeros en `#002733`
- Etiquetas en `#3A3A3A`
- Acento superior en `#F15B2B`

No se modificaron `queryRef`, medidas, tablas, columnas ni nombres tecnicos del modelo.

## Colores de graficos

Se restituyeron colores consistentes con Connect Assistance:

- Barras y lineas principales: `#F15B2B`
- Texto de ejes: `#3A3A3A`
- Lineas de grilla: `#F4F4F4`
- Leyendas ocultas cuando no aportan lectura

## Validaciones realizadas

- JSON valido en 201 archivos bajo `PBI/PBI_Indicadores.Report/definition/pages`.
- 16 segmentadores revisados con modo `Dropdown`.
- No quedan patrones corruptos conocidos en textos visibles de paginas PBIR.
- `pages.json` queda con `activePageName` en `Home`.
- No se modificaron Power Query, medidas DAX, relaciones ni tablas del modelo.
- No se modificaron archivos `Data/*.xlsx`.

La validacion contra schema PBIR oficial se mantiene como estructural de alto nivel, porque el mecanismo `ajv` temporal documentado en fases previas no resuelve de forma concluyente todas las referencias anidadas del schema oficial. No se agregaron dependencias al repositorio.

## Archivos modificados

- Visuales PBIR de paginas internas `p14_*`.
- `PBI/PBI_Indicadores.Report/definition/pages/pages.json`.
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/visuals/home_pilot_note/visual.json`, por sincronizacion automatica menor de Desktop.
- `Outputs/25_cierre_fase_14_ajustes_manual_slicers.md`.

## Resultado del commit

Commit creado correctamente con el mensaje:

`fix(report): sincronizar segmentadores y ajustar filtro de fecha`

## Estado final de git

`git status` quedo limpio despues del commit.

## Recomendacion

Despues de este commit, avanzar a Fase 15 para configurar navegacion entre paginas sin modificar el modelo semantico.
