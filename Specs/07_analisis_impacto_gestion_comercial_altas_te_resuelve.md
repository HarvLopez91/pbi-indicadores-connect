# Análisis de impacto — Gestión comercial de altas Te Resuelve

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Archivo analizado | `Data/Informe de Altas/INFORME ALTAS TE RESUELVE Cierre Julio.xlsx` |
| Fuente candidata | Hoja `Tabla1_2 (2)`, tabla formal `Insumo2` |
| Referencia de conciliación | Hoja `INFORME ALTAS MULTIASISTENCIAS` |
| Línea base histórica | `Specs/04_analisis_impacto_informe_altas_t_resuelve.md` |
| Fecha del análisis | 2026-08-11 |
| Estado | Fase 1 cerrada; diagnóstico sin implementación |

## 1. Resumen ejecutivo

La fuente de julio confirma que el indicador comercial debe calcularse como `SUM(ALTAS)` sobre la tabla formal `Insumo2` de la hoja `Tabla1_2 (2)`. Esta conclusión se sustenta en tres evidencias: `ALTAS` es una cantidad entera positiva, 3.476 filas representan más de una alta, y los pivotes del tablero visible usan `Insumo2` como origen y `Suma de ALTAS` como agregación. El total de julio calculado desde el detalle, 4.518 altas, coincide exactamente con el total general del pivote regional de `INFORME ALTAS MULTIASISTENCIAS`.

El trimestre abril-junio presenta una caída continua: 4.190, 4.171 y 3.700 altas. Julio revierte esa secuencia con 4.518 altas: +818 frente a junio (+22,1%) y +497,7 frente al promedio abril-junio (+12,4%). PUSHER 2 aporta +572 altas entre junio y julio, equivalentes al 69,9% del cambio neto total; los principales impulsores son ATENTO (+381) y ONE CONTACT (+201). Este resultado muestra contribución observada y asociación temporal, pero no demuestra causalidad de la gestión porque la fuente no contiene tráfico, base contactada, campañas, metas, capacidad, disponibilidad ni otras variables operativas.

El libro denominado “Cierre Julio” contiene además 559 altas parciales de agosto, con fecha máxima 2026-08-04. La implementación futura debe aplicar un corte explícito o un parámetro de periodo; de lo contrario, el reporte podría mezclar julio cerrado con agosto parcial.

La recomendación es crear una página nueva de gestión comercial y una tabla de hechos independiente, reutilizando `Dim_Calendario`, los patrones visuales, Home y la navegación. No se recomienda incorporar directamente los 167 valores comerciales de `DESCRIPCION` en la `Dim_CallCenter` actual, porque esa dimensión se construye desde encuestas y representa otro dominio. La clasificación PUSHER debe mantenerse en una tabla de mapeo gobernada y editable.

## 2. Antecedentes

### 2.1 Línea base histórica de junio

`Specs/04_analisis_impacto_informe_altas_t_resuelve.md` analizó el archivo `INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx`, modificado el 2026-07-09. Registró 23.015 filas útiles, 29.366 altas acumuladas, fechas entre 2026-01-01 y 2026-07-05, y 3.700 altas para junio. También estableció que una fila no equivale necesariamente a una venta, identificó datos personales en `JEFE`, `ESPECIALISTA` y `ASESOR`, y recomendó una tabla de hechos comercial independiente.

Ese archivo histórico ya no está en `Data/Informe de Altas`; la comparación se realiza contra las cifras y hallazgos documentados en `Specs/04`. El archivo actual suma 28.777 altas entre enero y junio y 564 entre el 1 y el 5 de julio: 29.341 hasta ese corte. La línea base documentó 29.366 altas hasta el 5 de julio, una diferencia de 25 que no puede reconstruirse sin el libro histórico. Los totales mensuales enero-junio publicados en `Specs/04` para los aliados principales sí coinciden con el archivo actual, pero el total acumulado histórico debe tratarse como una discrepancia no resuelta y no como una conciliación exacta.

### 2.2 Hallazgos que siguen vigentes

- La tabla de detalle es una base agregada/transaccional, no una fila por venta individual.
- La métrica correcta requiere sumar `ALTAS`, no contar filas.
- `DESCRIPCION` representa al aliado o call center comercial, pero no tiene equivalencia garantizada con `Dim_CallCenter`.
- `JEFE`, `ESPECIALISTA` y `ASESOR` son campos sensibles y no deben incorporarse a una página pública.
- La hoja visible está orientada al consumo humano; la tabla formal oculta es la fuente adecuada para ingesta.
- El modelo comercial debe permanecer separado de las tres tablas de encuestas.

### 2.3 Cambios frente a junio

| Elemento | Línea base de junio | Archivo actual | Cambio |
|---|---:|---:|---|
| Filas útiles | 23.015 | 26.496 | +3.481 |
| Altas acumuladas del archivo | 29.366 | 33.854 | +4.488 |
| Fecha máxima | 2026-07-05 | 2026-08-04 | Julio completo y agosto parcial |
| Altas de junio | 3.700 | 3.700 | Sin cambio |
| Altas de julio | Parcial al 05/07 | 4.518 | Mes completo disponible |
| Descripciones distintas | No consolidado en el documento | 167 | Catálogo real disponible |
| Nulos en `DESCRIPCION2` | 350 | 372 | +22 |
| Filas con `ALTAS > 1` | 3.022 | 3.476 | +454 |

La información nueva permite validar julio completo, medir la recuperación frente a junio, cuantificar el aporte por aliado y clasificar el portafolio por PUSHER. También obliga a actualizar el supuesto de “cierre julio”: el archivo incluye agosto parcial y no debe cargarse sin una regla de corte.

La variación de +4.488 entre archivos no debe interpretarse como suma pura de nuevas ventas: el cierre actual contiene +3.954 adicionales de julio respecto al tramo 1-5, 559 de agosto parcial y una diferencia neta de -25 en el tramo histórico hasta el 5 de julio.

## 3. Estado actual del PBIP

### 3.1 Páginas y navegación

El reporte contiene siete páginas activas:

| Nombre técnico | Nombre visible | Visuales | Observación |
|---|---|---:|---|
| `67eff42d82e1c9c15b84` | Home | 41 | Página inicial activa |
| `p14_resumen_ejecutivo` | Resumen ejecutivo | 24 | Reutilizable como patrón de síntesis |
| `p14_calidad_llamadas` | Calidad de llamadas | 22 | No debe modificarse en esta iniciativa |
| `p14_satisfaccion_capacitaciones_v2` | Satisfacción de capacitaciones | 47 | Fuera de alcance |
| `p14_motivacion_comercial` | Motivación comercial | 26 | Reutilizable como patrón visual, no como fuente |
| `p14_detalle_call_center` | Detalle por call center | 24 | El concepto de call center requiere homologación |
| `p14_notas_metodologicas` | Notas metodológicas | 45 | Deberá ampliarse si se implementa la fuente |

Home permanece como `activePageName` y el patrón de navegación mediante botones y zonas de interacción puede reutilizarse para una nueva página, sin alterar las páginas existentes.

### 3.2 Modelo semántico

El modelo actual contiene:

- Hechos: `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`.
- Dimensiones explícitas: `Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`, `Dim_MetricaSatisfaccion`.
- Tablas de medidas: `_Medidas Generales`, `_Medidas Calidad`, `_Medidas Capacitacion`, `_Medidas Motivacion`.
- Relaciones simples desde cada hecho a `Dim_Calendario` y `Dim_CallCenter`; Satisfacción y Motivación también se relacionan con `Dim_Jornada`.
- Power Query con patrón `RutaCarpetaData` → `Base_*` → `*_Limpio` → `Fact_*`.

