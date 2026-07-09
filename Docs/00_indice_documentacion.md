# Índice de documentación — `PBI_Indicadores`

Punto de entrada a toda la documentación técnica y funcional del informe Connect Assistance. Si es la primera vez que trabaja en este proyecto, empiece por el [README.md](../README.md) en la raíz del repositorio y vuelva aquí para profundizar según su rol (§2).

## 1. Documentos disponibles

| Documento | Para qué sirve |
|---|---|
| [`01_modelo_datos.md`](01_modelo_datos.md) | Estructura del modelo estrella: tablas de hechos, dimensiones, columnas técnicas y su origen, relaciones, convenciones de nombres, y el ruido conocido de las tablas automáticas de fecha. |
| [`02_catalogo_medidas_dax.md`](02_catalogo_medidas_dax.md) | Catálogo completo de las 25 medidas DAX: fórmula exacta, qué calcula cada una, en qué páginas/visuales se usa, formato y observaciones/limitaciones. |
| [`03_mapa_reporte_paginas_visuales.md`](03_mapa_reporte_paginas_visuales.md) | Recorrido por las 7 páginas del reporte: objetivo, indicadores, medidas, visuales principales, segmentadores, notas visibles y navegación de cada una. |
| [`04_fuentes_y_actualizacion_datos.md`](04_fuentes_y_actualizacion_datos.md) | Guía operativa paso a paso para reexportar los 3 Google Forms, colocarlos en `Data/`, actualizar el modelo en Power BI Desktop y qué validar después. |
| [`05_decisiones_limitaciones_pendientes.md`](05_decisiones_limitaciones_pendientes.md) | Decisiones de diseño tomadas (definitivas y provisionales), estado actualizado de las dependencias D1–D8, pendientes de negocio y riesgos de mantenimiento. |
| [`06_publicacion_powerbi.md`](06_publicacion_powerbi.md) | Enlace publicado del informe, consideración de gobierno de datos sobre el acceso público, y checklist de validación antes/después de publicar. |

Documentos relacionados fuera de esta carpeta:

- [`../README.md`](../README.md) — visión general del proyecto, estructura de carpetas y cómo abrirlo.
- [`../Specs/01_analisis_de_impacto_informe_powerbi_connect.md`](../Specs/01_analisis_de_impacto_informe_powerbi_connect.md) — diagnóstico original de los datos y el modelo propuesto (previo a la construcción).
- [`../Specs/02_plan_implementacion_informe_powerbi_connect.md`](../Specs/02_plan_implementacion_informe_powerbi_connect.md) — plan de 18 fases ejecutado para construir el informe.
- [`../Specs/03_documentacion_final_informe_powerbi_connect.md`](../Specs/03_documentacion_final_informe_powerbi_connect.md) — cierre formal de la implementación (inventario final, criterios de cierre, recomendación final).
- [`../Outputs/`](../Outputs/) — bitácora cronológica de cada fase/corrección ejecutada; es el changelog operativo del proyecto.

## 2. Ruta de lectura recomendada por rol

### Usuario de negocio (gerencia, PUSHER, líder de operación)

1. [README.md](../README.md) (raíz) — qué es el informe y su enlace publicado.
2. [03_mapa_reporte_paginas_visuales.md](03_mapa_reporte_paginas_visuales.md) — qué muestra cada página y qué preguntas responde.
3. [05_decisiones_limitaciones_pendientes.md](05_decisiones_limitaciones_pendientes.md) §4 — qué está pendiente de confirmar y por qué los indicadores no deben leerse como definitivos todavía.
4. [06_publicacion_powerbi.md](06_publicacion_powerbi.md) §1–2 — cómo acceder al informe publicado y la consideración de gobierno de datos.

### Desarrollador Power BI (mantenimiento del modelo/reporte)

1. [README.md](../README.md) (raíz) — estructura de carpetas y cómo abrir el PBIP.
2. [01_modelo_datos.md](01_modelo_datos.md) — estructura completa del modelo estrella.
3. [02_catalogo_medidas_dax.md](02_catalogo_medidas_dax.md) — catálogo de medidas antes de crear una medida nueva (evitar duplicar lógica ya existente).
4. [03_mapa_reporte_paginas_visuales.md](03_mapa_reporte_paginas_visuales.md) — dónde vive cada visual antes de modificar una página.
5. [05_decisiones_limitaciones_pendientes.md](05_decisiones_limitaciones_pendientes.md) — decisiones ya tomadas, para no revertirlas sin darse cuenta.
6. [../Specs/02_plan_implementacion_informe_powerbi_connect.md](../Specs/02_plan_implementacion_informe_powerbi_connect.md) y `../Outputs/` — contexto histórico completo de por qué el modelo/reporte luce como luce.

### Persona que solo necesita actualizar datos

1. [04_fuentes_y_actualizacion_datos.md](04_fuentes_y_actualizacion_datos.md) — guía completa, de principio a fin, escrita para no requerir conocimiento técnico de Power BI.
2. [06_publicacion_powerbi.md](06_publicacion_powerbi.md) §4 — checklist posterior a la actualización, si corresponde republicar.

## 3. Cómo mantener esta documentación viva

Esta carpeta debe actualizarse cuando cambien medidas, páginas o fuentes de datos (ver la sección "Mantenimiento" del [README.md](../README.md) raíz). No se debe sobrescribir el historial de `Outputs/` — cualquier cambio nuevo se documenta como un archivo `Outputs/NN_...md` adicional, y si el cambio afecta la estructura descrita aquí, se actualiza el documento `Docs/` correspondiente en el mismo commit.
