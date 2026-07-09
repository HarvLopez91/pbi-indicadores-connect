# Mapeo de columnas — Fase 5 (PROPUESTA, NO APLICADA)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Estado | **Propuesta de diseño — no aplicada al PBIP.** La Fase 5 quedó detenida (ver [Outputs/06_resultado_fase_5_normalizacion_nombres_tecnicos.md](06_resultado_fase_5_normalizacion_nombres_tecnicos.md)) antes de renombrar ninguna columna o consulta. Este documento existe para no perder el trabajo de diseño ya validado en `Specs/01` y agilizar la ejecución una vez se desbloquee la fase. |
| Base del mapeo | [Specs/01_analisis_de_impacto_informe_powerbi_connect.md](../Specs/01_analisis_de_impacto_informe_powerbi_connect.md), sección 4.3 (columnas técnicas recomendadas por tabla de hechos) |
| Columna "original" | Se usa el nombre **ya trimeado en la Fase 4** (`*_Limpio`), no el crudo del Excel (que además tenía espacios al inicio/fin) |
| Convención técnica | `PascalCase`, sin espacios, sin tildes, sin `¿?` — según `Specs/01` §4.5 |

---

## 1. `Base_MatrizCalidad` / `MatrizCalidad_Limpio` → `Fact_CalidadLlamadas`

| Columna original (post Fase 4) | Columna técnica propuesta | Tipo de dato esperado | Observación |
|---|---|---|---|
| Timestamp | `FechaHora` | datetime | Ya tipada como datetime en `MatrizCalidad_Limpio` |
| CALL CENTER | `CallCenter` | text | Ya normalizada a mayúsculas en `MatrizCalidad_Limpio` |
| Nombre del asesor | `NombreAsesor` | text | Variantes de capitalización/typos observadas (`Andres stitch`, `Oscar Tineo`); consolidación de alias NO es parte de esta fase |
| Líder / supervisor | `NombreLider` | text | Solo nombre de pila en los datos actuales (`Tatiana`, `Alvaro`) |
| Auditor / PUSHER | `NombreAuditor` | text | Único valor observado: `Jeisy Martinez` |
| ¿El asesor inició la llamada con buen tono, saludo, energía y seguridad? | `Preg_TonoSaludo` | number (mixto con texto `"N/A"`) | `"N/A"` aún no tratado como nulo — pendiente de Fase 6 |
| ¿Usó una frase de impacto para introducir la oferta? | `Preg_FraseImpacto` | number (mixto con `"N/A"`) | ídem |
| ¿Realizó preguntas para descubrir la necesidad del cliente? | `Preg_PreguntasNecesidad` | number (mixto con `"N/A"`) | ídem |
| ¿Conectó la necesidad del cliente con el beneficio del producto? | `Preg_ConexionBeneficio` | number (mixto con `"N/A"`) | ídem |
| ¿Explicó claramente qué incluye el producto sin saturar? | `Preg_ExplicacionProducto` | number (mixto con `"N/A"`) | ídem |
| ¿Manejó adecuadamente las objeciones del cliente? | `Preg_ManejoObjeciones` | number | Sin `"N/A"` en la muestra actual (3 filas) |
| ¿Realizó cierre comercial sin presionar? | `Preg_CierreComercial` | number (mixto con `"N/A"`) | ídem |
| ¿Confirmó condiciones, aceptación y cerró con calidad? | `Preg_ConfirmacionCierre` | number (mixto con `"N/A"`) | ídem |
| Observaciones | `Observaciones` | text | Texto libre largo, con saltos de línea |
| Objeción principal del cliente | `ObjecionPrincipal` | text | Pocos valores repetidos; candidato a lista controlada a futuro |
| ¿La llamada terminó en venta? | `TerminoEnVenta` | text (Sí/No); booleano en fase futura | Se mantiene como texto en esta fase, sin convertir a booleano todavía |
| *(columna nueva, creada en Fase 4)* Fecha | `Fecha` | date | Derivada de `Timestamp` |

**Nota de grano:** 1 fila = 1 llamada evaluada por el PUSHER. Coincide con el grano definido en `Specs/01` §4.2 — no se propone ningún cambio de grano en este mapeo.

## 2. `Base_SatisfaccionCapacitacion` / `SatisfaccionCapacitacion_Limpio` → `Fact_SatisfaccionCapacitacion`

