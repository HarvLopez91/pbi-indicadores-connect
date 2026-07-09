# Cierre — Fase 11: Validación visual de medidas en Power BI Desktop

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Cierre complementario de la Fase 11 — Validación de indicadores (ver [Outputs/15_resultado_fase_11_validacion_indicadores.md](15_resultado_fase_11_validacion_indicadores.md)) |
| Documentos de referencia | [Outputs/15_resultado_fase_11_validacion_indicadores.md](15_resultado_fase_11_validacion_indicadores.md), [Outputs/13_resultado_fase_10_medidas_dax.md](13_resultado_fase_10_medidas_dax.md) |
| Fecha de validación | 2026-07-08 |
| Realizada por | Usuario (validación visual directa en Power BI Desktop) |

---

## Confirmación de apertura correcta del PBIP

Confirmado. El usuario reportó que el PBIP abre correctamente en Power BI Desktop, sin errores de formato TMDL. Esto cierra formalmente la cadena de validación abierta desde `Outputs/14` (corrección del error de `description`): el archivo no solo parsea, sino que además calcula y muestra valores en el lienzo, lo cual es una confirmación más fuerte que la evidencia indirecta (resaves automáticos) usada en `Outputs/15`.

## Confirmación de existencia de las 25 medidas

Confirmado por dos vías:
1. **Estructural** (ya documentada en `Outputs/15`): 25 medidas repartidas en `_Medidas Generales` (7), `_Medidas Calidad` (6), `_Medidas Capacitacion` (6), `_Medidas Motivacion` (6).
2. **Visual**: la captura de pantalla del usuario muestra las 4 tablas de medidas expandidas en el panel de Datos (`_Medidas Calidad`, `_Medidas Capacitacion`, `_Medidas Generales`, `_Medidas Motivacion`), con sus medidas individuales listadas y visibles, confirmando que Power BI Desktop las reconoce como medidas utilizables (no como errores ni objetos ocultos indebidamente).

## Confirmación de validación visual realizada por el usuario

Confirmado. El usuario creó 4 tablas temporales en el lienzo de "Página 1" (una por grupo de medidas) y leyó los valores calculados directamente por el motor DAX de Power BI Desktop — la validación real que `Outputs/15` había dejado pendiente por no poder hacerla desde este entorno.

## Confirmación de coincidencia general entre valores esperados y valores observados

**Coincidencia general: Sí, con una excepción documentada** (`% Llamadas con Venta`, ver sección dedicada). De las medidas reportadas por el usuario, todas coinciden con los valores esperados de `Outputs/15` salvo esa.

## Tabla resumen de medidas principales validadas

| Medida | Valor esperado | Valor observado | Coincide |
|---|---|---|---|
| `Total Evaluaciones Calidad` | 3 | 3 | Sí |
| `Total Respuestas Capacitacion` | 32 | 32 | Sí |
| `Total Respuestas Motivacion` | 5 | 5 | Sí |
| `Total Registros Piloto` | 40 | 40 | Sí |
| `n Calidad` | "n=3" | "n=3" | Sí |
| `n Capacitacion` | "n=32" | "n=32" | Sí |
| `n Motivacion` | "n=5" | "n=5" | Sí |
| `Puntaje Obtenido Calidad` | 23 | 23,0 | Sí |
| `Preguntas Aplicables Calidad` | 17 | 17 | Sí |
| `Promedio Puntaje Calidad` | ≈1,35 | 1,4 | Sí (redondeo de formato a 1 decimal) |
| `% Llamadas con Venta` | 0,0% | **En blanco** | **No** — ver observación |
| `Objecion Principal` | "Muy caro" | "Muy caro" | Sí |
| `% Calidad Promedio Provisional` | En blanco (por diseño) | En blanco | Sí |
| `Indice Global Capacitacion` | ≈4,73 | ≈4,7 | Sí |
| `% Comentarios Capacitacion` | ≈40,6% | 40,6% | Sí |
| `Indice Global Motivacion` | ≈4,27 | ≈4,3 | Sí |
| `% Ambiente Motivado` | 40,0% | 40,0% | Sí |
| `% Comentarios Motivacion` | 100,0% | 100,0% | Sí |

Las demás medidas de capacitación/motivación (promedios individuales de `Satisfaccion`, `Claridad`, `Utilidad`, `Dinamismo`, `ClaridadUtilidad`, `Motivacion Promedio Actividad`) también aparecen visibles en la captura con valores consistentes con lo esperado (rango 3,8–4,8), aunque no se listaron uno a uno en el mensaje del usuario; se dan por validadas junto con sus índices globales, que sí dependen directamente de ellas y coinciden.

## Observación sobre `% Llamadas con Venta`

**Discrepancia confirmada: la medida aparece en blanco en Power BI Desktop, no como `0,0%`.**

