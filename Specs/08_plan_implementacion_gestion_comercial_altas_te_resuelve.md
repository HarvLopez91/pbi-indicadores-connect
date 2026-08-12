# Plan de implementación — Gestión comercial de altas Te Resuelve

- **Proyecto:** PBI_Indicadores
- **Fase:** 2 — Plan de implementación
- **Estado:** Propuesto para aprobación
- **Fecha:** 2026-08-11
**Antecedente:** `Specs/07_analisis_impacto_gestion_comercial_altas_te_resuelve.md`

## 1. Propósito y alcance

Este documento convierte el análisis aprobado de la Fase 1 en un plan técnico ejecutable, reversible y dividido en gates. La solución incorporará una página adicional para analizar las altas de Te Resuelve por periodo, aliado y PUSHER, sin modificar el dominio de las encuestas ni exponer información personal.

La regla oficial será:

```text
Altas = SUM(ALTAS)
```

La fuente operativa será la tabla formal `Insumo2`, ubicada en la hoja `Tabla1_2 (2)` del archivo de cierre julio. La hoja `INFORME ALTAS MULTIASISTENCIAS` se utilizará exclusivamente para conciliación.

Esta fase solo define el plan. No implementa Power Query, TMDL, DAX, relaciones, páginas, visuales ni navegación.

## 2. Decisiones de arquitectura

### 2.1 Objetos previstos

| Capa | Objeto | Decisión |
| --- | --- | --- |
| Parámetro | `Ruta_Informe_Altas` o integración con el parámetro de ruta existente | Centralizar la ubicación sin incrustar una ruta personal en la tabla final |
| Parámetro | `Periodo_Corte_Comercial` | Entero `AAAAMM`; valor inicial `202607`; identifica el último periodo oficial cerrado |
| Parámetro | `Fecha_Inicio_Gestion_Pusher_2` | Fecha `2026-07-01`; evita repetir una constante en varias consultas y medidas |
| Staging | `Base_AltasTeResuelve` | Leer `Insumo2`; carga deshabilitada; preservar trazabilidad de origen |
| Limpieza | `AltasTeResuelve_Limpio` | Tipos, nombres técnicos, privacidad, normalización y validaciones; carga deshabilitada |
| Mapeo | `Map_PusherAliado` | Catálogo gobernado de las 16 coincidencias exactas; carga deshabilitada |
| Control | `Control_Aliados_Sin_Clasificar` | Salida de calidad con los valores no clasificados; no se publica en el modelo |
| Dimensión | `Dim_Aliado` | Un aliado por `Descripcion` normalizada, con `Pusher` y estado de clasificación |
| Hecho | `Fact_AltasTeResuelve` | Grano transaccional de la fuente, sin `JEFE`, `ESPECIALISTA` ni `ASESOR` |
| Dimensión | `Dim_Calendario` | Reutilizar y ampliar su rango para incluir `FechaAlta` sin afectar las tres fuentes existentes |
| Medidas | `_Medidas_Altas` | Tabla exclusiva para las medidas comerciales |
| Reporte | `GestionComercialAltas` | Página adicional; nombre visible `Gestión comercial de altas` |

No se incorporarán los 167 valores comerciales a `Dim_CallCenter`. Una homologación entre el dominio comercial y el de encuestas requiere un catálogo maestro y queda fuera de alcance.

### 2.2 Interpretación temporal de PUSHER 2

La gestión de PUSHER 2 empieza el 1 de julio de 2026. Enero-junio no se presentará como gestión de PUSHER 2, sino como línea base histórica del portafolio actualmente asignado a PUSHER 2.

Se evaluaron tres ubicaciones para `Periodo_Gestion`:

| Alternativa | Ventajas | Riesgos | Decisión |
| --- | --- | --- | --- |
| Columna en el hecho | Fácil de filtrar | Duplica el mismo atributo en todas las filas y mezcla semántica temporal con el hecho | No recomendada |
| Medida | No aumenta columnas | No funciona como segmentador natural y complica títulos, matrices y navegación | Solo para narrativas derivadas |
| Atributo en `Dim_Calendario` | Único, reutilizable, filtrable y coherente con el modelo estrella | Requiere incorporar el parámetro a la consulta de calendario | **Recomendada** |

`Dim_Calendario[Periodo_Gestion]` tendrá conceptualmente:

- `Antes de gestion`, para fechas anteriores a `Fecha_Inicio_Gestion_Pusher_2`;
- `Desde inicio gestion`, desde el 2026-07-01.

El atributo indica tiempo, no autoría. Al combinarlo con `Pusher = PUSHER 2`, las etiquetas visibles dirán “línea base histórica del portafolio actualmente asignado a PUSHER 2” o “desde el inicio de la gestión”. No se atribuirá causalidad.

### 2.3 Periodos cerrados y periodos parciales

No se filtrará de forma permanente `MES > 202607`. Se cargarán todos los periodos disponibles, incluido agosto parcial, y se aplicará este diseño:

