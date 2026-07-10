# Análisis de Impacto — Informe de Altas T Resuelve

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de documento | Análisis de impacto para evolución `v1.1` |
| Archivo analizado | `Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx` |
| Fecha del análisis | 2026-07-10 |
| Estado | Diagnóstico sin implementación |

## 1. Resumen ejecutivo

El archivo de altas es una fuente comercial nueva y relevante para una evolución `v1.1` del informe. A diferencia de las tres fuentes actuales, que son encuestas piloto, este libro contiene una base transaccional/agregada de altas T Resuelve con 23.015 filas y 29.366 altas acumuladas entre enero y el 5 de julio de 2026. El cierre de junio se identifica claramente con 3.700 altas.

La fuente puede aportar seguimiento comercial real por aliado, región, canal, producto, jefe, especialista y asesor. Sin embargo, no debe integrarse directamente sin una fase de diseño: contiene datos personales, hojas ocultas, pivotes, tablas duplicadas con distinto orden, campos anchos de periodo en el contexto de negocio y un archivo anidado en `Data/` que hoy no queda cubierto por el patrón `.gitignore` `Data/*.xlsx`.

La recomendación técnica es avanzar, pero no como ajuste menor: conviene crear un plan de implementación `v1.1`, definir una plantilla estándar mensual, corregir la exclusión de Excel anidados y acordar reglas de privacidad antes de publicar nuevos visuales.

## 2. Contexto de negocio recibido

El contexto indica cierre de ventas de junio, proyección de metas de julio, aliados de mayor volumen con tendencia a la baja, aliados nuevos con crecimiento y metas por especialista. También menciona una regla de negocio: la meta de julio se calcula con base en el promedio del año y un incremento del 30%.

Por privacidad, este documento no reproduce nombres de especialistas, asesores ni jefes recibidos en el mensaje o encontrados en el archivo. Se documenta la estructura, el riesgo y la lógica de integración.

## 3. Descripción del archivo analizado

El libro pesa aproximadamente 4,4 MB, fue modificado el 2026-07-09 10:50:33 y contiene cuatro hojas:

| Hoja | Estado | Filas x columnas | Contenido principal |
|---|---:|---:|---|
| `INFORME ALTAS MULTIASISTENCIAS` | Visible | 2.532 x 23 | Reporte armado con pivotes/resúmenes y rankings. |
| `Tablas_back` | Oculta | 33 x 14 | Tablas auxiliares de pivotes de junio. |
| `Tabla1_2 (2)` | Oculta | 23.016 x 18 | Tabla formal `Insumo2`, detalle de altas con columnas adicionales. |
| `Tabla1` | Oculta | 91.044 x 16, datos hasta fila 23.016 | Tabla formal `Insumo`, detalle base de altas. |

`Tabla1` e `Insumo2` contienen el mismo conjunto de registros en columnas comunes, pero `Insumo2` agrega `FECHA_ALTA - Copy.1` y `UNIDAD_NEGOCIO2`.

## 4. Estructura del Excel

### Tabla transaccional principal

La tabla `Insumo` tiene 23.015 filas útiles y estos encabezados:

`ALTAS`, `FECHA_ALTA`, `MES`, `DIVISION`, `CANAL`, `CANAL2`, `TIPO_PAQUETE`, `DESCRIPCION`, `DESCRIPCION2`, `TIPO`, `SUBTIPO`, `SERVICIO`, `JEFE`, `ESPECIALISTA`, `ASESOR`, `UNIDAD_NEGOCIO`.

Campos relevantes:

- Fecha/periodo: `FECHA_ALTA`, `MES`.
- Aliado/call center: `DESCRIPCION`, `DESCRIPCION2`.
- Organización: `DIVISION`, `CANAL`, `CANAL2`, `UNIDAD_NEGOCIO`.
- Producto: `TIPO_PAQUETE`, `TIPO`, `SUBTIPO`, `SERVICIO`.
- Personas: `JEFE`, `ESPECIALISTA`, `ASESOR`.
- Métrica: `ALTAS`.

### Tablas internas detectadas

