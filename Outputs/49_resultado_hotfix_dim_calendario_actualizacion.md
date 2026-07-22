# Resultado — hotfix Dim_Calendario en actualización

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo | Hotfix posterior al cierre del plan de Satisfacción de capacitaciones |
| Fecha | 2026-07-22 |
| Rama | `main` |
| Commit base | `fb895c8 docs: cerrar plan de satisfaccion de capacitaciones` |
| Estado | Hotfix aprobado |

## Error reportado

Power BI Desktop mostró durante **Actualizar**:

```text
Dim_Calendario: No se pudo analizar la entrada proporcionada como un valor DateTime.
```

El cuadro indicó 10 consultas bloqueadas. Las tablas automáticas `DateTableTemplate` y `LocalDateTable` aparecieron esperando otras consultas; se trataron como consecuencia del fallo, no como causa raíz.

## Diagnóstico de origen

El incidente se presentó después de incorporar nuevas respuestas al archivo `Satisfacción capacitación (Responses).xlsx`. Negocio confirmó por conversación operativa que se había agregado información nueva; no se guarda captura de esa conversación en el repositorio.

Se inspeccionaron en modo lectura los tres Excel de `Data/`, leyendo directamente el XML interno del `.xlsx` para identificar tipo físico de celda.

| Fuente | Fila | Valor original | Tipo detectado | Conversión actual | Resultado |
|---|---:|---|---|---|---|
| `Matriz de calidad (Responses).xlsx` | 2-4 | seriales Excel `46205...46210` | serial numérico | `Table.TransformColumnTypes(... type datetime)` | Correcto |
| `Satisfacción capacitación (Responses).xlsx` | 2-85 | seriales Excel | serial numérico | `Table.TransformColumnTypes(... type datetime)` + `TimestampNormalizado` | Correcto |
| `Satisfacción capacitación (Responses).xlsx` | 86 | `S` | texto | `Table.TransformColumnTypes(... type datetime)` | **Error** |
| `Satisfacción capacitación (Responses).xlsx` | 87-99 | `7/15/2026 h:mm:ss` | texto `m/d/yyyy` | cultura implícita | Riesgo de error en cultura no `en-US` |
| `Encuesta satisfacción (Responses).xlsx` | 2-6 | seriales Excel `46205...` | serial numérico | `Table.TransformColumnTypes(... type datetime)` | Correcto |

Fila causante principal:

- Archivo: `Data/Satisfacción capacitación (Responses).xlsx`
- Hoja: `Form Responses 1`
- Fila Excel: `86`
- Celda: `A86`
- Columna: `Timestamp`
- Valor: `S`
- Resto de la fila: respuesta de capacitación de `ALMA`, jornada `Tarde`, formador `Jeisy Martinez`, con métricas Likert completas.

## Causa raíz

`SatisfaccionCapacitacion_Limpio` convertía `Timestamp` con:

```powerquery
Table.TransformColumnTypes(TextoLimpio, {{"Timestamp", type datetime}})
```

La celda `A86 = "S"` no es convertible a `datetime`. Además, las filas 87-99 llegaron como texto con formato estadounidense `m/d/yyyy h:mm:ss`, por lo que la conversión implícita queda expuesta a errores de cultura.

El error llega a `Dim_Calendario` porque su partición calcula:

```powerquery
FechaMinCapacitacion = List.Min(Fact_SatisfaccionCapacitacion[Fecha])
```

Si la columna `Fecha` contiene errores de conversión, `List.Min` evalúa esa lista y propaga el error de DateTime, bloqueando el calendario y todas las consultas dependientes.

## Corrección aplicada

### `SatisfaccionCapacitacion_Limpio`

Se reemplazó la conversión implícita de `Timestamp` por una conversión explícita por tipo:

- `datetime`: se conserva.
- `date`: `DateTime.From`.
- `number`: `DateTime.From` para serial Excel.
- `text`: limpieza con `Text.Trim(Text.Clean(...))`.
- texto vacío o token demasiado corto para representar fecha: `null`.
- texto fecha/hora válido: `DateTime.FromText(..., [Culture = "en-US"])`.
- tipo no soportado: error explícito.

Esto conserva la corrección ya validada de julio de 2026 en `TimestampNormalizado`.

### `Dim_Calendario`

La partición M real está en:

```text
PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl
```

Se ajustó el cálculo de mínimos para usar solo fechas no nulas:

- `FechasCalidadValidas = List.RemoveNulls(...)`
- `FechasCapacitacionValidas = List.RemoveNulls(...)`
- `FechasMotivacionValidas = List.RemoveNulls(...)`
- `FechaMinima` se calcula sobre mínimos válidos.

No se usó `try ... otherwise null` dentro del calendario para ocultar errores. Si aparece un error real de conversión, debe corregirse en la consulta de origen.

## Validación antes/después

Diagnóstico estático posterior al hotfix, simulado desde los tres Excel:

| Tabla | Filas | Tipos físicos en Timestamp | Nulos en Fecha esperados | Errores esperados | Fecha mínima | Fecha máxima |
|---|---:|---|---:|---:|---|---|
| `Fact_CalidadLlamadas` | 3 | 3 seriales | 0 | 0 | `2026-07-02` | `2026-07-07` |
| `Fact_SatisfaccionCapacitacion` | 98 | 84 seriales, 14 textos | 1 | 0 | `2026-07-04` | `2026-07-15` |
| `Fact_MotivacionActividad` | 5 | 5 seriales | 0 | 0 | `2026-07-02` | `2026-07-02` |

El único nulo esperado es la fila 86 de satisfacción, cuyo `Timestamp` original es `S`. La respuesta no se elimina; queda sin fecha hasta que el dato fuente sea corregido.

Rango esperado de `Dim_Calendario`:

- Fecha mínima: `2026-07-02`.
- Fecha máxima: fecha local de actualización (`Date.From(DateTime.LocalNow())`).
- Lista continua diaria, excluyendo nulos antes de `List.Min`.

## Archivos modificados

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Dim_Calendario.tmdl`
- `Outputs/49_resultado_hotfix_dim_calendario_actualizacion.md`

## Confirmaciones de alcance

- No se modificaron visuales.
- No se modificó DAX.
- No se modificaron relaciones.
- No se modificó navegación.
- No se modificaron archivos Excel.
- No se republicó en Power BI Service.

## Validación pendiente en Power BI Desktop

La validación en Power BI Desktop fue aprobada por el usuario después de ejecutar **Actualizar**.

Resultado confirmado:

- No apareció el error DateTime de `Dim_Calendario`.
- Las consultas dejaron de estar bloqueadas.
- Las 7 páginas cargaron.
- Satisfacción conservó sus fechas correctas.
- El gráfico por fecha incorporó correctamente `15/07`.
- Apareció el nuevo call center `ALMA`.
- Resultado final mostrado:
  - 6 capacitaciones.
  - 97 respuestas.
  - 6 call centers.
  - Última capacitación `15/07/2026`.
- Filtros, tablas, KPI y panel de satisfacción continuaron funcionando.

## Republicación posterior

Después de validar la actualización local, el informe fue republicado manualmente en Power BI Service. La republicación se documenta como operación posterior al cierre del plan, sin reabrir `SC-1` a `SC-9`.

## Estado final

`Hotfix aprobado`.