1. `Periodo_Corte_Comercial`, entero `AAAAMM`, identifica el último mes aprobado como cierre oficial; inicialmente `202607`.
2. `Dim_Calendario[Estado_Periodo]` clasifica los meses como `Cerrado`, `En curso` o `Posterior al corte sin validar` según el parámetro y el máximo periodo cargado.
3. `Dim_Calendario[Es_Periodo_Comparable]` será verdadero hasta el corte oficial y falso después.
4. La página abrirá con periodos comparables; cualquier dato posterior permanecerá cargado y podrá verse en una sección identificada como parcial.
5. Las medidas mensuales oficiales devolverán vacío o una advertencia cuando se intente comparar un mes parcial con uno completo, salvo una medida expresamente diseñada para seguimiento intrames.
6. Al cerrar agosto, solo se actualizará el archivo y `Periodo_Corte_Comercial` a `202608`; no se reconstruirá el modelo.

| Opción | Ventaja | Riesgo |
| --- | --- | --- |
| Eliminar meses posteriores al corte | Simplifica resultados iniciales | Pierde trazabilidad y obliga a cambiar la consulta cada mes; descartada |
| Solo parámetro de corte | Fácil de mantener | Por sí solo no comunica el estado en visuales |
| Solo bandera calculada con la fecha actual | Automática | Confunde disponibilidad con cierre oficial |
| Parámetro + estado + guardas DAX | Conserva datos, comunica parcialidad y permite actualizaciones | Exige validar el parámetro en cada refresh; **recomendada** |

El refresh deberá fallar o emitir una alerta si el corte no existe en la fuente, si es posterior al máximo periodo o si aparecen varios meses posteriores no validados.

### 2.4 Mapeo de aliados y PUSHER

`Descripcion` se normalizará con `Text.Trim`, `Text.Clean` y mayúsculas. El mapeo se hará por igualdad exacta; no habrá equivalencias aproximadas.

La arquitectura recomendada es:

1. `Map_PusherAliado`: consulta pequeña y gobernada con los 16 aliados aprobados y su PUSHER; no se carga al modelo.
2. `Dim_Aliado`: catálogo distinto de valores normalizados de la fuente, unido por la izquierda con el mapa.
3. Todo valor sin coincidencia recibe `Pusher = Sin asignar` y `Estado_Clasificacion = Pendiente`.
4. `Control_Aliados_Sin_Clasificar`: referencia de la dimensión con los 151 valores iniciales no clasificados, conteos y altas agregadas; carga deshabilitada y sin datos nominales.
5. `Cobertura_Clasificacion_Pusher_Pct`: porcentaje de altas asociadas a aliados clasificados sobre las altas totales en el contexto activo. Se podrá complementar con una cobertura por cantidad de aliados para control técnico.

Este patrón mantiene una sola relación de negocio (`Dim_Aliado` a hecho), evita una tabla puente innecesaria y permite editar el catálogo sin alterar el hecho. Cualquier nueva equivalencia exigirá aprobación funcional y coincidencia explícita.

### 2.5 Privacidad y publicación pública

`JEFE`, `ESPECIALISTA` y `ASESOR` podrán existir solo en la lectura temporal del staging si el conector los recibe, pero se eliminarán en `AltasTeResuelve_Limpio` antes de construir la tabla final. No estarán en `Fact_AltasTeResuelve`, `Dim_Aliado`, TMDL, visuales, tooltips, drillthrough ni exportaciones.

Antes de cerrar GC-8 se inspeccionarán todas las columnas cargadas y se hará una revisión manual en Power BI Desktop. Esta iniciativa no modifica otras páginas que puedan contener información individual; el riesgo general del enlace público se mantiene registrado.

### 2.6 Modelo estrella y relaciones

| Desde | Hacia | Cardinalidad | Filtro | Validación |
| --- | --- | --- | --- | --- |
| `Dim_Calendario[Fecha]` | `Fact_AltasTeResuelve[FechaAlta]` | 1:* | Unidireccional | Sin fechas huérfanas; tabla marcada como calendario |
| `Dim_Aliado[AliadoKey]` | `Fact_AltasTeResuelve[AliadoKey]` | 1:* | Unidireccional | Clave única y no nula en dimensión; cero hechos huérfanos |

No se crearán relaciones entre hechos, bidireccionales ni muchos a muchos. `AliadoKey` será la descripción normalizada estable; se ocultará al usuario. No se usará un índice que pueda cambiar entre refreshes.

`Dim_Calendario` conservará sus tres consumidores actuales. Su rango se calculará con el mínimo y máximo conjunto de las fuentes existentes y `Fact_AltasTeResuelve`, con una regla de límites documentada. Se ejecutarán conteos y pruebas de relaciones antes y después para detectar regresiones.

Auto date/time y las tres `LocalDateTable` son deuda técnica preexistente. Se verificará que la nueva relación utilice exclusivamente `Dim_Calendario`; no se desactivará Auto date/time ni se eliminarán tablas locales en esta iniciativa. Si bloquean el modelo, se abrirá un gate independiente y se pedirá autorización antes de intervenir.

## 3. Convenciones técnicas y de presentación

