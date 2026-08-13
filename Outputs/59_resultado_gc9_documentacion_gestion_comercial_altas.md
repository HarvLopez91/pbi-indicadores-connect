# Resultado GC-9 — Documentación de Gestión comercial de altas

## Objetivo

Crear una guía funcional y técnica definitiva de la página `GestionComercialAltas`, dirigida a PUSHER, líderes comerciales, analistas y a quien mantenga el PBIP, sin modificar PBIR, TMDL, Power Query, `Data/`, Excel, relaciones, medidas ni páginas.

## Documento creado

`Docs/GESTION_COMERCIAL_ALTAS_TE_RESUELVE.md` — 214 líneas, 16 secciones (objetivo, fuente, modelo, clasificación PUSHER, regla temporal PUSHER 2, filtros, indicadores, gráfico histórico, drivers/ranking, periodos parciales, actualización y mantenimiento, controles de calidad, privacidad, exportación/Focus mode, interpretación comercial, glosario).

## Fuentes utilizadas

`AGENTS.md`, `Specs/07_analisis_impacto_gestion_comercial_altas_te_resuelve.md`, `Outputs/56`, `Outputs/57`, `Outputs/58`, y lectura directa del repositorio como fuente de verdad: `_Medidas_Altas.tmdl` (36 medidas), `Dim_Aliado.tmdl`, `Dim_Calendario.tmdl`, `Fact_AltasTeResuelve.tmdl`, `expressions.tmdl` (`Base_AltasTeResuelve`, `AltasTeResuelve_Limpio`, `Map_PusherAliado`, parámetros `RutaCarpetaData`, `Periodo_Corte_Comercial`, `Fecha_Inicio_Gestion_Pusher_2`), y el PBIR actual de `GestionComercialAltas` (títulos, KPI, filtros, interacciones).

## Temas cubiertos

Regla de cálculo (`Altas_Total`), grano agregado, clasificación PUSHER por coincidencia exacta (sin fuzzy matching) contra `Map_PusherAliado`, regla temporal de PUSHER 2 (línea base antes del 01/07/2026 vs. desde inicio de gestión), los 4 filtros (Año, Mes, PUSHER, Aliado) con la excepción documentada del segmentador Mes sobre el gráfico histórico (`NoFilter` intencional), los 6 KPI de la página con su medida DAX exacta, el gráfico apilado con sus 4 series y casos de control (junio = 3.700, julio = 4.518), drivers/ranking con la advertencia sobre `Delta_Aliado_Mes` y `BLANK()`, tratamiento de agosto como periodo parcial (`Estado_Periodo`, `Es_Periodo_Comparable`), procedimiento real de actualización (incluyendo que `Map_PusherAliado` es hoy una lista fija en Power Query, no una tabla externa editable), cifras de control del corte vigente, privacidad, exportación/Focus mode con la limitación conocida del título del eje, y una sección explícita de qué SÍ y qué NO se puede concluir comercialmente.

## Validaciones realizadas

- Las 14 medidas citadas por nombre técnico se verificaron una por una contra `_Medidas_Altas.tmdl` real: las 14 existen.
- Los 3 parámetros citados (`RutaCarpetaData`, `Periodo_Corte_Comercial`, `Fecha_Inicio_Gestion_Pusher_2`) y `Map_PusherAliado` se verificaron contra `expressions.tmdl` real.
- Las columnas citadas (`Dim_Calendario[Anio/MesNombre/Periodo_Gestion/Estado_Periodo/Es_Periodo_Comparable]`, `Dim_Aliado[Descripcion/Pusher/AliadoKey/Estado_Clasificacion]`, `Fact_AltasTeResuelve[FechaAlta/AliadoKey/Altas]`) se verificaron contra los `.tmdl` reales.
- Los 8 textos visibles citados (título del gráfico, títulos de drivers/ranking, etiquetas de KPI, subtítulo de página) se verificaron con búsqueda literal contra los `visual.json` reales de `GestionComercialAltas`: las 8 coincidencias exactas.
- Cero coincidencias de mojibake (verificado a nivel de bytes UTF-8).
- Cero rutas locales o nombre de usuario de Windows expuestos.
- Cero afirmaciones causales asertadas; la única mención de "PUSHER 2 causó" aparece dentro de la lista explícita de frases prohibidas, no como afirmación.
- Sin contradicciones con las cifras de `Outputs/56` (conciliación), `Outputs/57` (construcción) y `Outputs/58` (validación funcional/visual).

## Privacidad

El documento no reproduce nombres de `JEFE`, `ESPECIALISTA` ni `ASESOR`, ni conteos de personas, ni datos fila a fila. Únicamente documenta que esas 3 columnas se excluyen del modelo publicado, y expone solo las 7 columnas agregadas ya aprobadas del modelo comercial (`FechaAlta`, `AliadoKey`, `Altas`, `Descripcion`, `Pusher`, `Estado_Clasificacion`).

## Ausencia de cambios técnicos

`git status --short --untracked-files=all` mostró únicamente el archivo de documentación nuevo antes de comitear. No se modificó ningún archivo de `PBI/` (PBIR ni TMDL), `Data/`, Excel, relaciones, medidas ni páginas del reporte.

## Commit de la guía

SHA: `21adf7b` — `docs: documenta gestion comercial de altas`. Publicado en `origin/main`.

## Estado final

- GC-9: **cerrado**.
- GC-10: **no iniciado**.
