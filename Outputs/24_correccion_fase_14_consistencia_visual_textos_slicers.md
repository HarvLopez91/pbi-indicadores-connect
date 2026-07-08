# Corrección - Fase 14: consistencia visual, textos y segmentadores

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Corrección transversal posterior a Fase 14 |
| Fecha | 2026-07-08 |
| Resultado | Se ajustaron textos visibles, segmentadores, tarjetas KPI, gráficos y tablas de páginas internas. |

## Estado inicial de `git status`

Al iniciar la corrección existían cambios pendientes generados por Power BI Desktop:

- Archivos PBIR de páginas internas con cambios automáticos en `visual.json` relacionados con `$schema`, `active: true` y orden de propiedades.
- `PBI/PBI_Indicadores.Report/definition/pages/pages.json` con cambio automático de `activePageName`.
- `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl` con metadatos lingüísticos automáticos de `PowerBI.VisualColumnRename`.
- `AGENTS.md` sin seguimiento.

`AGENTS.md`, `pages.json` y `cultures/es-ES.tmdl` no se incluyen en el commit de esta corrección.

## Páginas revisadas

Se revisaron las 6 páginas internas de Fase 14:

- `Resumen ejecutivo`
- `Calidad de llamadas`
- `Satisfacción de capacitaciones`
- `Motivación comercial`
- `Detalle por call center`
- `Notas metodológicas`

## Textos corregidos

Se corrigieron textos visibles corruptos o sin tildes en títulos de visuales, etiquetas y `displayName` visible, sin modificar `queryRef`, `nativeQueryRef`, nombres de medidas, tablas ni columnas.

Correcciones principales:

- Títulos de índice por call center y jornada quedaron como `Índice por call center` e `Índice por jornada`.
- Títulos de motivación por call center y jornada quedaron como `Motivación por call center` y `Motivación por jornada`.
- Etiquetas de capacitación quedaron como `Claridad Promedio Capacitación`, `Dinamismo Promedio Capacitación` y `Utilidad Promedio Capacitación`.
- Encabezados de conteo quedaron como `Capacitación n` y `Motivación n`.
- Etiquetas de líder quedaron como `Líder` y `Formador y líder`.

## Segmentadores ajustados

Se configuraron los 16 segmentadores de páginas internas con:

- `visual.objects.data.mode = 'Dropdown'`
- Altura compacta de `38`
- Fuente de ítems en `#002733`
- Fondo blanco y borde suave
- Etiquetas superiores compactas en gris oscuro

Segmentadores cubiertos:

- `Dim_Calendario[Fecha]`
- `Dim_CallCenter[CallCenter]`
- `Dim_Jornada[Jornada]` donde aplica

Ruta manual de verificación en Power BI Desktop: Formato > Configuración de la segmentación > Opciones > Estilo > Menú desplegable.

## Ajustes en `Notas metodológicas`

Los segmentadores de `Notas metodológicas` estaban ubicados en `y=150`, sobre la misma franja de las tarjetas KPI. Se movieron al encabezado derecho para dejarlos visibles y accesibles:

- `Fecha`: `x=730`, `y=60`, `width=158`, `height=38`, `z=71`
- `CallCenter`: `x=904`, `y=60`, `width=158`, `height=38`, `z=73`

También se redujo el ancho del subtítulo para evitar cruces con la zona de filtros. La validación geométrica confirmó `0` solapamientos entre segmentadores y tarjetas KPI.

## Ajustes de tarjetas KPI

En las páginas internas se ajustaron las tarjetas `cardVisual`:

- Valor principal: `26D`, color `#002733`, negrita.
- Etiqueta: `10D`, color `#3A3A3A`.
- Barra/acento superior: `#F15B2B`.
- Márgenes internos uniformes en `0L` para mejorar proporción y evitar recortes.

No se modificaron las medidas DAX ni los campos técnicos usados por los KPI.

## Ajustes de colores en gráficos y tablas

Se agregaron objetos de formato a gráficos de barras, columnas y líneas:

- Color principal de barras/series: `#F15B2B`.
- Ejes en gris oscuro `#3A3A3A`.
- Grillas en `#F4F4F4`.
- Leyenda desactivada cuando no aporta lectura.
- Líneas con grosor `3D`.

En tablas `tableEx` se aplicó:

- Encabezado oscuro `#002733`.
- Texto de encabezado blanco.
- Filas con texto `#002733`.
- Líneas horizontales suaves `#F4F4F4`.

No se encontraron referencias a azul genérico de Power BI en las páginas internas después del ajuste.

## Validaciones JSON/PBIR realizadas

Validaciones completadas:

- `201` JSON bajo `definition/pages/` parsean correctamente.
- `16/16` segmentadores internos tienen `mode='Dropdown'`.
- No quedan patrones corruptos conocidos en páginas internas ni en este documento.
- No hay solapamientos entre segmentadores y KPI en `Notas metodológicas`.
- No se encontraron colores azules genéricos `#118DFF`, `#12239E` o `#0078d4` en páginas internas.

Validación PBIR contra schema: se intentó usar `ajv-cli` con schemas oficiales descargados a carpeta temporal. El intento no fue concluyente porque los schemas oficiales resuelven referencias anidadas externas (`visualConfiguration`, `semanticQuery`, entre otras). No se agregaron dependencias ni archivos de validación al repositorio.

## Confirmación de no modificación del modelo semántico

La corrección no modificó:

- Power Query (`expressions.tmdl`)
- Medidas DAX
- Relaciones (`relationships.tmdl`)
- Tablas del modelo (`tables/*.tmdl`)
- Archivos `Data/*.xlsx`
- `Assets/`

Nota de control: `cultures/es-ES.tmdl` ya estaba modificado al inicio por Power BI Desktop y queda fuera del commit.

## Archivos modificados

El commit incluye:

- `PBI/PBI_Indicadores.Report/definition/pages/p14_*/visuals/*/visual.json`
- `Outputs/24_correccion_fase_14_consistencia_visual_textos_slicers.md`

No incluye:

- `AGENTS.md`
- `Data/*.xlsx`
- `PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl`
- `PBI/PBI_Indicadores.Report/definition/pages/pages.json`

## Resultado del commit

Mensaje de commit:

`fix(report): ajustar consistencia visual y textos de paginas internas`

El commit queda enfocado en visuales PBIR de páginas internas y este documento.

## Estado final de `git status`

Estado esperado tras el commit:

```text
 M PBI/PBI_Indicadores.Report/definition/pages/pages.json
 M PBI/PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl
?? AGENTS.md
```

Los cambios anteriores son pendientes preexistentes o automáticos de Power BI Desktop y no forman parte de esta corrección.

## Recomendación para avanzar a Fase 15

Antes de Fase 15, abrir el PBIP en Power BI Desktop y validar visualmente:

1. Que los segmentadores aparecen como menú desplegable.
2. Que `Notas metodológicas` muestra filtros visibles en el encabezado.
3. Que las tarjetas KPI no recortan etiquetas.
4. Que los gráficos usan la paleta Connect.
5. Que no aparecen textos con caracteres corruptos.

Después de esa validación, avanzar a Fase 15 para configurar navegación funcional.
