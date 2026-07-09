# Resultado — Fase 10: Creación de medidas DAX

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 10 — Creación de medidas DAX (ver [Specs/02_plan_implementacion_informe_powerbi_connect.md](../Specs/02_plan_implementacion_informe_powerbi_connect.md)) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, [Outputs/12_resultado_fase_9_modelo_relaciones.md](12_resultado_fase_9_modelo_relaciones.md) |
| Fecha | 2026-07-08 |
| Archivos nuevos | `tables/_Medidas Generales.tmdl`, `tables/_Medidas Calidad.tmdl`, `tables/_Medidas Capacitacion.tmdl`, `tables/_Medidas Motivacion.tmdl` |
| Archivo modificado | `model.tmdl` |

---

## Sincronización previa con Power BI Desktop

Antes de empezar, `git status` mostró un archivo nuevo que no había creado yo: `PBI/PBI_Indicadores.SemanticModel/diagramLayout.json`. Se investigó su contenido: es el metadato de posición/tamaño de los nodos en la vista de Modelo de Power BI Desktop (coordenadas `x`/`y`, zoom), generado al ver el diagrama tras confirmar visualmente las relaciones de la Fase 9. No forma parte de la definición TMDL del modelo y no tiene impacto funcional. Se comiteó por separado (commit `3a974a1`) antes de iniciar el trabajo propio de esta fase, siguiendo el mismo criterio de las fases anteriores.

## Estado inicial de `git status` (para la Fase 10 en sí)

Tras el commit de sincronización, el working tree quedó limpio antes de crear las medidas.

## Confirmación de modelo actualizado

- Se verificó que `model.tmdl` contiene `ref table` para las 6 tablas requeridas (`Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`, `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`).
- Se verificó que `relationships.tmdl` contiene las 8 relaciones creadas en la Fase 9 (más las 3 automáticas de Auto Date/Time, sin relación con esta fase).
- No fue posible abrir Power BI Desktop directamente desde este entorno para confirmar la alerta de "relaciones modificadas, actualizar manualmente" — el usuario ya había confirmado visualmente que el modelo y las relaciones abren correctamente antes de esta ejecución (contexto de la tarea), lo cual se tomó como la confirmación de partida. Ver sección de recomendaciones para la validación pendiente en Desktop tras esta fase.

## Tablas donde quedaron ubicadas las medidas

Se crearon las 4 tablas de medidas pedidas, **desde TMDL**, evaluando que el riesgo era manejable: se replicó exactamente el patrón ya probado con éxito en `Dim_CallCenter`/`Dim_Jornada` (tabla + columna + partición `mode: import`), usando una columna placeholder oculta y una fuente M trivial de 0 filas, en vez de experimentar con una tabla sin ninguna columna (territorio no probado en este proyecto).

| Tabla | Archivo | Medidas | Columna placeholder |
|---|---|---|---|
| `_Medidas Generales` | `tables/_Medidas Generales.tmdl` | 7 | `Columna1` (oculta, `int64`) |
| `_Medidas Calidad` | `tables/_Medidas Calidad.tmdl` | 6 | `Columna1` (oculta, `int64`) |
| `_Medidas Capacitacion` | `tables/_Medidas Capacitacion.tmdl` | 6 | `Columna1` (oculta, `int64`) |
| `_Medidas Motivacion` | `tables/_Medidas Motivacion.tmdl` | 6 | `Columna1` (oculta, `int64`) |

La tabla en sí **no** está oculta (para que las medidas sean visibles y usables en el panel de campos); solo se ocultó la columna placeholder `Columna1`, que no tiene ningún uso salvo darle una forma válida a la tabla. Se agregaron las 4 referencias correspondientes a `model.tmdl` (`ref table '_Medidas Generales'`, etc., con comillas simples por el espacio en el nombre).

## Medidas creadas por categoría

### Medidas generales (`_Medidas Generales`)

| Medida | Fórmula DAX | Formato |
|---|---|---|
| `Total Evaluaciones Calidad` | `COUNTROWS(Fact_CalidadLlamadas)` | `0` |
| `Total Respuestas Capacitacion` | `COUNTROWS(Fact_SatisfaccionCapacitacion)` | `0` |
| `Total Respuestas Motivacion` | `COUNTROWS(Fact_MotivacionActividad)` | `0` |
| `Total Registros Piloto` | `[Total Evaluaciones Calidad] + [Total Respuestas Capacitacion] + [Total Respuestas Motivacion]` | `0` |
| `n Calidad` | `"n=" & [Total Evaluaciones Calidad]` | texto |
| `n Capacitacion` | `"n=" & [Total Respuestas Capacitacion]` | texto |
| `n Motivacion` | `"n=" & [Total Respuestas Motivacion]` | texto |

