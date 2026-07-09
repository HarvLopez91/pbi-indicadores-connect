# Resultado — Fase 11: Validación de indicadores con los datos actuales

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 11 — Validación de indicadores con los datos actuales (ver [Specs/02_plan_implementacion_informe_powerbi_connect.md](../Specs/02_plan_implementacion_informe_powerbi_connect.md)) |
| Documentos de referencia | [Outputs/13_resultado_fase_10_medidas_dax.md](13_resultado_fase_10_medidas_dax.md), [Outputs/14_correccion_fase_10_description_tmdl.md](14_correccion_fase_10_description_tmdl.md) |
| Fecha | 2026-07-08 |
| Archivos modificados en esta ejecución | Ninguno propio de la fase (ver sección de sincronización previa) |

---

## Nota metodológica importante — léase antes del resto del documento

Esta fase pide explícitamente crear visuales temporales en Power BI Desktop (`TMP_Validacion_Medidas`, tarjetas) y leer los valores que el motor DAX realmente calcula. **No tengo forma de abrir, interactuar con, ni observar la interfaz de Power BI Desktop desde este entorno** — limitación documentada de forma consistente en cada fase de este proyecto desde `Outputs/03`. Por esa misma razón, **decidí no crear los visuales temporales de validación**: editar a mano el JSON de una página del reporte para agregar una tabla/tarjetas que después no puedo ver ni leer no aporta ningún valor real de validación, y sí introduce un riesgo innecesario (sería el primer archivo de página de reporte editado a mano en todo este proyecto, un tipo de archivo distinto al TMDL, sin ninguna validación previa de su sintaxis). Preferí no tocar páginas del reporte en absoluto, consistente con la instrucción de "no diseñes páginas finales" y con el criterio de no forzar cambios de bajo beneficio y riesgo desconocido.

**Lo que sí hice como mejor alternativa disponible:**
1. Confirmé estructuralmente (lectura de los archivos `.tmdl`) que las 25 medidas y las 4 tablas existen, con las fórmulas exactas documentadas en la Fase 10, sin alteraciones.
2. Recalculé en Python, releyendo los 3 archivos fuente reales en `Data/`, el valor que **debería** producir cada medida si el motor DAX las evalúa correctamente — replicando la misma lógica que ya usé para diseñar las fórmulas en la Fase 10. Los resultados son idénticos a los ya documentados en `Outputs/13`, confirmando que los datos fuente no cambiaron.
3. Interpreté la confirmación que ya diste en el contexto de esta tarea ("el PBIP abre correctamente y las tablas de medidas aparecen en el panel de datos") como la evidencia de apertura disponible para esta fase.
4. Encontré evidencia indirecta adicional: al revisar `git status` al inicio de esta fase, Power BI Desktop había vuelto a reescribir los 4 archivos de medidas (agregando su propio `lineageTag`), lo cual **solo es posible si el PBIP abrió y guardó sin el error de `description`** corregido en `Outputs/14`. Ese cambio se sincronizó en un commit separado antes de esta validación (ver abajo).

**La columna "Valor observado en Power BI Desktop" de la tabla de validación queda pendiente de tu confirmación** — no la completé con valores inventados. Esta es la limitación central que debes conocer antes de considerar esta fase "cerrada".

## Sincronización previa con Power BI Desktop

Antes de empezar, `git status` mostró cambios que no había hecho yo: Power BI Desktop reescribió las 4 tablas de medidas (agregó `lineageTag` propio a cada tabla/columna/medida, sin reintroducir `description`) y actualizó `PBI_QueryOrder` en `model.tmdl`. Se investigó, confirmó que era metadato sin impacto funcional (mismo patrón que en fases anteriores), y se comiteó por separado (commit `0fcf2b8`) antes del trabajo propio de esta fase.

## Estado inicial de `git status` (para la Fase 11 en sí)

Tras el commit de sincronización, el working tree quedó limpio.

## Confirmación de apertura del PBIP

