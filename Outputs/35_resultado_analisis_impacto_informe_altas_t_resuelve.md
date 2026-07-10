# Resultado operativo — Análisis de impacto informe de altas T Resuelve

## Estado inicial de git

Comandos ejecutados:

- `git status`
- `git branch --show-current`
- `git remote -v`
- `git ls-files Data`

Resultado:

- Rama actual: `main`.
- Remoto `origin`: `https://github.com/HarvLopez91/pbi-indicadores-connect.git`.
- `git ls-files Data` no retorna archivos; no hay Excel versionados desde `Data`.
- El working tree no inicio limpio porque aparece `Data/` sin seguimiento.
- Hallazgo: el patrón `.gitignore` actual `Data/*.xlsx` ignora Excel en la raíz de `Data`, pero no cubre el archivo anidado `Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx`.

No se agregaron archivos de `Data` al índice.

## Archivo analizado

`Data/Informe de Altas/INFORME ALTAS T RESUELVE CIERRE JUNIO 2026.xlsx`

Propiedades observadas:

- Tamaño aproximado: 4,4 MB.
- Última modificación: 2026-07-09 10:50:33.
- Fecha de corte visible dentro del informe: 2026-07-05.

## Documentos de referencia leídos

- `README.md`
- `Docs/00_indice_documentacion.md`
- `Docs/01_modelo_datos.md`
- `Docs/02_catalogo_medidas_dax.md`
- `Docs/03_mapa_reporte_paginas_visuales.md`
- `Docs/04_fuentes_y_actualizacion_datos.md`
- `Docs/05_decisiones_limitaciones_pendientes.md`
- `Docs/06_publicacion_powerbi.md`
- `Specs/01_analisis_de_impacto_informe_powerbi_connect.md`
- `Specs/02_plan_implementacion_informe_powerbi_connect.md`
- `Specs/03_documentacion_final_informe_powerbi_connect.md`

## Validaciones realizadas

- Inspección de hojas, dimensiones, tablas formales y hojas ocultas con `openpyxl`.
- Validación de rangos de fecha y periodo.
- Conteo de filas, columnas, nulos, tipos y cardinalidades.
- Comparación de `Tabla1` e `Insumo2` en columnas comunes.
- Agregación de altas por mes y aliado.
- Búsqueda de encabezados de metas dentro del Excel.
- Verificación de que no se modificaron archivos de Power BI.
- Verificación de que no se versionaron archivos Excel.

## Hojas y tablas detectadas

| Hoja | Estado | Resultado |
|---|---|---|
| `INFORME ALTAS MULTIASISTENCIAS` | Visible | Reporte con pivotes, fecha de corte y rankings nominales. |
| `Tablas_back` | Oculta | Pivotes auxiliares de junio por canal, servicio, tipo de paquete y día. |
| `Tabla1_2 (2)` | Oculta | Tabla formal `Insumo2`, 23.015 filas útiles y 18 columnas. |
| `Tabla1` | Oculta | Tabla formal `Insumo`, 23.015 filas útiles y 16 columnas. |

## Resumen de hallazgos

- El libro contiene cierre de junio con 3.700 altas.
- La base incluye enero-junio completos y julio parcial hasta 2026-07-05.
- Total acumulado en la base: 29.366 altas.
- Enero-junio suma 28.777 altas; julio parcial suma 589.
- `ALTAS` debe sumarse; no basta contar filas.
- La fuente contiene nombres reales de asesores, especialistas y jefes.
- No se encontraron tablas explícitas de metas por aliado o especialista dentro del Excel.
- El contexto de metas del mensaje debe considerarse información externa o fuente pendiente de estructurar.
- La caída promedio del 25% no se puede validar como regla única sin definir la base de comparación.
- La integración requiere modelo nuevo, no solo agregar visuales.

## Documentos creados

- `Specs/04_analisis_impacto_informe_altas_t_resuelve.md`
- `Outputs/35_resultado_analisis_impacto_informe_altas_t_resuelve.md`

## Confirmación de no modificación del informe

No se modificaron:

- `PBI/`
- PBIR
- TMDL
- Power Query
- Medidas DAX
- Relaciones
- Visuales
- Tema visual
- Archivos Excel de `Data`

## Confirmación de no versionamiento de Data

No se agregó ningún archivo de `Data` al índice de Git. El nuevo Excel permanece fuera del commit.

Riesgo documentado: el Excel anidado no está ignorado por el patrón actual `Data/*.xlsx`; se recomienda ajustar `.gitignore` en una tarea separada antes de cualquier integración.

## Estado final de git

Después del commit documental, `git status --short` conserva:

`?? Data/`

Ese estado corresponde al nuevo archivo local en `Data/Informe de Altas/`, no a cambios del análisis. No se agregó ningún archivo Excel al commit.

## Recomendación del siguiente paso

Crear un plan de implementación `v1.1` para altas T Resuelve, precedido por decisiones de negocio sobre privacidad, definición de metas, equivalencia aliado/call center y plantilla estándar mensual.

Commit creado:

`docs: analizar impacto de informe de altas t resuelve`