`Dim_Calendario` es reutilizable: contiene fecha, año, mes, año-mes, trimestre, semana y atributos diarios. Su consulta, sin embargo, calcula la fecha mínima solo desde los tres hechos actuales; una implementación comercial deberá incluir `Fact_AltasTeResuelve` en ese cálculo o redefinir el rango con una regla estable.

`Dim_CallCenter` no debe reutilizarse sin ajuste conceptual. Hoy combina valores distintos de las tres encuestas; la fuente comercial presenta 167 valores en `DESCRIPCION`, incluidos CAV, tiendas, aliados BPO y otras operaciones. Incorporarlos sin catálogo maestro cambiaría el dominio y podría afectar todos los visuales existentes.

El modelo conserva Auto date/time activo (`__PBI_TimeIntelligenceEnabled = 1`) y tres `LocalDateTable`, además de `Dim_Calendario`. Es un riesgo técnico preexistente; no se corrige en esta fase.

### 3.3 Publicación y privacidad

El reporte está documentado con un enlace de “Publicar en la Web”, sin autenticación. La nueva fuente contiene 70 jefes, 241 especialistas y 2.766 asesores distintos. Ninguno de esos campos debe cargarse o exponerse en la primera versión comercial. La página debe permanecer agregada por periodo, PUSHER y aliado.

## 4. Análisis del Excel de julio

### 4.1 Estructura del libro

| Hoja | Estado | Rango | Función |
|---|---|---|---|
| `Tablas_back` | Oculta | `A1:N34` | Cuatro pivotes auxiliares |
| `Tabla1_2 (2)` | Oculta | `A1:R26497` | Tabla formal `Insumo2`, 26.496 filas y 18 columnas |
| `Tabla1` | Oculta | `A1:P91044`; tabla `Insumo` hasta fila 26.497 | Copia de detalle con 16 columnas comunes |
| `INFORME ALTAS MULTIASISTENCIAS` | Visible | `A1:W31` | Tablero original con cuatro pivotes |

Los ocho pivotes del libro usan un único caché cuyo origen es la tabla formal `Insumo2`. Todos los campos de valores se agregan como suma de `ALTAS`.

### 4.2 Columnas de `Insumo2`

`ALTAS`, `FECHA_ALTA`, `MES`, `FECHA_ALTA - Copy.1`, `DIVISION`, `CANAL`, `CANAL2`, `TIPO_PAQUETE`, `DESCRIPCION`, `DESCRIPCION2`, `TIPO`, `SUBTIPO`, `SERVICIO`, `JEFE`, `ESPECIALISTA`, `UNIDAD_NEGOCIO`, `ASESOR`, `UNIDAD_NEGOCIO2`.

La implementación futura deberá normalizar estos nombres a nombres técnicos sin espacios ni signos, por ejemplo `Altas`, `FechaAlta`, `Mes`, `Descripcion` y `Pusher`.

### 4.3 Periodos y volumen

- Rango real: 2026-01-01 a 2026-08-04.
- Julio está completo: 2026-07-01 a 2026-07-31.
- Meses disponibles: enero-agosto de 2026.
- Filas útiles: 26.496.
- Total del archivo: 33.854 altas.
- Total enero-julio: 33.295 altas.
- Agosto parcial: 559 altas en 452 filas; debe excluirse de la lectura de cierre julio.

## 5. Comparación con el análisis histórico de junio

La línea base acertó en los puntos estructurales: grano agregado, necesidad de sumar `ALTAS`, sensibilidad de campos nominales, duplicación de tablas internas y separación conceptual entre aliado comercial y call center de encuestas.

Debe actualizarse en cuatro aspectos:

1. La regla `.gitignore` ya cubre recursivamente `Data/**/*.xlsx`; el riesgo de versionar este Excel anidado fue corregido antes de esta fase.
2. La fuente actual ya permite evaluar julio completo, no solo un avance al 5 de julio.
3. La iniciativa actual no depende de metas individuales ni de la regla de crecimiento del 30%; su foco es comparación mensual y clasificación PUSHER.
4. Los nombres solicitados para las dos PUSHER aparecen hoy exactamente después de `Trim`, `Clean` y mayúsculas; las abreviaciones históricas `ALMA EXP`, `IBR` y `CAPITALS` no deben aplicarse automáticamente al archivo actual.

## 6. Calidad y grano de los datos

### 6.1 Calidad observada

- `ALTAS`: 26.496 valores numéricos, mínimo 1, máximo 41; sin ceros, negativos ni nulos.
- 3.476 filas tienen `ALTAS > 1`; contar filas subestimaría el indicador.
- `FECHA_ALTA`: sin nulos; 216 fechas distintas.
- `MES`: ocho valores `yyyymm`; coincide con el año y mes de `FECHA_ALTA` en todas las filas.
- No se encontraron filas duplicadas exactas en las 18 columnas.
- Nulos: `DIVISION` 1; `DESCRIPCION2` 372; `UNIDAD_NEGOCIO2` 9.871.
- `TIPO`, `SUBTIPO`, `UNIDAD_NEGOCIO` y `UNIDAD_NEGOCIO2` tienen un único valor no nulo; aportan poco como filtros en el corte actual.
- `DESCRIPCION` no presenta nulos y tiene 167 valores distintos.

### 6.2 Grano

Una fila representa una combinación agregada de fecha, periodo, división, canal, aliado, producto y responsables comerciales, con una cantidad en `ALTAS`. No existe identificador de venta, cliente o transacción. La llave física es compuesta y no debe exponerse como clave de negocio.

## 7. Validación de `ALTAS`

`Ventas/Altas = SUM(ALTAS)` queda **confirmado**:

1. `ALTAS` es una cantidad positiva y puede ser mayor que uno.
2. Los pivotes del libro usan `Suma de ALTAS`.
3. La suma de julio desde el detalle es 4.518.
4. El total general de julio del pivote regional visible también es 4.518.
5. `Tabla1` y `Tabla1_2 (2)` coinciden en filas, total y totales mensuales.

Contar filas no es válido: julio contiene cantidades agregadas y el archivo completo tiene 3.476 filas con más de una alta.

## 8. Validación de `Tabla1_2 (2)`

La hoja `Tabla1_2 (2)` y la tabla `Insumo2` quedan **confirmadas** como fuente adecuada porque:

- Son el origen formal de los pivotes del tablero.
- Contienen todas las columnas comunes de `Tabla1` y dos campos adicionales.
- Sus 26.496 filas coinciden con la tabla formal `Insumo` en las columnas comunes.
- Sus totales mensuales coinciden con `Tabla1`.
- La estructura tabular tiene encabezado único y rango formal definido.

La ingesta futura debe seleccionar la tabla `Insumo2`, no depender de la posición física de la hoja ni de la hoja visible.

## 9. Conciliación con `INFORME ALTAS MULTIASISTENCIAS`

La hoja visible se utilizó únicamente como referencia de lógica y conciliación. El pivote regional muestra para julio:

| División | Altas julio |
|---|---:|
| Región Centro | 1.726 |
| Región Noroccidente | 800 |
| Región Oriente | 565 |
| Región Occidente | 998 |
| Región Costa | 428 |
| En blanco | 1 |
| **Total general** | **4.518** |

Este total coincide con `SUM(Insumo2[ALTAS])` filtrado por `MES = 202607`. La conciliación confirma la lógica general, pero no convierte la hoja visible en fuente de ingesta: contiene pivotes, rankings nominales y diseño orientado a lectura.

## 10. Análisis mensual enero-julio

| Mes | Altas | Diferencia mensual | Variación mensual |
|---|---:|---:|---:|
| Enero | 6.297 | — | — |
| Febrero | 5.310 | -987 | -15,7% |
| Marzo | 5.109 | -201 | -3,8% |
| Abril | 4.190 | -919 | -18,0% |
| Mayo | 4.171 | -19 | -0,5% |
| Junio | 3.700 | -471 | -11,3% |
| Julio | 4.518 | +818 | +22,1% |

