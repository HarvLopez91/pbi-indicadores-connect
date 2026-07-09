# Resultado — Fase 9: Diseño del modelo de datos y relaciones

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 9 — Diseño del modelo de datos y relaciones (ver [Specs/02_plan_implementacion_informe_powerbi_connect.md](../Specs/02_plan_implementacion_informe_powerbi_connect.md)) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, [Outputs/11_resultado_fase_8_creacion_dimensiones.md](11_resultado_fase_8_creacion_dimensiones.md) |
| Fecha | 2026-07-08 |
| Archivo modificado | `relationships.tmdl` |

---

## Sincronización previa con Power BI Desktop

Antes de empezar, `git status` mostró cambios que no había hecho yo: al confirmar visualmente que `Dim_*`/`Fact_*` aparecen en el panel de datos, Power BI Desktop volvió a reescribir el modelo (mismo patrón que en la Fase 8) — agregó su propio `lineageTag` a las 3 tablas de dimensión y actualizó el orden de `ref table`/`PBI_QueryOrder` en `model.tmdl`. Se investigó, confirmó que era metadato sin impacto funcional, y se comiteó por separado (commit `1945440`) antes de iniciar el trabajo propio de esta fase, siguiendo el mismo criterio aplicado en la Fase 8.

## Estado inicial de `git status` (para la Fase 9 en sí)

Tras el commit de sincronización, el working tree quedó limpio antes de crear las relaciones.

## Verificación de prerrequisitos

- `model.tmdl` contiene `ref table` para las 6 tablas requeridas: `Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`, `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad` — confirmado.
- Los 6 archivos `tables/*.tmdl` correspondientes existen — confirmado.

## Relaciones creadas

Se agregaron 8 relaciones nuevas a `relationships.tmdl` (que ya contenía 3 relaciones automáticas de Power BI Desktop, ver sección de tablas automáticas de fecha más abajo), completando exactamente las 8 pedidas:

| # | Relación | Columnas |
|---|---|---|
| 1 | `Dim_Calendario` ↔ `Fact_CalidadLlamadas` | `Fecha` |
| 2 | `Dim_Calendario` ↔ `Fact_SatisfaccionCapacitacion` | `Fecha` |
| 3 | `Dim_Calendario` ↔ `Fact_MotivacionActividad` | `Fecha` |
| 4 | `Dim_CallCenter` ↔ `Fact_CalidadLlamadas` | `CallCenter` |
| 5 | `Dim_CallCenter` ↔ `Fact_SatisfaccionCapacitacion` | `CallCenter` |
| 6 | `Dim_CallCenter` ↔ `Fact_MotivacionActividad` | `CallCenter` |
| 7 | `Dim_Jornada` ↔ `Fact_SatisfaccionCapacitacion` | `Jornada` |
| 8 | `Dim_Jornada` ↔ `Fact_MotivacionActividad` | `Jornada` |

**No se creó** relación entre `Dim_Jornada` y `Fact_CalidadLlamadas` (esa fuente no tiene columna `Jornada`), tal como indicaba la instrucción.

## Cardinalidad y dirección de filtro aplicadas

Cada relación se escribió con únicamente `fromColumn` (lado `Fact_*`, columna no única) y `toColumn` (lado `Dim_*`, columna única) — y `joinOnDateBehavior: datePartOnly` en las 3 relaciones de calendario. **No se declaró explícitamente `fromCardinality`/`toCardinality`/`crossFilteringBehavior`.**

**Motivo de esta decisión:** repliqué exactamente el patrón mínimo que Power BI Desktop ya usa y valida con éxito en este mismo archivo para las 3 relaciones automáticas de "Auto Date/Time" (`fromColumn`/`toColumn`/`joinOnDateBehavior`, sin cardinalidad ni dirección de filtro explícitas). Tras el incidente de `lineageTag` en la Fase 7 —una propiedad documentada en la especificación general de TOM/TMDL pero no aceptada por el parser de esta versión de Power BI Desktop—, decidí no arriesgar con propiedades de relación (`fromCardinality`, `toCardinality`, `crossFilteringBehavior`) cuya grafía y contexto exactos en TMDL no pude confirmar con certeza total en la documentación oficial. Cuando una propiedad no está declarada, el motor de Analysis Services/Power BI la determina automáticamente a partir de los datos reales; para esta forma de relación (columna única en la dimensión, columna repetida en los hechos) el resultado esperado por defecto es exactamente `1:*` con dirección de filtro única (`Dim_* → Fact_*`), que es lo pedido. **Esto queda pendiente de tu confirmación visual** en la vista de modelo de Power BI Desktop (ver sección de validaciones).