- Hoja visible: título, fecha de corte, pivote por región de junio y ranking de asesores/especialistas/jefes.
- `Tablas_back`: pivotes auxiliares de junio por canal, servicio, tipo de paquete y día.
- `Tabla1`/`Tabla1_2 (2)`: tablas formales ocultas para análisis.

No se encontraron encabezados explícitos de metas como `Promedio`, `Crecimiento`, `% Rep2` o `Meta total año` dentro del libro. La tabla de metas del mensaje debe tratarse como contexto externo o como una tabla no incluida en este archivo.

## 5. Diagnóstico de calidad de datos

La base tiene tipado relativamente consistente:

- `ALTAS`: entero, sin nulos, con valores entre 1 y 41.
- `FECHA_ALTA`: fecha, sin nulos, rango 2026-01-01 a 2026-07-05.
- `MES`: entero `yyyymm`, sin nulos, enero a julio de 2026.
- `DESCRIPCION2`: 350 nulos.
- `TIPO`, `SUBTIPO` y `UNIDAD_NEGOCIO`: constantes, por lo que aportan poco como segmentadores si no se esperan más valores.

No se detectaron filas duplicadas exactas ni duplicados por la combinación de columnas excepto `ALTAS`. El punto clave es que una fila no siempre representa una única alta: 3.022 filas tienen `ALTAS > 1`. Por tanto, las medidas deben sumar `ALTAS`, no contar filas.

## 6. Datos sensibles o personales

El archivo contiene nombres reales en campos de jefe, especialista y asesor. La hoja visible además muestra rankings nominales. Esto eleva el riesgo porque el informe actual está publicado mediante enlace público sin autenticación.

Recomendación de gobierno: no publicar páginas con nombres individuales salvo autorización explícita. Para una versión pública, agregar solo análisis agregado por aliado, región, canal y producto. El análisis por especialista debe quedar restringido, anonimizado o publicado en un workspace con control de acceso.

## 7. Grano de la información

El grano recomendado para modelar `Tabla1` es:

> una fila = una combinación de fecha, periodo, región, canal, aliado, producto y responsables comerciales, con una cantidad de altas en `ALTAS`.

No es una fila por cliente ni necesariamente una fila por venta individual. No hay identificador único de alta, venta, cliente o transacción.

## 8. Validación de la lógica de metas

La lógica de metas no puede validarse contra columnas del archivo porque las tablas de metas no están en el libro. Sí se puede simular la regla sobre las altas históricas:

- `Promedio` = promedio mensual enero-junio.
- `Crecimiento` = `Promedio * 30%`.
- `Meta julio` = `Promedio + Crecimiento`.

Ejemplos calculados desde `Tabla1`:

| Aliado | Total ene-jun | Promedio ene-jun | Meta julio simulada |
|---|---:|---:|---:|
| ATENTO | 6.396 | 1.066,0 | 1.385,8 |
| COS | 5.632 | 938,7 | 1.220,3 |
| GNP | 4.572 | 762,0 | 990,6 |
| IBR | 4.535 | 755,8 | 982,6 |
| ONE CONTACT | 2.807 | 467,8 | 608,2 |

El campo `Meta total año` mencionado en el contexto requiere aclaración: por la regla descrita parece representar una meta mensual de julio, no una meta anual completa.

## 9. Análisis de aliados

### Aliados de alto volumen

Tendencia enero-junio calculada con `ALTAS`:

| Aliado | Ene | Feb | Mar | Abr | May | Jun | Jun vs Ene | Jun vs prom. ene-may |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| ATENTO | 1.237 | 903 | 1.010 | 1.109 | 1.123 | 1.014 | -18,0% | -5,8% |
| COS | 1.275 | 1.452 | 1.067 | 790 | 586 | 462 | -63,8% | -55,3% |
| ONE CONTACT | 640 | 471 | 476 | 441 | 418 | 361 | -43,6% | -26,2% |
| GNP | 1.148 | 933 | 1.032 | 650 | 398 | 411 | -64,2% | -50,6% |
| IBR | 892 | 785 | 840 | 592 | 829 | 597 | -33,1% | -24,2% |

El archivo sí permite validar una caída, pero no una única disminución promedio del 25% sin definir la base. Frente al promedio enero-mayo, IBR está cerca de -25%, One Contact también, pero COS y GNP caen mucho más y ATENTO menos.

