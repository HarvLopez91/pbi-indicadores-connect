# Resultado — Fase SC-3: creación de medidas DAX (Satisfacción de capacitaciones)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | `SC-3` de [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Fases previas cerradas | `SC-1` (decisiones DEC-1 a DEC-4 confirmadas), `SC-2` (preparación técnica validada) |
| Fecha | 2026-07-21 |
| Alcance | Modificación de una sola tabla de medidas. No se modificó PBIR, Power Query, relaciones, visuales ni `Data/*.xlsx`. |

---

## Corrección posterior (validada en Power BI Desktop durante SC-4)

La validación manual de §3 originalmente reportaba `Ultima Capacitacion = 2026-10-07`, calculado leyendo directamente el XML interno del `.xlsx` sin librerías externas (openpyxl/pandas no estaban disponibles en este entorno). Ese cálculo externo **interpretó incorrectamente la fecha**.

Al ejecutar la misma medida dentro de Power BI Desktop (fase `SC-4`, ver `Outputs/40` §10), el resultado real del modelo es **`10/07/2026` (10 de julio de 2026)**, valor que además cae dentro del rango visible del segmentador de Fecha de la página (`02/07/2026`–`15/07/2026`) — consistente con el resto de los datos piloto, sin fechas futuras ni anómalas.

**El resultado del modelo en Power BI Desktop es la fuente de validación definitiva**, no la lectura externa del XML. Las secciones §3, §4 y §5 de este documento se corrigen a continuación para reflejar el valor correcto; el texto original tachado se conserva como referencia de lo que se reportó inicialmente y por qué se corrigió.

Esta corrección **no afecta** a `Call Centers Capacitados` (5) ni a `Capacitaciones Realizadas` (5) — ambas fueron confirmadas correctas por Power BI Desktop, coincidiendo con el cálculo manual original.

## 1. Estado inicial de `git status`

```
?? Data/
```

Working tree limpio salvo `Data/` (sin seguimiento por diseño), confirmado en `SC-2` y sin cambios desde entonces.

## 2. Medidas creadas

Las 3 medidas se agregaron a `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl`, después de `% Comentarios Capacitacion` (última medida existente) y antes de `column Columna1`. Ninguna incluye `lineageTag`, `description` ni `queryGroup` escritos a mano (Power BI Desktop los generará al guardar, según la convención del proyecto).

### `Call Centers Capacitados`
```dax
Call Centers Capacitados = DISTINCTCOUNT(Fact_SatisfaccionCapacitacion[CallCenter])
```
- `formatString: 0`

### `Ultima Capacitacion`
```dax
Ultima Capacitacion = MAX(Fact_SatisfaccionCapacitacion[Fecha])
```
- `formatString: Short Date`

### `Capacitaciones Realizadas`
```dax
Capacitaciones Realizadas =
    COUNTROWS(
        SUMMARIZE(
            Fact_SatisfaccionCapacitacion,
            Fact_SatisfaccionCapacitacion[Fecha],
            Fact_SatisfaccionCapacitacion[CallCenter],
            Fact_SatisfaccionCapacitacion[NombreFormador]
        )
    )
```
- `formatString: 0`
- Implementada con `COUNTROWS(SUMMARIZE(...))`, no con `DISTINCTCOUNTX`, conforme a la instrucción explícita del usuario.
- Clave de "capacitación única" = `Fecha + CallCenter + NombreFormador`, según DEC-1 (`SC-1`).
- **No se crearon medidas separadas** para "por fecha" ni "por call center": esta medida se reutilizará en `SC-5` colocando `Fecha` o `CallCenter` en el eje del visual correspondiente, tal como recomienda `Specs/05` §6.

## 3. Validación manual (fuente de datos real, sin exponer nombres)

Dado que este entorno no puede operar la interfaz de Power BI Desktop, la validación manual se hizo leyendo directamente `Data/Satisfacción capacitación (Responses).xlsx` (formato XML interno del `.xlsx`, sin librerías externas) y reproduciendo la misma lógica de agregación que cada medida nueva, **sin imprimir ningún nombre de formador/líder/asesor individual** en ningún momento (solo conteos agregados, consistente con el manejo de datos personales ya establecido en el proyecto).

| Medida | Cálculo manual esperado | Método |
|---|---|---|
| `Total Respuestas Capacitacion` (ya existente, usada como control) | 84 filas | Conteo directo de filas de datos en la hoja `Form Responses 1` |
| `Call Centers Capacitados` | **5** (`ATENTO`, `BRM`, `GNP`, `INTERACTIVO`, `ONE CONTACT`) | Valores distintos de la columna `CallCenter`, normalizados a mayúsculas |
| `Ultima Capacitacion` | ~~2026-10-07~~ → **corregido: `Ultima Capacitacion` real confirmada en Power BI Desktop = 10/07/2026 (10 de julio de 2026)**, ver "Corrección posterior" al inicio del documento | Máximo de la fecha (parte de fecha del `Timestamp`, sin hora) entre las 84 filas — el valor original de esta fila fue calculado incorrectamente por la lectura externa del XML, no por la fórmula DAX |
| `Capacitaciones Realizadas` | **5** | Combinaciones únicas de (Fecha, CallCenter, NombreFormador\*) entre las 84 filas — cada una de las 5 fechas observadas corresponde a un único call center y un único formador, con entre 7 y 24 respuestas cada una |

\* La columna `NombreFormador` se usó solo internamente (nunca impresa) para construir la clave de agrupación.