## Confirmación de las 8 relaciones del modelo estrella

Confirmado por conteo exacto en el archivo: `relationships.tmdl` tiene 11 bloques `relationship` en total (3 automáticas preexistentes de Auto Date/Time + 8 nuevas de esta fase). Se verificaron una por una las 8 nuevas contra la lista pedida — coinciden exactamente, sin relaciones adicionales ni faltantes.

## Revisión de tablas automáticas de fecha

Se confirmó que Power BI Desktop había creado, de forma automática (función "Auto Date/Time", ya observada y documentada desde la Fase 7):

- `DateTableTemplate_2973bde6-872f-4cb8-9c9b-5dae0a9694b2.tmdl` — plantilla oculta base.
- `LocalDateTable_225f0da6-...tmdl`, `LocalDateTable_082769f1-...tmdl`, `LocalDateTable_c16eb748-...tmdl` — una tabla de calendario oculta por cada columna `Fecha` de las 3 `Fact_*`.
- 3 relaciones automáticas en `relationships.tmdl` (`Fact_*.Fecha → LocalDateTable_*.Date`).
- Un bloque `variation` agregado por Power BI Desktop dentro de la columna `Fecha` de cada tabla `Fact_*` (visible desde la sincronización de la Fase 8), que referencia esa relación y una jerarquía de fechas automática.

**Identificación confirmada con seguridad:** son artefactos 100% generados por Auto Date/Time (atributos `isHidden`, `isPrivate`/`showAsVariationsOnly`, `dataCategory: PaddedDateTableDates`, nomenclatura estándar) — no hay ambigüedad sobre su origen. **Y ahora son efectivamente redundantes**, ya que `Dim_Calendario` tiene relaciones reales con las 3 `Fact_*` desde esta misma fase.

## Decisión tomada sobre `LocalDateTable_*` / `DateTableTemplate_*`

**No se eliminaron.** Aunque identificar su origen es seguro, eliminarlas de forma completa y segura requiere más que borrar los 4 archivos de tabla: también habría que quitar sus 3 relaciones de `relationships.tmdl`, sus 4 líneas `ref table` de `model.tmdl`, **y el bloque `variation` que Power BI Desktop agregó dentro de la columna `Fecha` de cada `Fact_*.tmdl`** — si se elimina la tabla oculta pero se deja ese `variation` apuntando a una relación/tabla que ya no existe, se produce exactamente el mismo tipo de error de referencia huérfana que causó el incidente de `queryGroup` en la Fase 3-4. Dado que dos incidentes de TMDL ya ocurrieron en este proyecto por escribir/quitar propiedades sin certeza total del efecto en cascada, preferí no forzar esta limpieza coordinada de múltiples archivos en esta fase.

**Queda pendiente hacerlo manualmente en Power BI Desktop:** Archivo → Opciones y configuración → Opciones → Archivo actual → Carga de datos → desmarcar "Detección de tabla de fechas y hora automáticas para este archivo". Al deshabilitar esta opción y actualizar, Power BI Desktop debería ofrecer limpiar las tablas ocultas y sus relaciones/variaciones de forma consistente por su cuenta, evitando el riesgo de referencias huérfanas que implicaría hacerlo a mano en TMDL.

## Estado de marcado de `Dim_Calendario` como tabla de fechas

**No se hizo desde TMDL.** Se investigó la documentación oficial de TMDL/TOM buscando específicamente la propiedad y sintaxis para "Mark as Date Table"; toda la documentación encontrada describe el procedimiento desde la interfaz de Power BI Desktop o desde Visual Studio/SSDT (Extensiones → Tabla → Fecha → Marcar como tabla de fechas), sin confirmar la representación exacta en TMDL con el nivel de certeza que exige evitar un tercer incidente de sintaxis.

