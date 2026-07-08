# Resultado — Fase 6: Tratamiento de valores nulos, `"N/A"`, alias de líderes y respuestas abiertas

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 6 — Tratamiento de valores nulos, `"N/A"` y respuestas abiertas (ver `Specs/02_plan_implementacion_informe_powerbi_connect.md`) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/07_resultado_fase_5_normalizacion_nombres_tecnicos_aplicada.md` |
| Fecha | 2026-07-08 |
| Archivo modificado | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` |

---

## Estado inicial de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado antes de iniciar. Último commit: `59ba739` (Fase 5 aplicada).

## Validación inicial de consultas `Fact_*`

No hay servidor MCP disponible (situación ya documentada en `Outputs/06`), y esta fase permite explícitamente validación estructural como alternativa. Se releyó `expressions.tmdl` completo y se confirmó:

- Las 3 expresiones `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` existen, con el código exacto documentado en `Outputs/07`, sin cambios inesperados desde el último commit.
- No fue necesario detenerse por error de nombre de columna — la estructura previa a esta fase estaba íntegra.

Como validación adicional específica de esta fase (no exigida por MCP ni por la interfaz, pero relevante porque las reglas de esta fase dependen de los valores reales de los datos), se releyeron los 3 archivos de `Data/` con Python para:
1. Confirmar que los 3 siguen accesibles (sin bloqueo).
2. Calcular conteos exactos de filas/celdas afectadas por cada regla de esta fase (detalle en cada sección).
3. Verificar si existían variantes adicionales de líderes más allá de las ya conocidas (instrucción 16).

## Tratamientos aplicados en `Fact_CalidadLlamadas`

Se agregaron 2 pasos al final de la consulta existente:

1. **`NAsComoNulos`** — `Table.ReplaceValue(..., "N/A", null, Replacer.ReplaceValue, {8 columnas Preg_*})`: reemplaza el texto exacto `"N/A"` por `null` real en las 8 columnas de checklist, sin tocar los valores numéricos existentes.
2. **`PreguntasComoNumero`** — `Table.TransformColumnTypes(..., {8 columnas Preg_* -> type number})`: fija el tipo numérico de las 8 columnas ahora que ya no contienen texto, dejándolas listas para cálculos DAX en fases posteriores (sin calcular todavía ningún puntaje ni porcentaje, como se pidió).

### Columnas donde `"N/A"` fue convertido a `null`

Las 8: `Preg_TonoSaludo`, `Preg_FraseImpacto`, `Preg_PreguntasNecesidad`, `Preg_ConexionBeneficio`, `Preg_ExplicacionProducto`, `Preg_ManejoObjeciones`, `Preg_CierreComercial`, `Preg_ConfirmacionCierre`.

### Conteo real (recalculado sobre los 3 registros actuales de `Matriz de calidad`)

| Columna | Celdas `"N/A"` convertidas a `null` |
|---|---|
| Preg_TonoSaludo | 0 |
| Preg_FraseImpacto | 0 |
| Preg_PreguntasNecesidad | 1 |
| Preg_ConexionBeneficio | 1 |
| Preg_ExplicacionProducto | 1 |
| Preg_ManejoObjeciones | 0 |
| Preg_CierreComercial | 2 |
| Preg_ConfirmacionCierre | 2 |
| **Total** | **7** |

No se convirtió ningún `"N/A"` a `0` (verificado explícitamente en el código: `Replacer.ReplaceValue` sustituye por `null`, no por `0`).

## Tratamientos aplicados en `Fact_SatisfaccionCapacitacion`

Se agregaron 6 pasos al final de la consulta existente, en este orden:

1. **`NombresPersonaNormalizados`** — aplica `Text.Trim(Text.Clean(...))` + `Text.Proper(...)` (formato tipo nombre propio) a `NombreAsesor`, `NombreLider` y `NombreFormador`, con protección ante nulos.
2. **`AliasLiderJuanEsteban`** — lista de las 4 variantes conocidas (ya normalizadas a formato propio) del líder `Juan Esteban Pérez Camargo`.
3. **`LiderConsolidado`** — reemplaza cualquier valor de `NombreLider` que coincida con la lista de alias por el nombre canónico `"Juan Esteban Pérez Camargo"`.
4. **`ColumnasCategoricasSinDato`** — lista de columnas categóricas/persona: `CallCenter`, `Jornada`, `NombreAsesor`, `NombreLider`, `NombreFormador`, `Duracion`.
5. **`NulosCategoricosMarcados`** — reemplaza `null` por `"Sin dato"` en esas 6 columnas (sin eliminar ninguna fila).
6. **`ComentarioUnificado`** — unifica `Comentario` (ver sección de respuestas abiertas más abajo).