**Resultado esperado vs. medida:** las fórmulas DAX creadas replican exactamente la misma lógica de agregación usada en esta validación manual (`DISTINCTCOUNT` sobre `CallCenter`, `MAX` sobre `Fecha`, conteo de combinaciones únicas de las 3 columnas de la clave). No hay divergencia lógica entre la fórmula y el cálculo manual. **Confirmado en vivo en `SC-4`** (ver `Outputs/40` §10): Power BI Desktop calculó `Call Centers Capacitados = 5`, `Capacitaciones Realizadas = 5` y `Ultima Capacitacion = 10/07/2026` — las 3 medidas funcionan correctamente. Solo el valor de `Ultima Capacitacion` reportado originalmente en esta tabla (2026-10-07) era incorrecto, por un error de la validación externa por XML, no de la medida.

## 4. Observaciones y riesgos documentados

- **`Capacitaciones Realizadas` depende de una clave compuesta provisional (DEC-1), no de un identificador real de sesión.** Si `NombreFormador` tuviera variantes de escritura no cubiertas por la tabla de alias (dependencia D5), el conteo podría sobreestimarse — en los datos actuales no se observó ese riesgo (cada una de las 5 fechas mapea a una única combinación de call center/formador), pero sigue siendo un supuesto de negocio, no un hecho verificado.
- **`Call Centers Capacitados` no excluye explícitamente `"Sin dato"`.** Actualmente hay **0 filas** con `CallCenter` vacío en la fuente, por lo que la medida no está inflada hoy. Si en el futuro aparece una respuesta sin `CallCenter` (que Power Query marca como `"Sin dato"`), la medida contaría ese valor como si fuera un call center real, sumando 1 de más. No se agregó un filtro adicional no solicitado — se documenta como riesgo latente a monitorear en la próxima actualización de datos.
- ~~`Ultima Capacitacion` = 2026-10-07 es posterior a la fecha de referencia del proyecto~~ — **riesgo descartado**: el valor real confirmado en Power BI Desktop es `10/07/2026` (10 de julio de 2026), anterior a la fecha de referencia del proyecto (2026-07-21) y dentro del rango visible del segmentador de Fecha de la página (`02/07/2026`–`15/07/2026`). No hay evidencia de datos de prueba ni fechas anómalas en el dataset piloto — el riesgo señalado originalmente fue producto de un error en la validación externa por XML, no un hallazgo real sobre los datos.
- **Discrepancia de nombre de call center**: los datos reales muestran `INTERACTIVO`, mientras el mockup de referencia usa `INTERASEO` — confirma lo ya señalado en `Specs/05`: los valores del mockup son ilustrativos de diseño, no deben tomarse como el resultado exacto esperado.

## 5. Resumen de validaciones

| Validación | Resultado |
|---|---|
| Archivo modificado | `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl` (único archivo) |
| Medidas creadas | `Call Centers Capacitados`, `Ultima Capacitacion`, `Capacitaciones Realizadas` (3 medidas) |
| Medidas existentes modificadas | No — las 6 medidas previas de `_Medidas Capacitacion` y las 25 medidas del catálogo original quedan intactas (confirmado por `git diff`, solo inserciones) |
| Cambios en PBIR | No (`git status --porcelain -- "PBI/PBI_Indicadores.Report/"` sin salida) |
| Cambios en Power Query / relaciones | No (`expressions.tmdl` y `relationships.tmdl` sin cambios) |
| Cambios en `Data/` | No (`Data/` sigue sin seguimiento, ningún archivo agregado al índice) |
| Resultado manual vs. medida | Confirmado en vivo en Power BI Desktop durante `SC-4`: `Call Centers Capacitados = 5`, `Capacitaciones Realizadas = 5`, `Ultima Capacitacion = 10/07/2026` (corregido; el valor original de 2026-10-07 reportado aquí fue un error de la validación externa por XML, no de la medida — ver "Corrección posterior") |
| Riesgos documentados | 3 vigentes (clave compuesta provisional de DEC-1, posible inflación por `"Sin dato"` en `CallCenter`, discrepancia de nombre de call center entre mockup y datos reales) + 1 descartado tras corrección (fecha máxima posterior a "hoy" del proyecto — ya no aplica) |
| ¿Se puede avanzar a SC-4? | **Sí** |

## 6. Confirmación de no modificación fuera de alcance

No se modificó:

- Ningún archivo de `PBI_Indicadores.Report/` (PBIR, páginas, visuales).
- `expressions.tmdl` (Power Query) ni `relationships.tmdl`.
- `Fact_CalidadLlamadas.tmdl`, `Fact_MotivacionActividad.tmdl`, ni ninguna otra tabla de hechos/medidas fuera de `_Medidas Capacitacion.tmdl`.
- Ningún archivo `Data/*.xlsx` (solo se leyó, nunca se escribió).

## 7. Estado final de `git status`

```
 M "PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl"
?? Data/
```

## 8. Commit

```
feat(dax): agregar medidas para satisfaccion capacitaciones
```

No se hizo push remoto.

## 9. Recomendación para continuar

**Actualizado tras `SC-4`:** la recomendación original (avanzar a `SC-4` y confirmar las 3 medidas en Power BI Desktop) ya se ejecutó y quedó completa — ver `Outputs/40` §10. Las 3 medidas funcionan correctamente; el único punto que requería corrección era el valor de `Ultima Capacitacion` reportado en este documento (ya corregido arriba), no la fórmula. `SC-5` queda desbloqueada en lo relativo a estas medidas.
