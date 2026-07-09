# Modelo de datos — `PBI_Indicadores`

Descripción técnica del modelo semántico (TMDL) del informe Connect Assistance. Fuente de verdad: `PBI/PBI_Indicadores.SemanticModel/definition/`. Este documento describe la estructura vigente al cierre de la Fase 18; para el detalle de fórmulas DAX ver [`02_catalogo_medidas_dax.md`](02_catalogo_medidas_dax.md).

## 1. Enfoque: modelo en estrella, 3 hechos independientes

El modelo sigue un esquema estrella clásico: **3 tablas de hechos** (una por encuesta/instrumento) que comparten **3 dimensiones** (`Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`) y **4 tablas de medidas** DAX. Las 3 tablas de hechos **no están unificadas** en una sola tabla: cada encuesta tiene grano, periodicidad y preguntas distintas (una llamada evaluada, una respuesta de capacitación, una respuesta de actividad comercial), por lo que fusionarlas obligaría a forzar preguntas no comparables en las mismas columnas. Esta separación es una decisión de diseño intencional, no un pendiente.

```
                    Dim_Calendario (Fecha)
                    Dim_CallCenter (CallCenter)
                    Dim_Jornada (Jornada)
                            │
        ┌───────────────────┼───────────────────┐
        │                    │                    │
Fact_CalidadLlamadas  Fact_SatisfaccionCapacitacion  Fact_MotivacionActividad
   (sin Jornada)
```

`Fact_CalidadLlamadas` no tiene relación con `Dim_Jornada` porque la Matriz de calidad de origen no captura esa pregunta — es un vacío conocido de la fuente, no un error de modelado.

## 2. Tablas de hechos

### 2.1 `Fact_CalidadLlamadas`

Grano: 1 fila = 1 llamada evaluada por el rol PUSHER (auditoría de calidad).

| Columna técnica | Tipo | Columna original en Excel |
|---|---|---|
| `FechaHora` | dateTime | `Timestamp` |
| `Fecha` | date (derivada de `FechaHora`) | — |
| `CallCenter` | string | `CALL CENTER` |
| `NombreAsesor` | string | `Nombre del asesor` |
| `NombreLider` | string | `Líder / supervisor` |
| `NombreAuditor` | string | `Auditor / PUSHER` |
| `Preg_TonoSaludo` | double | `¿El asesor inició la llamada con buen tono, saludo, energía y seguridad?` |
| `Preg_FraseImpacto` | double | `¿Usó una frase de impacto para introducir la oferta?` |
| `Preg_PreguntasNecesidad` | double | `¿Realizó preguntas para descubrir la necesidad del cliente?` |
| `Preg_ConexionBeneficio` | double | `¿Conectó la necesidad del cliente con el beneficio del producto?` |
| `Preg_ExplicacionProducto` | double | `¿Explicó claramente qué incluye el producto sin saturar?` |
| `Preg_ManejoObjeciones` | double | `¿Manejó adecuadamente las objeciones del cliente?` |
| `Preg_CierreComercial` | double | `¿Realizó cierre comercial sin presionar?` |
| `Preg_ConfirmacionCierre` | double | `¿Confirmó condiciones, aceptación y cerró con calidad?` |
| `ObjecionPrincipal` | string | `Objeción principal del cliente` |
| `TerminoEnVenta` | string (Sí/No) | `¿La llamada terminó en venta?` |
| `Observaciones` | string | `Observaciones` |

**Regla de negocio aplicada:** en las 8 columnas `Preg_*`, el texto literal `"N/A"` se reemplaza por `null` antes de convertir a número — nunca se trata como `0`. Esto es obligatorio para no penalizar preguntas no aplicables al calcular promedios/porcentajes.

**Sin `Jornada`**: intencional, ver §1.

### 2.2 `Fact_SatisfaccionCapacitacion`

Grano: 1 fila = 1 respuesta de encuesta de satisfacción de capacitación.

| Columna técnica | Tipo | Columna original en Excel |
|---|---|---|
| `FechaHora` | dateTime | `Timestamp` |
| `Fecha` | date (derivada) | — |
| `CallCenter` | string | `¿En qué call center trabajas?` |
| `Jornada` | string | `¿En qué jornada participaste?` |
| `NombreAsesor` | string | `Nombre completo (mayúscula)` |
| `NombreLider` | string | `Nombre del líder (Mayúscula)` |
| `NombreFormador` | string | `Nombre formador` |
| `SatisfaccionGeneral` | double (Likert) | `¿Qué tan satisfecho/a quedaste con la capacitación?` |
| `Claridad` | double (Likert) | `La capacitación fue clara y fácil de entender.` |
| `Utilidad` | double (Likert) | `El contenido fue útil para mi gestión comercial.` |
| `Dinamismo` | double (Likert) | `La capacitación fue dinámica y mantuvo mi atención.` |
| `Duracion` | string | `Duración` |
| `Comentario` | string | `¿Qué mejorarías o qué información te gustaría para una próxima visita?` |