Julio presenta la única variación mensual positiva del periodo analizado. Marzo no creció frente a febrero en el total general, a diferencia de la serie de siete aliados que aparece en los mockups; esta diferencia de universo debe quedar visible en el diseño futuro.

## 11. Análisis específico abril-julio

Abril, mayo y junio descienden consecutivamente: 4.190 → 4.171 → 3.700. La caída acumulada de abril a junio es de 490 altas (-11,7%).

| Indicador | Resultado |
|---|---:|
| Promedio abril-junio | 4.020,3 |
| Julio | 4.518 |
| Julio vs promedio abril-junio | +497,7 |
| Julio vs promedio abril-junio | +12,4% |
| Julio vs junio | +818 |
| Julio vs junio | +22,1% |

La recuperación es observable tanto frente al último mes como frente al promedio inmediato previo. No prueba que el nivel sea sostenible ni que la intervención de una PUSHER sea la causa única.

## 12. Clasificación por PUSHER

### 12.1 Coincidencias exactas

Después de aplicar únicamente `Text.Trim`, `Text.Clean` y mayúsculas, los 16 aliados solicitados aparecen exactamente en `DESCRIPCION`. No falta ningún aliado de la regla vigente y no fue necesario aplicar coincidencias aproximadas.

| PUSHER 1 | PUSHER 2 |
|---|---|
| MILLENIUM | BRM |
| VECTOR | CAV BOGOTA PLAZA IMPERIAL |
| INTELIGENCE BUSSINES RECOVERY COL | CAPITALS TELECOM BPO |
| ASISTE ING | ONE CONTACT |
| COS | INTERACTIVO |
| CAV BOGOTA CENTRO MAYOR | ATENTO |
| AIB | CAV BOGOTA PLAZA DE LAS AMERICAS |
|  | ALMAEXPERIENCE SAS |
|  | GNP |

Las variantes históricas `ALMA EXP`, `IBR` y `CAPITALS` no aparecen como valores exactos en el archivo actual; sí aparecen sus nombres completos `ALMAEXPERIENCE SAS`, `INTELIGENCE BUSSINES RECOVERY COL` y `CAPITALS TELECOM BPO`. No debe crearse un alias mientras el origen actual sea consistente, aunque la tabla de mapeo futura debería admitir alias gobernados para cierres anteriores.

### 12.2 Valores no clasificados

La fuente contiene 167 descripciones y la regla PUSHER clasifica 16; quedan 151 valores como `Sin asignar`. Entre los de mayor volumen enero-julio están:

| Descripción | Altas enero-julio |
|---|---:|
| `UNO 27` | 1.225 |
| `TRESUELVE HOGAR` | 1.169 |
| `TEAM COMUNICACIONES` | 120 |
| `INVERSIONES ARAUJO` | 98 |
| `CAV CUCUTA AV GRAN COLOMBIA` | 81 |

El catálogo completo de no clasificados debe conservarse como salida de control en la implementación de Power Query, sin forzar equivalencias. La primera página puede ofrecer una categoría `Sin asignar` y una alerta de cobertura.

## 13. Comparativo PUSHER 1 vs PUSHER 2

La clasificación se aplica por coincidencia exacta; el porcentaje restante corresponde a aliados sin asignar.

| Mes | PUSHER 1 | Part. | PUSHER 2 | Part. | Sin asignar | Part. |
|---|---:|---:|---:|---:|---:|---:|
| Enero | 2.298 | 36,5% | 3.094 | 49,1% | 905 | 14,4% |
| Febrero | 2.334 | 44,0% | 2.347 | 44,2% | 629 | 11,8% |
| Marzo | 2.003 | 39,2% | 2.570 | 50,3% | 536 | 10,5% |
| Abril | 1.473 | 35,2% | 2.226 | 53,1% | 491 | 11,7% |
| Mayo | 1.563 | 37,5% | 1.976 | 47,4% | 632 | 15,2% |
| Junio | 1.193 | 32,2% | 1.857 | 50,2% | 650 | 17,6% |
| Julio | 1.357 | 30,0% | 2.429 | 53,8% | 732 | 16,2% |

Entre junio y julio:

- PUSHER 1: +164 altas.
- PUSHER 2: +572 altas.
- Sin asignar: +82 altas.
- Cambio total: +818 altas.

PUSHER 2 representa 69,9% del cambio neto observado y aumenta su participación en 3,6 puntos porcentuales. PUSHER 1 también crece, por lo que la recuperación no es exclusiva de PUSHER 2.

## 14. Contribución de cada call center al cambio de julio

### 14.1 Principales aumentos junio-julio

| Descripción | PUSHER | Junio | Julio | Variación | Aporte al cambio neto |
|---|---|---:|---:|---:|---:|
| ATENTO | PUSHER 2 | 1.014 | 1.395 | +381 | 46,6% |
| ONE CONTACT | PUSHER 2 | 361 | 562 | +201 | 24,6% |
| INTELIGENCE BUSSINES RECOVERY COL | PUSHER 1 | 597 | 688 | +91 | 11,1% |
| UNO 27 | Sin asignar | 138 | 224 | +86 | 10,5% |
| BRM | PUSHER 2 | 20 | 50 | +30 | 3,7% |
| ALMAEXPERIENCE SAS | PUSHER 2 | 26 | 55 | +29 | 3,5% |
| COS | PUSHER 1 | 462 | 490 | +28 | 3,4% |
| MILLENIUM | PUSHER 1 | 33 | 59 | +26 | 3,2% |

ATENTO y ONE CONTACT aportan juntos +582 altas, 71,1% del cambio neto. Esta concentración debe mostrarse como driver observado, no como prueba causal.

### 14.2 Principales caídas junio-julio

| Descripción | PUSHER | Junio | Julio | Variación |
|---|---|---:|---:|---:|
| GNP | PUSHER 2 | 411 | 341 | -70 |
| CAV PEREIRA LA REBECA | Sin asignar | 8 | 2 | -6 |
| CAV BOGOTA KENNEDY | Sin asignar | 5 | 0 | -5 |
| CAV CUCUTA AV GRAN COLOMBIA | Sin asignar | 12 | 7 | -5 |
| CAV GIRARDOT | Sin asignar | 6 | 1 | -5 |
| TEAM COMUNICACIONES | Sin asignar | 9 | 4 | -5 |
| CAPITALS TELECOM BPO | PUSHER 2 | 16 | 12 | -4 |

El resultado agregado de PUSHER 2 oculta heterogeneidad: GNP y CAPITALS caen mientras ATENTO, ONE CONTACT, BRM, ALMAEXPERIENCE SAS e INTERACTIVO crecen.

## 15. Validación de los seis supuestos

| # | Supuesto | Clasificación | Evidencia y límite |
|---:|---|---|---|
| 1 | Abril, mayo y junio muestran descenso | **Confirmado** | 4.190 → 4.171 → 3.700; los dos cambios mensuales son negativos |
| 2 | Julio muestra recuperación | **Confirmado** | 4.518; +818 vs junio y +12,4% vs promedio abril-junio |
| 3 | PUSHER 2 explica principalmente el incremento de julio | **Parcialmente confirmado** | Aporta +572, 69,9% del cambio neto; también crecen PUSHER 1 (+164) y sin asignar (+82). Es contribución observada, no causalidad |
| 4 | `ALTAS` es la métrica correcta | **Confirmado** | Cantidad positiva, filas con valores >1, pivotes con `Suma de ALTAS` y conciliación exacta de julio |
| 5 | `Tabla1_2 (2)` es la fuente adecuada | **Confirmado** | Contiene `Insumo2`, origen del caché de pivotes; coincide con `Tabla1` y agrega dos campos |
| 6 | `INFORME ALTAS MULTIASISTENCIAS` permite conciliar la lógica | **Confirmado** | Su pivote regional suma 4.518 en julio, igual al detalle; sirve como referencia, no como fuente de ingesta |

