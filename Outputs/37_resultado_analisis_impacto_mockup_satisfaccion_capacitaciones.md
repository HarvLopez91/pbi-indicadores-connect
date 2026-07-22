# Resultado — Análisis de impacto: mockup de Satisfacción de capacitaciones

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de tarea | Análisis de impacto exclusivamente (sin implementación) |
| Documento principal | [`Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md`](../Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md) |
| Mockup analizado | [`Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png`](../Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png) (reubicado previamente, `Outputs/36`) |
| Fecha | 2026-07-21 |
| Alcance | Documentación exclusivamente. No se modificó ningún archivo PBIR, TMDL, Power Query, DAX, relaciones, visuales ni `Data/*.xlsx`. No se creó ninguna medida DAX. |

---

## 1. Estado inicial de `git status`

```
?? Data/
```

Working tree limpio salvo `Data/` (sin seguimiento por diseño, excluida vía `.gitignore`). Confirmado antes de iniciar.

## 2. Fuentes revisadas

- **Página actual**: `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/page.json` y los 29 `visual.json` de su carpeta `visuals/` (inspección directa, sin modificarlos).
- **Modelo**: `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_SatisfaccionCapacitacion.tmdl` (columnas y partición Power Query, solo lectura).
- **Mockup**: `Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png` (lectura visual directa de la imagen).
- **Documentación de referencia**: [Docs/01_modelo_datos.md](../Docs/01_modelo_datos.md), [Docs/02_catalogo_medidas_dax.md](../Docs/02_catalogo_medidas_dax.md), [Docs/03_mapa_reporte_paginas_visuales.md](../Docs/03_mapa_reporte_paginas_visuales.md), [Docs/05_decisiones_limitaciones_pendientes.md](../Docs/05_decisiones_limitaciones_pendientes.md), [Docs/06_publicacion_powerbi.md](../Docs/06_publicacion_powerbi.md).

## 3. Método de comparación

1. Se inventarió cada visual real de la página (`sc_kpi_*`, `sc_chart_*`, `sc_tabla_formador`, segmentadores, encabezado, nota) leyendo los `visual.json` vigentes, no solo el `Docs/03` (para confirmar campos y medidas exactas usadas en cada visual, incluyendo el modo `Dropdown` del segmentador de Fecha).
2. Se describió cada elemento visible del mockup (KPI, gráficos, tabla, tabla de comentarios, segmentadores, notas).
3. Se clasificó cada elemento del mockup en tres categorías: **ya existe**, **debe ajustarse**, **falta**.
4. Se validó cada uno de los 6 indicadores solicitados por el usuario contra las columnas reales de `Fact_SatisfaccionCapacitacion.tmdl` y el catálogo de 25 medidas vigente.

## 4. Hallazgo principal

`Fact_SatisfaccionCapacitacion` tiene grano **1 fila = 1 respuesta de encuesta**, no de sesión de capacitación, y no existe ninguna columna en el origen que identifique una sesión de capacitación como entidad propia. Esto bloquea 3 de los 6 indicadores solicitados (Capacitaciones realizadas, Capacitaciones por fecha, Capacitaciones por call center) hasta que negocio defina qué combinación de columnas constituye "una capacitación única" — documentado como dependencia nueva candidata (no formalizada aún en `Docs/05`) en `Specs/05` §7.

Los otros 3 indicadores sí son viables:
- **Satisfacción por call center seleccionado**: viable hoy con medidas ya existentes, sin DAX nuevo.
- **Comentarios destacados**: viable hoy con columnas ya existentes, sin DAX nuevo.
- **Última capacitación**: requiere 1 medida nueva simple (`MAX(Fecha)`), sin dependencia de negocio.

## 5. Resumen de medidas

- **7 medidas existentes reutilizables** sin ningún cambio (`Satisfaccion Promedio Capacitacion`, `Claridad Promedio Capacitacion`, `Utilidad Promedio Capacitacion`, `Dinamismo Promedio Capacitacion`, `Total Respuestas Capacitacion`, `% Comentarios Capacitacion`, `Indice Global Capacitacion`).
- **5 medidas nuevas identificadas, ninguna creada en esta tarea** (`Capacitaciones Realizadas`, `Capacitaciones por Fecha`, `Capacitaciones por Call Center` — las 3 bloqueadas por la dependencia de grano; `Call Centers Capacitados`, `Ultima Capacitacion` — técnicamente simples, sin bloqueo de negocio). Detalle completo en `Specs/05` §6.

## 6. Recomendación de implementación

`Specs/05` §8 recomienda **crear una copia de página para prototipar el rediseño**, no editar `p14_satisfaccion_capacitaciones` en sitio, dado que la página actual está validada (Fase 17) y publicada en el enlace público activo, y el alcance del cambio no es cosmético (reestructura la fila de KPI, reemplaza un gráfico, cambia el grano de la tabla principal, y depende de una decisión de negocio no resuelta).

## 7. Documentos creados

- `Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md`
- `Outputs/37_resultado_analisis_impacto_mockup_satisfaccion_capacitaciones.md` (este documento)

## 8. Confirmación de no modificación del informe

No se modificó:

- Ningún archivo bajo `PBI/` (PBIR, TMDL, Power Query, medidas DAX, relaciones, visuales, tema).
- Ningún archivo `Data/*.xlsx`.
- No se creó ninguna medida DAX nueva — las 5 identificadas en §5 quedan solo documentadas como propuesta.

## 9. Confirmación de no versionamiento de `Data/`

No se agregó ningún archivo de `Data/` al índice de Git. `git status --porcelain -- Data/` no cambió respecto al estado inicial.

## 10. Estado final de `git status`

```
 M Data (sin cambios, preexistente, no agregado al commit)
?? Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md
?? Outputs/37_resultado_analisis_impacto_mockup_satisfaccion_capacitaciones.md
```

## 11. Commit sugerido

`docs: analizar impacto de mockup satisfaccion capacitaciones`

No se hizo push remoto.

## 12. Recomendaciones futuras

1. Llevar la dependencia candidata D9 (definición de "capacitación única") a negocio antes de continuar con cualquier implementación — bloquea la mitad de los indicadores solicitados.
2. Si negocio confirma D9 y autoriza medidas nuevas, seguir el plan de 5 fases descrito en `Specs/05` §9 (decisiones → autorización de medidas → prototipo en copia de página → validación → reemplazo y documentación).
3. Formalizar la dependencia D9 en `Docs/05_decisiones_limitaciones_pendientes.md` en la fase en que se resuelva o se decida explícitamente diferirla, siguiendo el mismo formato de D1–D8 ya usado en ese documento.
