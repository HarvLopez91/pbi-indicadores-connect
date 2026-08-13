# Gestión comercial de altas Te Resuelve

Guía funcional y técnica de la página **Gestión comercial de altas** (`GestionComercialAltas`), dirigida a PUSHER, líderes comerciales, analistas y a quien mantenga el PBIP en el futuro.

## 1. Objetivo

La página permite analizar históricamente las altas de Te Resuelve por **periodo**, **PUSHER** y **aliado**. Con ella se puede observar volumen, cambio mensual, distribución por PUSHER, drivers (aliados que más suben o bajan) y comportamiento individual de cada aliado.

La página **no demuestra causalidad**: muestra asociación temporal y contribución observada, no prueba que una gestión específica haya "causado" un resultado.

## 2. Fuente de información

- **Fuente operativa**: tabla formal `Insumo2` del archivo vigente de altas, ubicado en la subcarpeta `Informe de Altas` dentro de la carpeta de datos configurada del proyecto (parámetro `RutaCarpetaData`). El nombre del archivo y la ruta completa no se documentan aquí por ser información local del equipo donde corre el proyecto.
- **Regla de cálculo**: `Altas = SUM(Fact_AltasTeResuelve[Altas])`.
- **Grano**: una fila del hecho puede representar **múltiples altas** (la columna `Altas` es una cantidad, no un indicador binario). No interpretar una fila como una venta o un cliente individual — no existe identificador de venta ni de cliente en la fuente.
- La consulta de origen (`Base_AltasTeResuelve`) valida en tiempo de refresh que exista un único archivo con el nombre esperado, una única tabla `Insumo2`, las columnas obligatorias (`ALTAS`, `DESCRIPCION`, `FECHA_ALTA`, `MES`) y que el periodo de corte configurado exista en los datos; si alguna condición falla, el refresh se detiene con un error explícito en vez de cargar datos parciales o incorrectos.
- Solo las filas que pasan la validación de calidad (`Altas` numérica no nula, `FechaAlta` válida, `Mes` válido y coincidente con `FechaAlta`, `Descripcion` no vacía) llegan a `Fact_AltasTeResuelve`. Las filas inválidas se descartan antes del hecho final.

## 3. Modelo

| Objeto | Contenido |
|---|---|
| `Fact_AltasTeResuelve` | Una fila por combinación agregada de fecha/aliado con `FechaAlta`, `AliadoKey`, `Altas`. |
| `Dim_Aliado` | Una fila por aliado (`Descripcion`), con `AliadoKey`, `Pusher` y `Estado_Clasificacion`. |
| `Dim_Calendario` | Calendario compartido con el resto del reporte; para Altas aporta `Periodo_Gestion`, `Estado_Periodo` y `Es_Periodo_Comparable`. |
| `_Medidas_Altas` | Tabla exclusiva de medidas DAX de esta iniciativa. |

Relaciones: `Dim_Calendario` 1:* `Fact_AltasTeResuelve` (por `Fecha`/`FechaAlta`) y `Dim_Aliado` 1:* `Fact_AltasTeResuelve` (por `AliadoKey`), ambas unidireccionales. No existen relaciones entre `Fact_AltasTeResuelve` y las tablas de hechos de las otras encuestas, ni relación directa con `Dim_CallCenter` (dominios distintos: aliados comerciales vs. call centers de las encuestas de calidad/satisfacción/motivación).

## 4. Clasificación PUSHER

Cada aliado (`Dim_Aliado[Descripcion]`) se clasifica en `Pusher` como **PUSHER 1**, **PUSHER 2** o **Sin asignar**.

El mapeo se resuelve por **coincidencia exacta** contra una lista fija de aliados (`Map_PusherAliado`), después de normalizar el texto con `Trim`, `Clean` y mayúsculas. **No hay coincidencia aproximada** ("fuzzy matching"): un aliado que no aparece exactamente en la lista queda como `Sin asignar`, sin importar cuán parecido sea su nombre a uno clasificado.

La **cobertura de clasificación** (medida `Cobertura_Clasificacion_Pusher_Pct`, visible como "Altas en aliados con PUSHER") es dinámica: cambia según los filtros activos (mes, PUSHER, aliado). Ejemplos observados:

- Julio 2026: 83,80 %.
- Agosto 2026 (parcial): 85,15 %.

Esta cobertura **no es un indicador de desempeño**. Es la proporción de altas que hoy caen en un aliado ya clasificado como PUSHER 1 o PUSHER 2, frente al total del universo (incluyendo `Sin asignar`).

## 5. Regla temporal PUSHER 2

**Punto crítico de interpretación.** La gestión de PUSHER 2 inicia el **1 de julio de 2026** (parámetro `Fecha_Inicio_Gestion_Pusher_2`).

- **Antes de julio**: las altas de los aliados hoy asignados a PUSHER 2 deben leerse exclusivamente como **"Línea base histórica del portafolio P2"** — volumen que ya existía antes de que empezara cualquier gestión activa.
- **Desde julio**: esas mismas altas se identifican como **"PUSHER 2 · desde inicio de gestión"**.

Ambas series provienen del mismo `Pusher = "PUSHER 2"`, separadas únicamente por `Dim_Calendario[Periodo_Gestion]` (columna calculada a partir de `Fecha_Inicio_Gestion_Pusher_2`), nunca se solapan en el mismo mes.

**No debe afirmarse**:
- que PUSHER 2 causó el crecimiento de julio;
- que el aumento fue "gracias a" PUSHER 2;
- ninguna relación causa-efecto no demostrada por la fuente (que no contiene tráfico, campañas, metas ni capacidad operativa).

**Sí puede usarse**: recuperación observada, cambio observado, contribución al cambio, comportamiento desde el inicio de gestión.

## 6. Filtros

| Filtro | Campo | Comportamiento |
|---|---|---|
| Año | `Dim_Calendario[Anio]` | Filtra todo el contenido de la página. |
| Mes | `Dim_Calendario[MesNombre]` | Filtra los KPI, drivers y ranking. **No filtra el gráfico histórico principal** (excepción intencional, ver abajo). |
| PUSHER | `Dim_Aliado[Pusher]` | Filtra todo el contenido de la página. |
| Aliado | `Dim_Aliado[Descripcion]` | Filtra todo el contenido de la página. |

**Decisión funcional importante**: el segmentador Mes está configurado explícitamente como `NoFilter` hacia el gráfico "Evolución mensual y distribución por PUSHER". El gráfico conserva siempre el contexto histórico completo (todos los meses disponibles) para permitir comparar la evolución, mientras los demás KPI y visuales sí responden al mes seleccionado.

## 7. Indicadores de la página

### Altas del periodo
`Altas_Total` — `SUM(Fact_AltasTeResuelve[Altas])` bajo los filtros activos.

### Cambio vs. mes anterior
`Diferencia_Altas_Mes` — diferencia absoluta contra el mes comparable anterior. Solo se calcula cuando hay exactamente un mes seleccionado y ese mes es comparable (ver §10); de lo contrario queda en blanco.

### Variación vs. mes anterior
`Variacion_Altas_Mes_Pct` — cambio relativo (`Diferencia_Altas_Mes` ÷ `Altas_Mes_Anterior`).

### Promedio histórico previo
`Promedio_Altas_Hasta_Mes_Anterior` — promedio de los totales mensuales desde enero del mismo año hasta el mes anterior al seleccionado (solo meses cerrados/comparables). Para julio 2026: **4.796**.

### Meta +30 %
`Meta_Altas_Promedio_Mas_30_Pct` — `Promedio_Altas_Hasta_Mes_Anterior × 1,30`. Para julio 2026: **6.235**. Es una **referencia/proyección comercial interna**, no una predicción estadística ni un compromiso.

### Altas en aliados con PUSHER
`Cobertura_Clasificacion_Pusher_Pct` — proporción de altas asociadas a aliados clasificados como PUSHER 1 o PUSHER 2. Julio 2026: **83,80 %**. No debe confundirse con la participación de PUSHER 2 en el total (§4) ni con la contribución de PUSHER 2 al cambio mensual (§9).

## 8. Gráfico histórico

**Evolución mensual y distribución por PUSHER** — columnas apiladas por mes, con 4 series:

- PUSHER 1 (`Altas_Pusher_1`)
- Línea base portafolio P2 (`Altas_Portafolio_Pusher_2_Linea_Base`)
- PUSHER 2 · desde inicio de gestión (`Altas_Pusher_2_Desde_Gestion`)
- Sin asignar (`Altas_Sin_Asignar`)

Cada columna muestra la etiqueta de valor de cada segmento y el total mensual encima de la columna. Casos de control:

- Junio: 1.193 + 1.857 + 650 = **3.700**.
- Julio: 1.357 + 2.429 + 732 = **4.518**.

## 9. Drivers y ranking

- **Mayores contribuciones positivas** / **Principales caídas**: dos gráficos de barras que muestran los aliados con mayor cambio mensual positivo/negativo (`Delta_Aliado_Mes`, limitado a los 2 primeros de cada lado mediante `Ranking_Driver_Positivo`/`Ranking_Driver_Negativo`). Representan **cambio mensual observado**, no una medición de esfuerzo o desempeño individual.

  Ejemplo julio 2026: ATENTO +381, ONE CONTACT +201, GNP -70.

- **Ranking de aliados · volumen y cambio mensual**: tabla con Aliado, PUSHER, Altas, Mes anterior, Diferencia y Variación % para todos los aliados con datos en el periodo filtrado.

**Particularidad técnica**: no debe sumarse `Delta_Aliado_Mes` directamente como total general cuando existan aliados sin base en el mes anterior, porque esa medida conserva `BLANK()` para esos casos (no los trata como cero). El total oficial del cambio mensual es `Diferencia_Altas_Mes`, no la suma de los deltas por aliado.

## 10. Periodos parciales

`Dim_Calendario` clasifica cada mes según el parámetro `Periodo_Corte_Comercial` (mes-año del último cierre oficial, formato `AAAAMM`):

- `Estado_Periodo`: "Cerrado" (≤ corte), "En curso" (el mes más reciente disponible en la fuente, posterior al corte), "Posterior al corte sin validar" u otros estados de control.
- `Es_Periodo_Comparable`: verdadero solo para meses cerrados (≤ corte). Las medidas de cambio/variación mensual solo calculan cuando el mes seleccionado es comparable.

Al momento de esta guía, agosto 2026 es un **periodo parcial en curso**. Ejemplo validado:

| Indicador | Agosto (parcial) |
|---|---:|
| Altas | 559 |
| Cambio vs. mes anterior | *(en blanco)* |
| Variación vs. mes anterior | *(en blanco)* |
| Promedio histórico previo | 4.756 |
| Meta +30 % | 6.183 |
| Altas en aliados con PUSHER | 85,15 % |

Las altas de un periodo en curso **sí pueden mostrarse** (son datos reales ya cargados), pero las **comparaciones mensuales no deben leerse como cierre** — por eso las medidas de cambio/variación quedan en blanco automáticamente en vez de comparar un mes incompleto contra uno cerrado.

Al cerrar un nuevo mes, el paso operativo es actualizar `Periodo_Corte_Comercial` al nuevo `AAAAMM` de cierre (ver §11). No existe hoy un procedimiento automático de cierre de mes distinto a ese parámetro.

## 11. Actualización y mantenimiento

Procedimiento soportado por la implementación actual:

1. Actualizar el archivo fuente autorizado en la carpeta configurada (`RutaCarpetaData\Informe de Altas`), respetando el nombre de archivo esperado.
2. Confirmar que la tabla formal `Insumo2` conserva las columnas obligatorias (`ALTAS`, `DESCRIPCION`, `FECHA_ALTA`, `MES`) — si falta alguna, el refresh se detiene con error.
3. Actualizar el parámetro `Periodo_Corte_Comercial` (Power Query) cuando exista un nuevo cierre oficial.
4. Actualizar la lista `Map_PusherAliado` (Power Query) **solo con aprobación funcional explícita** — hoy es una lista fija de coincidencias exactas embebida en la consulta, no una tabla externa editable por negocio.
5. Ejecutar el refresh del modelo.
6. Validar los totales contra la fuente (`Altas_Total` del mes cerrado más reciente vs. el pivote de referencia del Excel).
7. Revisar los aliados `Sin asignar` de mayor volumen — la consulta de control interna identifica los aliados sin clasificación PUSHER.
8. Validar el mes nuevo en Power BI Desktop antes de publicar.
9. Publicar únicamente después de completar el QA visual y funcional.