La fuente no permite demostrar que la incorporación de PUSHER 2 causó la recuperación. Solo permite observar simultaneidad temporal, participación y contribución aritmética.

## 16. Análisis de los mockups

### 16.1 Estructura y jerarquía reutilizable

Ambos mockups proponen una composición ejecutiva adecuada:

- Encabezado con identidad Connect, título, subtítulo y navegación a Home.
- Panel de filtros lateral o superior.
- Primera fila de KPI con altas, variación, promedio, mejor mes, cobertura y mensaje clave.
- Tendencia mensual como visual principal.
- Drivers positivos y negativos de junio a julio.
- Comparativo antes/después y participación PUSHER.
- Ranking o detalle por aliado.
- Resumen ejecutivo y nota metodológica visible.

Conviene replicar la jerarquía, el uso de naranja como acento, fondo blanco, tipografía oscura, tarjetas con aire visual, señales verde/rojo y una narrativa central basada en abril-julio.

### 16.2 Elementos que deben adaptarse

- Sustituir `Persona 1`/`Persona 2` por `PUSHER 1`/`PUSHER 2` en todas las etiquetas.
- Evitar frases causales como “demostrando que muevo más las ventas” o “tú mueves más las ventas”. Usar “variación observada”, “contribución al cambio” y “señal asociada al periodo”.
- El segundo mockup representa siete aliados de PUSHER 2; la regla vigente tiene nueve, al incluir dos CAV adicionales. El diseño debe calcular sobre los nueve o permitir filtrar la cobertura.
- La serie de PUSHER 2 de siete aliados que muestran los mockups concilia con los datos: 3.082, 2.339, 2.562, 2.224, 1.971, 1.851 y 2.424. No equivale al total de los nueve aliados ni al total general.
- Algunas cifras acumuladas del primer mockup no concilian con sus propias series mensuales; por tanto, ninguna cifra ilustrativa debe copiarse al reporte.
- El total general incluye 151 aliados sin clasificación. Una comparación PUSHER 1 vs PUSHER 2 debe mostrar la cobertura o incluir `Sin asignar`; no debe presentar ambas PUSHER como 100% del universo si no lo son.
- La matriz debe limitarse a aliado, PUSHER y métricas agregadas; no debe incluir jefe, especialista ni asesor.
- La nota metodológica debe advertir que la asociación temporal no implica causalidad.

## 17. Objetos reutilizables del modelo

| Objeto actual | Reutilización propuesta | Condición |
|---|---|---|
| `Dim_Calendario` | Relacionar con `FechaAlta` | Ampliar su consulta para considerar la nueva fuente y marcarla como tabla de fechas |
| Home | Agregar navegación a la página nueva | Mantener Home como página activa y no saturar el lienzo |
| Patrones de botones y zonas de interacción | Reutilizar estilos y comportamiento | Crear objetos nuevos, no retargetear páginas existentes |
| Tema Connect | Reutilizar colores y tipografía | Mantener naranja, blanco y gris/negro |
| `RutaCarpetaData` | Reutilizar como parámetro raíz | Construir la ruta relativa a `Informe de Altas` |
| Patrón `Base_*` → `*_Limpio` → `Fact_*` | Reutilizar | Evitar dependencia directa de la hoja visible |
| Tablas de medidas existentes | No modificar | Crear `_Medidas Altas` o `_Medidas Comercial` según decisión del plan |
| `Dim_CallCenter` | No reutilizar directamente | Requiere catálogo/homologación; preferir `Dim_Aliado` |

## 18. Brechas de implementación

1. No existe consulta para el archivo de altas.
2. No existe `Fact_AltasTeResuelve`.
3. No existe `Dim_Aliado` ni una tabla gobernada de clasificación PUSHER.
4. No existe regla explícita para excluir agosto parcial o seleccionar el cierre.
5. No existen medidas comerciales ni tabla de medidas correspondiente.
6. No existe página de gestión comercial ni navegación desde Home.
7. Los 151 aliados sin asignar requieren una salida de control.
8. Falta confirmar si los dos CAV adicionales de PUSHER 2 deben formar parte del mensaje ejecutivo o solo de la clasificación completa.
9. Auto date/time permanece activo y crea tablas locales redundantes.
10. La publicación pública exige excluir columnas personales y revisar el contenido antes de republicar.

## 19. Impacto en Power Query

La futura implementación deberá:

- Crear `Base_AltasTeResuelve` desde la tabla `Insumo2`.
- Validar existencia del archivo, tabla y columnas `ALTAS`, `FECHA_ALTA`, `MES` y `DESCRIPCION`.
- Crear `AltasTeResuelve_Limpio` con nombres técnicos, tipos explícitos y normalización `Trim`/`Clean`/mayúsculas.
- Aplicar una regla de corte explícita. Para el cierre actual, usar `MES <= 202607` o un parámetro de periodo; no depender del nombre del archivo.
- Crear o combinar una tabla `Map_PusherAliado` visible y auditable con coincidencias exactas y categoría `Sin asignar`.
- Generar una consulta de control para aliados no clasificados.
- Eliminar de la tabla final `JEFE`, `ESPECIALISTA` y `ASESOR`; conservarlos solo en staging deshabilitado si existe una necesidad técnica aprobada.
- Mantener `ALTAS` como número entero y `FECHA_ALTA` como fecha.
- Registrar nulos y transformaciones sin imprimir registros personales.

## 20. Impacto en el modelo semántico

Modelo recomendado:

- `Fact_AltasTeResuelve`: una fila por combinación agregada del origen, con `Altas`, `FechaAlta`, `Mes`, `Descripcion` y atributos comerciales necesarios.
- `Dim_Aliado`: una fila por `Descripcion` normalizada; incluir `Pusher` o relacionarse con `Map_PusherAliado` según el gobierno elegido.
- `Dim_Calendario`: reutilizada con relación uno a muchos y filtro unidireccional.
- `Map_PusherAliado`: preferiblemente tabla de mapeo gobernada. Si cada aliado solo puede pertenecer a una PUSHER, puede incorporarse como atributo de `Dim_Aliado`; no se requiere puente muchos a muchos.
- `_Medidas Altas` o `_Medidas Comercial`: tabla exclusiva de medidas.

No crear relaciones entre hechos ni relaciones bidireccionales. La relación `Dim_CallCenter`–`Fact_AltasTeResuelve` debe posponerse hasta disponer de catálogo maestro; una relación por texto directa podría generar huérfanos y alterar el dominio de las encuestas.

## 21. Impacto en DAX

La convención técnica corregida separa nombre técnico y etiqueta visible. Los nombres técnicos no tendrán espacios, tildes ni caracteres especiales:

| Nombre técnico propuesto | Etiqueta visible |
|---|---|
| `Altas_Total` | Altas totales |
| `Altas_Mes_Anterior` | Altas mes anterior |
| `Diferencia_Altas_Mes` | Diferencia mensual |
| `Variacion_Altas_Mes_Pct` | Variación mensual % |
| `Crecimiento_Altas_Mes` | Crecimiento mensual |
| `Promedio_Altas_Abr_Jun` | Promedio abril-junio |
| `Altas_Julio` | Altas julio |
| `Variacion_Julio_Vs_Promedio_Abr_Jun_Pct` | Julio vs promedio abril-junio % |
| `Altas_Pusher_1` | Altas PUSHER 1 |
| `Altas_Pusher_2` | Altas PUSHER 2 |
| `Participacion_Pusher_1_Pct` | Participación PUSHER 1 % |
| `Participacion_Pusher_2_Pct` | Participación PUSHER 2 % |
| `Diferencia_Pusher_2_Vs_Pusher_1` | Diferencia PUSHER 2 vs PUSHER 1 |
| `Call_Center_Top_Altas` | Call center con más altas |
| `Tendencia_Altas` | Tendencia de altas |
| `Impacto_Observado_Pusher_2_Desde_Julio` | Impacto observado PUSHER 2 desde julio |

