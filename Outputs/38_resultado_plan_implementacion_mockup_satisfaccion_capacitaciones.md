# Resultado — Plan de implementación: mockup de Satisfacción de capacitaciones

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de tarea | Planificación exclusivamente (sin implementación) |
| Documento principal | [`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`](../Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md) |
| Documento base | [`Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md`](../Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md) |
| Fecha | 2026-07-21 |
| Alcance | Documentación exclusivamente. No se modificó ningún archivo PBIR, TMDL, Power Query, DAX, relaciones, visuales ni `Data/*.xlsx`. No se creó ninguna medida DAX. No se ejecutó ninguna fase del plan. |

---

## 1. Estado inicial de `git status`

```
?? Data/
```

Working tree limpio salvo `Data/` (sin seguimiento por diseño). Confirmado antes de iniciar.

## 2. Fuente del plan

El plan se construyó exclusivamente a partir de [`Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md`](../Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md), sin reabrir el análisis del mockup ni el modelo — las conclusiones de `Specs/05` (indicadores bloqueados/viables, medidas existentes/nuevas, recomendación de copia de página) se tomaron como entrada fija.

## 3. Estructura del plan creado

`Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md` define **9 fases** (`SC-1` a `SC-9`), numeradas con el prefijo `SC` (Satisfacción de Capacitaciones) para no colisionar con las 18 fases históricas de `Specs/02`:

1. **SC-1** — Confirmación de decisiones de negocio (4 decisiones: definición de "capacitación única", tabla por call center vs. formador/líder, gráfico de jornada, estilo del segmentador de Fecha).
2. **SC-2** — Preparación técnica (Git, rama, protección de la página publicada).
3. **SC-3** — Creación condicionada de medidas DAX nuevas (2 siempre, 1+variantes solo si se resuelve DEC-1).
4. **SC-4** — Duplicado de la página actual en una copia de trabajo, sin tocar la original.
5. **SC-5** — Rediseño visual de la copia según el mockup, bloque por bloque.
6. **SC-6** — Configuración de interacciones cruzadas entre visuales de la copia.
7. **SC-7** — Validación técnica, funcional y visual de la copia.
8. **SC-8** — Actualización de `Docs/02`, `Docs/03`, `Docs/05` y bitácora en `Outputs/`.
9. **SC-9** — Decisión de reemplazo/coexistencia/descarte y, si aplica, publicación.

Cada fase incluye los 8 elementos solicitados: objetivo, actividades, restricciones, archivos que podrían modificarse, validaciones, riesgos, resultado esperado y un prompt de ejecución específico y autocontenible.

## 4. Puntos de diseño relevantes del plan

- **Condicionamiento explícito por DEC-1**: `SC-3` y `SC-5` incluyen instrucciones explícitas de no crear/mostrar los indicadores de "capacitaciones" (sesiones) si la definición de negocio sigue sin resolver, evitando que una futura ejecución del plan invente un supuesto no autorizado.
- **Aislamiento de la página publicada**: todas las fases de cambio (`SC-3` a `SC-6`) trabajan exclusivamente sobre una copia de página (`p14_satisfaccion_capacitaciones_v2`, nombre tentativo); `SC-2`, `SC-4` y `SC-7` incluyen verificaciones explícitas de que la página original no cambió.
- **Reemplazo como decisión separada**: `SC-9` no asume que la copia reemplaza a la original — exige confirmación explícita del usuario y advierte si "coexistir" reabriría la decisión ya cerrada de 7 páginas (`Specs/03` §2).
- **Trazabilidad de decisiones**: `SC-1` alimenta directamente `SC-3` y `SC-5`; `SC-8` formaliza esas decisiones en `Docs/05` con el mismo formato ya usado para las dependencias D1–D8.

## 5. Documentos creados

- `Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md`
- `Outputs/38_resultado_plan_implementacion_mockup_satisfaccion_capacitaciones.md` (este documento)

## 6. Confirmación de no modificación del informe

No se modificó:

- Ningún archivo bajo `PBI/` (PBIR, TMDL, Power Query, medidas DAX, relaciones, visuales, tema).
- Ningún archivo `Data/*.xlsx`.
- No se creó ninguna medida DAX — las medidas propuestas en `SC-3` quedan solo documentadas como fórmulas de referencia dentro del plan, no implementadas.
- No se ejecutó ninguna fase del plan (`SC-1` a `SC-9`) — el documento es exclusivamente la hoja de ruta.

## 7. Confirmación de no versionamiento de `Data/`

No se agregó ningún archivo de `Data/` al índice de Git. `git status --porcelain -- Data/` no cambió respecto al estado inicial.

## 8. Estado final de `git status`

```
?? Data/ (sin cambios, preexistente, no agregado al commit)
?? Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
?? Outputs/38_resultado_plan_implementacion_mockup_satisfaccion_capacitaciones.md
```

## 9. Commit sugerido

`docs: planificar implementacion mockup satisfaccion capacitaciones`

No se hizo push remoto.

## 10. Recomendaciones futuras

1. Ejecutar `SC-1` en la próxima sesión de trabajo con el usuario/negocio antes de tocar cualquier archivo del PBIP — es la fase que desbloquea o acota el resto del plan.
2. No saltar directamente a `SC-3`/`SC-5` sin pasar por `SC-1` y `SC-2`, aunque parezca más rápido — el plan depende de que las decisiones de negocio y el estado de Git estén confirmados primero.
3. Revisar `Specs/06` §11 ("Criterios para avanzar o detenerse") antes de cada fase, especialmente si `DEC-1` sigue sin resolver al momento de ejecutar `SC-3`.