- Tablas: `Fact_`, `Dim_`, `Map_` y `_Medidas` según el patrón existente.
- Medidas: palabras separadas por guion bajo, sin tildes, eñes ni caracteres especiales.
- Columnas nuevas: nombres técnicos sin espacios ni caracteres especiales y coherentes con el PBIP.
- Etiquetas visibles: español colombiano natural, con tildes.
- Porcentajes: proporciones entre 0 y 1, formateadas como porcentaje; nunca volver a dividir entre 100.
- El término visible será `PUSHER`; no se usará “persona”.
- La narrativa distinguirá variación observada, asociación temporal, contribución y causalidad.

## 4. Catálogo planificado de medidas

Todas las medidas vivirán en `_Medidas_Altas`. Las fórmulas son pseudocódigo y deberán ajustarse a los nombres físicos confirmados. Las medidas de tiempo exigirán un contexto mensual inequívoco; cuando no exista, devolverán `BLANK()` o una etiqueta neutra. Toda división usará `DIVIDE(..., ..., BLANK())`.

| Medida técnica | Objetivo y definición funcional | DAX o pseudocódigo | Contexto, filtros y validación |
| --- | --- | --- | --- |
| `Altas_Total` | Sumar las altas del contexto activo. | `SUM(Fact_AltasTeResuelve[Altas])` | Respeta periodo, aliado y PUSHER. Conciliar total del archivo, 33.295 enero-julio y 4.518 julio. |
| `Altas_Mes_Anterior` | Obtener el total del mes calendario anterior comparable. | `CALCULATE([Altas_Total], DATEADD(Dim_Calendario[Fecha], -1, MONTH))` con guarda de mes cerrado | Un único mes; vacío si el periodo previo no existe o la comparación es parcial. Validar enero-julio contra Excel. |
| `Diferencia_Altas_Mes` | Cambio absoluto contra el mes anterior. | `[Altas_Total] - [Altas_Mes_Anterior]` | Mantiene filtros de aliado/PUSHER. Validar junio-julio = 818. |
| `Variacion_Altas_Mes_Pct` | Cambio relativo mensual. | `DIVIDE([Diferencia_Altas_Mes], [Altas_Mes_Anterior], BLANK())` | Vacío con denominador cero. Validar junio-julio = 22,1 %. |
| `Crecimiento_Altas_Mes` | Etiqueta de dirección del cambio. | `SWITCH(TRUE(), delta>0,"Crecimiento",delta<0,"Decrecimiento",delta=0,"Sin cambio",BLANK())` | No agrega causalidad. Validar estados enero-julio. |
| `Promedio_Altas_Abr_Jun` | Línea base mensual promedio abril-junio de 2026. | `AVERAGEX({202604,202605,202606}, CALCULATE([Altas_Total], periodo actual))` | Fija el periodo, conserva aliado/PUSHER. Validar total = 4.020,33. |
| `Altas_Julio` | Total fijo de julio de 2026. | `CALCULATE([Altas_Total], Periodo=202607)` | Conserva filtros de aliado/PUSHER. Validar total = 4.518. |
| `Variacion_Julio_Vs_Promedio_Abr_Jun_Pct` | Comparar julio con la línea base mensual. | `DIVIDE([Altas_Julio]-[Promedio_Altas_Abr_Jun],[Promedio_Altas_Abr_Jun],BLANK())` | Validar total = 12,4 %; vacío con base cero. |
| `Altas_Pusher_1` | Volumen de aliados clasificados PUSHER 1. | `CALCULATE([Altas_Total], Dim_Aliado[Pusher]="PUSHER 1")` | Respeta fecha y aliado compatible. Validar julio = 1.357. |
| `Altas_Pusher_2` | Volumen del portafolio asignado a PUSHER 2. | `CALCULATE([Altas_Total], Dim_Aliado[Pusher]="PUSHER 2")` | Antes de julio se etiqueta como línea base histórica. Validar julio = 2.429. |
| `Participacion_Pusher_1_Pct` | Participación de PUSHER 1 sobre todo el universo, incluido `Sin asignar`. | `DIVIDE([Altas_Pusher_1], CALCULATE([Altas_Total], REMOVEFILTERS(Dim_Aliado[Pusher])), BLANK())` | Conserva periodo/aliado externo. Validar julio = 30,04 %. |
| `Participacion_Pusher_2_Pct` | Participación de PUSHER 2 sobre todo el universo. | Igual a la anterior filtrando PUSHER 2 | Validar julio = 53,76 %; no confundir con participación entre solo clasificados. |
| `Diferencia_Pusher_2_Vs_Pusher_1` | Diferencia descriptiva de volumen entre portafolios. | `[Altas_Pusher_2]-[Altas_Pusher_1]` | No se usa sola como evidencia de impacto. Validar por mes. |
| `Delta_Pusher_1_Mes` | Cambio absoluto mensual de PUSHER 1. | `[Altas_Pusher_1]-CALCULATE([Altas_Pusher_1], mes anterior)` | Solo meses comparables. Validar junio-julio = 164. |
| `Delta_Pusher_2_Mes` | Cambio absoluto mensual del portafolio PUSHER 2. | `[Altas_Pusher_2]-CALCULATE([Altas_Pusher_2], mes anterior)` | Junio es línea base; julio es primer mes de gestión. Validar = 572. |
| `Variacion_Pusher_1_Pct` | Variación relativa mensual de PUSHER 1. | `DIVIDE([Delta_Pusher_1_Mes], P1 mes anterior, BLANK())` | Vacío con base cero o periodo parcial. Validar junio-julio. |
| `Variacion_Pusher_2_Pct` | Variación relativa mensual del portafolio PUSHER 2. | `DIVIDE([Delta_Pusher_2_Mes], P2 mes anterior, BLANK())` | Etiquetar asociación temporal. Validar junio-julio. |
| `Contribucion_Cambio_Pusher_2_Pct` | Proporción del cambio neto mensual explicada aritméticamente por el delta de PUSHER 2. | `DIVIDE([Delta_Pusher_2_Mes],[Diferencia_Altas_Mes],BLANK())` | Puede superar 100 % o ser negativa por compensaciones; vacío si cambio neto es cero. Validar julio = 69,9 %. |
| `Call_Center_Top_Altas` | Devolver el aliado con más altas en el contexto. | `TOPN(1, VALUES(Dim_Aliado[Descripcion]), [Altas_Total], DESC, Descripcion, ASC)` + `CONCATENATEX` | Dinámico; desempate determinista. Validar contra ranking Excel del filtro activo. |
| `Call_Center_Mayor_Crecimiento` | Devolver el aliado con mayor delta absoluto. | `TOPN(1, ADDCOLUMNS(aliados,"Delta",[Diferencia_Altas_Mes]), [Delta], DESC, Descripcion, ASC)` | Excluir total y exigir dos meses comparables. Validar julio sin hardcodear ATENTO. |
| `Tendencia_Altas` | Etiquetar la pendiente del periodo seleccionado. | Pendiente de una regresión mensual (`LINESTX`) o, si no es viable, comparación del primer y último mes con mínimo tres puntos | Respeta filtros; nunca usa agosto parcial en tendencia cerrada. Validar signo contra serie Excel. |
| `Impacto_Observado_Pusher_2_Desde_Julio` | Medir el delta de julio del portafolio PUSHER 2 frente a su promedio abril-junio. | `P2 julio - AVERAGEX(abril-junio, [Altas_Pusher_2])` | Valor descriptivo, no causal; fijo al hito, conserva filtros de aliado. Validar contra Excel. |
| `Cobertura_Clasificacion_Pusher_Pct` | Porcentaje ponderado de altas cuyo aliado está clasificado como PUSHER 1 o 2. | `DIVIDE(CALCULATE([Altas_Total], Pusher<>"Sin asignar"), CALCULATE([Altas_Total], REMOVEFILTERS(Pusher)), BLANK())` | Respeta periodo; denominador incluye no clasificados. Validar totales por mes. |

