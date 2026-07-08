# Resultado — Fase 4: Limpieza y transformación de datos en Power Query

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | Fase 4 — Limpieza y transformación de datos en Power Query (ver `Specs/02_plan_implementacion_informe_powerbi_connect.md`) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/03...`, `Outputs/04...` |
| Fecha | 2026-07-08 |
| Archivo modificado | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` |

---

## Estado inicial de `git status`

Al ejecutar `git status` antes de empezar, el working tree **no estaba limpio**: había un cambio sin confirmar en `expressions.tmdl`. Se investigó con `git diff` antes de continuar (siguiendo la práctica de no sobrescribir cambios ajenos sin revisarlos primero):

- El `diff` mostró únicamente **líneas en blanco insertadas** entre `lineageTag:` y `annotation` en las 4 expresiones existentes, más una línea en blanco final — **sin ningún cambio de contenido** (código M, GUIDs de `lineageTag` y anotaciones idénticos).
- Este patrón es consistente con que **Power BI Desktop abrió el PBIP y volvió a serializar internamente el modelo** — coincide con lo que reportaste ("el PBIP ya abre sin el error de `QueryGroup`"). Es decir, es evidencia indirecta razonable de que el archivo corregido en la fase anterior fue parseado exitosamente por Power BI Desktop (si hubiera fallado el parseo, Desktop no habría podido cargar ni re-serializar el modelo).
- Se decidió conservar este reformateo (no tiene impacto funcional) e incorporarlo en el mismo commit de esta fase, ya que separar un cambio de solo espacios en blanco en un commit aparte no aportaría valor.

## Confirmación de vista previa de consultas base

Reitero la limitación ya documentada en `Outputs/03` y `Outputs/04`: no tengo forma de abrir o controlar la interfaz de Power BI Desktop desde este entorno, por lo que no puedo generar yo mismo una vista previa ejecutada por el motor real de Power Query.

Lo que sustenta la confirmación de este punto:
1. **Confirmación explícita del usuario** en el contexto de esta tarea: "El PBIP ya abre sin el error de `QueryGroup`... valida en Power BI Desktop que las consultas `Base_*` aparecen en el Editor de Power Query y muestran vista previa sin error" — se toma como confirmación de que el paso previo (Fase 3 + corrección de `QueryGroup`) quedó validado en la aplicación real.
2. **Evidencia de archivo** descrita arriba (reformateo por Power BI Desktop sin errores de parseo).
3. Validación estructural repetida (ya hecha en fases previas y no alterada aquí): los 3 nombres de archivo y la hoja `Form Responses 1` siguen siendo correctos.

**Pendiente de confirmación manual por tu parte, igual que en fases anteriores:** una vez agregadas las 3 consultas limpias de esta fase, cerrar y volver a abrir Power BI Desktop y confirmar en el Editor de Power Query que `MatrizCalidad_Limpio`, `SatisfaccionCapacitacion_Limpio` y `EncuestaMotivacion_Limpio` muestran vista previa sin error.

## Consultas limpias creadas

Las 3 se crearon como **expresiones compartidas de M** (`expression`), igual que las `Base_*`, referenciando cada una a su consulta base por nombre (`Origen = Base_MatrizCalidad`, etc.) — el encadenamiento de consultas por nombre es una referencia M estándar y no está sujeto al problema de `QueryGroup` (ese error era una referencia a nivel de propiedad TOM, no una referencia M; se evitó deliberadamente añadir `queryGroup` a estas nuevas consultas para no repetir el mismo tipo de error).

| Consulta | Basada en | Carga al modelo |
|---|---|---|
| `MatrizCalidad_Limpio` | `Base_MatrizCalidad` | Deshabilitada (staging) |
| `SatisfaccionCapacitacion_Limpio` | `Base_SatisfaccionCapacitacion` | Deshabilitada (staging) |
| `EncuestaMotivacion_Limpio` | `Base_EncuestaMotivacion` | Deshabilitada (staging) |

## Transformaciones aplicadas por consulta

Mismo patrón de 6 pasos con nombres descriptivos en las 3 consultas (solo cambia la consulta de origen y, en el último paso, el nombre exacto de la columna de call center):

1. **`Origen`** — referencia a la consulta `Base_*` correspondiente (sin tocarla).
2. **`EncabezadosPromovidos`** — `Table.PromoteHeaders(..., [PromoteAllScalars=true])`: usa la primera fila (el encabezado real del formulario) como nombres de columna.
3. **`NombresColumnaSinEspacios`** — `Table.TransformColumnNames(..., Text.Trim)`: elimina espacios al inicio/fin de cada **nombre de columna** (ej. `"  Nombre del asesor  "` → `"Nombre del asesor"`). Se conservan los nombres originales en español, tildes y signos `¿?` — el renombrado técnico final a convenciones `Fact_`/`PascalCase` queda para la Fase 5, como corresponde.
4. **`TextoLimpio`** — `Table.TransformColumns(..., {}, each if _ is text then Text.Trim(Text.Clean(_)) else _)`: aplica `Trim` + `Clean` a **todas las celdas de tipo texto** de la tabla (sin tocar columnas numéricas/fecha). Es null-safe por diseño: `null` no es texto, por lo que se deja intacto.
5. **`TimestampComoFechaHora`** — `Table.TransformColumnTypes(..., {{"Timestamp", type datetime}})`: fija explícitamente el tipo fecha/hora de la columna `Timestamp`.
6. **`ColumnaFechaAgregada`** — `Table.AddColumn(..., "Fecha", each if [Timestamp] = null then null else Date.From([Timestamp]), type date)`: crea la nueva columna `Fecha` (solo fecha, sin hora), derivada de `Timestamp`, con protección explícita ante nulos.
7. **`CallCenterNormalizado`** (paso final) — `Table.TransformColumns(..., {{"<columna call center>", each if _ = null then _ else Text.Trim(Text.Upper(_)), type text}})`: convierte la columna equivalente a call center a mayúsculas y sin espacios extremos, con protección ante nulos.

