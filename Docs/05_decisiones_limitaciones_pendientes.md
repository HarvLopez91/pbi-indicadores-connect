# Decisiones, limitaciones y pendientes

Registro consolidado de las decisiones de diseño tomadas durante la construcción del informe (Fases 1 a 18, más la iniciativa `SC-1` a `SC-9` de rediseño y reemplazo de Satisfacción de capacitaciones), cuáles son definitivas y cuáles son provisionales por depender de confirmación de negocio, más el estado actualizado de las dependencias D1–D9.

## 1. Decisiones de diseño tomadas (definitivas)

Estas decisiones ya están implementadas y no dependen de ninguna confirmación pendiente:

1. **Modelo en estrella con 3 tablas de hechos independientes**, sin unificarlas: cada encuesta tiene grano y preguntas distintas (Fase 7–9).
2. **`Dim_CallCenter` y `Dim_Jornada` dinámicas** (unión de valores distintos observados en los hechos), nunca listas fijas — un call center o jornada nuevo se incorpora solo con la próxima actualización (Fase 8).
3. **`Dim_Calendario` se extiende hasta `HOY()`**, no hasta la fecha máxima observada en los datos, para que el calendario se autoextienda en cada actualización (Fase 8).
4. **`"N/A"` en el checklist de calidad se trata como valor nulo, nunca como `0`** — evita penalizar preguntas no aplicables en los promedios (Fase 6).
5. **Fila con datos nulos en `Fact_SatisfaccionCapacitacion`**: se decidió **marcarla como `"Sin dato"`** en las columnas categóricas (`CallCenter`, `Jornada`, nombres, `Duracion`) en vez de excluirla del modelo (Fase 6).
6. **Comentarios abiertos**: variantes de "sin respuesta" (`N/A`, `NA`, `Nada`, `Ninguna`, `No`, `Sin observaciones`, vacío) unificadas al valor estándar `"Sin comentario"` (Fase 6).
7. **Columna de nombre de `Fact_MotivacionActividad`**: llega 100% vacía desde el origen y se **elimina explícitamente** en Power Query (no se conserva como columna fantasma) — la encuesta es, en la práctica, anónima (Fase 6).
8. **`% Calidad Promedio Provisional` se implementó como `BLANK()` explícito**, no se omitió la medida — deja visible en el modelo el placeholder de una medida que negocio aún no puede calcular correctamente (Fase 10).
9. **Tema visual con colores reales de marca** (`#F15B2B` naranja, `#002733` oscuro), obtenidos de un logo SVG a color entregado durante la Fase 12 — reemplazaron la paleta placeholder inicial (`#F37021`/`#1F1F1F`).
10. **Navegación reforzada con "hitzones"**: rectángulos transparentes superpuestos a cada tarjeta/botón de navegación, con el mismo `visualLink`, para que el clic funcione en toda el área visual y no solo en los sub-elementos individuales (corrección QA, `Outputs/28`).
11. **Segmentadores en modo menú desplegable (`Dropdown`)**, en la misma posición relativa en todas las páginas internas. Aplica a las **7 páginas oficiales**; la página oficial de Satisfacción de capacitaciones conserva la excepción documentada de Fecha en comportamiento visual `Between` — ver `DEC-4` en §3.
12. **Etiquetas de datos activas en los 8 gráficos** del informe, con estilo Connect (`#002733`, tamaño moderado).
13. **No se construyó una página separada "Detalle por asesor/líder"**: el desglose nominal por asesor queda cubierto dentro de `cl_tabla_asesor` (Calidad de llamadas). La tabla nominal de formador/líder de Satisfacción de capacitaciones fue retirada del PBIR activo en `SC-9`; el informe final mantiene **7 páginas**, no las 8 originalmente esbozadas en `Specs/01`/`Specs/02`.
14. **No se creó `Dim_Colaborador` maestro** con ID único — evaluado y descartado para esta primera implementación por falta de fuente de nómina/WFM; queda como iniciativa futura.
15. **No se implementó conexión automática a Google Forms/Sheets** — la ingesta sigue siendo manual vía los 3 archivos Excel exportados en `Data/`.

## 2. Decisiones provisionales (dependen de confirmación de negocio)

| Decisión provisional | Supuesto actual | Qué cambia si se confirma |
|---|---|---|
| Puntaje máximo por pregunta del checklist de calidad | No se implementó `% Calidad Promedio` real; la medida existe como `BLANK()` | Se podría calcular `% Calidad Promedio` real dividiendo el puntaje obtenido entre el máximo aplicable por rúbrica |
| Catálogo oficial de call centers/jornadas | Se usa el catálogo dinámico (unión de valores observados) | El catálogo dinámico seguiría funcionando igual; se podría añadir una validación cruzada contra la lista oficial para detectar errores de tipeo |
| Nombres estándar de líderes/formadores | Se aplicó una tabla de alias solo para las variantes de un líder ya detectadas en el diagnóstico inicial | Cualquier variante nueva de nombre que aparezca en futuras respuestas requiere ampliar manualmente la tabla de alias en Power Query |
| Colores/logo oficiales de marca | **Resuelta** desde la Fase 12 con un logo SVG a color real | No aplica — ya no es un supuesto, ver dependencia D6 abajo |