La variación debe usar `DIVIDE`; las medidas mensuales deben respetar contexto de filtro. La medida de impacto debe devolver un valor o texto descriptivo de asociación, nunca una afirmación causal.

## 22. Impacto en reporte y navegación

Se recomienda crear una página nueva `GestionComercialAltas`, visible como `Gestión comercial de altas`, y mantener intactas las siete páginas actuales. La página debe:

- Conservar el botón de regreso a Home y el patrón de navegación actual.
- Priorizar una tendencia enero-julio y una lectura específica abril-julio.
- Mostrar filtros de mes, aliado y PUSHER; región/canal solo si se valida su utilidad.
- Mostrar KPI de altas totales, variación, diferencia, altas PUSHER 2, participación PUSHER 2 y principal driver.
- Separar drivers positivos y negativos de junio-julio.
- Incluir matriz por aliado con PUSHER, altas mensuales, diferencia y variación.
- Mostrar cobertura de clasificación y categoría `Sin asignar`.
- Incluir nota metodológica sobre corte de datos y causalidad.

No se recomienda modificar Resumen ejecutivo ni Detalle por call center en la primera versión. La integración transversal puede evaluarse después de validar la página nueva.

## 23. Riesgos técnicos, comerciales y de privacidad

### Técnicos

- Agosto parcial puede contaminar el cierre de julio.
- El nombre mensual del archivo puede cambiar y romper una ruta fija.
- `Dim_CallCenter` y `DESCRIPCION` no comparten un catálogo maestro.
- Auto date/time mantiene tablas locales redundantes.
- Power BI Desktop puede reescribir TMDL/JSON automáticamente.
- Las cifras de los mockups no deben tratarse como fuente de verdad.

### Comerciales

- El crecimiento se concentra en ATENTO y ONE CONTACT; no es homogéneo.
- GNP y CAPITALS caen en julio dentro de PUSHER 2.
- La cobertura PUSHER solo clasifica 16 de 167 descripciones.
- No existen variables operativas suficientes para atribuir causalidad.
- La serie de siete aliados del mockup no representa las nueve asignaciones vigentes ni el total general.

### Privacidad

- El origen contiene nombres de jefes, especialistas y asesores.
- El tablero visible del Excel incluye rankings nominales.
- El reporte se distribuye públicamente sin autenticación.
- La primera versión debe excluir columnas personales y rankings individuales.

## 24. Hallazgos fuera de alcance

Durante el preflight se detectó que la fuente local de Satisfacción de capacitaciones alcanza el 2026-07-28, mientras el slicer oficial persistido en `HEAD` conserva el rango hasta 2026-07-22. Este desfase es preexistente y no pertenece a Gestión Comercial. No se modificó la página `p14_satisfaccion_capacitaciones_v2` ni su slicer.

También permanece fuera de alcance la eliminación de Auto date/time y de las tres `LocalDateTable`. Debe planificarse como corrección técnica separada para evitar mezclar riesgos con la integración comercial.

## 25. Recomendación técnica

Avanzar a una Fase 2 de planificación detallada, con estas decisiones:

1. Crear una página nueva, no adaptar una existente.
2. Implementar `Fact_AltasTeResuelve`, `Dim_Aliado` y una clasificación PUSHER gobernada.
3. Reutilizar `Dim_Calendario`, Home, navegación y tema visual, sin reutilizar directamente `Dim_CallCenter`.
4. Cargar solo enero-julio para el cierre actual mediante un parámetro o regla explícita; documentar agosto parcial.
5. Excluir campos personales desde la tabla final.
6. Mostrar `Sin asignar` y cobertura de clasificación.
7. Implementar nombres técnicos con guion bajo y etiquetas visibles en español colombiano.
8. Validar cada medida contra el detalle y el pivote visible antes de diseñar visuales.
9. Evitar cualquier afirmación causal; presentar contribución observada y asociación temporal.

## 26. Criterios para autorizar la Fase 2

Autorizar el plan de implementación únicamente si se confirma:

- La regla de corte para archivos que incluyen periodos posteriores al cierre.
- Que los nueve aliados de PUSHER 2 y los siete de PUSHER 1 constituyen el catálogo vigente.
- Que los 151 valores restantes deben quedar como `Sin asignar` hasta nueva homologación.
- Que la página será agregada y no cargará `JEFE`, `ESPECIALISTA` ni `ASESOR`.
- Que se creará `Dim_Aliado` o equivalente en vez de ampliar directamente `Dim_CallCenter`.
- El nombre de la tabla de medidas comercial (`_Medidas Altas` o `_Medidas Comercial`).
- Que la nueva página se agregará a Home sin modificar las páginas funcionales existentes.
- Que la validación se hará primero en Power BI Desktop local antes de cualquier republicación pública.
- Que Auto date/time se tratará en una intervención separada o con un subpaso explícitamente autorizado.

## Anexo A. Catálogo agregado de `DESCRIPCION`

El valor exacto y el valor normalizado coinciden para las 167 descripciones después de aplicar únicamente limpieza de caracteres de control, espacios y mayúsculas. La tabla incluye totales agregados y no expone personas ni registros individuales.