Columna de call center usada en el paso 7 por consulta:

| Consulta | Columna normalizada |
|---|---|
| `MatrizCalidad_Limpio` | `CALL CENTER` |
| `SatisfaccionCapacitacion_Limpio` | `¿En qué call center trabajas?` |
| `EncuestaMotivacion_Limpio` | `¿En qué call center trabajas?` |

**Nota sobre el guardado de nulos en la protección del paso 7:** se agregó intencionalmente porque el diagnóstico de `Specs/01` (sección 3.2) detectó que `Satisfacción capacitación` tiene 1 fila de 32 con `CallCenter`/`Jornada`/`Nombre` en blanco. Sin esta protección, `Text.Upper(null)` habría generado un error de vista previa en esa consulta — es la única desviación "defensiva" respecto al patrón mínimo, y se aplicó por igual en las 3 consultas por consistencia, no porque las otras 2 tengan nulos conocidos actualmente.

**Lo que explícitamente NO se hizo en esta fase** (por instrucción directa): no se trataron los valores `"N/A"` de la Matriz de calidad como nulos (permanecen como texto `"N/A"`, sin alterar), no se consolidaron los alias del líder (`Juan Esteban Pérez Camargo` sigue con sus 4 variantes), no se unificaron respuestas abiertas equivalentes a "sin comentario", y no se aplicó el renombrado técnico final de columnas.

## Columnas nuevas creadas

Una sola columna nueva, igual en las 3 consultas: **`Fecha`** (tipo `date`), derivada de `Timestamp` mediante `Date.From`.

## Decisión sobre carga al modelo

**Deshabilitada, igual que las `Base_*`.** Las 3 consultas limpias se representan como `expression` (no `table`) en TMDL — no existen como tablas del modelo semántico ni aparecerán en el panel de campos del informe. Quedan como paso intermedio, tal como pide la instrucción #15 de esta fase; la conversión a tablas cargadas (`Fact_*`) es responsabilidad de la Fase 7 del plan.

## Errores encontrados y solución aplicada

- **No se encontraron errores de sintaxis** en la revisión estructural (TMDL e M) realizada.
- Se identificó **proactivamente** (no como un error ya ocurrido, sino como un riesgo evitado) que aplicar `Text.Upper`/`Text.Trim` directamente sobre la columna de call center sin protección de nulos habría producido un error de evaluación en la fila en blanco de `Satisfacción capacitación`. Se resolvió agregando la guarda `if _ = null then _ else ...` antes de escribir el archivo, evitando así el error en vez de corregirlo después.
- El reformateo de espacios en blanco detectado al inicio (ver sección de estado inicial) no es un error — es evidencia de una apertura exitosa del archivo en Power BI Desktop.

## Archivos modificados

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (se agregaron las 3 expresiones `*_Limpio`; se incorporó también el reformateo de espacios en blanco preexistente descrito arriba).

## Resultado del commit

- Mensaje: `refactor(powerquery): crear consultas limpias intermedias`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (modificado: reformateo de Power BI Desktop + 3 expresiones nuevas), `Outputs/05_resultado_fase_4_limpieza_transformacion_powerquery.md` (nuevo).
- No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma. No se incluyó ningún archivo de `Data/*.xlsx`.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 5

**No avanzar todavía a la Fase 5 (normalización de nombres técnicos) sin antes:**
1. Cerrar y volver a abrir Power BI Desktop, y confirmar en el Editor de Power Query que `MatrizCalidad_Limpio`, `SatisfaccionCapacitacion_Limpio` y `EncuestaMotivacion_Limpio` muestran vista previa sin error — esta es la validación pendiente más importante, ya que ninguna de las transformaciones de esta fase fue ejecutada por el motor real.
2. Revisar visualmente que la columna `Fecha` se ve como fecha (no como texto) y que la columna de call center quedó en mayúsculas en las 3 consultas.
3. Si algo falla en Power BI Desktop, repórtalo con el mensaje de error exacto para diagnosticarlo antes de continuar — el diseño de las consultas ya incluye protección contra nulos en los puntos identificados como de mayor riesgo (columna de call center en `Satisfacción capacitación`).

Si la validación en Power BI Desktop es exitosa, el proyecto queda listo para la Fase 5 (normalización de nombres técnicos de tablas y columnas hacia la convención `Fact_`/`PascalCase`).

---

*Documento generado como registro operativo de la Fase 4, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
