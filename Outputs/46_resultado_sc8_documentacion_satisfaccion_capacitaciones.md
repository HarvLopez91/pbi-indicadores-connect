# Resultado — Fase SC-8: documentación

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | `SC-8` de [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Fecha | 2026-07-22 |
| Alcance | Actualizar `Docs/02`, `Docs/03`, `Docs/05` para reflejar el estado real del repositorio tras `SC-1`–`SC-7`, y crear esta bitácora consolidada. **No se modificó ningún archivo dentro de `PBI/`** (ni Power Query, ni DAX, ni PBIR, ni relaciones, ni visuales, ni navegación), ni `Data/`, ni `Docs/06`. No se creó commit. No se hizo push. No se avanzó a `SC-9`. |
| Fases previas cerradas | `SC-1` (decisiones DEC-1 a DEC-4), `SC-2` (preparación técnica), `SC-3` (medidas, commit `d0c86f8`), `SC-4` (copia de página, commit `ba4666e`), `SC-5` (rediseño visual, commit `d4b58fd`), `SC-6` (interacciones, commit `1bf9198`), `SC-7` (validación técnica/funcional/visual, aprobada, commits `e34d803`/`1bf9198`/`e64b4b6`) |

---

## 1. Validación inicial

```
$ git rev-parse --show-toplevel
C:/Users/eclavijo/OneDrive/PBI_Indicadores

$ git branch --show-current
main

$ git status --short --untracked-files=all
(sin salida — working tree limpio)

$ git log --oneline -10
e64b4b6 chore(modelo): sincronizar layout de tabla de metricas
1bf9198 feat(report): configurar y validar interacciones de satisfaccion
e34d803 chore(git): proteger archivos de Data en subcarpetas
d4b58fd feat(report): redisenar satisfaccion de capacitaciones
5469ded fix(data): restaurar rutas locales de fuentes OneDrive
f481d12 chore(modelo): sincronizar metadatos tras validacion en desktop
28664d2 data(powerquery): migrar origenes de datos a rutas web de onedrive
d2131cd docs: corregir validacion previa a sc5
c2d457c chore(pbi): sincronizar metadatos tras validacion en desktop
ba4666e feat(report): crear copia pagina satisfaccion capacitaciones
```

Confirmado: rama `main`, working tree limpio antes de iniciar, `SC-6`/`SC-7` cerradas y versionadas (`e34d803`, `1bf9198`, `e64b4b6`), `Data/` correctamente ignorada (`git status -- Data/` no devuelve nada). Ningún cambio pendiente en `PBI/` al iniciar esta fase.

## 2. Síntesis de `SC-1` a `SC-7`

Sin duplicar las bitácoras completas — enlaces a la fuente de cada fase:

| Fase | Resultado | Bitácora |
|---|---|---|
| `SC-1` | Decisiones DEC-1 a DEC-4 confirmadas (clave de capacitación, tabla de detalle sin nombres, retiro del gráfico de Jornada, comportamiento `Between` de Fecha) | Documentadas en `Specs/06` y referenciadas en `SC-3`/`SC-4` |
| `SC-2` | Preparación técnica de la copia validada | — |
| `SC-3` | Medidas DAX iniciales creadas y validadas | [Outputs/39](39_resultado_sc3_medidas_satisfaccion_capacitaciones.md) |
| `SC-4` | Copia de página creada (`p14_satisfaccion_capacitaciones_v2`); validación en vivo de `Ultima Capacitacion` (`10/07/2026`, vía consulta DAX en Desktop) y corrección de `DEC-4` (conservar `Between`) | [Outputs/40](40_resultado_sc4_copia_pagina_satisfaccion_capacitaciones.md) |
| `SC-5` | Rediseño visual aprobado tras múltiples revisiones (hasta la Revisión 10); columna `Fecha Eje`, panel de satisfacción de 4 métricas, reversión de artefactos Auto Date/Time no usados, y **corrección de causa raíz de fechas en Power Query** (`TimestampNormalizado`, Revisión 10) | [Outputs/43](43_resultado_sc5_rediseno_visual_satisfaccion_capacitaciones.md) |
| `SC-6` | 167 interacciones `DataFilter`/`NoFilter` configuradas | [Outputs/44](44_resultado_sc6_interacciones_satisfaccion_capacitaciones.md) |
| `SC-7` | Validación técnica, funcional, visual y de navegación **aprobada** por el usuario en Power BI Desktop; corrección de `.gitignore` (`Data/**/*.xlsx`) | [Outputs/45](45_resultado_sc7_validacion_tecnica_funcional_visual.md) |

### Objetos de modelo creados durante la iniciativa

- **Medidas** (`_Medidas Capacitacion`): `Call Centers Capacitados`, `Ultima Capacitacion`, `Capacitaciones Realizadas`, `Valor Metrica Satisfaccion`, `Ultima Capacitacion Texto` — documentadas con fórmula exacta en [Docs/02](../Docs/02_catalogo_medidas_dax.md).
- **Objetos de soporte (no medidas)**: `Dim_Calendario[Fecha Eje]` (columna calculada **textual**, etiqueta de eje `dd/MM`, ordenada por `Fecha`), `Fact_SatisfaccionCapacitacion[Fecha Texto]` (columna calculada de presentación), `Dim_MetricaSatisfaccion` (tabla desconectada de 4 filas) — documentados en la misma sección de `Docs/02`.
- **Página**: `p14_satisfaccion_capacitaciones_v2`, documentada en [Docs/03](../Docs/03_mapa_reporte_paginas_visuales.md) §8, **no** enlazada desde Home ni contada entre las 7 páginas oficiales.

### Corrección de fechas del piloto

Durante `SC-4` se validó en vivo, mediante consulta DAX en Power BI Desktop, que `Ultima Capacitacion` calculaba `10/07/2026` — corrigiendo una validación externa previa e incorrecta (lectura directa del XML del `.xlsx`, sin `openpyxl`/`pandas` disponibles) que había interpretado la fecha al revés (`Outputs/40`). La **causa raíz real** (tres seriales de Excel del piloto que Power Query interpretaba como agosto/septiembre/octubre en vez de julio) se diagnosticó y corrigió posteriormente durante `SC-5`, Revisión 10, agregando el paso `TimestampNormalizado` en `expressions.tmdl` (`SatisfaccionCapacitacion_Limpio`). `Ultima Capacitacion`/`Ultima Capacitacion Texto` reflejan `10/07/2026` como la fecha más reciente desde entonces.

### Protección de datos

`.gitignore` se corrigió en `SC-7` de `Data/*.xlsx` a `Data/**/*.xlsx`, cubriendo exportaciones en subcarpetas de `Data/` (hallazgo detectado durante la validación, ver [Outputs/45](45_resultado_sc7_validacion_tecnica_funcional_visual.md) §10/§15).

### Estado de la página original y Home

Sin cambios en ningún momento de `SC-1` a `SC-7`: `p14_satisfaccion_capacitaciones` (original) y `67eff42d82e1c9c15b84` (Home) permanecen intactas. Home sigue apuntando a la página original — comportamiento esperado, no un error, hasta que `SC-9` decida el reemplazo, coexistencia o descarte.

## 3. Documentos actualizados en `SC-8`

### `Docs/02_catalogo_medidas_dax.md`

- Conteo total actualizado de 25 a **30 medidas** (recalculado desde los `.tmdl` reales, no conservando el conteo histórico de la Fase 18).
- Agregadas las 5 medidas nuevas de `_Medidas Capacitacion` con fórmula DAX exacta (copiada carácter por carácter del `.tmdl`), formato, qué calcula, visuales donde se usa, dependencias y limitaciones.
- Agregada la subsección "Objetos de soporte técnico (no son medidas)" documentando `Fecha Eje`, `Fecha Texto` y `Dim_MetricaSatisfaccion` sin contarlos como medidas.

### `Docs/03_mapa_reporte_paginas_visuales.md`

- Agregada la sección **"8. Satisfacción de capacitaciones (v2 - borrador)"**, aclarando explícitamente que es una página de trabajo, no una de las 7 oficiales, no enlazada desde Home.
- Inventario por grupos funcionales (encabezado/navegación, segmentadores, KPI, visuales analíticos, nota metodológica, interacciones) leído directamente de los `visual.json` reales — sin enumerar los 47 visuales uno por uno, siguiendo el formato existente del documento.
- Notas aclaratorias en "Resumen de navegación global" y "Resumen de segmentadores por página" para que los conteos de las 7 páginas oficiales (42 navegaciones, 16 segmentadores) no queden ambiguos frente a la página `v2`.

### `Docs/05_decisiones_limitaciones_pendientes.md`

- Nueva sección **"3. Decisiones de la iniciativa `SC-1`–`SC-7`"** formalizando `DEC-1` (clave de capacitación provisional), `DEC-2` (tabla de detalle sin nombres), `DEC-3` (gráfico de Jornada retirado) y `DEC-4` (comportamiento `Between` de Fecha, no extendido a otras páginas).
- Tabla de dependencias renombrada a "D1–D9": agregada **`D9`** (identificador oficial de sesión de capacitación), estado asignado **"Mitigada provisionalmente / pendiente de definición oficial"** — explícitamente no cerrada.
- Notas agregadas en "Pendientes de negocio" y "Riesgos de mantenimiento" sobre `D9` y la corrección de `.gitignore`.

## 4. Medidas documentadas

| Medida | Tabla | Estado en `Docs/02` |
|---|---|---|
| `Call Centers Capacitados` | `_Medidas Capacitacion` | Nueva |
| `Ultima Capacitacion` | `_Medidas Capacitacion` | Nueva |
| `Capacitaciones Realizadas` | `_Medidas Capacitacion` | Nueva |
| `Valor Metrica Satisfaccion` | `_Medidas Capacitacion` | Nueva |
| `Ultima Capacitacion Texto` | `_Medidas Capacitacion` | Nueva |

Ninguna medida preexistente fue modificada ni duplicada. Total de medidas del modelo tras `SC-8`: **30** (verificado contando directamente los bloques `measure` en los 4 archivos `_Medidas *.tmdl`).

## 5. Estado asignado a `D9`

**Mitigada provisionalmente / pendiente de definición oficial** — no se marca como cerrada. La clave `Fecha + CallCenter + NombreFormador` es un sustituto operativo válido para el piloto actual, pero puede sobre/subestimar el conteo real de sesiones de capacitación si aparecen variantes de nombre sin alias o sesiones simultáneas del mismo formador el mismo día en el mismo call center.

## 6. Nuevo archivo de `Outputs/`

`Outputs/46_resultado_sc8_documentacion_satisfaccion_capacitaciones.md` (este documento) — `46` confirmado como el siguiente número consecutivo real tras `45`.

## 7. Validaciones realizadas

| # | Validación | Resultado |
|---|---|---|
| 1 | Fórmulas de `Docs/02` coinciden carácter por carácter con el `.tmdl` real | **Aprobado** — copiadas directamente de `_Medidas Capacitacion.tmdl` |
| 2 | Ninguna medida documentada dos veces | **Aprobado** — las 5 nuevas no existían previamente en `Docs/02` |
| 3 | Conteo total de medidas actualizado | **Aprobado** — 25 → 30 |
| 4 | `Docs/03` refleja la página real, no el mockup aspiracional | **Aprobado** — inventario construido leyendo los `visual.json` reales de `p14_satisfaccion_capacitaciones_v2`, no `Specs/05`/mockup |
| 5 | Home sigue apuntando a la original | **Aprobado** — reconfirmado (`home_nav_03_*` → `p14_satisfaccion_capacitaciones`), sin cambios desde `Outputs/45` |
| 6 | `DEC-1` a `DEC-4` y `D9` consistentes entre sí | **Aprobado** — `D9` referencia `DEC-1`, ninguna decisión se documentó como cerrada si dependía de negocio |
| 7 | Enlaces relativos Markdown revisados | **Aprobado** — enlaces entre `Docs/02`↔`Docs/03`↔`Docs/05` y hacia `Outputs/39`–`45` verificados |
| 8 | Textos corruptos (`�`, `Ã`, `capacitaci?n`, `satisfacci?n`) | **Aprobado** — cero coincidencias en los 3 documentos `Docs/` editados y en este archivo |
| 9 | Nombres personales de formadores/líderes/asesores en textos nuevos | **Aprobado** — cero coincidencias (los textos nuevos hablan de "formador"/"líder" en abstracto, sin nombres propios) |
| 10 | Valores piloto (`5`, `84`, `4,8`) no documentados como permanentes | **Aprobado** — no se citaron valores numéricos actuales del piloto en `Docs/`; donde se mencionan cifras de ejemplo (`Outputs/45`) ya estaban etiquetadas como capturas de un momento específico, no como valores de referencia |
| 11 | Ningún archivo de `PBI/` modificado | **Aprobado** — ver §8 |
| 12 | `Data/` no modificado | **Aprobado** — ver §8 |

## 8. Confirmación — `PBI/` y `Data/` sin cambios

```
$ git status --short --untracked-files=all
 M Docs/02_catalogo_medidas_dax.md
 M Docs/03_mapa_reporte_paginas_visuales.md
 M Docs/05_decisiones_limitaciones_pendientes.md
?? Outputs/46_resultado_sc8_documentacion_satisfaccion_capacitaciones.md

$ git diff --stat -- PBI/
(sin salida — sin cambios)

$ git status --short --untracked-files=all -- Data/
(sin salida — sin cambios)
```

## 9. Diferencias o inconsistencias encontradas

La primera versión de esta fase (antes de la corrección solicitada por el usuario) contenía 5 inconsistencias, todas corregidas en esta versión:

1. **Resumen final de `Docs/02` desactualizado** — mostraba el conteo previo a `SC-8` (25 medidas, 6 en `_Medidas Capacitacion`) en vez del real (30 medidas, 11 en `_Medidas Capacitacion`). Corregido: tabla recalculada (30/28/2), con `% Calidad Promedio Provisional` y `Ultima Capacitacion` explícitas como las 2 medidas sin enlace directo.
2. **Usos por visual no actualizados para medidas preexistentes** — `Total Respuestas Capacitacion`, los 4 promedios Likert de capacitación y `% Comentarios Capacitacion` solo listaban visuales de la página original. Corregido: se agregaron los visuales de `v2` (`sc_kpi_respuestas`, `sc_kpi_respuestas_panel`, `sc_kpi_satisfaccion`, columnas de `sc_tabla_callcenter`, `sc_kpi_comentarios`) y la dependencia de los 4 promedios respecto a `Valor Metrica Satisfaccion`.
3. **`Dim_Calendario[Fecha Eje]` documentada con una definición obsoleta** (columna de tipo fecha vía `DATE(YEAR/MONTH/DAY)`) — la implementación final (`SC-5` Revisión 9) la convirtió en columna **textual** (`dataType: string`, `FORMAT(DAY...)&"/"&FORMAT(MONTH...)`, `sortByColumn: Fecha`). Corregido en `Docs/02` y `Docs/03`.
4. **Numeración duplicada en `Docs/05`** — dos secciones `## 4`. Corregido: "Pendientes de negocio" pasó a `## 5`, "Riesgos de mantenimiento" a `## 6`; referencia cruzada interna actualizada.
5. **Cronología imprecisa en este documento** — atribuía a `SC-4` la corrección definitiva de fechas del piloto y describía `SC-5` como "aprobado tras 8 revisiones". Corregido: `SC-4` solo validó en vivo el valor `10/07/2026`; la causa raíz (Power Query, `TimestampNormalizado`) se corrigió en `SC-5` Revisión 10, tras un total de 10 revisiones documentadas en `Outputs/43`.

Verificado tras las correcciones: el estado real del repositorio (TMDL, `visual.json`, `page.json`) coincide ahora con lo documentado en `Docs/02`, `Docs/03`, `Docs/05` y con las bitácoras `Outputs/39`–`45`.

## 10. Recomendación

**`SC-8` puede aprobarse.** `Docs/02`, `Docs/03` y `Docs/05` reflejan el estado real y verificado del repositorio tras `SC-1`–`SC-7`. No se modificó `PBI/`, `Data/` ni `Docs/06` (la publicación depende de `SC-9`, no ejecutada). No se creó commit — pendiente de tu revisión antes de comitear.

**Pendiente real para `SC-9`:** decidir si `p14_satisfaccion_capacitaciones_v2` reemplaza a la página original, coexiste con ella, o se descarta; y si se actualiza `Docs/06` según esa decisión.