| Columna original (post Fase 4) | Columna técnica propuesta | Tipo de dato esperado | Observación |
|---|---|---|---|
| Timestamp | `FechaHora` | datetime | |
| ¿En qué call center trabajas? | `CallCenter` | text | Ya normalizada a mayúsculas en `SatisfaccionCapacitacion_Limpio` |
| ¿En qué jornada participaste? | `Jornada` | text | Valores: `Mañana`/`Tarde`; 1 fila con valor nulo conocida |
| Nombre completo (mayúscula) | `NombreAsesor` | text | Capitalización inconsistente pese al nombre del campo original |
| Nombre del líder (Mayúscula) | `NombreLider` | text | 4 variantes detectadas para la misma persona (`Specs/01` §3.2); consolidación de alias NO es parte de esta fase |
| Nombre formador | `NombreFormador` | text | Único valor: `Jeisy Martinez` |
| ¿Qué tan satisfecho/a quedaste con la capacitación? | `SatisfaccionGeneral` | number | Escala 1–5 |
| La capacitación fue clara y fácil de entender. | `Claridad` | number | Escala 1–5 |
| El contenido fue útil para mi gestión comercial. | `Utilidad` | number | Escala 1–5 |
| La capacitación fue dinámica y mantuvo mi atención. | `Dinamismo` | number | Escala 1–5 |
| Duración | `Duracion` | text | Categórica (`1 hora`, `30 minutos`) |
| ¿Qué mejorarías o qué información te gustaría para una próxima visita? | `Comentario` | text | Variantes de "sin respuesta" (`Na`, `N/A`, `Nada`...) pendientes de unificar — Fase 6, no esta fase |
| *(columna nueva, creada en Fase 4)* Fecha | `Fecha` | date | Derivada de `Timestamp` |

**Nota de grano:** 1 fila = 1 respuesta de encuesta de capacitación. Coincide con `Specs/01` §4.2.

## 3. `Base_EncuestaMotivacion` / `EncuestaMotivacion_Limpio` → `Fact_MotivacionActividad`

| Columna original (post Fase 4) | Columna técnica propuesta | Tipo de dato esperado | Observación |
|---|---|---|---|
| Timestamp | `FechaHora` | datetime | |
| ¿En qué call center trabajas? | `CallCenter` | text | Ya normalizada a mayúsculas en `EncuestaMotivacion_Limpio` |
| ¿En qué jornada participaste? | `Jornada` | text | Solo `Mañana` observado en la muestra actual (5 filas) |
| En general, ¿Qué tan satisfecho/a quedaste con la actividad? | `SatisfaccionGeneral` | number | Escala 1–5 |
| La actividad fue clara, dinámica y útil para tu trabajo. | `ClaridadUtilidad` | number | Escala 1–5 |
| Después de la actividad, ¿te sentiste más motivado/a para vender? | `MotivacionPostActividad` | number | Escala 1–5 |
| ¿Cómo describirías el ambiente de tu equipo? | `AmbienteEquipo` | text | Categórica: `Motivado`/`Colaborativo`/`Presionado` |
| ¿Qué tipo de actividades crees que funcionan mejor con tu equipo? | `TipoActividadPreferida` | text | Texto libre con alta variedad potencial |
| ¿Qué mejorarías o qué actividad te gustaría para una próxima visita? | `Comentario` | text | Texto libre; unificación de "sin respuesta" pendiente — Fase 6 |
| Nombre completo (Mayúscula) | *(sin mapear)* | text | **100% vacía** (5/5 filas). No forma parte del grano recomendado en `Specs/01` §4.2/4.3 (la encuesta es anónima). Se recomienda no renombrarla/cargarla, u ocultarla explícitamente si se conserva por trazabilidad |
| *(columna nueva, creada en Fase 4)* Fecha | `Fecha` | date | Derivada de `Timestamp` |

**Nota de grano:** 1 fila = 1 respuesta de encuesta de actividad comercial, sin identificación de asesor. Coincide con `Specs/01` §4.2 y con la limitación ya documentada (sin desglose por asesor posible en esta fuente).

---

## Cómo usar este documento cuando se desbloquee la Fase 5

1. Confirmar que este mapeo sigue vigente (no debería cambiar salvo que el volumen de datos revele nuevas columnas o cambios de estructura en los formularios).
2. Aplicar el renombrado de consulta y de columnas exactamente como está aquí, generando `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` a partir de las consultas `*_Limpio`.
3. Mantener sin renombrar el tratamiento de `"N/A"`, alias de líder y respuestas abiertas — eso sigue correspondiendo a fases posteriores (6 en adelante), no a la Fase 5.

---

*Documento generado como registro operativo de la Fase 5 (bloqueada), según la regla documental vigente: los resultados de ejecución/diseño de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
