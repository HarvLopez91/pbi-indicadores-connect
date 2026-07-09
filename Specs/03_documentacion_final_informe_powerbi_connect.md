# Documentación final — Informe Power BI Connect Assistance S.A.S.

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Documentos base | [`01_analisis_de_impacto_informe_powerbi_connect.md`](01_analisis_de_impacto_informe_powerbi_connect.md), [`02_plan_implementacion_informe_powerbi_connect.md`](02_plan_implementacion_informe_powerbi_connect.md) |
| Cliente / contexto | Connect Assistance S.A.S. — call centers asociados a Claro |
| Tipo de documento | Cierre formal de la implementación (Fase 18) |
| Fecha | 2026-07-09 |
| Repositorio | [github.com/HarvLopez91/pbi-indicadores-connect](https://github.com/HarvLopez91/pbi-indicadores-connect) (rama `main`) |
| Estado | **Cerrado** — implementación completa, informe validado (Fase 17), documentación final publicada (Fase 18) y repositorio sincronizado en GitHub |

---

## 1. Resumen ejecutivo

El informe `PBI_Indicadores` fue construido desde un PBIP inicialmente vacío hasta un informe funcional de 7 páginas, con modelo en estrella (3 hechos, 3 dimensiones, 25 medidas DAX), identidad visual de marca Connect, navegación completa entre Home y páginas internas, y notas metodológicas dinámicas que documentan las limitaciones del piloto sin depender de conteos escritos a mano. El proyecto siguió las 18 fases descritas en [02_plan_implementacion_informe_powerbi_connect.md](02_plan_implementacion_informe_powerbi_connect.md), todas versionadas en Git con commits descriptivos en español. El informe ya fue validado técnica, funcional y visualmente (Fase 17, `Outputs/31`) y está publicado mediante un enlace de "Publicar en la Web" de Power BI Service.

El volumen de datos sigue siendo de fase piloto (3 evaluaciones de calidad, 32 respuestas de capacitación, 5 respuestas de motivación al momento del diagnóstico inicial, con crecimiento esperado en cada actualización) — el informe está diseñado para comunicar esa condición de forma explícita y dinámica, no para ocultarla.

## 2. Alcance final implementado

- Modelo semántico con 3 tablas de hechos, 3 dimensiones compartidas y 4 tablas de medidas (25 medidas DAX).
- 7 páginas de reporte: Home, Resumen ejecutivo, Calidad de llamadas, Satisfacción de capacitaciones, Motivación comercial, Detalle por call center, Notas metodológicas.
- Tema visual de marca Connect (naranja `#F15B2B`, oscuro `#002733`) aplicado de forma centralizada vía tema JSON, no hardcodeado por visual.
- Navegación funcional en ambas direcciones (Home ↔ páginas internas), reforzada con zonas clicables transparentes ("hitzones") sobre cada tarjeta/botón.
- Advertencias de bajo volumen (`n=`) y de limitaciones de fuente (encuesta de motivación anónima) visibles en Home, páginas internas y Notas metodológicas, todas mediante medidas DAX dinámicas.
- Publicación activa en Power BI Service vía enlace de "Publicar en la Web".

**Desviación documentada respecto al plan original:** `Specs/01`/`Specs/02` proponían una página adicional "Detalle por asesor/líder". En la implementación final ese desglose se cubrió con tablas dentro de "Calidad de llamadas" (`cl_tabla_asesor`) y "Satisfacción de capacitaciones" (`sc_tabla_formador`), sin una octava página dedicada. El informe final tiene 7 páginas, no 8.

**Fuera de alcance** (confirmado como no implementado, consistente con lo ya definido en `Specs/02` §3): conexión automática a Google Forms/Sheets, `Dim_Colaborador` maestro con ID único, RLS (seguridad a nivel de fila), y limpieza de las tablas automáticas de fecha (Auto Date/Time) — ver §10.

## 3. Inventario final del modelo

Detalle completo de columnas y orígenes en [`../Docs/01_modelo_datos.md`](../Docs/01_modelo_datos.md). Resumen:

| Tabla | Tipo | Grano / contenido | Origen |
|---|---|---|---|
| `Fact_CalidadLlamadas` | Hechos | 1 fila = 1 llamada evaluada | `Data/Matriz de calidad (Responses).xlsx` |
| `Fact_SatisfaccionCapacitacion` | Hechos | 1 fila = 1 respuesta de encuesta de capacitación | `Data/Satisfacción capacitación (Responses).xlsx` |
| `Fact_MotivacionActividad` | Hechos | 1 fila = 1 respuesta de encuesta de motivación | `Data/Encuesta satisfacción (Responses).xlsx` |
| `Dim_Calendario` | Dimensión | Tabla continua de fechas, desde el mínimo observado hasta `HOY()` | Generada en Power Query |
| `Dim_CallCenter` | Dimensión | Valores distintos de `CallCenter`, unión dinámica de las 3 Fact | Generada en Power Query |
| `Dim_Jornada` | Dimensión | Valores distintos de `Jornada`, unión dinámica de 2 Fact (no incluye `Fact_CalidadLlamadas`) | Generada en Power Query |
| `_Medidas Generales` / `_Medidas Calidad` / `_Medidas Capacitacion` / `_Medidas Motivacion` | Medidas | 25 medidas DAX, sin datos reales (tabla placeholder de 0 filas) | N/A |

**Columnas principales por tabla de hechos** (nombre técnico, `PascalCase`, sin tildes ni espacios): ver la tabla completa columna-por-columna con su origen exacto del Excel en [Docs/01_modelo_datos.md](../Docs/01_modelo_datos.md) §2.

**Relaciones:** 8 relaciones de negocio (`Dim_Calendario`/`Dim_CallCenter` → las 3 Fact; `Dim_Jornada` → 2 Fact) más 3 relaciones auxiliares hacia tablas ocultas de Auto Date/Time (ruido conocido, ver §10). Todas `1:*`, dirección de filtro única, sin ambigüedad — confirmado en la validación de la Fase 17.

## 4. Catálogo resumido de medidas DAX

Catálogo completo con fórmula exacta de cada una de las 25 medidas en [`../Docs/02_catalogo_medidas_dax.md`](../Docs/02_catalogo_medidas_dax.md). Resumen por familia:

| Tabla de medidas | Medidas | Propósito |
|---|---|---|
| `_Medidas Generales` (7) | `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion`, `Total Registros Piloto`, `n Calidad`, `n Capacitacion`, `n Motivacion` | Conteos base transversales y leyendas dinámicas `n=` |
| `_Medidas Calidad` (6) | `Puntaje Obtenido Calidad`, `Preguntas Aplicables Calidad`, `Promedio Puntaje Calidad`, `% Llamadas con Venta`, `Objecion Principal`, `% Calidad Promedio Provisional` | Indicadores de la auditoría de calidad de llamadas |
| `_Medidas Capacitacion` (6) | `Satisfaccion Promedio Capacitacion`, `Claridad Promedio Capacitacion`, `Utilidad Promedio Capacitacion`, `Dinamismo Promedio Capacitacion`, `Indice Global Capacitacion`, `% Comentarios Capacitacion` | Indicadores de satisfacción de capacitación |
| `_Medidas Motivacion` (6) | `Satisfaccion Promedio Actividad`, `Claridad Utilidad Promedio Actividad`, `Motivacion Promedio Actividad`, `Indice Global Motivacion`, `% Ambiente Motivado`, `% Comentarios Motivacion` | Indicadores de motivación de actividades comerciales |

De las 25 medidas, **24 están enlazadas a algún visual del reporte**; `% Calidad Promedio Provisional` retorna `BLANK()` intencionalmente y solo se documenta en texto (ver §10).

## 5. Mapa de páginas y navegación

Detalle completo (objetivo, indicadores, visuales, segmentadores, notas) de cada página en [`../Docs/03_mapa_reporte_paginas_visuales.md`](../Docs/03_mapa_reporte_paginas_visuales.md).

```
                    ┌───────────────┐
                    │     Home      │  (landing page, 6 KPI, sin segmentadores)
                    └───────┬───────┘
       ┌───────────┬────────┼────────┬───────────┬───────────────┐
       ▼           ▼        ▼        ▼           ▼               ▼
  Resumen      Calidad   Satisfacción Motivación  Detalle por  Notas
  ejecutivo    de        de           comercial   call center  metodológicas
               llamadas  capacitaciones
       │           │        │        │           │               │
       └───────────┴────────┴────────┴───────────┴───────────────┘
                    (botón "Volver a Home" en cada página interna)
```

Navegación implementada con `visualLink` tipo `PageNavigation` en 42 visuales (24 en Home, 18 en páginas internas), reforzada con zonas clicables transparentes ("hitzones") que cubren el área completa de cada tarjeta/botón — corrección aplicada tras detectar que la navegación por sub-elementos apilados dejaba zonas no clicables (`Outputs/28`).

## 6. Tema visual y recursos usados

- **Tema JSON personalizado**: `Assets/theme/connect_assistance_theme.json`, registrado en el reporte como `Connect_Assistance5317256897086657.json` (recurso registrado en `PBI_Indicadores.Report/StaticResources/RegisteredResources/`), sobre la base `CY25SU11`.
- **Paleta de marca real**: naranja `#F15B2B` (acento — contraste 3.35:1 sobre blanco, válido para texto grande/elementos de acento) y oscuro `#002733` (texto principal — contraste 15.69:1 sobre blanco), obtenidos de un logo SVG a color entregado durante la Fase 12. Reemplazaron la paleta placeholder inicial (`#F37021`/`#1F1F1F`).
- **Grises de apoyo**: `#E7E7E7` (bordes), `#F4F4F4` (cuadrícula/fondos suaves), `#3A3A3A` (texto secundario), `#FAFAFA`/`#FFF3EE` (fondos suaves de acento).
- **Logo**: `logo_connect_naranja_20260708.png`, registrado como recurso del reporte y usado en el encabezado de Home.
- **Assets fuente**: `Assets/logos/` (SVG e imagen del logo) y `Assets/imagenes/` (recursos gráficos adicionales de marca, no todos necesariamente usados en el reporte actual).

## 7. Fuentes de datos y actualización

Guía operativa completa en [`../Docs/04_fuentes_y_actualizacion_datos.md`](../Docs/04_fuentes_y_actualizacion_datos.md). Resumen: 3 archivos Excel exportados manualmente desde Google Forms, ubicados en `Data/` (excluida de git), conectados vía el parámetro de Power Query `RutaCarpetaData` (sin rutas absolutas embebidas en cada consulta). Los archivos deben estar cerrados en Excel antes de actualizar en Power BI Desktop. No existe conexión automática a la fuente — cada actualización de datos requiere reexportar manualmente desde Google Forms y actualizar en Desktop.

## 8. Publicación del informe

**Enlace publicado:**
```
https://app.powerbi.com/view?r=eyJrIjoiZGI2ZjNiYmItODQ0Yy00M2Y1LThkNTYtZGQ5NDIxYWExNjk3IiwidCI6Ijc1NDEyNGJlLTM2NGItNDg1MS1hYzA3LTc0ZjljZGJhYzM0ZiIsImMiOjR9&pageName=67eff42d82e1c9c15b84
```

Publicado vía "Publicar en la Web" de Power BI Service, con página inicial Home (`pageName=67eff42d82e1c9c15b84`). Este mecanismo de publicación **no requiere autenticación** — ver la consideración de gobierno de datos en [Docs/06_publicacion_powerbi.md](../Docs/06_publicacion_powerbi.md) §2, dado que 2 tablas del informe (`cl_tabla_asesor`, `sc_tabla_formador`) muestran nombres reales de asesores/líderes/formadores.

## 9. Decisiones técnicas relevantes

- **PBIP/TMDL**: modelo semántico versionado en formato TMDL (no `.pbix` binario), permitiendo diffs legibles en Git. No se escribe `lineageTag`/`description`/`queryGroup` a mano en bloques nuevos — Power BI Desktop los genera al guardar; escribirlos manualmente rompe el analizador de la vista previa de Desktop (`UnknownKeyword`), error detectado y corregido 3 veces durante la implementación (Fases 3, 7 y 10).
- **PBIR**: definición del reporte en JSON versionable por página/visual (`pages/<pageId>/visuals/<visualName>/visual.json`), sin manifiesto central de visuales — cada carpeta nueva se autodescubre.
- **Git**: repositorio inicializado desde la Fase 2, con convención de commits tipo conventional-commit en español (`feat(modelo):`, `fix(report):`, `chore(report):`, `docs:`, etc.). Los cambios automáticos de Power BI Desktop se comitean por separado de los cambios intencionales, patrón aplicado de forma consistente en las 18 fases.
- **Tema JSON**: colores centralizados en `Assets/theme/connect_assistance_theme.json`, nunca hardcodeados por visual individual — permite actualizar la paleta en un solo lugar.
- **Hitzones de navegación**: técnica de rectángulo transparente (`fill.transparency: 100D`) superpuesto con z-index superior a cada tarjeta/botón de navegación, con el mismo `visualLink`, para garantizar un área clicable continua sin depender de la alineación exacta entre sub-elementos.
- **Segmentadores en modo `Dropdown`**: los 16 segmentadores del reporte usan menú desplegable (no lista expandida), en la misma posición relativa en todas las páginas, para una experiencia consistente.
- **Etiquetas de datos**: activas en los 8 gráficos cuantitativos del reporte (`barChart`/`columnChart`/`lineChart`), color `#002733`, tamaño `9D` — estilo Connect sin saturar.
- **Notas dinámicas**: ninguna nota metodológica ni advertencia del reporte contiene un conteo escrito a mano; todos los volúmenes se muestran vía medidas DAX (`Total *`, `n Calidad`/`n Capacitacion`/`n Motivacion`) enlazadas a tarjetas, para que el texto siga siendo válido aunque cambien los datos.

## 10. Limitaciones y pendientes

Detalle completo en [`../Docs/05_decisiones_limitaciones_pendientes.md`](../Docs/05_decisiones_limitaciones_pendientes.md). Resumen:

- Datos de fase piloto — volumen bajo, sujeto a crecimiento; los indicadores no deben leerse como definitivos.
- Encuesta de motivación anónima — sin desglose por asesor individual.
- `% Calidad Promedio Provisional` — pendiente de la rúbrica oficial de puntaje máximo por pregunta (dependencia D3).
- `% Llamadas con Venta` — observación pendiente si sigue apareciendo en blanco en ciertos contextos de filtro.
- Catálogo oficial de call centers (D4) y confirmación oficial de alias de líderes/formadores (D5) — pendientes de negocio.
- Tablas automáticas de Auto Date/Time — ruido conocido, limpieza pendiente manual desde Power BI Desktop.
- Medida `Fecha Corte Datos` — propuesta y no implementada (ver `Outputs/29`).
- Gobierno de la publicación pública — pendiente de confirmación de negocio (ver §8).

## 11. Estado actualizado de dependencias D1–D8

| # | Dependencia | Estado |
|---|---|---|
| D1 | Power BI Desktop con soporte PBIP + TMDL | **Resuelta** |
| D2 | Archivos de `Data/` cerrados al actualizar | Gestionada (riesgo operativo permanente, no eliminable) |
| D3 | Rúbrica de puntaje máximo por pregunta de calidad | **Pendiente** |
| D4 | Catálogo oficial de call centers y jornadas | **Pendiente** |
| D5 | Nombres estándar de líderes/formadores | Parcialmente resuelta |
| D6 | Logo oficial y HEX exactos de marca | **Resuelta** (Fase 12) |
| D7 | Versionamiento con Git | **Resuelta** (Fase 2) |
| D8 | Volumen de datos piloto no representativo | Constatada y gestionada activamente (no es un estado "resoluble") |

Detalle y justificación de cada estado en [Docs/05_decisiones_limitaciones_pendientes.md](../Docs/05_decisiones_limitaciones_pendientes.md) §3.

## 12. Guía de mantenimiento

- No versionar `Data/*.xlsx`.
- No escribir `lineageTag`/`description`/`queryGroup` a mano en TMDL.
- Sincronizar los cambios automáticos de Power BI Desktop en un commit `chore(...)` separado antes de iniciar trabajo intencional en cada sesión.
- Documentar cada cambio relevante como un nuevo `Outputs/NN_...md`, sin sobrescribir el historial.
- Actualizar `Docs/` cuando cambien medidas, páginas o fuentes de datos.
- Seguir la guía operativa de actualización de datos en [Docs/04_fuentes_y_actualizacion_datos.md](../Docs/04_fuentes_y_actualizacion_datos.md) para cada reexportación de los Google Forms.
- Revisar [Docs/05_decisiones_limitaciones_pendientes.md](../Docs/05_decisiones_limitaciones_pendientes.md) antes de tratar cualquier supuesto provisional (D3, D4, D5) como definitivo.

## 13. Criterios de cierre

Estado frente a los criterios de cierre definidos en `Specs/02` §9:

- [x] Las 18 fases fueron ejecutadas y sus validaciones respectivas aprobadas (Fase 17, `Outputs/31`, sin hallazgos críticos).
- [x] El modelo semántico no presenta relaciones ambiguas ni tablas sin relación (confirmado en Fase 17).
- [x] Las medidas DAX del catálogo están implementadas, documentadas (`Docs/02`) y fueron validadas contra los datos reales (Fase 11).
- [x] Las páginas del informe están construidas, respetando el máximo de 4-7 visuales principales por página (7 páginas construidas; ver desviación de alcance documentada en §2 respecto a las 8 originalmente esbozadas).
- [x] El tema visual Connect está aplicado de forma consistente en todo el informe, con colores reales de marca (no placeholder).
- [x] La navegación entre Home y páginas internas funciona en ambas direcciones, reforzada con zonas clicables completas.
- [x] Las advertencias de bajo volumen (`n=`) y de limitación de la encuesta anónima de motivación son visibles y dinámicas.
- [x] La documentación final (`Specs/03`, este documento, más `Docs/00`–`06` y [README.md](../README.md)) está completa y refleja el estado real del modelo, incluyendo decisiones provisionales pendientes de confirmación de negocio.
- [x] El proyecto está versionado en Git con historial completo de commits de cierre por fase.

**Pendiente de confirmación del usuario, no bloqueante para este cierre documental** (ya señalado en `Outputs/31` §13): apertura sin errores en una sesión limpia de Power BI Desktop, cálculo en vivo de las tarjetas KPI, comportamiento interactivo real de segmentadores y navegación — validaciones que requieren la interfaz gráfica de Power BI Desktop, no ejecutable desde este entorno.

## 14. Recomendación final

La implementación inicial del informe `PBI_Indicadores` se considera **cerrada** conforme a los criterios de la sección 13. El informe está construido, validado estructuralmente, documentado y publicado. Se recomienda:

1. Que el usuario confirme visualmente en Power BI Desktop los puntos pendientes señalados en `Outputs/31` §13 (apertura sin errores, cálculo en vivo, interacción real de segmentadores y navegación).
2. Que negocio resuelva en paralelo las dependencias D3, D4 y D5 (rúbrica de calidad, catálogo oficial, alias de líderes) para poder retirar los supuestos provisionales del informe.
3. Que se confirme el modelo de gobierno de la publicación pública (§8) antes de distribuir el enlace ampliamente.
4. Que cualquier trabajo futuro (conexión automática a la fuente, `Dim_Colaborador` maestro, RLS, limpieza de Auto Date/Time, nuevas páginas o medidas) se trate como una nueva iniciativa versionada, no como una modificación silenciosa de este cierre — siguiendo el mismo patrón de fases y documentación ya establecido en este proyecto.

No se requieren más cambios de código para dar por cerrada esta primera implementación.