**Queda pendiente hacerlo manualmente en Power BI Desktop:** pestaña Modelado → seleccionar `Dim_Calendario` → "Marcar como tabla de fechas" → columna `Fecha`.

## Validación de ausencia de relaciones ambiguas

- **Sin relaciones activas entre tablas `Fact_*` entre sí** — confirmado programáticamente (se analizaron las 11 relaciones del archivo; ninguna tiene ambos extremos con prefijo `Fact_`).
- **Sin relaciones circulares** — cada `Fact_*` se conecta únicamente a las dimensiones correspondientes (`Dim_Calendario`, `Dim_CallCenter`, y `Dim_Jornada` donde aplica), nunca entre sí ni de vuelta a una dimensión ya conectada por otra ruta.
- **Sin rutas ambiguas**: no existe ningún par de tablas conectado por más de un camino de relaciones, por lo que no hay ambigüedad de filtrado posible en este modelo.
- Todas las relaciones de esta fase apuntan **desde** la tabla de hechos (`fromColumn`) **hacia** la dimensión (`toColumn`) — el sentido conceptual de filtrado (dimensión filtra a hechos) es el que corresponde a esta forma de relación por defecto del motor, no invertido.

## Validación de ausencia de `lineageTag` y `queryGroup` agregados manualmente

- Búsqueda global de `lineageTag` en `relationships.tmdl`: **0 coincidencias**.
- Búsqueda global de `queryGroup` en `relationships.tmdl`: **0 coincidencias**.
- No se modificó ningún archivo `tables/*.tmdl` en esta fase — por lo tanto no se introdujo ninguna línea `lineageTag`/`queryGroup` nueva en ningún otro archivo tampoco.

## Archivos modificados en el PBIP

- `PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl` (modificado: 8 relaciones nuevas agregadas a las 3 preexistentes).

*(En un commit previo y separado (`1945440`) se incorporó la sincronización automática de Power BI Desktop tras la Fase 8 — ver sección correspondiente arriba.)*

## Errores encontrados y solución aplicada

- No se encontraron errores durante la construcción de esta fase.
- Se aplicó la misma disciplina de las fases anteriores: no se agregó `lineageTag` ni `queryGroup`, y se evitó declarar propiedades de relación (`fromCardinality`, `toCardinality`, `crossFilteringBehavior`) sin certeza total de su sintaxis exacta en esta versión del parser de Power BI Desktop, prefiriendo el patrón mínimo ya comprobado como funcional en este mismo archivo.
- No fue necesario detenerse por ninguna referencia de columna inválida.

## Resultado del commit

- Mensaje: `feat(modelo): crear relaciones del modelo estrella`.
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl` (modificado), `Outputs/12_resultado_fase_9_modelo_relaciones.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` (respecto a los archivos de esta fase) — confirmado tras el commit. [CLAUDE.md](../CLAUDE.md) se gestiona en un commit separado, ver `Outputs/` u otro registro si aplica.

## Recomendación para avanzar o no a Fase 10

**Antes de avanzar:**
1. Cerrar y volver a abrir Power BI Desktop y confirmar que el PBIP abre sin errores.
2. En la vista de modelo, confirmar visualmente que aparecen las 8 relaciones nuevas, cada una mostrando cardinalidad `1` en el lado de la dimensión y `*` en el lado de hechos, con una sola flecha de dirección de filtro (de dimensión hacia hechos).
3. Confirmar que `Dim_CallCenter` filtra correctamente las 3 `Fact_*`, que `Dim_Jornada` filtra solo `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad`, y que `Dim_Calendario` filtra las 3 `Fact_*` (por ejemplo, con una tabla o segmentador de prueba temporal, eliminada después de confirmar).
4. Decidir si se procede con la limpieza manual de "Auto Date/Time" (deshabilitar la opción en Power BI Desktop) y el marcado manual de `Dim_Calendario` como tabla de fechas — ninguna de las dos es bloqueante para continuar, pero se recomiendan antes de construir medidas de inteligencia de tiempo en fases futuras.

Si la validación es exitosa, el proyecto queda listo para la **Fase 10 — Creación de medidas DAX**. **No se avanzó a la Fase 10 en esta ejecución**, conforme a la restricción indicada.

---

*Documento generado como registro operativo de la Fase 9, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