### Medidas de calidad de llamadas (`_Medidas Calidad`)

| Medida | Fórmula DAX (resumida) | Formato |
|---|---|---|
| `Puntaje Obtenido Calidad` | Suma de `SUM()` de las 8 columnas `Preg_*` de `Fact_CalidadLlamadas` | `0.0` |
| `Preguntas Aplicables Calidad` | Suma de `COUNT()` de las 8 columnas `Preg_*` | `0` |
| `Promedio Puntaje Calidad` | `DIVIDE([Puntaje Obtenido Calidad], [Preguntas Aplicables Calidad])` | `0.0` |
| `% Llamadas con Venta` | `DIVIDE(CALCULATE(COUNTROWS(...), TerminoEnVenta IN {"Sí","Si"}), COUNTROWS(...))` | `0.0%` |
| `Objecion Principal` | `VAR`/`FILTER`/`ADDCOLUMNS`/`TOPN`/`MAXX` sobre `ObjecionPrincipal`, excluyendo blancos | (sin formato numérico, texto) |
| `% Calidad Promedio Provisional` | `BLANK()` — ver sección de medidas provisionales | `0.0%` |

Fórmula completa de `Puntaje Obtenido Calidad` (las demás de esta tabla siguen el mismo patrón de las 8 columnas):
```dax
Puntaje Obtenido Calidad =
    SUM(Fact_CalidadLlamadas[Preg_TonoSaludo]) +
    SUM(Fact_CalidadLlamadas[Preg_FraseImpacto]) +
    SUM(Fact_CalidadLlamadas[Preg_PreguntasNecesidad]) +
    SUM(Fact_CalidadLlamadas[Preg_ConexionBeneficio]) +
    SUM(Fact_CalidadLlamadas[Preg_ExplicacionProducto]) +
    SUM(Fact_CalidadLlamadas[Preg_ManejoObjeciones]) +
    SUM(Fact_CalidadLlamadas[Preg_CierreComercial]) +
    SUM(Fact_CalidadLlamadas[Preg_ConfirmacionCierre])
```
`SUM()` ignora automáticamente los `null` (los "N/A" ya convertidos en la Fase 6) sin convertirlos en error ni en penalización — exactamente lo pedido. `Preguntas Aplicables Calidad` usa la misma estructura con `COUNT()` en vez de `SUM()`.

Fórmula completa de `Objecion Principal`:
```dax
Objecion Principal =
    VAR ResumenObjeciones =
        FILTER(
            ADDCOLUMNS(
                VALUES(Fact_CalidadLlamadas[ObjecionPrincipal]),
                "Conteo", CALCULATE(COUNTROWS(Fact_CalidadLlamadas))
            ),
            NOT ISBLANK(Fact_CalidadLlamadas[ObjecionPrincipal]) && Fact_CalidadLlamadas[ObjecionPrincipal] <> ""
        )
    RETURN
        IF(
            ISEMPTY(ResumenObjeciones),
            BLANK(),
            MAXX(TOPN(1, ResumenObjeciones, [Conteo], DESC), Fact_CalidadLlamadas[ObjecionPrincipal])
        )
```

### Medidas de satisfacción de capacitaciones (`_Medidas Capacitacion`)

| Medida | Fórmula DAX | Formato |
|---|---|---|
| `Satisfaccion Promedio Capacitacion` | `AVERAGE(Fact_SatisfaccionCapacitacion[SatisfaccionGeneral])` | `0.0` |
| `Claridad Promedio Capacitacion` | `AVERAGE(Fact_SatisfaccionCapacitacion[Claridad])` | `0.0` |
| `Utilidad Promedio Capacitacion` | `AVERAGE(Fact_SatisfaccionCapacitacion[Utilidad])` | `0.0` |
| `Dinamismo Promedio Capacitacion` | `AVERAGE(Fact_SatisfaccionCapacitacion[Dinamismo])` | `0.0` |
| `Indice Global Capacitacion` | `DIVIDE([Satisfaccion...] + [Claridad...] + [Utilidad...] + [Dinamismo...], 4)` | `0.0` |
| `% Comentarios Capacitacion` | `DIVIDE(CALCULATE(COUNTROWS(...), Comentario <> "Sin comentario"), COUNTROWS(...))` | `0.0%` |

### Medidas de motivación y actividad comercial (`_Medidas Motivacion`)