**Reglas de negocio aplicadas:**
- `NombreAsesor`/`NombreLider`/`NombreFormador`: `Trim` + `Clean` + `Proper Case`.
- `NombreLider`: las variantes ya detectadas de un mismo líder (tildes, orden de apellidos, errores de tipeo) se consolidan a un único nombre estándar mediante una tabla de alias en Power Query, definida directamente en la partición de `Fact_SatisfaccionCapacitacion`.
- Valores nulos en columnas categóricas (`CallCenter`, `Jornada`, nombres, `Duracion`) se marcan como `"Sin dato"` en vez de dejarse en blanco.
- `Comentario`: variantes de "sin respuesta" (`N/A`, `NA`, `Nada`, `Ninguna`, `No`, `Sin observaciones`, vacío) se unifican al valor estándar `"Sin comentario"`.

### 2.3 `Fact_MotivacionActividad`

Grano: 1 fila = 1 respuesta de encuesta de motivación de actividades comerciales.

| Columna técnica | Tipo | Columna original en Excel |
|---|---|---|
| `FechaHora` | dateTime | `Timestamp` |
| `Fecha` | date (derivada) | — |
| `CallCenter` | string | `¿En qué call center trabajas?` |
| `Jornada` | string | `¿En qué jornada participaste?` |
| `SatisfaccionGeneral` | double (Likert) | `En general, ¿Qué tan satisfecho/a quedaste con la actividad?` |
| `ClaridadUtilidad` | double (Likert) | `La actividad fue clara, dinámica y útil para tu trabajo.` |
| `MotivacionPostActividad` | double (Likert) | `Después de la actividad, ¿te sentiste más motivado/a para vender?` |
| `AmbienteEquipo` | string | `¿Cómo describirías el ambiente de tu equipo?` |
| `TipoActividadPreferida` | string | `¿Qué tipo de actividades crees que funcionan mejor con tu equipo?` |
| `Comentario` | string | `¿Qué mejorarías o qué actividad te gustaría para una próxima visita?` |

**Particularidad:** la columna `Nombre completo (Mayúscula)` del Excel de origen llega 100% vacía — se elimina explícitamente en Power Query (`ColumnaSinDatosEliminada`) porque no aporta información y la encuesta es, en la práctica, anónima. Ver [`05_decisiones_limitaciones_pendientes.md`](05_decisiones_limitaciones_pendientes.md).

## 3. Dimensiones

### 3.1 `Dim_Calendario`

Tabla continua de fechas generada en Power Query (no una lista fija). Rango: desde la `Fecha` mínima observada entre las 3 tablas de hechos, hasta `DateTime.LocalNow()` (hoy) — **no** hasta la fecha máxima observada en los datos. Esto hace que el calendario se autoextienda en cada actualización sin necesidad de tocarlo manualmente.

Columnas: `Fecha`, `Anio`, `MesNumero`, `MesNombre` (español, `es-CO`), `AnioMes`, `Trimestre`, `SemanaAnio`, `DiaMes`, `DiaSemanaNumero`, `DiaSemanaNombre` (español), `EsFinDeSemana`.

### 3.2 `Dim_CallCenter`

Columna única `CallCenter`. Se construye por **unión (`Table.Combine`) de los valores distintos** de `CallCenter` observados en las 3 tablas de hechos, sin nulos, sin duplicados, orden alfabético. No es una lista fija: un call center nuevo que aparezca en cualquiera de los 3 Excel se incorpora automáticamente en la siguiente actualización.

### 3.3 `Dim_Jornada`

Columna única `Jornada`. Se construye por unión de los valores distintos de `Jornada` de `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` únicamente (`Fact_CalidadLlamadas` no aporta, porque no tiene esa columna). Misma lógica dinámica que `Dim_CallCenter`.

## 4. Tablas de medidas

Las medidas DAX están organizadas en 4 tablas separadas por área, con el prefijo `_` para que aparezcan primero en el panel de campos de Power BI Desktop. Cada una es una tabla "vacía" (una columna oculta `Columna1` sin datos reales) que solo aloja medidas — patrón estándar para no mezclar medidas con columnas de una tabla de hechos.

