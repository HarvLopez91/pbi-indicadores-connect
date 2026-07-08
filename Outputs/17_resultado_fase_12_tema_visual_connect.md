# Resultado — Fase 12: Creación de tema visual Connect Assistance

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 12 — Creación de tema visual Connect Assistance (ver `Specs/02_plan_implementacion_informe_powerbi_connect.md`) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/16_cierre_fase_11_validacion_visual_powerbi.md` |
| Fecha | 2026-07-08 |
| Archivo nuevo | `Assets/theme/connect_assistance_theme.json` |

---

## Nota de proceso: el inventario de `Assets` cambió a mitad de la ejecución

Al iniciar esta fase, `Assets/logos/` solo tenía 1 archivo (un logo monocromático blanco) y `Assets/imagenes/` no existía. Se documentó el tema con la paleta placeholder sugerida sobre esa base. Antes de comitear, una revisión de `git status` mostró **6 archivos adicionales** que no estaban ahí al inicio (5 imágenes más un segundo logo) — muy probablemente sincronizados por OneDrive con algo de retraso mientras se trabajaba en esta fase. Se investigó el contenido nuevo antes de continuar (no se comiteó nada a ciegas), y **el segundo logo reveló colores de marca reales**, lo cual cambió una decisión de diseño importante (ver sección de paleta). Este documento refleja el inventario final completo, no el estado parcial inicial.

## Estado inicial de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado antes de iniciar.

## Confirmación de existencia de carpeta `Assets`

Ya existía, creada por el usuario, con `Assets/logos/` ya presente. Se crearon las 2 subcarpetas faltantes:

- `Assets/logos/` (ya existía)
- `Assets/imagenes/` (creada en esta fase)
- `Assets/theme/` (creada en esta fase)

## Recursos visuales encontrados en `Assets` (inventario completo)

### Logos

| Archivo | Ruta | Formato | Contenido |
|---|---|---|---|
| `64b580afe0544a5492bb389f_logo-completo.svg` | `Assets/logos/` | SVG, 202×48 | Variante **monocromática blanca** (todos los `path` en `fill="white"`) — pensada para fondos de color u oscuros |
| `64b580afe0544a5492bb38a3_logo-completo.svg` | `Assets/logos/` | SVG, 267×65 | Variante **a color**: el wordmark "Connect" en `fill="#F15B2B"` (naranja) y "Assistance" en `fill="#002733"` (verde azulado muy oscuro, no negro puro). **Esta es la fuente de la que se extrajo la paleta de marca real usada en el tema** (ver siguiente sección) |
| `6973ca8b4e3df02ed6efdaa7_logo_connect_naranja.png` | `Assets/logos/` | PNG, 460×79 | Apareció durante la ejecución (después de los 2 SVG). El nombre ("naranja") corrobora el hallazgo de color: no se extrajo el HEX por muestreo de píxeles (no hay librería de imágenes disponible en este entorno y el PNG rasterizado sería menos preciso que la declaración exacta ya obtenida del SVG) |

### Imágenes

| Archivo | Dimensiones | Peso | Observación |
|---|---|---|---|
| `64b580afe0544a5492bb382c_Cambio_de_llanta Image@3x.png` | 672×496 | 151 KB | Nombre sugiere ilustración de servicio "cambio de llanta" |
| `64b580afe0544a5492bb382d_Moto_Sr_Connect%403x-p-800.png` | 800×590 | 83 KB | Nombre sugiere mascota/ilustración "Sr. Connect" en moto |
| `64b580afe0544a5492bb38dc_Group 8950 (1).png` | 948×808 | 69 KB | Exportación tipo "Group" (patrón típico de Figma/diseño) |
| `64b580afe0544a5492bb38dd_Group 8949.png` | 1076×688 | 210 KB | Exportación tipo "Group" |
| `64b580afe0544a5492bb38f5_Group 8951.png` | 900×615 | 208 KB | Exportación tipo "Group" |
| `64b580afe0544a5492bb38f6_Group 8955.png` | 799×746 | 197 KB | Exportación tipo "Group" |

Todos los nombres de archivo comparten el prefijo `64b580afe0544a5492bb...`, consistente con una exportación masiva desde un mismo CMS/sitio web (patrón típico de Webflow). No se abrió el contenido visual de las 6 imágenes en esta fase (son ilustraciones/fotografías, no archivos de texto con información de color extraíble como los SVG) — quedan disponibles en `Assets/imagenes/` para uso en el futuro Home/páginas internas (Fase 13 en adelante).

## Tema JSON creado

`Assets/theme/connect_assistance_theme.json` — validado como JSON sintácticamente correcto (`json.load` sin errores). Sigue el esquema oficial de temas de reporte de Power BI documentado por Microsoft (`name`, `dataColors`, colores estructurales, `textClasses`, `visualStyles`).

## Paleta de colores usada

**Se reemplazó la paleta placeholder sugerida en la instrucción por colores reales extraídos del logo a color oficial**, siguiendo el criterio de la propia instrucción (#12): "si encuentras un logo... con colores diferentes, no inventes colores exactos" — al encontrarse colores reales embebidos en el logo (no inventados por mí), se priorizaron sobre la sugerencia genérica.

| Color | HEX usado | Origen | Uso principal en el tema |
|---|---|---|---|
| Naranja Connect | `#F15B2B` | **Extraído del logo a color** (`fill` del wordmark "Connect") | `tableAccent`, borde de tarjetas KPI, `dataColors[0]`, barras/columnas de gráficos, `maximum` |
| Verde azulado oscuro (texto principal) | `#002733` | **Extraído del logo a color** (`fill` del wordmark "Assistance") | `foreground`, `firstLevelElements`, títulos, números de tarjetas KPI, encabezados de tabla |
| Gris oscuro | `#3A3A3A` | Neutro genérico (sin confirmar en marca) | `secondLevelElements`, texto secundario |
| Gris medio | `#8A8A8A` | Neutro genérico (sin confirmar en marca) | `fourthLevelElements`, color "neutral" |
| Gris claro | `#F4F4F4` | Neutro genérico (sin confirmar en marca) | `secondaryBackground`, fondos secundarios, líneas de cuadrícula |
| Blanco | `#FFFFFF` | Estándar | `background`, fondo de página/tarjetas |

