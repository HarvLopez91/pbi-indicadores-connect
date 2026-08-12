# Resultado GC-1 — Ingesta de altas Te Resuelve

## Objetivo

Establecer una lectura controlada y trazable de la tabla formal `Insumo2`, con parámetros mantenibles y validaciones estructurales, sin cargar todavía una tabla final al modelo semántico.

## Implementación

- **Commit:** `0716f8a23524d4a69f645ec3b2afba9ab2668aac`
- **Archivos modificados:**
  - `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
  - `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl`
- **Objetos creados:**
  - `Ruta_Informe_Altas`, expresión compartida derivada de `RutaCarpetaData`;
  - `Periodo_Corte_Comercial = 202607`;
  - `Fecha_Inicio_Gestion_Pusher_2 = 2026-07-01`;
  - `Base_AltasTeResuelve`, staging con carga deshabilitada.

## Validaciones

- Fuente localizada por nombre mediante la tabla formal `Insumo2`.
- Columnas obligatorias confirmadas: `ALTAS`, `DESCRIPCION`, `FECHA_ALTA` y `MES`.
- Periodos detectados: `202601` a `202608`.
- Periodo oficial de corte confirmado: `202607`.
- El gate manual M-1/M-2 confirmó en Power BI Desktop que la vista previa actualiza sin errores, usa la fuente esperada y mantiene `Base_AltasTeResuelve` sin carga.
- No se crearon `Fact_AltasTeResuelve`, `Dim_Aliado`, medidas DAX ni relaciones.
- No se modificaron páginas, navegación, Satisfacción ni otros objetos del reporte.
- El Excel fuente permanece fuera de Git, cubierto por `Data/**/*.xlsx`.

## Riesgos y decisiones

- Agosto de 2026 está disponible de forma parcial y se conserva; no se elimina mediante filtros fijos.
- `Periodo_Corte_Comercial` gobierna el último periodo oficial cerrado y deberá actualizarse solo después de aprobación del cierre mensual.
- `Ruta_Informe_Altas` no duplica una ruta personal absoluta: se deriva del parámetro existente `RutaCarpetaData`.
- La huella operativa de fuente combina nombre, tamaño y fecha de modificación; no se versionan filas ni datos personales.
- La limpieza, normalización, privacidad y control de filas inválidas corresponden a GC-2.

## Estado final

**GC-1 cerrado. GC-2 no iniciado.**