Reglas de gobierno:

- No agregar un aliado a un PUSHER por coincidencia aproximada de nombre.
- Cualquier equivalencia nueva en `Map_PusherAliado` debe aprobarse explícitamente antes de editar la consulta.
- Un mes en curso no debe convertirse automáticamente en "cerrado" solo porque ya está disponible en la fuente — requiere actualizar `Periodo_Corte_Comercial` de forma deliberada.

## 12. Controles de calidad (cifras del corte actual)

Estas cifras son una **referencia del corte vigente al momento de esta guía**, no valores permanentes — cambiarán con cada actualización de la fuente.

| Indicador | Valor |
|---|---:|
| Total archivo vigente | 33.854 |
| Total enero-julio | 33.295 |
| Junio | 3.700 |
| Julio | 4.518 |
| Cambio junio→julio | +818 |
| Variación junio→julio | +22,11 % |
| Driver ATENTO | +381 |
| Driver ONE CONTACT | +201 |
| Driver GNP | -70 |

## 13. Privacidad

`Gestión comercial de altas` **no expone**: `JEFE`, `ESPECIALISTA`, `ASESOR`, nombres individuales, rutas locales, metadata operativa ni datos fila a fila de personas. Estas tres columnas sensibles se eliminan explícitamente en la limpieza de Power Query, antes de llegar al modelo publicado.

Modelo comercial aprobado y expuesto en el reporte:

- **Hecho**: `FechaAlta`, `AliadoKey`, `Altas`.
- **Dimensión de aliado**: `AliadoKey`, `Descripcion`, `Pusher`, `Estado_Clasificacion`.

Ningún dato personal se reproduce en esta guía.

## 14. Exportación y Focus mode

El reporte permite exportación resumida (`exportDataMode = AllowSummarized`): solo la vista agregada visible de cada visual, nunca filas de detalle de la fuente. Como el modelo comercial no contiene campos personales, ninguna exportación posible desde esta página puede revelar datos individuales.

**Focus mode** está habilitado en: gráfico histórico, drivers positivos, drivers negativos y ranking de aliados.

**Limitación conocida**: en Focus mode del gráfico histórico, Power BI puede mostrar un título vertical truncado en el eje aunque la propiedad de título del eje esté desactivada. No afecta la vista normal del reporte ni ninguna cifra. No se abrirá una tarea técnica adicional por este punto.

## 15. Interpretación comercial

**Qué SÍ se puede concluir**:

- Julio 2026 muestra una recuperación observada frente a junio.
- Determinados aliados (ATENTO, ONE CONTACT) explican una parte relevante del cambio observado en julio.
- El portafolio de PUSHER 2 puede analizarse por separado antes y después del inicio de su gestión.

**Qué NO se puede concluir**:

- Causalidad directa entre la gestión de PUSHER 2 y el crecimiento.
- Que PUSHER 2 sea responsable único del aumento (PUSHER 1 y el universo sin asignar también crecieron en el mismo periodo).
- Que cobertura de clasificación equivalga a impacto o desempeño.
- Que un mes parcial (como agosto en curso) sea comparable con un mes cerrado.

## 16. Glosario breve

- **Alta**: unidad de conteo de la fuente comercial; una fila puede representar varias altas.
- **Aliado**: call center o canal comercial identificado por `Descripcion` en la fuente de altas (dominio distinto al de `Dim_CallCenter`, usada por las encuestas).
- **PUSHER**: clasificación comercial de un aliado (PUSHER 1, PUSHER 2 o Sin asignar).
- **Línea base**: volumen histórico de un aliado de PUSHER 2 antes del inicio de su gestión activa.
- **Periodo comparable**: mes cerrado, apto para cálculos de cambio/variación mensual.
- **Cobertura**: proporción de altas en aliados ya clasificados con PUSHER, frente al universo total.
- **Driver**: aliado cuyo cambio mensual (positivo o negativo) más contribuye al cambio total observado.