`dataColors` completo (recalculado con tintes/sombras derivados matemáticamente del naranja real, no del placeholder anterior): `#F15B2B, #002733, #8A8A8A, #F69475, #A9401E, #D9D9D9, #3A3A3A, #F9BDAA`.

**Nota importante — sigue siendo provisional:** encontrar estos dos colores en el logo es una señal mucho más fuerte que la paleta sugerida genérica, pero **no equivale a un manual de marca oficial confirmado**. No sabemos, por ejemplo, si `#002733` es realmente el color "de texto" oficial de la marca o solo el color de esa versión específica del logo, ni si existen tonos secundarios de marca (grises propios, un segundo color de acento) fuera de lo que aparece en este archivo puntual. La dependencia D6 (`Specs/02` §4, manual de marca / HEX oficiales) **sigue abierta**, pero esta fase deja el tema mucho mejor calibrado que con la paleta genérica original.

## Justificación visual

**Contraste medido con los colores reales de marca** (no con el placeholder original, ya descartado):

| Par | Contraste | Cumple AA texto normal (4.5:1) | Cumple AA texto grande (3:1) |
|---|---|---|---|
| Naranja marca `#F15B2B` sobre blanco | **3.35:1** | No | **Sí** (con margen) |
| Verde azulado oscuro `#002733` sobre blanco | **15.69:1** | Sí | Sí |
| Blanco sobre verde azulado oscuro | 15.69:1 | Sí | Sí |
| Naranja sobre verde azulado oscuro | 4.69:1 | Sí | Sí |

Dato relevante: el naranja **real** de marca (`#F15B2B`, 3.35:1) tiene mejor contraste que el placeholder original (`#F37021`, que medía 2.94:1 y no llegaba ni al umbral de texto grande) — un beneficio adicional, no solo estético, de haber encontrado el color real. Aun así, **se mantuvo la misma decisión conservadora** que con el placeholder: el naranja no se usa como color de texto de lectura general (`label`/`title`), reservándose para acentos (bordes, barras/columnas de datos, `tableAccent`) donde el contraste de texto no aplica. Los números grandes de las tarjetas KPI (`callout`) usan el verde azulado oscuro `#002733` (15.69:1, contraste excelente), priorizando la "lectura ejecutiva, alto contraste" pedida por la instrucción por encima de forzar el naranja incluso donde técnicamente ya pasaría el umbral de texto grande.

**Regla 60/30/10 aplicada:** blanco y grises/verde-azulado oscuro dominan como fondo y texto, el naranja aparece solo en puntos de énfasis. **Diseño limpio y ejecutivo:** bordes sutiles en gris claro con esquinas redondeadas (`radius: 4`), encabezados de tabla en el oscuro de marca con texto blanco.

## Configuraciones incluidas en el tema