| Valor exacto | Normalizado | PUSHER | Ene-jul | Abr | May | Jun | Jul | ? jul-jun |
|---|---|---|---:|---:|---:|---:|---:|---:|
| AIB | AIB | PUSHER 1 | 392 | 30 | 68 | 95 | 98 | +3 |
| ALMAEXPERIENCE SAS | ALMAEXPERIENCE SAS | PUSHER 2 | 187 | 16 | 22 | 26 | 55 | +29 |
| ALTYCOM | ALTYCOM | Sin asignar | 5 | 1 | 0 | 0 | 0 | +0 |
| ANCLA TELECOMUNICACIONES | ANCLA TELECOMUNICACIONES | Sin asignar | 14 | 1 | 3 | 4 | 0 | -4 |
| ARCOS | ARCOS | Sin asignar | 1 | 0 | 1 | 0 | 0 | +0 |
| ASISTE ING | ASISTE ING | PUSHER 1 | 13 | 1 | 1 | 0 | 8 | +8 |
| ATENTO | ATENTO | PUSHER 2 | 7791 | 1109 | 1123 | 1014 | 1395 | +381 |
| BPO GLOBAL SERVICES | BPO GLOBAL SERVICES | Sin asignar | 4 | 0 | 1 | 1 | 2 | +1 |
| BRM | BRM | PUSHER 2 | 142 | 6 | 8 | 20 | 50 | +30 |
| CAPITALS TELECOM BPO | CAPITALS TELECOM BPO | PUSHER 2 | 28 | 0 | 0 | 16 | 12 | -4 |
| CATALITICO | CATALITICO | Sin asignar | 4 | 0 | 0 | 0 | 4 | +4 |
| CAV ACACIAS | CAV ACACIAS | Sin asignar | 8 | 0 | 0 | 0 | 1 | +1 |
| CAV AGUACHICA | CAV AGUACHICA | Sin asignar | 1 | 0 | 0 | 0 | 1 | +1 |
| CAV APARTADO | CAV APARTADO | Sin asignar | 7 | 1 | 0 | 0 | 1 | +1 |
| CAV ARMENIA CENTRO | CAV ARMENIA CENTRO | Sin asignar | 7 | 0 | 1 | 0 | 4 | +4 |
| CAV ARMENIA PORTAL QUINDIO | CAV ARMENIA PORTAL QUINDIO | Sin asignar | 21 | 1 | 0 | 5 | 1 | -4 |
| CAV BARRANCABERMEJA | CAV BARRANCABERMEJA | Sin asignar | 16 | 0 | 3 | 2 | 0 | -2 |
| CAV BARRANQUILLA CENTRO | CAV BARRANQUILLA CENTRO | Sin asignar | 4 | 1 | 0 | 1 | 0 | -1 |
| CAV BARRANQUILLA METROPOLITANO | CAV BARRANQUILLA METROPOLITANO | Sin asignar | 6 | 0 | 0 | 0 | 1 | +1 |
| CAV BARRANQUILLA PARQUE ALEGRA | CAV BARRANQUILLA PARQUE ALEGRA | Sin asignar | 4 | 0 | 1 | 0 | 0 | +0 |
| CAV BARRANQUILLA PRADO | CAV BARRANQUILLA PRADO | Sin asignar | 25 | 1 | 1 | 2 | 0 | -2 |
| CAV BOGOTA ALAMOS | CAV BOGOTA ALAMOS | Sin asignar | 6 | 0 | 2 | 1 | 0 | -1 |
| CAV BOGOTA ANDINO | CAV BOGOTA ANDINO | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| CAV BOGOTA CALLE 140 | CAV BOGOTA CALLE 140 | Sin asignar | 27 | 2 | 0 | 3 | 2 | -1 |
| CAV BOGOTA CALLE 94 | CAV BOGOTA CALLE 94 | Sin asignar | 12 | 3 | 1 | 0 | 1 | +1 |
| CAV BOGOTA CENTRO MAYOR | CAV BOGOTA CENTRO MAYOR | PUSHER 1 | 26 | 4 | 5 | 4 | 1 | -3 |
| CAV BOGOTA CHAPINERO | CAV BOGOTA CHAPINERO | Sin asignar | 26 | 0 | 1 | 1 | 1 | +0 |
| CAV BOGOTA COLINA | CAV BOGOTA COLINA | Sin asignar | 5 | 0 | 2 | 0 | 1 | +1 |
| CAV BOGOTA CRA 7 | CAV BOGOTA CRA 7 | Sin asignar | 24 | 3 | 2 | 2 | 2 | +0 |
| CAV BOGOTA FLORESTA | CAV BOGOTA FLORESTA | Sin asignar | 2 | 1 | 0 | 0 | 1 | +1 |
| CAV BOGOTA FONTIBON | CAV BOGOTA FONTIBON | Sin asignar | 8 | 2 | 0 | 0 | 1 | +1 |
| CAV BOGOTA KENNEDY | CAV BOGOTA KENNEDY | Sin asignar | 37 | 5 | 0 | 5 | 0 | -5 |
| CAV BOGOTA LA FELICIDAD | CAV BOGOTA LA FELICIDAD | Sin asignar | 13 | 2 | 1 | 3 | 0 | -3 |
| CAV BOGOTA MALL PLAZA NQS | CAV BOGOTA MALL PLAZA NQS | Sin asignar | 7 | 0 | 0 | 0 | 2 | +2 |
| CAV BOGOTA PASEO DEL RIO | CAV BOGOTA PASEO DEL RIO | Sin asignar | 5 | 1 | 0 | 0 | 0 | +0 |
| CAV BOGOTA PLAZA CLARO | CAV BOGOTA PLAZA CLARO | Sin asignar | 42 | 4 | 7 | 6 | 4 | -2 |
| CAV BOGOTA PLAZA DE LAS AMERICAS | CAV BOGOTA PLAZA DE LAS AMERICAS | PUSHER 2 | 5 | 0 | 0 | 3 | 0 | -3 |
| CAV BOGOTA PLAZA IMPERIAL | CAV BOGOTA PLAZA IMPERIAL | PUSHER 2 | 41 | 2 | 5 | 3 | 5 | +2 |
| CAV BOGOTA RESTREPO | CAV BOGOTA RESTREPO | Sin asignar | 34 | 0 | 2 | 3 | 3 | +0 |
| CAV BOGOTA SANTAFE | CAV BOGOTA SANTAFE | Sin asignar | 19 | 1 | 0 | 2 | 1 | -1 |
| CAV BOGOTA TITAN PLAZA | CAV BOGOTA TITAN PLAZA | Sin asignar | 8 | 1 | 1 | 1 | 0 | -1 |
| CAV BOGOTA TOBERIN | CAV BOGOTA TOBERIN | Sin asignar | 24 | 0 | 2 | 2 | 5 | +3 |
| CAV BOGOTA UNICENTRO | CAV BOGOTA UNICENTRO | Sin asignar | 42 | 0 | 0 | 1 | 0 | -1 |
| CAV BOGOTA VENECIA | CAV BOGOTA VENECIA | Sin asignar | 8 | 1 | 0 | 1 | 3 | +2 |
| CAV BUCARAMANGA CABECERA | CAV BUCARAMANGA CABECERA | Sin asignar | 61 | 5 | 11 | 5 | 6 | +1 |
| CAV BUCARAMANGA OMNICENTRO | CAV BUCARAMANGA OMNICENTRO | Sin asignar | 8 | 1 | 2 | 0 | 2 | +2 |
| CAV BUENAVENTURA MALECON | CAV BUENAVENTURA MALECON | Sin asignar | 3 | 0 | 0 | 0 | 0 | +0 |
| CAV CALI CHIPICHAPE | CAV CALI CHIPICHAPE | Sin asignar | 19 | 6 | 2 | 2 | 3 | +1 |
| CAV CALI GRAN COMERCIO | CAV CALI GRAN COMERCIO | Sin asignar | 4 | 0 | 0 | 0 | 0 | +0 |
| CAV CALI LA ESTACION | CAV CALI LA ESTACION | Sin asignar | 2 | 0 | 0 | 1 | 0 | -1 |
| CAV CALI NORTE | CAV CALI NORTE | Sin asignar | 22 | 1 | 3 | 1 | 0 | -1 |
| CAV CALI SUR | CAV CALI SUR | Sin asignar | 13 | 0 | 2 | 1 | 1 | +0 |
| CAV CARTAGENA EJECUTIVOS | CAV CARTAGENA EJECUTIVOS | Sin asignar | 5 | 0 | 1 | 0 | 0 | +0 |
| CAV CARTAGO | CAV CARTAGO | Sin asignar | 8 | 1 | 0 | 0 | 2 | +2 |
| CAV CHIA FONTANAR | CAV CHIA FONTANAR | Sin asignar | 4 | 0 | 1 | 0 | 1 | +1 |
| CAV CHIA LA LIBERTAD | CAV CHIA LA LIBERTAD | Sin asignar | 31 | 1 | 2 | 3 | 1 | -2 |
| CAV CUCUTA AV GRAN COLOMBIA | CAV CUCUTA AV GRAN COLOMBIA | Sin asignar | 81 | 7 | 11 | 12 | 7 | -5 |
| CAV CUCUTA CENTRO | CAV CUCUTA CENTRO | Sin asignar | 62 | 3 | 4 | 6 | 9 | +3 |
| CAV DUITAMA | CAV DUITAMA | Sin asignar | 18 | 1 | 1 | 0 | 4 | +4 |
| CAV ENVIGADO VIVA | CAV ENVIGADO VIVA | Sin asignar | 7 | 0 | 1 | 1 | 0 | -1 |
| CAV FLORENCIA | CAV FLORENCIA | Sin asignar | 5 | 1 | 2 | 1 | 0 | -1 |
| CAV FLORIDABLANCA | CAV FLORIDABLANCA | Sin asignar | 28 | 3 | 0 | 1 | 3 | +2 |
| CAV FUSAGASUGA | CAV FUSAGASUGA | Sin asignar | 22 | 1 | 0 | 2 | 2 | +0 |
| CAV GIRARDOT | CAV GIRARDOT | Sin asignar | 32 | 1 | 6 | 6 | 1 | -5 |
| CAV IBAGUE CENTRO | CAV IBAGUE CENTRO | Sin asignar | 13 | 1 | 1 | 4 | 1 | -3 |
| CAV IBAGUE FONTAINEBLEAU | CAV IBAGUE FONTAINEBLEAU | Sin asignar | 39 | 4 | 2 | 5 | 3 | -2 |
| CAV MANIZALES CENTRO | CAV MANIZALES CENTRO | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| CAV MANIZALES EL CABLE | CAV MANIZALES EL CABLE | Sin asignar | 6 | 1 | 2 | 0 | 1 | +1 |
| CAV MEDELLIN AV COLOMBIA | CAV MEDELLIN AV COLOMBIA | Sin asignar | 19 | 2 | 1 | 0 | 2 | +2 |
| CAV MEDELLIN FLORIDA | CAV MEDELLIN FLORIDA | Sin asignar | 4 | 1 | 2 | 0 | 0 | +0 |
| CAV MEDELLIN LA CENTRAL | CAV MEDELLIN LA CENTRAL | Sin asignar | 3 | 0 | 0 | 1 | 0 | -1 |
| CAV MEDELLIN MAYORCA | CAV MEDELLIN MAYORCA | Sin asignar | 10 | 1 | 1 | 2 | 1 | -1 |
| CAV MEDELLIN PARQUE FABRICATO | CAV MEDELLIN PARQUE FABRICATO | Sin asignar | 8 | 0 | 1 | 0 | 0 | +0 |
| CAV MEDELLIN PREMIUM | CAV MEDELLIN PREMIUM | Sin asignar | 17 | 3 | 3 | 4 | 0 | -4 |
| CAV MEDELLIN PUERTA DEL NORTE | CAV MEDELLIN PUERTA DEL NORTE | Sin asignar | 20 | 4 | 0 | 1 | 3 | +2 |
| CAV MEDELLIN SANTAFE | CAV MEDELLIN SANTAFE | Sin asignar | 1 | 1 | 0 | 0 | 0 | +0 |
| CAV MOLINOS MEDELLIN | CAV MOLINOS MEDELLIN | Sin asignar | 18 | 11 | 3 | 2 | 0 | -2 |
| CAV MONTERIA ALAMEDA | CAV MONTERIA ALAMEDA | Sin asignar | 3 | 0 | 0 | 0 | 1 | +1 |
| CAV MOSQUERA | CAV MOSQUERA | Sin asignar | 10 | 1 | 0 | 0 | 0 | +0 |
| CAV NEIVA CENTRO | CAV NEIVA CENTRO | Sin asignar | 15 | 1 | 1 | 0 | 1 | +1 |
| CAV NEIVA SAN PEDRO | CAV NEIVA SAN PEDRO | Sin asignar | 13 | 1 | 3 | 1 | 4 | +3 |
| CAV PALMETO CALI | CAV PALMETO CALI | Sin asignar | 41 | 1 | 0 | 1 | 5 | +4 |
| CAV PALMIRA CENTRO | CAV PALMIRA CENTRO | Sin asignar | 29 | 1 | 2 | 4 | 9 | +5 |
| CAV PASTO | CAV PASTO | Sin asignar | 21 | 1 | 3 | 2 | 0 | -2 |
| CAV PEREIRA ESTACION CENTRAL | CAV PEREIRA ESTACION CENTRAL | Sin asignar | 65 | 9 | 7 | 8 | 8 | +0 |
| CAV PEREIRA LA REBECA | CAV PEREIRA LA REBECA | Sin asignar | 34 | 5 | 1 | 8 | 2 | -6 |
| CAV POPAYAN CAMPANARIO | CAV POPAYAN CAMPANARIO | Sin asignar | 13 | 3 | 3 | 1 | 3 | +2 |
| CAV POPAYAN PLAZA COLONIAL | CAV POPAYAN PLAZA COLONIAL | Sin asignar | 20 | 2 | 2 | 4 | 4 | +0 |
| CAV RIOHACHA | CAV RIOHACHA | Sin asignar | 1 | 0 | 1 | 0 | 0 | +0 |
| CAV RIONEGRO | CAV RIONEGRO | Sin asignar | 7 | 0 | 1 | 0 | 0 | +0 |
| CAV SANTA MARTA BUENAVISTA | CAV SANTA MARTA BUENAVISTA | Sin asignar | 4 | 0 | 1 | 0 | 3 | +3 |
| CAV SINCELEJO GUACARI | CAV SINCELEJO GUACARI | Sin asignar | 27 | 2 | 3 | 3 | 4 | +1 |
| CAV SOACHA MERCURIO II | CAV SOACHA MERCURIO II | Sin asignar | 20 | 3 | 1 | 2 | 3 | +1 |
| CAV SOGAMOSO | CAV SOGAMOSO | Sin asignar | 27 | 0 | 2 | 2 | 3 | +1 |
| CAV TULUA SALESIANO | CAV TULUA SALESIANO | Sin asignar | 15 | 4 | 3 | 3 | 1 | -2 |
| CAV VALLEDUPAR GUATAPURI | CAV VALLEDUPAR GUATAPURI | Sin asignar | 12 | 2 | 1 | 2 | 2 | +0 |
| CAV VALLEDUPAR MAYALES | CAV VALLEDUPAR MAYALES | Sin asignar | 15 | 3 | 0 | 2 | 0 | -2 |
| CAV VILLAVICENCIO ORIENTE | CAV VILLAVICENCIO ORIENTE | Sin asignar | 10 | 0 | 1 | 0 | 1 | +1 |
| CAV VILLAVICENCIO VILLA CENTRO | CAV VILLAVICENCIO VILLA CENTRO | Sin asignar | 20 | 2 | 1 | 0 | 1 | +1 |
| CAV VIVA TUNJA | CAV VIVA TUNJA | Sin asignar | 42 | 7 | 5 | 3 | 4 | +1 |
| CAV YOPAL | CAV YOPAL | Sin asignar | 53 | 3 | 3 | 0 | 2 | +2 |
| CAV ZIPAQUIRA | CAV ZIPAQUIRA | Sin asignar | 28 | 0 | 7 | 1 | 2 | +1 |
| CELLPLUS SAS | CELLPLUS SAS | Sin asignar | 8 | 2 | 0 | 0 | 2 | +2 |
| CELTEL SA | CELTEL SA | Sin asignar | 3 | 0 | 0 | 0 | 0 | +0 |
| CELULLANO | CELULLANO | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| CELUNION | CELUNION | Sin asignar | 1 | 0 | 0 | 1 | 0 | -1 |
| CINCO SAS | CINCO SAS | Sin asignar | 1 | 0 | 1 | 0 | 0 | +0 |
| COMNORTE | COMNORTE | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| COMTEL SAS | COMTEL SAS | Sin asignar | 9 | 1 | 4 | 2 | 1 | -1 |
| COMUNICACIONES COLOMBIA MOVIL | COMUNICACIONES COLOMBIA MOVIL | Sin asignar | 1 | 0 | 0 | 1 | 0 | -1 |
| CONECTIVIDAD MOVIL | CONECTIVIDAD MOVIL | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| CONEXIONES DIGITALES | CONEXIONES DIGITALES | Sin asignar | 3 | 1 | 0 | 0 | 1 | +1 |
| CONFEMOVIL SAS | CONFEMOVIL SAS | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| CONTACTMASTER BPO | CONTACTMASTER BPO | Sin asignar | 1 | 0 | 1 | 0 | 0 | +0 |
| COS | COS | PUSHER 1 | 6120 | 790 | 584 | 462 | 490 | +28 |
| CYM MOVIL COMUNICACIONES | CYM MOVIL COMUNICACIONES | Sin asignar | 10 | 0 | 1 | 0 | 0 | +0 |
| DIGITAL CONTACT CENTER | DIGITAL CONTACT CENTER | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| DISTRIBUCIONES DEL QUINDIO | DISTRIBUCIONES DEL QUINDIO | Sin asignar | 8 | 0 | 3 | 3 | 1 | -2 |
| FOTOSPORT SAS | FOTOSPORT SAS | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| FRONTERA CELULAR LTDA | FRONTERA CELULAR LTDA | Sin asignar | 2 | 0 | 0 | 0 | 1 | +1 |
| GALCOM | GALCOM | Sin asignar | 1 | 1 | 0 | 0 | 0 | +0 |
| GEDIMA | GEDIMA | Sin asignar | 7 | 3 | 1 | 0 | 0 | +0 |
| GLOBEXCALL | GLOBEXCALL | Sin asignar | 1 | 1 | 0 | 0 | 0 | +0 |
| GNP | GNP | PUSHER 2 | 4913 | 650 | 398 | 411 | 341 | -70 |
| ICELL SA | ICELL SA | Sin asignar | 8 | 0 | 3 | 1 | 0 | -1 |
| INTELIGENCE BUSSINES RECOVERY COL | INTELIGENCE BUSSINES RECOVERY COL | PUSHER 1 | 5223 | 592 | 829 | 597 | 688 | +91 |
| INTERACTIVO | INTERACTIVO | PUSHER 2 | 23 | 2 | 2 | 3 | 9 | +6 |
| INVERCELL DEL CARIBE | INVERCELL DEL CARIBE | Sin asignar | 4 | 0 | 0 | 0 | 0 | +0 |
| INVERSIONES ARAUJO | INVERSIONES ARAUJO | Sin asignar | 98 | 39 | 34 | 10 | 6 | -4 |
| INVERSIONES BMS SAS | INVERSIONES BMS SAS | Sin asignar | 2 | 0 | 2 | 0 | 0 | +0 |
| INVERSIONES GERA SAS | INVERSIONES GERA SAS | Sin asignar | 40 | 5 | 0 | 0 | 0 | +0 |
| INVERSIONES OBAMAR SAS | INVERSIONES OBAMAR SAS | Sin asignar | 7 | 0 | 1 | 0 | 0 | +0 |
| JESMAR | JESMAR | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| LEFCOM | LEFCOM | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| MC MULTICELL SAS | MC MULTICELL SAS | Sin asignar | 2 | 0 | 1 | 0 | 1 | +1 |
| MILLENIUM | MILLENIUM | PUSHER 1 | 426 | 55 | 74 | 33 | 59 | +26 |
| MOVILCO SAS | MOVILCO SAS | Sin asignar | 32 | 5 | 3 | 2 | 4 | +2 |
| NEXA | NEXA | Sin asignar | 10 | 0 | 0 | 0 | 0 | +0 |
| OCANAXEL LTDA | OCANAXEL LTDA | Sin asignar | 2 | 0 | 0 | 2 | 0 | -2 |
| OIN SAS | OIN SAS | Sin asignar | 6 | 4 | 0 | 0 | 1 | +1 |
| ONE CONTACT | ONE CONTACT | PUSHER 2 | 3369 | 441 | 418 | 361 | 562 | +201 |
| ONSALES | ONSALES | Sin asignar | 1 | 1 | 0 | 0 | 0 | +0 |
| OPERACIONES | OPERACIONES | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| RHANDOM SAS | RHANDOM SAS | Sin asignar | 60 | 13 | 13 | 3 | 3 | +0 |
| RIO COMUNICACIONES PUERTO BERRIO | RIO COMUNICACIONES PUERTO BERRIO | Sin asignar | 5 | 0 | 0 | 1 | 0 | -1 |
| RYL TELECOMUNICACIONES | RYL TELECOMUNICACIONES | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| SER COMUNICACIONES | SER COMUNICACIONES | Sin asignar | 2 | 0 | 1 | 0 | 0 | +0 |
| SMARTMOBILE SAS | SMARTMOBILE SAS | Sin asignar | 2 | 1 | 1 | 0 | 0 | +0 |
| SOUL CENTRO DE TECNOLOGIA SAS | SOUL CENTRO DE TECNOLOGIA SAS | Sin asignar | 6 | 1 | 0 | 1 | 0 | -1 |
| SPC COMERCIAL SAS | SPC COMERCIAL SAS | Sin asignar | 4 | 0 | 0 | 3 | 1 | -2 |
| TEAM COMUNICACIONES | TEAM COMUNICACIONES | Sin asignar | 120 | 21 | 24 | 9 | 4 | -5 |
| TECPHONE | TECPHONE | Sin asignar | 9 | 2 | 1 | 1 | 0 | -1 |
| TELCOS | TELCOS | Sin asignar | 1 | 0 | 1 | 0 | 0 | +0 |
| TEYTEL SAS | TEYTEL SAS | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| TIENDA BOGOTA ENSUEÑO | TIENDA BOGOTA ENSUEÑO | Sin asignar | 2 | 0 | 0 | 0 | 0 | +0 |
| TIENDA BOGOTA GRAN PLAZA BOSA | TIENDA BOGOTA GRAN PLAZA BOSA | Sin asignar | 2 | 1 | 0 | 0 | 0 | +0 |
| TIENDA BOGOTA HAYUELOS | TIENDA BOGOTA HAYUELOS | Sin asignar | 3 | 0 | 0 | 1 | 2 | +1 |
| TIENDA BOGOTA PORTAL 80 | TIENDA BOGOTA PORTAL 80 | Sin asignar | 28 | 2 | 2 | 1 | 1 | +0 |
| TIENDA PASTO | TIENDA PASTO | Sin asignar | 5 | 0 | 0 | 0 | 0 | +0 |
| TIENDA PEREIRA ARBOLEDA | TIENDA PEREIRA ARBOLEDA | Sin asignar | 1 | 0 | 0 | 0 | 0 | +0 |
| TIENDA SANTA MARTA | TIENDA SANTA MARTA | Sin asignar | 7 | 2 | 3 | 1 | 1 | +0 |
| TIENDA SOACHA VENTURA | TIENDA SOACHA VENTURA | Sin asignar | 1 | 0 | 1 | 0 | 0 | +0 |
| TRESUELVE HOGAR | TRESUELVE HOGAR | Sin asignar | 1169 | 107 | 188 | 315 | 318 | +3 |
| TRESUELVE MULTIASISTENCIA | TRESUELVE MULTIASISTENCIA | Sin asignar | 13 | 0 | 0 | 0 | 1 | +1 |
| UNO 27 | UNO 27 | Sin asignar | 1225 | 132 | 189 | 138 | 224 | +86 |
| VECTOR | VECTOR | PUSHER 1 | 21 | 1 | 2 | 2 | 13 | +11 |
| ZENIX SAS | ZENIX SAS | Sin asignar | 15 | 2 | 4 | 0 | 1 | +1 |

## Cierre de la Fase 1

Este análisis no modificó el PBIP, TMDL, Power Query, DAX, relaciones, páginas, navegación, mockups ni archivos de `Data`. La Fase 2 no fue iniciada.