### Decisión sobre valores nulos en `CallCenter`, `Jornada` y `NombreAsesor`

**No se eliminó ninguna fila.** Se confirmó que exactamente **1 fila de 32** tiene `CallCenter`, `Jornada`, `NombreAsesor`, `NombreLider` y `NombreFormador` en blanco simultáneamente (es la misma fila de respuesta incompleta ya detectada en el diagnóstico original, `Specs/01` §3.2). Esa fila permanece en el modelo; sus valores nulos en esas 5 columnas ahora se muestran como `"Sin dato"` en vez de en blanco, para que no rompa ni distorsione agrupaciones visuales (ej. una tabla "por call center" no mostrará una fila vacía sin etiqueta).

`Duracion` tiene **2 nulos** en total: la misma fila incompleta, más 1 fila adicional distinta donde solo `Duracion` está en blanco (el resto de esa fila sí tiene datos). Ambas quedan como `"Sin dato"` en esa columna.

### Conteo de filas afectadas por "Sin dato"

| Columna | Filas marcadas como "Sin dato" |
|---|---|
| CallCenter | 1 |
| Jornada | 1 |
| NombreAsesor | 1 |
| NombreLider | 1 |
| NombreFormador | 1 |
| Duracion | 2 |

### Tabla de alias aplicada para `NombreLider`

| Variante (post Trim/Clean/Proper) | Se consolida en |
|---|---|
| `Juan Esteban Perez Camargo` | `Juan Esteban Pérez Camargo` |
| `Juan Esteban Pérez Caramargo` | `Juan Esteban Pérez Camargo` |
| `Juan Esteban Camargo` | `Juan Esteban Pérez Camargo` |
| `Juan Esteban Cámargo` | `Juan Esteban Pérez Camargo` |

Estas son exactamente las 4 variantes ya identificadas en `Specs/01` §3.2; se confirmó releyendo los datos reales que **siguen siendo las mismas 4** (no aparecieron variantes adicionales de este líder específico más allá de las ya conocidas — respuesta a la instrucción 16: no hubo nada nuevo que agregar a este mapeo puntual).

### ⚠️ Hallazgo adicional no solicitado explícitamente — requiere tu decisión

Al recalcular los 22 valores distintos reales de `NombreLider` (el diagnóstico original de `Specs/01` solo había mostrado una muestra de 5 de 24), se detectaron **dos grupos adicionales de variantes que no fueron consolidados en esta fase**, porque la instrucción 16 solo pedía variantes "del mismo líder" (Juan Esteban) y consolidar identidades de personas sin confirmación de negocio es una decisión sensible (riesgo de atribuir mal el desempeño a la persona equivocada):

**Grupo "María del Pilar" (11 variantes observadas)** — posible mismo líder, pero con **discrepancia de apellido** entre variantes (`Castañeda` vs. `Castaño`), lo cual podría ser un typo o podría ser información genuinamente distinta:
```
Maria del pila, Maria del pilar, MARIA DEL PILAR, Maria del Pilar Castañeda,
MARIA DEL PILAR CASTAÑO, María, María del Pilar, María Del Pilar,
María del Pilar castaño, MARÍA DEL PILAR CASTAÑO, Maria castaño,
MARIA CASTAÑO, Maria Castaño, María castaño, María Castaño, MARÍA CASTAÑO
```
**Grupo "Juan Pérez" (2 variantes)** — ambigüedad real: podría ser una forma abreviada de `Juan Esteban Pérez Camargo`, o podría ser una persona distinta con nombre similar:
```
JUAN PEREZ, JUAN PÉREZ
```

**No se aplicó ninguna consolidación automática a estos 2 grupos.** Quedan con sus valores originales (solo con Trim/Clean/Proper aplicado, sin fusionar). Se recomienda confirmar con negocio (dependencia D5 ya registrada en `Specs/02` §4) antes de consolidarlos en una fase posterior — un mapeo incorrecto podría mezclar el desempeño de dos personas distintas bajo un solo nombre, o mantener separado a alguien que en realidad es la misma persona.

## Tratamientos aplicados en `Fact_MotivacionActividad`

Se agregaron 2 pasos: la definición de la lista `ValoresSinComentario` y la unificación de `Comentario` (misma lógica que en `Fact_SatisfaccionCapacitacion`, ver sección de respuestas abiertas). No se aplicó ningún tratamiento de nulos en `CallCenter`/`Jornada` en esta consulta porque, a diferencia de `Fact_SatisfaccionCapacitacion`, no se detectaron valores nulos en esos campos en los 5 registros actuales.

### Confirmación de encuesta anónima