### Aliados nuevos o en crecimiento

| Aliado | Total ene-jun | Observación |
|---|---:|---|
| VECTOR | 8 | Presencia mínima desde marzo. |
| ALMA EXP | 132 | Tendencia positiva moderada; junio supera el promedio ene-may. |
| ASISTE ING | 5 | Volumen muy bajo; no permite concluir crecimiento. |
| CAPITALS | 16 | Aparece en junio; requiere seguimiento posterior. |
| AQI | 0 | No se detectó por ese nombre en el archivo. Aparece `AIB` como aliado distinto; requiere confirmación si es el mismo. |

## 10. Posible modelo de datos propuesto

Opción recomendada para `v1.1`:

- `Fact_AltasTResuelve`: tabla de altas desde `Insumo`.
- `Fact_MetasAliado`: tabla de metas mensuales por aliado, si negocio entrega una fuente estructurada.
- `Fact_MetasEspecialista`: tabla de metas por especialista y aliado, con tratamiento de privacidad.
- `Dim_Aliado`: catálogo normalizado de aliados/call centers comerciales.
- `Dim_Calendario`: reutilizar la existente.
- `Dim_Canal`, `Dim_Region`, `Dim_Servicio`, `Dim_TipoPaquete`: dimensiones opcionales si se crean páginas comerciales.
- `Dim_Especialista`, `Dim_Asesor`, `Dim_Jefe`: solo si hay gobierno de acceso o anonimización.

## 11. Relación con el modelo actual

El modelo actual usa `Dim_CallCenter` como dimensión dinámica de encuestas. El nuevo archivo usa `DESCRIPCION`/`DESCRIPCION2` como aliado u operación comercial. Algunos valores coinciden conceptualmente con call center, pero no debe asumirse equivalencia total.

Se recomienda crear `Dim_Aliado` y una tabla de mapeo `Mapa_Aliado_CallCenter` si negocio confirma que ambos conceptos se deben comparar. No se recomienda mezclar directamente `DESCRIPCION` en la actual `Dim_CallCenter` sin catálogo maestro.

## 12. Posibles indicadores DAX

- `Total Altas = SUM(Fact_AltasTResuelve[Altas])`
- `Altas Junio`
- `Altas Año Acumulado`
- `Altas Julio Parcial`
- `Promedio Mensual Altas`
- `Crecimiento Esperado 30%`
- `Meta Julio`
- `Cumplimiento Meta`
- `Brecha Frente a Meta`
- `Variación Mensual`
- `Variación Junio vs Promedio`
- `Participación por Aliado`
- `Ranking Aliados`
- `Aliados en Caída`
- `Aliados en Crecimiento`
- `Meta por Especialista`
- `Cumplimiento por Especialista`

## 13. Posibles visuales o páginas

Páginas actuales impactadas:

- Home: KPIs comerciales agregados, solo si se evita saturación.
- Resumen ejecutivo: cierre comercial mensual y tendencia.
- Detalle por call center: comparación por aliado/call center, previa homologación.
- Notas metodológicas: fuente de altas, corte, cálculo de metas y privacidad.

Páginas nuevas sugeridas:

- `Altas T Resuelve`
- `Metas comerciales`
- `Aliados y tendencia`
- `Cierre junio y proyección julio`
- `Calidad vs Altas`

La página `Especialistas` solo debería existir en una versión restringida, no en publicación pública.

## 14. Impacto sobre Power Query

La integración requeriría:

- Leer desde carpeta `Data/Informe de Altas`.
- Usar una función de ingesta preparada para archivos mensuales.
- Elegir una sola tabla fuente entre `Insumo` e `Insumo2`.
- Despivotar cualquier tabla de metas que venga en formato ancho por mes.
- Normalizar aliados (`ALMAEXPERIENCE SAS` vs `ALMA EXPERIENCE SAS`, IBR, COS, CAPITALS).
- Excluir o anonimizar campos personales según decisión de gobierno.

## 15. Impacto sobre DAX

Requiere una nueva familia de medidas, idealmente `_Medidas Altas` o `_Medidas Comercial`. Las medidas existentes de calidad, capacitación y motivación no deben modificarse para la primera integración.

## 16. Impacto sobre relaciones

Relaciones probables:

- `Dim_Calendario[Fecha]` -> `Fact_AltasTResuelve[FechaAlta]`
- `Dim_Aliado[Aliado]` -> `Fact_AltasTResuelve[Aliado]`
- `Dim_Aliado[Aliado]` -> `Fact_MetasAliado[Aliado]`
- `Dim_Especialista[Especialista]` -> `Fact_MetasEspecialista[Especialista]`, solo si se permite análisis nominal.

Evitar relaciones bidireccionales y evitar relacionar hechos entre sí.

## 17. Impacto sobre documentación

Si se implementa, actualizar:

- `README.md`
- `Docs/01_modelo_datos.md`
- `Docs/02_catalogo_medidas_dax.md`
- `Docs/03_mapa_reporte_paginas_visuales.md`
- `Docs/04_fuentes_y_actualizacion_datos.md`
- `Docs/05_decisiones_limitaciones_pendientes.md`
- `Docs/06_publicacion_powerbi.md`
- Nuevo plan `Specs/05_plan_implementacion_v1_1_altas_t_resuelve.md`

## 18. Riesgos de privacidad y gobierno

- Nombres de asesores, especialistas y jefes.
- Metas individuales asociadas a personas.
- Ranking de desempeño individual.
- Publicación pública actual sin autenticación.
- Posible información comercial sensible por aliado.

Decisión requerida: definir si la versión pública puede mostrar solo agregado por aliado o si se migrará a un workspace con permisos.

## 19. Riesgos técnicos

- Varias tablas en un mismo libro.
- Hojas ocultas y pivotes.
- Tabla visible orientada a consumo humano, no a ingesta.
- Columnas por mes en la tabla de metas recibida por contexto.
- Nombres de aliados inconsistentes.
- Archivo mensual con nombre variable.
- Excel anidado no ignorado por el patrón actual `.gitignore`.

## 20. Decisiones requeridas de negocio

1. Confirmar catálogo oficial de aliados y equivalencias con call centers.
2. Confirmar si `AQI` corresponde a algún nombre distinto encontrado en el archivo.
3. Confirmar definición exacta de caída promedio del 25%.
4. Confirmar si `Meta total año` significa meta de julio o meta anual.
5. Confirmar si metas por especialista pueden publicarse.
6. Definir si se continuará con enlace público o acceso autenticado.
7. Definir plantilla estándar mensual para próximos cierres.

## 21. Recomendación técnica

Conviene integrar esta fuente como versión `v1.1`, pero no de inmediato sobre el PBIP. El siguiente paso debe ser un plan de implementación y una plantilla de fuente estándar. La fuente aporta valor alto porque conecta calidad/formación/motivación con resultado comercial, pero también introduce un nivel de sensibilidad superior al modelo actual.

## 22. Plan sugerido de implementación futura

1. Corregir exclusión de Excel anidados en `Data/`.
2. Definir contrato de archivo mensual.
3. Crear `Specs/05_plan_implementacion_v1_1_altas_t_resuelve.md`.
4. Implementar ingesta Power Query en staging.
5. Crear `Fact_AltasTResuelve` y `Dim_Aliado`.
6. Validar metas con fuente estructurada.
7. Crear medidas `_Medidas Altas`.
8. Diseñar páginas comerciales.
9. Validar privacidad antes de publicar.
10. Actualizar `Docs/` y publicar solo tras aprobación.

## 23. Criterios para continuar

Continuar si:

- Negocio confirma la definición de meta y caída.
- Se recibe o aprueba una fuente estructurada de metas.
- Se autoriza el tratamiento de datos personales o se decide anonimizar.
- Se corrige la regla de exclusión de Excel anidados.
- Se valida visualmente el archivo en Power BI Desktop antes de publicar.

No continuar si:

- La meta por especialista debe publicarse en un enlace público sin control de acceso.
- No hay claridad sobre la equivalencia aliado/call center.
- El archivo seguirá cambiando de estructura cada mes sin plantilla.

## 24. Cierre

Este análisis no modificó `PBI/`, PBIR, TMDL, Power Query, medidas DAX, relaciones, visuales ni archivos Excel. Solo se evaluó el impacto potencial de la nueva fuente para una evolución futura del proyecto.