Ver nota metodológica arriba: confirmación basada en (a) tu confirmación directa en el contexto de esta tarea, y (b) la evidencia indirecta del resave exitoso de Power BI Desktop tras la corrección de `description`.

## Confirmación de existencia de las 25 medidas

Verificado por lectura directa de los 4 archivos `.tmdl`:

| Tabla | Medidas encontradas |
|---|---|
| `_Medidas Generales` | 7 |
| `_Medidas Calidad` | 6 |
| `_Medidas Capacitacion` | 6 |
| `_Medidas Motivacion` | 6 |
| **Total** | **25** ✅ |

Las 4 referencias `ref table '_Medidas ...'` están presentes en `model.tmdl`. Ninguna fórmula DAX fue modificada desde la Fase 10 (confirmado leyendo el contenido completo de los 4 archivos tras la sincronización de Power BI Desktop).

## Tabla de validación

**Valor esperado**: recalculado en Python contra los datos reales de `Data/` (idéntico a `Outputs/13`, dato fuente sin cambios). **Valor observado en Power BI**: pendiente — ver nota metodológica.

| Categoría | Medida | Valor esperado | Valor observado en Power BI | Coincide | Observación |
|---|---|---|---|---|---|
| General | `Total Evaluaciones Calidad` | 3 | Pendiente | Pendiente | — |
| General | `Total Respuestas Capacitacion` | 32 | Pendiente | Pendiente | — |
| General | `Total Respuestas Motivacion` | 5 | Pendiente | Pendiente | — |
| General | `Total Registros Piloto` | 40 | Pendiente | Pendiente | — |
| General | `n Calidad` | "n=3" | Pendiente | Pendiente | — |
| General | `n Capacitacion` | "n=32" | Pendiente | Pendiente | — |
| General | `n Motivacion` | "n=5" | Pendiente | Pendiente | — |
| Calidad | `Puntaje Obtenido Calidad` | 23 | Pendiente | Pendiente | — |
| Calidad | `Preguntas Aplicables Calidad` | 17 | Pendiente | Pendiente | — |
| Calidad | `Promedio Puntaje Calidad` | ≈1.35 (1.3529) | Pendiente | Pendiente | — |
| Calidad | `% Llamadas con Venta` | 0.0% | Pendiente | Pendiente | Las 3 evaluaciones actuales son "No" |
| Calidad | `Objecion Principal` | "Muy caro" | Pendiente | Pendiente | 2 de 3 objeciones registradas |
| Calidad | `% Calidad Promedio Provisional` | BLANK() | **Confirmado por lectura de fórmula: `= BLANK()`** | **Sí** (garantizado por diseño) | Ver sección dedicada abajo — no depende del motor DAX, es una constante |
| Capacitación | `Satisfaccion Promedio Capacitacion` | ≈4.77 (4.7742) | Pendiente | Pendiente | — |
| Capacitación | `Claridad Promedio Capacitacion` | ≈4.65 (4.6452) | Pendiente | Pendiente | — |
| Capacitación | `Utilidad Promedio Capacitacion` | ≈4.74 (4.7419) | Pendiente | Pendiente | — |
| Capacitación | `Dinamismo Promedio Capacitacion` | ≈4.74 (4.7419) | Pendiente | Pendiente | — |
| Capacitación | `Indice Global Capacitacion` | ≈4.73 (4.7258) | Pendiente | Pendiente | — |
| Capacitación | `% Comentarios Capacitacion` | ≈40.6% (40.62%) | Pendiente | Pendiente | 13 de 32 |
| Motivación | `Satisfaccion Promedio Actividad` | 4.6 | Pendiente | Pendiente | — |
| Motivación | `Claridad Utilidad Promedio Actividad` | 4.4 | Pendiente | Pendiente | — |
| Motivación | `Motivacion Promedio Actividad` | 3.8 | Pendiente | Pendiente | — |
| Motivación | `Indice Global Motivacion` | ≈4.27 (4.2667) | Pendiente | Pendiente | — |
| Motivación | `% Ambiente Motivado` | 40.0% | Pendiente | Pendiente | 2 de 5 |
| Motivación | `% Comentarios Motivacion` | 100.0% | Pendiente | Pendiente | 5 de 5 |