- `_Medidas Generales` — conteos base y medidas `n=` transversales.
- `_Medidas Calidad` — medidas sobre `Fact_CalidadLlamadas`.
- `_Medidas Capacitacion` — medidas sobre `Fact_SatisfaccionCapacitacion`.
- `_Medidas Motivacion` — medidas sobre `Fact_MotivacionActividad`.

El catálogo completo, con fórmula DAX exacta de cada medida, está en [`02_catalogo_medidas_dax.md`](02_catalogo_medidas_dax.md).

## 5. Relaciones

11 relaciones activas en `relationships.tmdl`, todas de cardinalidad `1:*` (una dimensión filtra muchas filas de hechos), dirección de filtro única, sin relaciones entre las 3 tablas de hechos entre sí:

| Desde | Hacia | Comentario |
|---|---|---|
| `Dim_Calendario[Fecha]` | `Fact_CalidadLlamadas[Fecha]` | |
| `Dim_Calendario[Fecha]` | `Fact_SatisfaccionCapacitacion[Fecha]` | |
| `Dim_Calendario[Fecha]` | `Fact_MotivacionActividad[Fecha]` | |
| `Dim_CallCenter[CallCenter]` | `Fact_CalidadLlamadas[CallCenter]` | |
| `Dim_CallCenter[CallCenter]` | `Fact_SatisfaccionCapacitacion[CallCenter]` | |
| `Dim_CallCenter[CallCenter]` | `Fact_MotivacionActividad[CallCenter]` | |
| `Dim_Jornada[Jornada]` | `Fact_SatisfaccionCapacitacion[Jornada]` | |
| `Dim_Jornada[Jornada]` | `Fact_MotivacionActividad[Jornada]` | Sin relación equivalente para `Fact_CalidadLlamadas` (no tiene columna `Jornada`) |

No hay relaciones ambiguas ni inactivas no intencionadas (confirmado en la Fase 17, `Outputs/31`).

## 6. Tablas automáticas de fecha (Auto Date/Time) — ruido conocido

Además de las 8 relaciones de negocio, el modelo tiene **3 relaciones adicionales** hacia tablas ocultas generadas automáticamente por la función "Auto Date/Time" de Power BI Desktop:

- `DateTableTemplate_2973bde6-872f-4cb8-9c9b-5dae0a9694b2`
- `LocalDateTable_225f0da6-3e13-4994-aec4-f14191d7c9b9` (asociada a `Fact_CalidadLlamadas[Fecha]`)
- `LocalDateTable_082769f1-b472-466c-aa67-659058d901ad` (asociada a `Fact_SatisfaccionCapacitacion[Fecha]`)
- `LocalDateTable_c16eb748-6320-4b4d-aa26-794390d09a95` (asociada a `Fact_MotivacionActividad[Fecha]`)

**Estas tablas están presentes al cierre de la Fase 18** y son redundantes desde que `Dim_Calendario` tiene relaciones reales con las 3 tablas de hechos (desde la Fase 9). Es "ruido conocido", no un error. Limpiarlas requiere:

1. Deshabilitar la detección automática de tabla de fechas en Power BI Desktop (Archivo → Opciones y configuración → Opciones → Carga de datos → desmarcar "Detección automática de tabla de fechas/hora nuevas").
2. Quitar manualmente el bloque `variation` que Power BI Desktop agrega en la columna `Fecha` de cada `Fact_*` (apunta a estas tablas ocultas como jerarquía por defecto).

Esta limpieza **queda pendiente**, debe hacerse manualmente desde la interfaz de Power BI Desktop (no a mano en TMDL, por riesgo de referencia huérfana). Ver [`05_decisiones_limitaciones_pendientes.md`](05_decisiones_limitaciones_pendientes.md).

## 7. Convenciones de nombres

- Tablas de hechos: prefijo `Fact_`.
- Dimensiones: prefijo `Dim_`.
- Tablas de medidas: prefijo `_Medidas <Área>` (el guion bajo las ordena primero en el panel de campos).
- Columnas técnicas: `PascalCase`, sin espacios, sin tildes, sin `¿`/`?`.
- Pipeline de Power Query por fuente: `Base_<Nombre>` (staging crudo, carga deshabilitada) → `<Nombre>_Limpio` (Trim/Clean/tipado) → partición final `Fact_<Nombre>` (renombrado técnico + reglas de negocio).
- No se escribe `lineageTag`, `description` ni `queryGroup` a mano en archivos `.tmdl` — Power BI Desktop los genera automáticamente al guardar; escribirlos a mano rompe el analizador de la vista previa de Desktop (`UnknownKeyword`).