Medidas auxiliares requeridas para drivers dinámicos:

- `Delta_Aliado_Mes`: variación absoluta por aliado;
- `Variacion_Aliado_Mes_Pct`: variación relativa por aliado;
- `Contribucion_Aliado_Cambio_Pct`: delta del aliado dividido por el cambio neto del universo;
- `Ranking_Driver_Positivo` y `Ranking_Driver_Negativo`: orden dinámico, con filtros de periodo, PUSHER y aliado.

Las medidas auxiliares permitirán mostrar aumentos y caídas sin hardcodear ATENTO, ONE CONTACT, GNP, CAPITALS ni otros hallazgos del archivo actual.

## 5. Marco analítico del comparativo PUSHER

La página mostrará siete perspectivas complementarias:

1. volumen absoluto por portafolio;
2. diferencia frente al mes anterior;
3. variación porcentual frente al mes anterior;
4. cambio frente a la línea base abril-junio;
5. contribución al cambio neto;
6. participación sobre todo el universo, incluido `Sin asignar`;
7. cobertura de clasificación.

Ninguna se presentará aisladamente como prueba causal. Las expresiones permitidas incluyen “recuperación observada”, “contribución al cambio”, “variación asociada al periodo”, “desde el inicio de la gestión” y “el portafolio actualmente asignado a PUSHER 2”. Se excluyen afirmaciones como “PUSHER 2 causó el crecimiento” o “PUSHER 2 movió X ventas”.

## 6. Diseño previsto de la página

La página técnica será `GestionComercialAltas` y su nombre visible, `Gestión comercial de altas`. Será adicional: no reemplazará ni eliminará páginas. Home continuará como página inicial y se mantendrá el patrón actual con botón `Volver a Home`.

Los mockups en `Assets/mockups/Mockup-gestion-comercial-1.png` y `Assets/mockups/Mockup-gestion-comercial-2.png` se usarán como referencia de composición, jerarquía, espacios e identidad Connect, nunca como fuente de cifras.

### 6.1 Contenido

- Segmentadores: mes/periodo, aliado o call center y PUSHER.
- KPI: altas totales, variación mensual, diferencia mensual, altas PUSHER 2, contribución de PUSHER 2 al cambio y aliado con mayor crecimiento.
- Tendencia mensual enero-julio y meses futuros, diferenciando periodos cerrados y parciales.
- Comparativo abril-mayo-junio-julio.
- Comparativo “antes de gestión” frente a “desde inicio de gestión”.
- PUSHER 1 frente a PUSHER 2 desde las siete perspectivas definidas.
- Drivers positivos y negativos dinámicos.
- Ranking de aliados y matriz por aliado.
- Indicador de cobertura de clasificación.
- Bloque de lectura ejecutiva y nota metodológica sobre asociación y causalidad.

