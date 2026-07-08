# Resultado — Fase 7: Creación de tablas de hechos

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 7 — Creación de tablas de hechos (ver `Specs/02_plan_implementacion_informe_powerbi_connect.md`) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/08_resultado_fase_6_tratamiento_nulos_na_respuestas_abiertas.md` |
| Fecha | 2026-07-08 |
| Archivos modificados/creados | `expressions.tmdl`, `model.tmdl`, `tables/Fact_CalidadLlamadas.tmdl` (nuevo), `tables/Fact_SatisfaccionCapacitacion.tmdl` (nuevo), `tables/Fact_MotivacionActividad.tmdl` (nuevo) |

---

## Nota de proceso (transparencia)

Esta fase implicó, por primera vez en el proyecto, convertir consultas de staging (`expression`, sin carga) en **tablas reales del modelo** (`table`, con columnas y partición). A diferencia de una `expression`, un objeto `table` en TMDL exige declarar explícitamente cada columna con su `dataType`, lo cual no puede "inferirse" desde un editor de texto sin ejecutar el motor real de Power Query. Se abordó así:

1. Se reconstruyó el esquema de columnas de cada tabla a partir del código M ya validado en las Fases 4-6 (que define explícitamente qué tipo debe tener cada columna transformada).
2. Se releyeron los 3 archivos de `Data/` con Python para confirmar, columna por columna, que **no existen valores atípicos** que pudieran romper un `Table.TransformColumnTypes` a número (por ejemplo, texto distinto de `"N/A"` en una columna de puntaje) — resultado: **cero valores problemáticos** en las 8 columnas `Preg_*` y las 7 columnas Likert de las otras 2 fuentes.
3. Se verificaron los conteos de filas reales contra los esperados por la instrucción: coinciden exactamente (3 / 32 / 5).

Con esto, la confianza en que las 3 tablas cargarán sin error en Power BI Desktop es alta, pero **la confirmación visual final en la aplicación (panel de datos, vista previa) sigue pendiente de tu validación**, igual que en fases anteriores, porque no tengo forma de abrir Power BI Desktop desde este entorno.

**Nota de disciplina de proceso:** en esta ejecución no ejecuté `git status` como primer paso literal antes de empezar a investigar la sintaxis TMDL de tablas — lo ejecuté después de hacer los cambios. Verifiqué retroactivamente que el working tree solo contiene exactamente los cambios de esta fase (nada preexistente sin commitear), por lo que no hay impacto en el resultado, pero lo señalo para que quede registrado.

## Estado inicial de `git status`

Verificado (de forma retroactiva, ver nota arriba) que antes de esta fase el working tree estaba limpio: el único commit previo relevante es `c2737b9` (Fase 6), y los únicos cambios detectados ahora corresponden exactamente a los archivos de esta fase.

## Validación de vista previa de las consultas `Fact_*`

No hay servidor MCP disponible ni pude abrir Power BI Desktop directamente (limitación ya documentada). Como alternativa estructural (permitida explícitamente en fases previas de este proyecto cuando MCP no está disponible):

- Se releyó `expressions.tmdl` antes de modificarlo y se confirmó que las 3 expresiones `Fact_*` de la Fase 6 estaban íntegras, sin errores de sintaxis.
- Se recalculó el pipeline completo de transformaciones sobre los datos reales (Python, simulando la misma lógica M) para las 8 columnas `Preg_*` de `Fact_CalidadLlamadas` y las 7 columnas Likert de las otras 2 tablas: **0 valores no numéricos y no-`"N/A"` encontrados** — es decir, los `Table.TransformColumnTypes` a `number`/`double` no deberían fallar.
- No se encontró ningún error de nombre de columna ni de sintaxis; no fue necesario detenerse.

## Tablas cargadas al modelo

Se crearon 3 archivos nuevos en `PBI/PBI_Indicadores.SemanticModel/definition/tables/`, cada uno representando una tabla TMDL completa (columnas + partición `mode: import`):

| Tabla | Archivo | Basada en (M) | Columnas |
|---|---|---|---|
| `Fact_CalidadLlamadas` | `tables/Fact_CalidadLlamadas.tmdl` | `MatrizCalidad_Limpio` (vía el mismo código M ya validado en Fase 6) | 17 |
| `Fact_SatisfaccionCapacitacion` | `tables/Fact_SatisfaccionCapacitacion.tmdl` | `SatisfaccionCapacitacion_Limpio` | 13 |
| `Fact_MotivacionActividad` | `tables/Fact_MotivacionActividad.tmdl` | `EncuestaMotivacion_Limpio` | 10 |

Las 3 expresiones `expression Fact_CalidadLlamadas`, `expression Fact_SatisfaccionCapacitacion` y `expression Fact_MotivacionActividad` se **eliminaron** de `expressions.tmdl` (un mismo nombre no puede existir simultáneamente como `expression` sin carga y como `table` cargada) — su código M se trasladó íntegro a la partición de cada tabla, sin alterar ninguna transformación ya validada en las Fases 4-6.

Se actualizó `model.tmdl` agregando las 3 referencias obligatorias (`ref table ...`), requeridas por TMDL para cualquier objeto que viva en su propio archivo dentro de la carpeta `tables/`:
```
ref table Fact_CalidadLlamadas
ref table Fact_SatisfaccionCapacitacion
ref table Fact_MotivacionActividad
```

## Consultas staging/intermedias conservadas sin carga

`expressions.tmdl` ahora contiene exactamente 7 expresiones (verificado), todas sin carga al modelo (staging), sin ningún cambio respecto a las Fases 3-4:

- `RutaCarpetaData` (parámetro)
- `Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion`, `Base_EncuestaMotivacion`
- `MatrizCalidad_Limpio`, `SatisfaccionCapacitacion_Limpio`, `EncuestaMotivacion_Limpio`

Ninguna se eliminó — se conservan íntegras para trazabilidad, tal como pedía la instrucción.

### Sobre agrupar `Fact_*` en una carpeta visual "Hechos"

La instrucción lo pedía como opcional ("si es posible"). **No se implementó** en esta fase: la organización de queries en Power Query Editor usa la propiedad `queryGroup`, que fue precisamente la causa del incidente corregido en `Outputs/04` (una referencia a un objeto de grupo no declarado rompió la apertura del PBIP). No encontré en la documentación oficial de TMDL una confirmación clara y segura de cómo declarar ese objeto `queryGroup` correctamente, y dado el antecedente, preferí no arriesgar otro error de apertura. **Recomendación:** puedes agrupar las 3 tablas manualmente en Power BI Desktop (seleccionarlas en el Editor de Poder Query → clic derecho → "Mover a grupo" → nombrarlo "Hechos") de forma segura desde la interfaz, sin este riesgo.

## Tipos de datos verificados por tabla

Se declararon explícitamente en TMDL **y** se aplicaron realmente en el código M (doble verificación: metadato + transformación real, no solo una declaración):

### `Fact_CalidadLlamadas`
| Columna(s) | Tipo TMDL | Cómo se logra |
|---|---|---|
| FechaHora | `dateTime` | Ya casteada en Fase 4 (`type datetime`), reafirmada en esta fase |
| Fecha | `dateTime` (formatString `Short Date`, se muestra solo la fecha — TOM no tiene un tipo "date" separado de `dateTime`) | Creada en Fase 4 con `type date` |
| CallCenter, NombreAsesor, NombreLider, NombreAuditor, ObjecionPrincipal, TerminoEnVenta, Observaciones | `string` | Casteadas explícitamente a `type text` en el nuevo paso `TiposFinalesAplicados` |
| 8 columnas `Preg_*` | `double` | `"N/A"` → `null` (Fase 6) + `Table.TransformColumnTypes(..., type number)` |

### `Fact_SatisfaccionCapacitacion`
| Columna(s) | Tipo TMDL | Cómo se logra |
|---|---|---|
| FechaHora, Fecha | `dateTime` | Igual que arriba |
| CallCenter, Jornada, NombreAsesor, NombreLider, NombreFormador, Duracion, Comentario | `string` | Casteadas en `TiposFinalesAplicados` |
| SatisfaccionGeneral, Claridad, Utilidad, Dinamismo | `double` | Nuevo paso `LikertComoNumero` — no estaban explícitamente tipadas en fases previas; se verificó contra los datos reales que ningún valor rompe el cast |

### `Fact_MotivacionActividad`
| Columna(s) | Tipo TMDL | Cómo se logra |
|---|---|---|
| FechaHora, Fecha | `dateTime` | Igual que arriba |
| CallCenter, Jornada, AmbienteEquipo, TipoActividadPreferida, Comentario | `string` | Casteadas en `TiposFinalesAplicados` |
| SatisfaccionGeneral, ClaridadUtilidad, MotivacionPostActividad | `double` | Nuevo paso `LikertComoNumero`, misma verificación |

Todas las columnas quedaron con `summarizeBy: none` (decisión explícita, no solicitada literalmente pero alineada con "no crear medidas todavía" y buena práctica: evita que Power BI aplique una agregación automática por `SUM` en columnas como puntajes Likert, donde sumar no tiene sentido semántico — las medidas explícitas se construirán en una fase posterior con la agregación correcta para cada caso, ej. `AVERAGE`).

## Conteo de filas por tabla

Recalculado sobre los datos reales actuales (no puedo leerlo desde el panel de Power BI Desktop, pero sí desde el archivo fuente, que es lo que el motor de Power Query procesará):

| Tabla | Filas esperadas (instrucción) | Filas reales verificadas |
|---|---|---|
| `Fact_CalidadLlamadas` | 3 | **3** ✅ |
| `Fact_SatisfaccionCapacitacion` | 32 | **32** ✅ |
| `Fact_MotivacionActividad` | 5 | **5** ✅ |

Ninguna transformación de las Fases 4-7 elimina filas (se marcaron nulos como `"Sin dato"`/`"Sin comentario"` en vez de descartarlas), por lo que el conteo debe coincidir exactamente cuando se actualice en Power BI Desktop.

## Archivos modificados en el PBIP

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (modificado: se removieron las 3 expresiones `Fact_*`, quedan 7 expresiones de staging).
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (modificado: se agregaron 3 líneas `ref table`).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_CalidadLlamadas.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_MotivacionActividad.tmdl` (nuevo).

