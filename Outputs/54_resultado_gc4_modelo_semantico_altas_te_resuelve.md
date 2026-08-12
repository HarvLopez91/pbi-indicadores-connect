# Resultado GC-4 — Modelo semántico de altas Te Resuelve

## Objetivo

Integrar las altas de Te Resuelve al modelo estrella mediante una tabla de hechos de grano agregado, reutilizando las dimensiones aprobadas y conservando sin cambios el reporte existente.

## Implementación técnica

- Commit técnico: `5d082b7b39f121a38bfe2d168e2f6bead1c0d25b`.
- Se creó `Fact_AltasTeResuelve` con 26.496 filas y 33.854 altas.
- La fact conserva el grano de fila agregada de `Insumo2`: una fila puede representar múltiples altas y el volumen se calcula mediante `SUM(Altas)`.
- La lista blanca de la fact contiene únicamente `FechaAlta`, `AliadoKey` y `Altas`.
- Se reutilizó `Dim_Aliado`, con 167 claves únicas, sin nulos, duplicados ni hechos huérfanos.
- Se modificó de forma controlada `Dim_Calendario` para incorporar la cuarta fuente y mantener el rango del gate entre 01/01/2026 y 11/08/2026.
- Se agregaron `Periodo_Gestion`, `Estado_Periodo` y `Es_Periodo_Comparable`.
- Se crearon dos relaciones activas, uno a muchos y con filtro unidireccional desde dimensión hacia hecho:
  - `Dim_Calendario[Fecha]` → `Fact_AltasTeResuelve[FechaAlta]`.
  - `Dim_Aliado[AliadoKey]` → `Fact_AltasTeResuelve[AliadoKey]`.
- Se confirmaron cero fechas huérfanas y cero aliados huérfanos.
- Las `LocalDateTable` permanecieron en 3; no se modificó Auto date/time.

## Validaciones

- Enero–julio: 33.295 altas.
- Julio: 4.518 altas.
- Agosto parcial: 452 filas y 559 altas.
- Junio: `Antes de gestion`, `Cerrado`, comparable.
- Julio: `Desde inicio gestion`, `Cerrado`, comparable.
- Agosto: `Desde inicio gestion`, `En curso`, no comparable.
- Regresión: `Fact_CalidadLlamadas` = 3, `Fact_SatisfaccionCapacitacion` = 150 y `Fact_MotivacionActividad` = 5.
- El gate manual y el gate MCP finalizaron aprobados con observación.

## Observación del gate

Durante el diagnóstico se observó una instancia MCP anterior con 145 filas de Satisfacción. La fuente física actual, la consulta limpia y la instancia refrescada confirmaron 150 filas y fecha máxima 28/07/2026. El estado anterior se clasificó como materialización o caché previo, sin evidencia de regresión causada por GC-4.

## Exclusiones y estado final

- Sin cambios en páginas, navegación, `Data`, Excel fuente ni objetos de Satisfacción.
- Sin datos personales ni registros individuales incorporados a la fact comercial.
- `_Medidas_Altas` y las medidas comerciales se difirieron a GC-5.
- Estado: **GC-4 cerrado / GC-5 no iniciado**.