Las cifras de agosto parcial se rotularán de forma visible y no compartirán comparativos oficiales con meses completos.

## 7. Plan por fases y gates

Cada fase comienza solo cuando la anterior cumple su criterio de cierre. Antes de cada commit se ejecutarán `git diff --check`, revisión de archivos, revisión de staging y validaciones específicas. Los datos fuente nunca se versionarán.

### GC-1 — Ingesta

**Objetivo:** establecer una lectura reproducible de `Insumo2` sin cargar todavía la tabla final.

**Prerrequisitos:** archivo vigente accesible; PBIP cerrado durante edición externa; ruta aprobada; copia de trabajo limpia.

**Archivos previstos:** expresiones Power Query del modelo semántico y documentación técnica mínima. No se versiona el Excel.

**Acciones:**

- definir `Ruta_Informe_Altas`, `Periodo_Corte_Comercial = 202607` y `Fecha_Inicio_Gestion_Pusher_2 = 2026-07-01`;
- crear `Base_AltasTeResuelve` con carga deshabilitada;
- leer por nombre la tabla formal `Insumo2` y corroborar que pertenece a `Tabla1_2 (2)`;
- validar presencia y tipo inicial de `ALTAS`, `DESCRIPCION`, fecha y `MES`;
- registrar nombre, fecha de modificación y huella de la fuente para trazabilidad, sin versionar su contenido.

**Validaciones automáticas:** existencia de archivo, objeto `Insumo2`, columnas obligatorias, al menos una fila, ausencia de errores de lectura y corte presente.

**Validaciones manuales:** abrir Desktop, configurar credenciales/ruta si aplica y confirmar vista previa sin errores.

**Riesgos:** ruta personal de OneDrive, cambio de nombre del objeto, bloqueo del archivo y deriva de esquema.

**Criterio de cierre:** staging refresca, sus validaciones pasan y permanece con carga deshabilitada.

**Commit sugerido:** `feat(data): agrega staging de altas te resuelve`. **Push:** sí, tras gate.
**Rollback:** revertir solo las expresiones/parámetros de GC-1 y confirmar que los hechos existentes refrescan.

### GC-2 — Limpieza y normalización

**Objetivo:** producir un conjunto tipado, seguro y preparado para el modelo.

**Prerrequisitos:** GC-1 aprobado.

**Archivos previstos:** expresiones Power Query y registro de validación de esquema.

**Acciones:**

- crear `AltasTeResuelve_Limpio` con carga deshabilitada;
- normalizar nombres técnicos, `Descripcion`, fechas, `MES` y `Altas`;
- comprobar que `MES` corresponde a `FechaAlta`;
- eliminar `JEFE`, `ESPECIALISTA` y `ASESOR` antes de cualquier salida cargada;
- mantener todos los periodos y derivar los estados mediante el corte;
- rechazar o aislar filas con fecha, descripción o altas inválidas, conservando conteos agregados de exclusión.

**Validaciones automáticas:** tipos, nulos en claves, correspondencia fecha-periodo, sumas antes/después, columnas sensibles ausentes y estados de periodo válidos.

**Validaciones manuales:** revisar reglas de exclusión y rotulación de agosto parcial.

**Riesgos:** pérdida de altas por coerción, fechas regionales y columnas nuevas en futuras versiones.

**Criterio de cierre:** suma conciliada, cero columnas sensibles en la salida y reglas de corte verificadas.

**Commit sugerido:** `feat(data): normaliza altas te resuelve`. **Push:** sí, tras gate.
**Rollback:** retirar la consulta limpia y volver al staging sin alterar otras consultas.

### GC-3 — Mapeo PUSHER y Dim_Aliado

**Objetivo:** gobernar la clasificación exacta y crear el dominio comercial independiente.

**Prerrequisitos:** GC-2 aprobado y catálogo de 16 aliados confirmado.

**Archivos previstos:** expresiones Power Query, definición TMDL de `Dim_Aliado` y control agregado de no clasificados.

**Acciones:**

- crear `Map_PusherAliado` con las 16 coincidencias exactas;
- generar `Dim_Aliado` con `AliadoKey`, descripción visible, `Pusher` y `Estado_Clasificacion`;
- asignar los demás valores a `Sin asignar`, sin coincidencias aproximadas;
- crear `Control_Aliados_Sin_Clasificar` con carga deshabilitada;
- preparar el KPI ponderado de cobertura para GC-5.

**Validaciones automáticas:** clave única, 16 coincidencias iniciales, cero duplicados en mapa, 151 valores iniciales sin asignar y conciliación de altas por categoría.

**Validaciones manuales:** negocio aprueba cualquier cambio futuro del catálogo; revisar variantes sin forzar equivalencias.

**Riesgos:** nuevas grafías, cambio de portafolio y falsa equivalencia con call centers de encuestas.

**Criterio de cierre:** dimensión única, mapa exacto trazable y control de pendientes reproducible.

**Commit sugerido:** `feat(model): agrega dimension de aliados y mapeo pusher`. **Push:** sí.
**Rollback:** eliminar dimensión y mapa nuevos; no tocar `Dim_CallCenter`.