## Errores encontrados y solución aplicada

- No se encontraron errores durante la construcción. Validaciones de sintaxis realizadas:
  - Balance de paréntesis y llaves por archivo de tabla: `Fact_CalidadLlamadas.tmdl` (4/4, 36/36), `Fact_SatisfaccionCapacitacion.tmdl` (25/25, 39/39), `Fact_MotivacionActividad.tmdl` (12/12, 26/26).
  - Conteo de `lineageTag` por archivo coincide exactamente con el número de objetos declarados (tabla + partición + columnas): 19, 15 y 12 respectivamente (46 en total, uno por cada objeto nuevo).
  - `grep` de `queryGroup` en todo `PBI/`: **0 coincidencias** (se evitó deliberadamente repetir el patrón que causó el incidente de la Fase 3/4).
  - `expressions.tmdl` verificado con exactamente 7 expresiones remanentes, ninguna con `Fact_` en el nombre.

## Resultado del commit

- Mensaje: `feat(modelo): cargar tablas de hechos al modelo semantico`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (modificado), `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (modificado), `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_CalidadLlamadas.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_MotivacionActividad.tmdl` (nuevo), `Outputs/09_resultado_fase_7_creacion_tablas_hechos.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 8

**Antes de avanzar:**
1. Cerrar y volver a abrir Power BI Desktop (las tablas se crearon por edición externa del TMDL, lo que requiere reinicio de la aplicación para reflejarse) y confirmar que `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` aparecen en el panel de datos/modelo, con vista previa sin error.
2. Verificar los conteos de fila reales en la aplicación (3 / 32 / 5) contra lo documentado aquí.
3. Verificar visualmente los tipos de dato en el panel de columnas (ícono de calendario para `FechaHora`/`Fecha`, `#` para las columnas numéricas, `ABC` para texto).
4. Confirmar que `Base_*` y `*_Limpio` **no** aparecen como tablas cargadas (deben seguir en cursiva/atenuadas en el panel de consultas, indicando "carga deshabilitada").

Si la validación en Power BI Desktop es exitosa, el proyecto queda listo para la **Fase 8 — Creación de dimensiones** (`Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`), tal como está definida en `Specs/02`. **No se avanzó a la Fase 8 en esta ejecución**, conforme a la restricción indicada.

---

*Documento generado como registro operativo de la Fase 7, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