Se confirmó (releyendo `expressions.tmdl` y los datos reales) que:
- La columna `Nombre completo (Mayúscula)` **no existe** en `Fact_MotivacionActividad` — fue eliminada en la Fase 5 (`Table.RemoveColumns`) y **no se recreó** en esta fase.
- Se revalidó contra los datos reales: las 5 filas actuales tienen esa columna 100% vacía en la fuente original, confirmando que la decisión de la Fase 5 sigue siendo correcta.
- **Esta fuente no permite análisis por asesor individual** — la encuesta de motivación es anónima por diseño del formulario. Cualquier página o visual futuro construido sobre `Fact_MotivacionActividad` debe limitarse a desgloses por `CallCenter`/`Jornada`, nunca por asesor (ya documentado como limitación en `Specs/01` §3.3 y §6, y en `Outputs/06_mapeo_columnas_fase_5.md`).

## Tratamiento aplicado a comentarios abiertos

Misma lógica aplicada en `Fact_SatisfaccionCapacitacion.Comentario` y `Fact_MotivacionActividad.Comentario`:

```
Comentario = 
  si es null -> "Sin comentario"
  si (Trim) es cadena vacía -> "Sin comentario"
  si (Trim + mayúsculas) coincide con {"N/A","NA","NADA","NINGUNA","NO","SIN OBSERVACIONES"} -> "Sin comentario"
  en cualquier otro caso -> Trim + Clean del texto original (sin alterar el contenido real)
```

La comparación se hizo insensible a mayúsculas/minúsculas (`Text.Upper`) para capturar variantes como `"N/a"`, `"Na"`, `"Ninguna"` tal como aparecen realmente en los datos, sin exigir coincidencia exacta de caso.

### Conteo de valores modificados

**`Fact_SatisfaccionCapacitacion.Comentario`** (32 filas totales):
- Convertidos a `"Sin comentario"`: **19** (10 nulos/vacíos + 9 ocurrencias de `"N/a"`, `"N/A"`, `"Na"`, `"Nada"`, `"Ninguna"` repartidas en varias filas).
- Comentarios reales conservados (con Trim/Clean, sin alterar contenido): **13**.

**`Fact_MotivacionActividad.Comentario`** (5 filas totales):
- Convertidos a `"Sin comentario"`: **0**.
- Comentarios reales conservados: **5** (`Ok`, `Más premios`, `Dinero`, `Premios`, y una repetición).

**Nota de alcance:** se detectó 1 valor real (`"."`, un solo punto) en `Fact_SatisfaccionCapacitacion.Comentario` que semánticamente no aporta información pero **no está en la lista explícita de valores a unificar** de la instrucción — se conservó tal cual, sin extender la regla más allá de lo pedido. Queda como observación menor para una futura fase de limpieza de texto si se desea.

## Archivos modificados

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (se extendieron las 3 expresiones `Fact_*` con los pasos descritos arriba; `Base_*` y `*_Limpio` quedaron sin cambios).

## Errores encontrados y solución aplicada

- No se encontraron errores de sintaxis tras la revisión estructural (paréntesis y llaves balanceados: 78/78 y 82/82 respectivamente; 10 `lineageTag` para 10 expresiones, consistente).
- Se verificó con `grep` que no se introdujo `queryGroup` (causa del incidente de la Fase 3, ya corregido en `Outputs/04`).
- No se encontraron discrepancias entre los nombres de columna usados en el código M y los confirmados en los datos reales.

## Resultado del commit

- Mensaje: `refactor(powerquery): tratar nulos na alias y comentarios abiertos`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (modificado: pasos nuevos en las 3 expresiones `Fact_*`), `Outputs/08_resultado_fase_6_tratamiento_nulos_na_respuestas_abiertas.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 7

**Antes de avanzar:**
1. Confirmar en Power BI Desktop que las 3 consultas `Fact_*` siguen mostrando vista previa sin error tras estos cambios (recomendado dado que se agregaron varios pasos nuevos con transformaciones de tipo).
2. Decidir qué hacer con el hallazgo de los grupos "María del Pilar" y "Juan Pérez" — confirmarlos con negocio antes de una eventual consolidación en una fase futura (no bloquea la Fase 7, pero sí queda como deuda de calidad de datos pendiente).

Si la validación en Power BI Desktop es exitosa, el proyecto queda listo para la **Fase 7 — Creación de tablas de hechos** (activar la carga al modelo de `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad`, y comenzar con las dimensiones `Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`), tal como está definida en `Specs/02`. **No se avanzó a la Fase 7 en esta ejecución**, conforme a la restricción indicada.

---

*Documento generado como registro operativo de la Fase 6, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