### GC-4 — Modelo semántico

**Objetivo:** incorporar el hecho comercial al modelo estrella.

**Prerrequisitos:** GC-3 aprobado y Desktop disponible para validación.

**Archivos previstos:** TMDL de `Fact_AltasTeResuelve`, `Dim_Aliado`, `Dim_Calendario`, relaciones y modelo.

**Acciones:**

- cargar `Fact_AltasTeResuelve` sin columnas sensibles;
- ampliar el rango de `Dim_Calendario` de forma conjunta con las fuentes existentes;
- agregar `Periodo_Gestion`, `Estado_Periodo` y `Es_Periodo_Comparable` a calendario;
- crear las dos relaciones 1:* unidireccionales;
- ocultar claves y campos técnicos;
- crear `_Medidas_Altas` vacía o mínima para recibir GC-5;
- verificar Auto date/time sin corregirlo.

**Validaciones automáticas:** unicidad, huérfanos, cardinalidad, dirección, rango calendario, ausencia de relaciones entre hechos y diff TMDL limitado.

**Validaciones manuales:** abrir PBIP, refrescar, revisar diagrama, marcar calendario si procede y confirmar que las páginas existentes funcionan.

**Riesgos:** regresión del calendario compartido, generación automática de TMDL/LocalDateTable y cambios colaterales de Desktop.

**Criterio de cierre:** modelo abre y refresca; relaciones válidas; fuentes actuales conservan resultados.

**Commit sugerido:** `feat(model): integra altas te resuelve al modelo estrella`. **Push:** sí.
**Rollback:** revertir objetos y relaciones nuevos, y restaurar únicamente el cambio controlado de calendario.

### GC-5 — Medidas DAX

**Objetivo:** implementar el catálogo de medidas base, temporales, PUSHER, impacto observado y drivers.

**Prerrequisitos:** GC-4 aprobado y totales del modelo conciliados.

**Archivos previstos:** TMDL de `_Medidas_Altas` y documentación técnica de medidas.

**Acciones:** implementar las 23 medidas obligatorias y auxiliares de drivers; asignar formatos, carpetas de visualización y descripciones; aplicar guardas de periodo parcial y división por cero.

**Validaciones automáticas:** sintaxis DAX, dependencias, formato, nombres técnicos, resultados de casos de prueba y ausencia de errores/infinitos.

**Validaciones manuales:** evaluar medidas en Desktop con filtros de mes, aliado y PUSHER; comprobar etiquetas y casos sin base.

**Riesgos:** contexto múltiple de meses, contribuciones superiores al 100 % por compensación y lectura causal de una medida descriptiva.

**Criterio de cierre:** todas las medidas responden a sus matrices de prueba y no comparan agosto parcial como mes cerrado.

**Commit sugerido:** `feat(dax): agrega medidas de gestion comercial de altas`. **Push:** sí.
**Rollback:** revertir exclusivamente `_Medidas_Altas` y sus referencias visuales aún inexistentes.

### GC-6 — Conciliación técnica

**Objetivo:** demostrar trazabilidad entre Excel, Power Query y DAX antes de diseñar la página.

**Prerrequisitos:** GC-5 aprobado.

**Archivos previstos:** script o matriz de validación agregada y evidencia técnica sin datos personales.

**Acciones:** comparar total del archivo, enero-julio, abril-julio, PUSHER, no clasificados y principales aliados; conciliar julio contra `INFORME ALTAS MULTIASISTENCIAS`; registrar agosto como parcial; mantener la diferencia histórica de 25 como riesgo de versión, sin ajustes artificiales.

**Validaciones automáticas:** diferencia cero para sumas de la fuente vigente y tolerancia documentada para proporciones; pruebas de junio-julio y julio contra promedio abril-junio.

**Validaciones manuales:** revisión cruzada de pivote visible del Excel y resultados de Desktop.

**Riesgos:** cambio silencioso del archivo, caché de refresh y divergencia de filtros entre pivote y detalle.

**Criterio de cierre:** matriz firmada con diferencias explicadas y cero discrepancias no justificadas.

**Commit sugerido:** `test: concilia altas te resuelve entre excel y power bi`. **Push:** sí.
**Rollback:** retirar artefactos de prueba; no alterar cifras para hacerlas coincidir.

### GC-7 — Construcción de página

**Objetivo:** crear la página adicional con composición ejecutiva y navegación consistente.

**Prerrequisitos:** GC-6 aprobado y mockups versionados.

**Archivos previstos:** definición PBIR de `GestionComercialAltas`, registro de páginas, recursos estrictamente necesarios y botón nuevo en Home.

**Acciones:** construir filtros, KPI, tendencias, comparativos, drivers, ranking, matriz, cobertura, lectura ejecutiva y nota metodológica; configurar interacciones; agregar navegación Home ↔ Gestión comercial; conservar Home como `activePageName`.

**Validaciones automáticas:** referencias de campos/medidas, IDs únicos, JSON válido, página registrada, Home activa y ausencia de campos sensibles.

**Validaciones manuales:** revisar diseño contra mockups, resoluciones, colores Connect, textos en español, filtros, tooltips e interacciones.