Cálculo manual esperado: de las 3 evaluaciones actuales de `Fact_CalidadLlamadas`, las 3 tienen `TerminoEnVenta = "No"` (confirmado leyendo los datos fuente). La fórmula es:

```dax
% Llamadas con Venta =
    DIVIDE(
        CALCULATE(COUNTROWS(Fact_CalidadLlamadas), Fact_CalidadLlamadas[TerminoEnVenta] IN {"Sí", "Si"}),
        COUNTROWS(Fact_CalidadLlamadas)
    )
```

Matemáticamente, el numerador (`CALCULATE(COUNTROWS(...), TerminoEnVenta IN {"Sí","Si"})`) debería evaluar a `0` (ninguna de las 3 filas coincide con "Sí"/"Si"), y `DIVIDE(0, 3)` debería devolver `0`, mostrado como `0,0%` con el formato aplicado — **no `BLANK()`**, ya que `DIVIDE` solo devuelve blanco cuando el denominador es `0` (aquí el denominador es 3, no cero). Sin embargo, el resultado observado en Power BI Desktop es blanco.

**Hipótesis (sin confirmar, no aplicada):** es posible que `CALCULATE(COUNTROWS(...), columna IN {...})`, cuando ningún valor de la columna en toda la tabla coincide con el conjunto del filtro, devuelva `BLANK()` en el numerador en vez de `0` en esta versión del motor — en vez de `0` "real". Una alternativa más robusta frente a este tipo de casos extremos, que evita depender de `CALCULATE`+`COUNTROWS` sobre un filtro que puede resultar vacío, sería:

```dax
% Llamadas con Venta =
    VAR VentasConfirmadas =
        SUMX(Fact_CalidadLlamadas, IF(Fact_CalidadLlamadas[TerminoEnVenta] IN {"Sí", "Si"}, 1, 0))
    RETURN
        DIVIDE(VentasConfirmadas, COUNTROWS(Fact_CalidadLlamadas))
```

`SUMX` con un `IF` fila por fila garantiza que el numerador sea `0` (no blanco) cuando ninguna fila cumple la condición, ya que sigue sumando ceros explícitos en vez de depender de un conteo condicionado por `CALCULATE`.

**No se aplicó este cambio** — se documenta únicamente como propuesta para una fase de corrección posterior, con tu autorización explícita, tal como se pidió. Es una diferencia menor y no bloqueante: con el volumen piloto actual (3 evaluaciones, todas sin venta), tanto "0,0%" como "en blanco" comunican la misma realidad de negocio (ninguna venta registrada); la diferencia importa para la robustez de la medida a futuro, no para la interpretación actual de los datos.

## Confirmación de que los visuales temporales fueron eliminados del lienzo antes de avanzar

**Confirmado por evidencia de archivo, no solo por declaración del usuario.** Al revisar `git status` antes de esta ejecución, se encontró un cambio en `PBI/PBI_Indicadores.Report/definition/report.json`, pero al inspeccionar el diff se confirmó que se trata únicamente de un estado de interfaz trivial (`outspacePane` → `expanded: false`, el panel de filtros/salida colapsado). El archivo `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/page.json` — que es donde se persisten los visuales reales de la página — **permanece exactamente igual** al estado vacío original del proyecto (sin ningún visual). Esto confirma de forma independiente que las 4 tablas temporales de validación **no quedaron guardadas en el PBIP**: se usaron solo en el lienzo de Power BI Desktop para la lectura visual y no se persistieron (ya sea porque se eliminaron antes de guardar, o porque no llegaron a guardarse). Ese cambio trivial de `report.json` se sincronizó en un commit separado (`ab7d77e`) antes de este documento.

## Archivos modificados en esta ejecución

Ninguno propio de este cierre más allá de la documentación. El único cambio de código fue la sincronización ya comiteada por separado (`ab7d77e`, cambio de UI trivial en `report.json`).

## Resultado del commit

- Mensaje: `test(dax): cerrar validacion visual de medidas`.
- Archivos incluidos: `Outputs/16_cierre_fase_11_validacion_visual_powerbi.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar a Fase 12

**Sí, se recomienda avanzar a la Fase 12.** La validación de la Fase 11 queda cerrada con 24 de 25 medidas confirmadas visualmente coincidentes con lo esperado, y la única discrepancia (`% Llamadas con Venta` en blanco en vez de `0,0%`) es menor, no bloqueante, y ya queda documentada con una propuesta de corrección concreta pendiente de tu autorización para una fase posterior (no necesariamente antes de continuar con el diseño de páginas/tema visual). Recomiendo llevar el ajuste de `% Llamadas con Venta` como una tarea puntual registrada, a resolver cuando se retomen ajustes de medidas o en una fase de pulido antes de la entrega final — no es necesario detener el avance del informe por esto.

---

*Documento generado como registro operativo complementario de la Fase 11, según la regla documental vigente: los resultados de ejecución/validación de fases se documentan en `Outputs/`.*