Todos los valores esperados de esta tabla coinciden exactamente con los que tú mismo listaste en las instrucciones de esta fase (recalculados de forma independiente en Python, no copiados), lo cual es una validación cruzada adicional de que el diseño de las fórmulas de la Fase 10 es consistente con lo esperado.

## Medidas con error

Ninguna — no se detectó ningún error de sintaxis DAX en la revisión estructural de los 4 archivos (paréntesis/llaves balanceados, referencias `[Medida]` entre medidas verificadas contra los 25 nombres definidos). No hay reporte de error desde Power BI Desktop en el contexto de esta tarea más allá del ya corregido en `Outputs/14`.

## Medidas con diferencia frente al esperado

No se puede determinar en esta ejecución — depende de la columna "Valor observado en Power BI", pendiente de tu confirmación (ver nota metodológica).

## Estado de `% Calidad Promedio Provisional`

**Confirmado en blanco por diseño, no por observación en Power BI Desktop.** Se releyó el archivo `tables/_Medidas Calidad.tmdl` tras la sincronización de esta fase y la fórmula sigue siendo exactamente `measure '% Calidad Promedio Provisional' = BLANK()` — una constante DAX que no depende de ningún dato ni contexto de filtro, por lo que su resultado en Power BI Desktop será blanco de forma garantizada, sin necesidad de confirmación visual. Esto es intencional: sigue pendiente la rúbrica oficial de puntaje máximo por pregunta del checklist de calidad (dependencia D3, `Specs/02` §4).

## Confirmación de eliminación de visuales temporales

No aplica — **no se crearon visuales temporales** en esta ejecución (ver nota metodológica). No hay nada que eliminar ni que documentar en ese sentido; no se tocó ningún archivo de página del reporte.

## Archivos modificados

Ninguno propio de esta fase. El único cambio de esta ejecución fue la sincronización de los archivos que Power BI Desktop ya había reescrito por su cuenta (commit `0fcf2b8`, ya documentado arriba), previo al trabajo de validación en sí.

## Resultado del commit

Dado que esta fase, tal como se ejecutó, no generó cambios propios en el PBIP (solo la sincronización previa, ya comiteada por separado), **no hay un commit adicional de "resultado de Fase 11"** más allá de este documento de `Outputs/`. Se comitea únicamente este archivo de documentación.

- Mensaje: `test(dax): validar medidas contra datos piloto`.
- Archivos incluidos: `Outputs/15_resultado_fase_11_validacion_indicadores.md` (nuevo).
- No se incluyó ningún archivo de `Data/*.xlsx`. No se realizó `push` a ningún remoto.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Recomendación para avanzar o no a Fase 12

**No avanzar a la Fase 12 sin antes completar la validación real que esta fase no pudo hacer por sí sola.** Pasos concretos que te pido:

1. En Power BI Desktop, crea una tabla o matriz temporal (o usa la vista de datos) mostrando las 25 medidas de las 4 tablas `_Medidas *`.
2. Compara los valores mostrados contra la columna "Valor esperado" de la tabla de esta fase.
3. Si todos coinciden: elimina el visual temporal (si creaste uno) y confírmamelo para dar por cerrada la Fase 11 y avanzar a la Fase 12.
4. Si alguno **no** coincide: repórtame el nombre exacto de la medida y el valor que Power BI muestra — no corregiré ninguna fórmula automáticamente, documentaré la diferencia y propondré un ajuste puntual para tu aprobación, tal como pediste.
5. Presta especial atención a `Objecion Principal`: es la medida con la lógica más compleja (`SUMMARIZE`/`TOPN`/`MAXX` sobre valores distintos); si algo va a fallar por una diferencia de comportamiento del motor real, es la candidata más probable.

---

*Documento generado como registro operativo de la Fase 11, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