| Elemento pedido | Cómo se resolvió |
|---|---|
| Fondo de página | `background: #FFFFFF` |
| Texto | `foreground: #002733` + `firstLevelElements`/`secondLevelElements` + `textClasses.label` |
| Títulos | `textClasses.title` (verde azulado oscuro, negrita, 14pt) |
| Tarjetas KPI | `visualStyles.card` y `visualStyles.cardVisual` (borde naranja de marca, número grande en `#002733` negrita, etiqueta de categoría en gris oscuro) — cubre tarjeta clásica y rediseñada |
| Segmentadores | `visualStyles.slicer` (encabezado gris claro con texto oscuro, ítems en blanco/oscuro) |
| Gráficos de barras/columnas | `visualStyles.barChart`, `columnChart`, `clusteredBarChart`, `clusteredColumnChart` (color de datos naranja de marca, ejes en gris oscuro, cuadrícula gris claro) |
| Tablas y matrices | `visualStyles.tableEx` y `pivotTable`: encabezado en `#002733` con texto blanco negrita, cuadrícula horizontal gris claro sin líneas verticales |
| Tooltips | Cubiertos automáticamente por los colores estructurales (`background`, `firstLevelElements`, `secondaryBackground`), sin necesitar una sección `visualStyles` dedicada, según la documentación oficial |
| Bordes y fondos de contenedores | Bloque `"*": {"*": {...}}` aplicado a todos los visuales: borde gris claro sutil, esquinas redondeadas, fondo blanco, "outspace" (lienzo de página) en blanco |

## Nivel de confianza de cada sección (transparencia)

- **Alta confianza**: estructura general de colores (`name`, `dataColors`, `background`, `foreground`, `tableAccent`, colores `firstLevelElements`–`fourthLevelElements`, `good`/`neutral`/`bad`, `maximum`/`center`/`minimum`) y `textClasses` — verificados contra ejemplos oficiales de Microsoft.
- **Confianza moderada**: nombres exactos de `cardName`/`propertyName` dentro de `visualStyles` por tipo de visual (`card`, `slicer`, `barChart`, `tableEx`, etc.) — basados en convenciones ampliamente usadas en temas publicados, no verificados contra el esquema JSON oficial descargable.

**Si al importar el tema aparece un error de validación**, repórtame el mensaje exacto y la sección señalada — es más probable que provenga de `visualStyles` que de la sección de colores.

## Decisión sobre aplicación automática o importación manual del tema

**Importación manual.** No se modificó `report.json` para referenciar este tema automáticamente: el mecanismo real de registro de un tema importado por Power BI Desktop (copia a `StaticResources/RegisteredResources/` con convención interna específica) no se verificó con certeza, y es un tipo de archivo nunca antes editado en este proyecto. Se prefirió no forzar una edición no verificada, siguiendo la misma disciplina de los incidentes de `lineageTag`/`description` en TMDL.

**Pasos para importar manualmente:** Power BI Desktop → pestaña **Vista** → **Temas** → **Examinar temas** → seleccionar `Assets/theme/connect_assistance_theme.json` → confirmar importación exitosa → guardar.

## Archivos modificados o creados

- `Assets/theme/connect_assistance_theme.json` (nuevo).
- `Assets/imagenes/` (carpeta nueva, con 6 imágenes ya provistas por el usuario).
- `Assets/logos/` — sin cambios de mi parte; contiene los 2 logos provistos por el usuario (1 ya existía al inicio de la fase, el segundo apareció durante la ejecución).

No se modificó ningún archivo del modelo semántico, Power Query, medidas, relaciones, ni `Data/*.xlsx`. No se crearon páginas ni visuales del reporte.

## Resultado del commit

- Mensaje: `style(theme): crear tema visual connect assistance`.
- Archivos incluidos: `Assets/theme/connect_assistance_theme.json` (nuevo), `Assets/logos/*.svg` (2 archivos, ya provistos por el usuario, incorporados al control de versiones por ser activos de marca, no datos personales), `Assets/imagenes/*.png` (6 archivos, mismo criterio), `Outputs/17_resultado_fase_12_tema_visual_connect.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar a Fase 13

**Antes de avanzar:**
1. Importar el tema manualmente en Power BI Desktop (pasos arriba) y confirmar que se aplica sin error de validación.
2. Revisar visualmente el resultado sobre un visual de prueba temporal — no pude generar esa vista previa yo mismo.
3. Si es posible, confirmar con el área de marca de Connect Assistance si `#F15B2B` y `#002733` son efectivamente los colores oficiales (encontrados en el logo, pero no en un manual de marca formal), y si existen colores secundarios oficiales más allá de los grises neutros usados aquí.
4. Las 6 imágenes ya disponibles en `Assets/imagenes/` (ilustraciones tipo "Cambio de llanta", "Moto Sr. Connect", y 4 exportaciones "Group") son candidatas directas para el diseño del Home tipo landing page de la Fase 13.

Si el tema se importa correctamente, el proyecto queda listo para la **Fase 13 — Diseño del Home principal tipo landing page**. **No se avanzó a la Fase 13 en esta ejecución.**

---

*Documento generado como registro operativo de la Fase 12, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