## 3. Decisiones de la iniciativa `SC-1`–`SC-9` (rediseño y reemplazo de Satisfacción de capacitaciones)

Estas decisiones aplican a la página oficial `p14_satisfaccion_capacitaciones_v2` (ver [Docs/03](03_mapa_reporte_paginas_visuales.md) §4), que reemplazó a la página original en `SC-9`.

### DEC-1 — Clave de capacitación única

**Decisión aplicada:** una sesión de capacitación se identifica mediante la combinación `Fecha + CallCenter + NombreFormador` (usada por la medida `Capacitaciones Realizadas`, ver [Docs/02](02_catalogo_medidas_dax.md)).

**Estado:** confirmada para esta implementación, pero **provisional desde el punto de vista de negocio** — pendiente de validación oficial o de que el origen entregue un identificador de sesión explícito. Ver dependencia `D9` abajo.

### DEC-2 — Tabla de detalle sin nombres personales

**Decisión aplicada:** en la página oficial, la tabla `sc_tabla_callcenter` (detalle por call center) **reemplaza** a la tabla nominal de formador/líder (`sc_tabla_formador`) de la página original.

**Efecto:** la tabla nominal de formador/líder deja de formar parte del informe activo. El respaldo queda disponible mediante Git en commits anteriores, no dentro del PBIR publicado.

### DEC-3 — Gráfico de Jornada retirado del lienzo principal

**Decisión aplicada:** el gráfico por Jornada (equivalente a `sc_chart_jornada` de la original) fue retirado del lienzo principal de la página `v2` durante el rediseño (`SC-5`).

**Estado:** no implementado actualmente. Queda como posible elemento futuro vía tooltip, drillthrough o página secundaria — ninguna de esas alternativas está construida hoy.

### DEC-4 — Comportamiento visual de Fecha (`Between`)

**Decisión aplicada:** se conserva en la página `v2` el comportamiento visual de rango (`Between`, dos casillas de fecha) del segmentador de Fecha, confirmado en `SC-7`.

**Detalle técnico:** el PBIR del visual (`sc_slicer_fecha`) declara `mode: 'Dropdown'`, pero Power BI Desktop renderiza la columna de fecha (`Dim_Calendario[Fecha]`) como control de rango con dos casillas independientemente de ese valor declarado. Esta observación **no se extiende automáticamente** a otras páginas ni segmentadores de fecha del informe — cada uno debe verificarse por separado si se cuestiona su comportamiento.

### SC-9 — Reemplazo definitivo de la página

**Decisión aplicada:** `p14_satisfaccion_capacitaciones_v2` reemplaza definitivamente a `p14_satisfaccion_capacitaciones` como página oficial de Satisfacción de capacitaciones.

**Implementación:** la página validada conserva su nombre técnico `p14_satisfaccion_capacitaciones_v2`, cambia su nombre visible a `Satisfacción de capacitaciones`, Home navega hacia ella y la carpeta original `p14_satisfaccion_capacitaciones` se retira del PBIR activo.

**Motivo:** Git conserva el respaldo histórico; mantener dentro del informe activo una página antigua con tabla nominal de formador/líder aumenta innecesariamente el riesgo de exposición al usar un enlace público sin autenticación.

**Límite:** esta decisión no cierra `D9`: la clave `Fecha + CallCenter + NombreFormador` sigue siendo provisional hasta que negocio defina o entregue un identificador oficial de sesión.

## 4. Estado actualizado de dependencias D1–D9 (`Specs/02` §4 y `SC-1`–`SC-7`)

| # | Dependencia | Estado original | Estado actual |
|---|---|---|---|
| D1 | Power BI Desktop con soporte PBIP + TMDL habilitado | Por confirmar en Fase 1 | **Resuelta** — confirmado en la Fase 1 y usado sin problemas en todas las fases posteriores |
| D2 | Archivos de `Data/` cerrados al actualizar | Riesgo conocido | **Gestionada, no eliminable** — es un riesgo operativo permanente mientras la fuente sea Excel local; mitigación documentada en [Docs/04_fuentes_y_actualizacion_datos.md](04_fuentes_y_actualizacion_datos.md) §3 |
| D3 | Rúbrica de puntaje máximo por pregunta de calidad | Pendiente | **Sigue pendiente** — `% Calidad Promedio Provisional` continúa en `BLANK()` |
| D4 | Catálogo oficial de call centers y jornadas | Pendiente | **Sigue pendiente** — se usa el catálogo dinámico como sustituto funcional |
| D5 | Nombres estándar de líderes/formadores | Pendiente | **Parcialmente resuelta** — tabla de alias aplicada para las variantes ya detectadas; confirmación oficial de negocio sigue pendiente para casos futuros |
| D6 | Logo oficial y HEX exactos de marca | Pendiente | **Resuelta** en la Fase 12 — logo real y colores reales (`#F15B2B`, `#002733`) aplicados en el tema |
| D7 | Versionamiento con Git antes de cambios estructurales | Pendiente de decisión | **Resuelta** — repositorio Git activo desde la Fase 2, todas las fases versionadas con commits descriptivos |
| D8 | Volumen de datos piloto no representativo | Confirmado en el diagnóstico | **Constatada y gestionada activamente**, no es un estado que se "resuelva": el informe comunica explícitamente la fase piloto en Home y en Notas metodológicas, con `n=` dinámico visible |
| D9 | Identificador oficial de sesión de capacitación (clave `Fecha + CallCenter + NombreFormador` usada por `Capacitaciones Realizadas`, `DEC-1`) | No existía — introducida en `SC-3` | **Mitigada provisionalmente / pendiente de definición oficial** — la clave compuesta funciona como sustituto operativo, pero puede sobre/subestimar el conteo real de sesiones si hay variantes de nombre sin alias o sesiones simultáneas del mismo formador el mismo día en el mismo call center. No se considera cerrada hasta que el origen entregue un identificador de sesión explícito |