| Medida | Fórmula DAX | Formato |
|---|---|---|
| `Satisfaccion Promedio Actividad` | `AVERAGE(Fact_MotivacionActividad[SatisfaccionGeneral])` | `0.0` |
| `Claridad Utilidad Promedio Actividad` | `AVERAGE(Fact_MotivacionActividad[ClaridadUtilidad])` | `0.0` |
| `Motivacion Promedio Actividad` | `AVERAGE(Fact_MotivacionActividad[MotivacionPostActividad])` | `0.0` |
| `Indice Global Motivacion` | `DIVIDE([Satisfaccion...] + [Claridad Utilidad...] + [Motivacion...], 3)` | `0.0` |
| `% Ambiente Motivado` | `DIVIDE(CALCULATE(COUNTROWS(...), AmbienteEquipo = "Motivado"), COUNTROWS(...))` | `0.0%` |
| `% Comentarios Motivacion` | `DIVIDE(CALCULATE(COUNTROWS(...), Comentario <> "Sin comentario"), COUNTROWS(...))` | `0.0%` |

**Total: 25 medidas** (7 + 6 + 6 + 6), confirmado por conteo automático en los 4 archivos.

## Formato aplicado

- Conteos (`Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`, `Total Registros Piloto`, `Preguntas Aplicables Calidad`): `formatString: 0` (entero sin decimales).
- Promedios/puntajes (Likert y `Puntaje Obtenido Calidad`/`Promedio Puntaje Calidad`): `formatString: 0.0` (1 decimal).
- Porcentajes: `formatString: 0.0%` (1 decimal).
- Medidas de texto (`n Calidad`, `n Capacitacion`, `n Motivacion`, `Objecion Principal`): sin `formatString` numérico — su tipo de resultado es texto por la propia expresión DAX (concatenación con `&` o valor extraído de una columna de texto), tal como corresponde en TOM.

## Descripciones agregadas

Se agregó `description:` a las 25 medidas (propiedad estándar de TOM/`MetadataObject`, de menor riesgo que `lineageTag` por ser una propiedad universal de texto, no una referencia de identidad). Cada descripción indica qué mide, la fuente/tabla usada y, cuando aplica, la limitación de volumen piloto o el carácter provisional. Ejemplos:

- `Total Respuestas Motivacion`: *"Número total de respuestas a la encuesta de motivación de actividades comerciales (Fact_MotivacionActividad). Volumen piloto actual: 5 respuestas - muestra muy pequeña."*
- `Satisfaccion Promedio Actividad`: *"...Basado en Fact_MotivacionActividad (n=5 actualmente - muestra muy pequeña, interpretar con cautela)."*
- `% Calidad Promedio Provisional`: ver sección siguiente (descripción completa del supuesto pendiente).

## Medidas provisionales o pendientes de confirmación

**`% Calidad Promedio Provisional`** — se creó la medida (para dejar el nombre y el lugar reservados en el modelo, tal como sugiere la instrucción), pero su fórmula es literalmente `BLANK()`, es decir, **siempre en blanco a propósito**. Motivo: las 8 columnas `Preg_*` del checklist de calidad tienen puntajes máximos distintos entre sí (observado en los datos: algunas preguntas llegan a 1 punto, otras a 3), y la rúbrica oficial de puntaje máximo por pregunta sigue pendiente de confirmación de negocio (dependencia **D3**, `Specs/02` §4). Sin esa rúbrica, cualquier normalización de `Promedio Puntaje Calidad` a un porcentaje sería un supuesto arbitrario que podría inducir a error sobre la calidad real. Se documenta explícitamente en la `description` de la medida y aquí: **no debe presentarse como indicador definitivo**; debe reemplazarse por la fórmula correcta una vez la rúbrica esté confirmada.

No hay otras medidas provisionales en esta fase — las 24 restantes se calculan directamente sobre datos ya limpios y sin supuestos pendientes, aunque **todas** heredan la limitación general de bajo volumen piloto (documentada en `Specs/02` y reflejada en las medidas `n Calidad`/`n Capacitacion`/`n Motivacion`).

## Validación de conteos base

Recalculado sobre los datos reales actuales (mismo método usado en fases anteriores, ya que no puedo ejecutar el motor DAX de Power BI Desktop directamente):

| Medida | Valor esperado |
|---|---|
| `Total Evaluaciones Calidad` | **3** ✅ |
| `Total Respuestas Capacitacion` | **32** ✅ |
| `Total Respuestas Motivacion` | **5** ✅ |
| `Total Registros Piloto` | **40** |

Adicionalmente, se calcularon a mano (Python, replicando la lógica DAX) los valores esperados del resto de medidas, para poder verificarlas apenas se confirme en Power BI Desktop:

| Medida | Valor esperado |
|---|---|
| `Puntaje Obtenido Calidad` | 23 |
| `Preguntas Aplicables Calidad` | 17 |
| `Promedio Puntaje Calidad` | ≈1.35 |
| `% Llamadas con Venta` | 0.0% (las 3 evaluaciones actuales son "No") |
| `Objecion Principal` | "Muy caro" (2 de 3 objeciones registradas) |
| `Satisfaccion/Claridad/Utilidad/Dinamismo Promedio Capacitacion` | ≈4.77 / 4.65 / 4.74 / 4.74 |
| `Indice Global Capacitacion` | ≈4.73 |
| `% Comentarios Capacitacion` | 13/32 ≈ 40.6% |
| `Satisfaccion/Claridad Utilidad/Motivacion Promedio Actividad` | 4.6 / 4.4 / 3.8 |
| `Indice Global Motivacion` | ≈4.27 |
| `% Ambiente Motivado` | 2/5 = 40.0% |
| `% Comentarios Motivacion` | 5/5 = 100.0% |

## Errores encontrados y solución aplicada

- **No se encontró ningún error de sintaxis** en la revisión estructural. Se validó con Python: paréntesis y llaves balanceados en los 4 archivos (`_Medidas Generales.tmdl` 11/11 y 1/1; `_Medidas Calidad.tmdl` 43/43 y 2/2; `_Medidas Capacitacion.tmdl` 17/17 y 1/1; `_Medidas Motivacion.tmdl` 22/22 y 1/1), 25 medidas definidas y localizadas correctamente en los 4 archivos.
- **Decisión de diseño para minimizar riesgo:** para las medidas DAX multilínea (`Puntaje Obtenido Calidad`, `Preguntas Aplicables Calidad`, `Objecion Principal`), inicialmente se consideró usar el delimitador de comillas triples (```` ``` ````) documentado en TMDL para expresiones verbatim, pero se descartó por no tener un ejemplo 100% inequívoco de su sintaxis exacta en la documentación revisada. En su lugar, se usó el patrón de indentación simple (sin comillas triples) ya probado con éxito 6+ veces en este proyecto para el código M de `Fact_*`/`Dim_*`, aplicando la misma regla de indentación (cuerpo multilínea = 2 niveles más profundo que la línea de declaración de la medida).
- Se verificaron las referencias cruzadas entre medidas (`[Total Evaluaciones Calidad]` dentro de `Total Registros Piloto`, `[Puntaje Obtenido Calidad]`/`[Preguntas Aplicables Calidad]` dentro de `Promedio Puntaje Calidad`, etc.) contra los 25 nombres de medida definidos — todas coinciden exactamente. La única coincidencia marcada por la verificación automática (`[Conteo]` dentro de `Objecion Principal`) es una referencia válida a una columna virtual creada por `ADDCOLUMNS` dentro de la misma medida, no una medida del modelo — falso positivo esperado de la heurística de verificación, no un error real.
- No se agregó ninguna línea `lineageTag` ni `queryGroup` — confirmado por búsqueda global en los 4 archivos nuevos (0 coincidencias en ambos casos).

## Archivos modificados en el PBIP

- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Generales.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Calidad.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Motivacion.tmdl` (nuevo).
- `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (modificado: 4 líneas `ref table` agregadas).

*(En un commit previo y separado (`3a974a1`) se incorporó la sincronización de `diagramLayout.json` generado por Power BI Desktop — ver sección correspondiente arriba.)*

No se crearon relaciones nuevas, no se crearon visuales, no se modificaron páginas del reporte ni archivos `Data/*.xlsx`.

## Resultado del commit

- Mensaje: `feat(dax): crear medidas base del informe connect`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Generales.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Calidad.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Motivacion.tmdl` (nuevo), `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl` (modificado), `Outputs/13_resultado_fase_10_medidas_dax.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 11

**Antes de avanzar:**
1. Cerrar y volver a abrir Power BI Desktop y confirmar que el PBIP abre sin errores tras agregar las 25 medidas.
2. Si aparece la alerta de "una o varias relaciones se han modificado y es necesario actualizar manualmente", ejecutar la actualización antes de validar las medidas.
3. Confirmar en el panel de campos que las 4 tablas de medidas aparecen (visibles, con sus medidas), y que la columna `Columna1` de cada una está oculta.
4. Colocar cada medida en una tarjeta o tabla de prueba temporal y comparar contra los valores esperados de la tabla de validación de esta fase (eliminar la tarjeta/tabla de prueba después de confirmar — no debe quedar como visual permanente).
5. Confirmar especialmente que `% Calidad Promedio Provisional` se muestra en blanco (comportamiento esperado, no un error).

Si la validación es exitosa, el proyecto queda listo para la **Fase 11 — Validación de indicadores con los datos actuales** (que en este proyecto se adelantó parcialmente en esta misma fase mediante el cálculo manual de valores esperados, pero la confirmación visual en Power BI Desktop sigue pendiente). **No se avanzó a la Fase 11 en esta ejecución**, conforme a la restricción indicada.

---

*Documento generado como registro operativo de la Fase 10, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
