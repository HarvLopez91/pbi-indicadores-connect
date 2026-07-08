# Resultado — Fase 5 (aplicada): Normalización de nombres técnicos de tablas y columnas

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 5 — Normalización de nombres técnicos de tablas y columnas (ver `Specs/02_plan_implementacion_informe_powerbi_connect.md`) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/05...`, `Outputs/06_resultado_fase_5_normalizacion_nombres_tecnicos.md`, `Outputs/06_mapeo_columnas_fase_5.md` |
| Fecha | 2026-07-08 |
| Estado | **Ejecutada** (desbloqueada sin MCP, con validación manual del usuario) |
| Archivo modificado | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` |

---

## Estado inicial de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado antes de iniciar. El último commit era `a3ca418` (documentación del bloqueo de la Fase 5 por falta de MCP).

## Motivo por el cual se desbloqueó la Fase 5 sin MCP

En la ejecución anterior (`Outputs/06_resultado_fase_5_normalizacion_nombres_tecnicos.md`) la fase quedó detenida porque no existe ningún servidor MCP en este entorno capaz de inspeccionar una sesión activa de Power BI Desktop. En esta ejecución, el usuario **autorizó explícitamente continuar sin MCP**, aportando validación manual propia:

- Confirmó que el PBIP abre correctamente en Power BI Desktop.
- Confirmó que el Editor de Power Query muestra las consultas creadas en la Fase 4.
- Confirmó que al menos una consulta `*_Limpio` mostró vista previa correcta.
- Indicó explícitamente que la validación por MCP se resolverá más adelante y no debe bloquear esta fase.

Esto reemplaza, para esta ejecución puntual, la validación por MCP exigida originalmente — es una decisión del usuario, no una que yo haya tomado unilateralmente. Se documenta aquí para trazabilidad.

## Consultas `Fact_*` creadas

Las 3 se crearon como **expresiones compartidas de M** (`expression`, no `table`), igual que en las fases anteriores, cada una referenciando su consulta `*_Limpio` correspondiente por nombre y aplicando `Table.RenameColumns` con el mapeo de `Outputs/06_mapeo_columnas_fase_5.md`:

| Consulta | Basada en | Carga al modelo |
|---|---|---|
| `Fact_CalidadLlamadas` | `MatrizCalidad_Limpio` | Deshabilitada (staging, hasta Fase 7) |
| `Fact_SatisfaccionCapacitacion` | `SatisfaccionCapacitacion_Limpio` | Deshabilitada (staging, hasta Fase 7) |
| `Fact_MotivacionActividad` | `EncuestaMotivacion_Limpio` | Deshabilitada (staging, hasta Fase 7) |

Las consultas `Base_*` y `*_Limpio` de las Fases 3 y 4 se conservan intactas y sin cambios, como corresponde a su rol de staging/intermedias.

## Mapeo de columnas aplicado por cada tabla

### `Fact_CalidadLlamadas` (desde `MatrizCalidad_Limpio`)

| Columna original | Columna técnica |
|---|---|
| Timestamp | `FechaHora` |
| CALL CENTER | `CallCenter` |
| Nombre del asesor | `NombreAsesor` |
| Líder / supervisor | `NombreLider` |
| Auditor / PUSHER | `NombreAuditor` |
| ¿El asesor inició la llamada con buen tono, saludo, energía y seguridad? | `Preg_TonoSaludo` |
| ¿Usó una frase de impacto para introducir la oferta? | `Preg_FraseImpacto` |
| ¿Realizó preguntas para descubrir la necesidad del cliente? | `Preg_PreguntasNecesidad` |
| ¿Conectó la necesidad del cliente con el beneficio del producto? | `Preg_ConexionBeneficio` |
| ¿Explicó claramente qué incluye el producto sin saturar? | `Preg_ExplicacionProducto` |
| ¿Manejó adecuadamente las objeciones del cliente? | `Preg_ManejoObjeciones` |
| ¿Realizó cierre comercial sin presionar? | `Preg_CierreComercial` |
| ¿Confirmó condiciones, aceptación y cerró con calidad? | `Preg_ConfirmacionCierre` |
| Objeción principal del cliente | `ObjecionPrincipal` |
| ¿La llamada terminó en venta? | `TerminoEnVenta` |
| Observaciones | *(sin cambio — ya cumplía la convención)* |
| Fecha | *(sin cambio — ya cumplía la convención, creada en Fase 4)* |

### `Fact_SatisfaccionCapacitacion` (desde `SatisfaccionCapacitacion_Limpio`)

| Columna original | Columna técnica |
|---|---|
| Timestamp | `FechaHora` |
| ¿En qué call center trabajas? | `CallCenter` |
| ¿En qué jornada participaste? | `Jornada` |
| Nombre completo (mayúscula) | `NombreAsesor` |
| Nombre del líder (Mayúscula) | `NombreLider` |
| Nombre formador | `NombreFormador` |
| ¿Qué tan satisfecho/a quedaste con la capacitación? | `SatisfaccionGeneral` |
| La capacitación fue clara y fácil de entender. | `Claridad` |
| El contenido fue útil para mi gestión comercial. | `Utilidad` |
| La capacitación fue dinámica y mantuvo mi atención. | `Dinamismo` |
| Duración | `Duracion` |
| ¿Qué mejorarías o qué información te gustaría para una próxima visita? | `Comentario` |
| Fecha | *(sin cambio — ya cumplía la convención, creada en Fase 4)* |