**Riesgos:** cambios automáticos de Desktop, saturación visual, contraste insuficiente y navegación rota.

**Criterio de cierre:** página funcional aprobada visualmente, sin reemplazar páginas y con navegación bidireccional.

**Commit sugerido:** `feat(report): crea pagina gestion comercial de altas`. **Push:** sí.
**Rollback:** eliminar solo la página y su botón de Home, manteniendo el modelo validado.

### GC-8 — Validación funcional y visual

**Objetivo:** probar el comportamiento integral y la seguridad de publicación.

**Prerrequisitos:** GC-7 aprobado.

**Archivos previstos:** matriz de pruebas y, solo si son necesarias, correcciones atómicas del PBIP.

**Acciones:** probar refresh, filtros cruzados, estados de periodo, medidas, drivers, navegación, narrativa y privacidad; inspeccionar exportación y tooltips; ejecutar regresión de las siete páginas previas.

**Validaciones automáticas:** diff, referencias rotas, columnas sensibles ausentes, medidas sin error y conteos de objetos.

**Validaciones manuales:** Desktop, filtros, interacciones, Home ↔ Gestión comercial, diseño, narrativa y revisión de ausencia de datos personales.

**Riesgos:** una validación visual no detectable estáticamente y cambios no relacionados generados por Desktop.

**Criterio de cierre:** acta de pruebas sin defectos críticos y aprobación funcional/visual del usuario.

**Commit sugerido:** `test(report): valida gestion comercial de altas`. **Push:** sí si genera evidencia o correcciones; si no hay cambios versionables, registrar el gate en GC-9/GC-10.
**Rollback:** revertir cada corrección fallida de forma aislada; conservar el último commit validado.

### GC-9 — Documentación

**Objetivo:** entregar una guía funcional y técnica comprensible para PUSHER 2 y líderes.

**Prerrequisitos:** GC-8 aprobado.

**Archivos previstos:** `Docs/GESTION_COMERCIAL_ALTAS_TE_RESUELVE.md` y referencias técnicas necesarias.

**Acciones:** documentar fuente, reglas, clasificación, KPI, filtros, periodos parciales, interpretación, causalidad, mantenimiento del corte y aliados sin asignar.

**Validaciones automáticas:** enlaces válidos, nombres coincidentes con el modelo y revisión ortográfica básica.

**Validaciones manuales:** usuario comercial confirma claridad y utilidad de la guía.

**Riesgos:** divergencia entre documentación y versión final.

**Criterio de cierre:** documentación aprobada y consistente con Desktop.

**Commit sugerido:** `docs: documenta gestion comercial de altas`. **Push:** sí.
**Rollback:** revertir solo el documento y corregirlo sin alterar el modelo.

### GC-10 — Cierre y publicación

**Objetivo:** cerrar técnicamente, versionar la evidencia final y ejecutar la publicación manual controlada.

**Prerrequisitos:** GC-9 aprobado, working tree conocido y autorización explícita para publicar.

**Archivos previstos:** `Specs/09_validacion_final_gestion_comercial_altas_te_resuelve.md` y ajustes finales aprobados.

**Acciones:** validar PBIP, refresh, conciliación, privacidad, Git y documentación; registrar SHA; publicar manualmente al servicio; comprobar el enlace público en navegación privada.

**Validaciones automáticas:** `git diff --check`, estado, historial, archivos versionados, ausencia de Excel/Data y consistencia estática del PBIP.

**Validaciones manuales:** refresh real, publicación, navegación privada, Home, filtros, visuales, narrativa y ausencia de datos personales.

**Riesgos:** caché del servicio, credenciales, enlace público y diferencia entre Desktop y artefacto publicado.

**Criterio de cierre:** documento final aprobado, repositorio limpio, remoto sincronizado y publicación validada por el usuario.

**Commit sugerido:** `docs: agrega validacion final gestion comercial altas`. **Push:** sí.
**Rollback:** restaurar la versión publicada anterior desde el servicio o republicar el último PBIP validado; revertir Git solo con un commit explícito.

## 8. Gates manuales del usuario

Codex realizará edición controlada, validaciones estáticas, conciliaciones agregadas, revisión de diffs y operaciones Git autorizadas. El usuario deberá intervenir en estos gates:

| Gate | Actividad manual | Momento |
| --- | --- | --- |
| M-1 | Abrir Power BI Desktop cuando se requiera refresh real | GC-1, GC-4 y GC-8 |
| M-2 | Validar credenciales, ruta y actualización sin errores | GC-1 y GC-4 |
| M-3 | Revisar el diseño visual frente a los mockups | GC-7 y GC-8 |
| M-4 | Probar filtros, cruces e interacciones | GC-7 y GC-8 |
| M-5 | Validar navegación Home ↔ Gestión comercial y que Home siga inicial | GC-7 y GC-8 |
| M-6 | Confirmar que no haya datos personales en modelo, visuales ni exportación | GC-4 y GC-8 |
| M-7 | Aprobar la narrativa comercial y la interpretación no causal | GC-8 y GC-9 |
| M-8 | Republicar manualmente en Power BI Service | GC-10 |
| M-9 | Validar el enlace público en navegación privada | GC-10 |