## 5. Pendientes de negocio (requieren decisión fuera de este repositorio)

- **Rúbrica de puntaje máximo por pregunta** del checklist de calidad (D3) — bloquea `% Calidad Promedio Provisional`.
- **Catálogo oficial de call centers** vigentes (D4).
- **Confirmación oficial de alias de líderes/formadores** más allá de las variantes ya detectadas (D5).
- **Limpieza de las tablas automáticas de fecha (Auto Date/Time)**: técnicamente pendiente, no de negocio — requiere que el usuario deshabilite la detección automática de tabla de fechas desde Power BI Desktop y quite el bloque `variation` de cada `Fact_*`. Ver [Docs/01_modelo_datos.md](01_modelo_datos.md) §6.
- **Posible ajuste de `% Llamadas con Venta`**: sigue documentado como observación pendiente si continúa apareciendo en blanco en vez de `0,0%` en ciertos contextos de filtro. Alternativa propuesta (no aplicada): reescribir la medida con `SUMX` sobre una columna booleana en vez de `CALCULATE` + `COUNTROWS` con `IN`.
- **Posible medida `Fecha Corte Datos`**: propuesta en la Fase 16 (`Outputs/29`) como `MAX(Dim_Calendario[Fecha])`, para mostrar la fecha de corte de los datos en Notas metodológicas sin texto fijo. No implementada — pendiente de autorización explícita, ya que la instrucción del proyecto es no crear medidas DAX nuevas sin aprobación.
- **Gobierno de publicación**: el informe está publicado mediante un enlace de "Publicar en la Web" (`app.powerbi.com/view?r=...`), que **no requiere autenticación** para ser visto por cualquier persona con el enlace. Tras `SC-9`, la página oficial de Satisfacción de capacitaciones ya no contiene la tabla nominal de formador/líder; sin embargo, `cl_tabla_asesor` (Calidad de llamadas) conserva `NombreAsesor`. Se recomienda que negocio confirme si ese nivel de exposición pública es aceptable o si el informe debe migrarse a un espacio de trabajo de Power BI Service con control de acceso (ver [Docs/06_publicacion_powerbi.md](06_publicacion_powerbi.md)).
- **Identificador oficial de sesión de capacitación** (D9) — la clave provisional `Fecha + CallCenter + NombreFormador` usada por `Capacitaciones Realizadas` en la página `v2` puede sobre/subestimar el conteo real de sesiones (ver `DEC-1`).

## 6. Riesgos de mantenimiento

- **`Data/*.xlsx` no está versionado** — solo se respalda vía sincronización de OneDrive. Si OneDrive falla o el archivo se sobrescribe por error, no hay historial en Git para recuperarlo. El patrón de `.gitignore` se amplió de `Data/*.xlsx` a `Data/**/*.xlsx` durante `SC-7` (ver [Outputs/45](../Outputs/45_resultado_sc7_validacion_tecnica_funcional_visual.md)) porque el patrón anterior no cubría exportaciones ubicadas en subcarpetas de `Data/`.
- **Bloqueo de archivo**: si algún Excel de `Data/` queda abierto durante una actualización, la actualización falla (ver `Docs/04`).
- **Fragmentación por nombre**: con el volumen actual, cualquier variante nueva de escritura de un nombre de asesor no cubierta por una regla de limpieza puede fragmentar filas en `cl_tabla_asesor`. Para capacitaciones, los nombres de formador/líder ya no se exponen en la página activa, pero siguen participando en la clave provisional de `Capacitaciones Realizadas` hasta resolver `D9`.
- **Cambios automáticos de Power BI Desktop**: cada apertura/guardado puede reescribir archivos TMDL/JSON (metadatos, `lineageTag`, orden de página activa). Si no se revisa `git status` con regularidad, estos cambios pueden mezclarse con cambios intencionales futuros y dificultar la revisión de diffs.
- **Documentación desactualizada**: si se agregan/quitan medidas, páginas o fuentes sin actualizar la carpeta `Docs/`, la documentación deja de reflejar el estado real del modelo. Ver la recomendación de mantenimiento en el [README.md](../README.md) raíz.
- **Exposición pública de nombres reales** vía el enlace de "Publicar en la Web" activo (ver §5) — riesgo de gobierno de datos, no técnico.