### `Fact_MotivacionActividad` (desde `EncuestaMotivacion_Limpio`)

| Columna original | Columna técnica |
|---|---|
| Timestamp | `FechaHora` |
| ¿En qué call center trabajas? | `CallCenter` |
| ¿En qué jornada participaste? | `Jornada` |
| En general, ¿Qué tan satisfecho/a quedaste con la actividad? | `SatisfaccionGeneral` |
| La actividad fue clara, dinámica y útil para tu trabajo. | `ClaridadUtilidad` |
| Después de la actividad, ¿te sentiste más motivado/a para vender? | `MotivacionPostActividad` |
| ¿Cómo describirías el ambiente de tu equipo? | `AmbienteEquipo` |
| ¿Qué tipo de actividades crees que funcionan mejor con tu equipo? | `TipoActividadPreferida` |
| ¿Qué mejorarías o qué actividad te gustaría para una próxima visita? | `Comentario` |
| Fecha | *(sin cambio — ya cumplía la convención, creada en Fase 4)* |

## Columnas omitidas y justificación

**`Nombre completo (Mayúscula)` en `Fact_MotivacionActividad`** — se **eliminó** (`Table.RemoveColumns`), no se renombró. Justificación, ya documentada en `Outputs/06_mapeo_columnas_fase_5.md`:
- La columna está **100% vacía** (5/5 filas, confirmado en `Specs/01` §3.3).
- No forma parte del grano/columnas recomendadas para `Fact_MotivacionActividad` en `Specs/01` §4.2/4.3 — la encuesta de motivación es anónima por diseño.
- Mantenerla sin datos y sin nombre técnico habría dejado una columna "huérfana" en la tabla de hechos final; eliminarla es más limpio que renombrarla o dejarla oculta.

No se omitió ninguna otra columna en las 3 consultas — todas las columnas de `MatrizCalidad_Limpio` y `SatisfaccionCapacitacion_Limpio` fueron renombradas o ya cumplían la convención (`Observaciones`, `Fecha`).

## Decisión sobre carga al modelo

**Deshabilitada, igual que en fases anteriores.** Las 3 consultas `Fact_*` se representan como `expression` (no `table`) en TMDL — no existen todavía como tablas del modelo semántico ni aparecen en el panel de campos. Esto es intencional y coincide con la instrucción #16 de esta fase: la carga final al modelo, junto con la creación de `Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada` y las relaciones, corresponde a la Fase 7 del plan (`Specs/02`).

## Errores encontrados y solución aplicada

- **No se encontraron errores de sintaxis** en la revisión estructural (TMDL e M) realizada tras escribir el archivo.
- Se prestó atención especial a la **coincidencia exacta de nombres de columna** entre `Table.RenameColumns`/`Table.RemoveColumns` y los encabezados reales post-Fase 4 (incluyendo diferencias sutiles como `"Nombre completo (mayúscula)"` con minúscula en la encuesta de capacitación vs. `"Nombre completo (Mayúscula)"` con mayúscula en la encuesta de motivación — son formularios distintos con capitalización distinta en el título de esa pregunta, no un error de transcripción). `Table.RenameColumns`/`Table.RemoveColumns` fallan con error si el nombre no existe exactamente, así que esta verificación era crítica antes de guardar.
- Se confirmó, mediante `grep`, que no se introdujo accidentalmente ninguna propiedad `queryGroup` (la causa del incidente corregido en `Outputs/04`).

## Archivos modificados

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (se agregaron las 3 expresiones `Fact_*`; `Base_*` y `*_Limpio` quedaron sin cambios).

## Resultado del commit

- Mensaje: `refactor(powerquery): aplicar normalizacion tecnica de consultas fact`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (modificado: 3 expresiones `Fact_*` agregadas), `Outputs/07_resultado_fase_5_normalizacion_nombres_tecnicos_aplicada.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 6

**No avanzar todavía a la Fase 6 sin antes:**
1. Confirmar en Power BI Desktop (Editor de Power Query) que `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` aparecen y muestran vista previa sin error, con los nombres de columna ya en `PascalCase`.
2. Verificar visualmente que `Fact_MotivacionActividad` ya no tiene la columna `Nombre completo (Mayúscula)`.
3. Si Power BI Desktop reporta algún nombre de columna no encontrado, es señal de que alguno de los encabezados originales no coincide exactamente con lo mapeado aquí — repórtalo con el mensaje de error exacto para corregir el nombre puntual, sin necesidad de rehacer el resto del mapeo.

Si la validación es exitosa, el proyecto queda listo para la Fase 6 (tratamiento de valores nulos, `"N/A"`, alias de líderes y respuestas abiertas), que sigue pendiente en su totalidad — nada de eso se tocó en esta fase.

---

*Documento generado como registro operativo de la Fase 5 (aplicada), según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