No se abrirán simultáneamente los mismos archivos desde Power BI Desktop y un editor externo.

## 9. Matriz de trazabilidad y aceptación

| Requisito | Implementación prevista | Evidencia de aceptación |
| --- | --- | --- |
| `Altas = SUM(ALTAS)` | `Altas_Total` sobre el hecho | Total y meses iguales a Excel |
| Fuente `Insumo2` | Staging por objeto formal | Validación de esquema |
| Hoja de dashboard solo para conciliar | GC-6 | Julio 4.518 conciliado |
| Inicio PUSHER 2 en julio | Parámetro y `Periodo_Gestion` | Etiquetas temporales correctas |
| Agosto parcial | Corte, estado y guardas | No se compara como mes cerrado |
| 16 aliados clasificados | Mapa exacto gobernado | 16 coincidencias; restantes `Sin asignar` |
| Dominio comercial separado | `Dim_Aliado` | `Dim_CallCenter` sin cambios |
| Privacidad | Exclusión antes de carga | Columnas sensibles ausentes |
| Calendario compartido | Relación explícita | Sin huérfanos ni regresiones |
| Drivers dinámicos | Medidas por aliado | Ranking responde a filtros |
| Nueva página | `GestionComercialAltas` | Home intacta y navegación válida |
| Narrativa no causal | Textos y nota metodológica | Aprobación comercial |

## 10. Riesgos y controles transversales

| Riesgo | Control | Gate |
| --- | --- | --- |
| Diferencia histórica de 25 altas frente al archivo anterior | Mantenerla como riesgo de versión; usar el cierre julio vigente; no corregir cifras | GC-6 |
| Archivo cambia sin aviso | Registrar huella, fecha y métricas de control | GC-1/GC-6 |
| Mes parcial interpretado como cierre | Parámetro, estado, filtro predeterminado y guarda DAX | GC-2/GC-5/GC-8 |
| Mapeo incompleto | `Sin asignar`, control y KPI de cobertura | GC-3/GC-5 |
| Afirmación causal | Lenguaje permitido, nota y aprobación funcional | GC-7/GC-8 |
| Exposición personal en publicación pública | Eliminación antes de carga y revisión de exportación | GC-2/GC-8 |
| Regresión en calendario | Pruebas de tres fuentes actuales antes/después | GC-4 |
| Auto date/time y `LocalDateTable` | No intervenir; verificar uso exclusivo de calendario explícito; gate separado si bloquea | GC-4 |
| Cambios automáticos de Desktop | PBIP cerrado durante edición, diff por archivo y commits atómicos | Todas |
| Confusión con `Dim_CallCenter` | Mantener `Dim_Aliado`; homologación futura fuera de alcance | GC-3/GC-4 |

## 11. Exclusiones confirmadas

- No modificar `Dim_CallCenter` para incorporar aliados comerciales.
- No crear equivalencias aproximadas de `Descripcion`.
- No excluir físicamente agosto ni futuros periodos por una regla fija.
- No incluir `JEFE`, `ESPECIALISTA` ni `ASESOR` en el modelo publicado.
- No modificar ni eliminar páginas existentes.
- No corregir Auto date/time ni las `LocalDateTable` dentro de esta iniciativa.
- No modificar la página de Satisfacción; su fuente llega al 2026-07-28 y el slicer persistido en `HEAD` termina el 2026-07-22.
- No resolver artificialmente la diferencia histórica de 25 altas.
- No publicar ni versionar el Excel fuente.
- No afirmar causalidad.

## 12. Estrategia Git y rollback global

Cada GC tendrá un commit atómico después de sus validaciones y un push solo cuando cierre su gate. Antes de cada commit se confirmará que ningún archivo ajeno esté staged. Los cambios automáticos no funcionales de Power BI Desktop se separarán y no se mezclarán con la iniciativa.

El rollback preferido será por reversión lógica del commit de la fase afectada, nunca mediante limpieza destructiva del working tree. Si una fase falla, se conservará la última versión validada y no se iniciará la siguiente.

## 13. Criterios para autorizar GC-1

GC-1 podrá iniciar únicamente cuando el usuario apruebe:

1. `Periodo_Corte_Comercial = 202607` como corte inicial oficial.
2. `Fecha_Inicio_Gestion_Pusher_2 = 2026-07-01`.
3. `Periodo_Gestion` en `Dim_Calendario`.
4. `Map_PusherAliado` con carga deshabilitada y atributo consolidado en `Dim_Aliado`.
5. `Sin asignar` para los 151 valores sin coincidencia exacta inicial.
6. Cobertura ponderada por altas como definición del KPI principal.
7. Exclusión de `JEFE`, `ESPECIALISTA` y `ASESOR` antes de la carga final.
8. `_Medidas_Altas` como tabla comercial de medidas.
9. El catálogo de medidas y las guardas para periodos parciales.
10. Los gates manuales de Desktop, privacidad, narrativa y publicación.

## Cierre de la Fase 2

Este documento define el plan; no ejecuta GC-1 ni modifica Power Query, TMDL, DAX, relaciones, páginas, visuales, navegación, archivos de `Data` o la página de Satisfacción.
