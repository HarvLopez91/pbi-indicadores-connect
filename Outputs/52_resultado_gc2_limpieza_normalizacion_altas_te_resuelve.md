# Resultado GC-2 — Limpieza y normalización de altas Te Resuelve

## Objetivo

Producir una consulta limpia, tipada y segura a partir de `Base_AltasTeResuelve`, conservando todas las filas y periodos y sin cargar todavía una tabla final al modelo semántico.

## Implementación

- **Commit técnico:** `de9318080795e7c56e8a080e98b7cb7803f3bf27`
- **Objeto creado:** `AltasTeResuelve_Limpio`, con carga deshabilitada.
- **Objeto ajustado:** `PBI_QueryOrder`, únicamente para registrar el orden de la nueva consulta.
- **Archivos técnicos:**
  - `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`;
  - `PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl`.

## Reglas aplicadas

- `Altas`: número entero.
- `FechaAlta`: fecha.
- `Mes`: número entero `AAAAMM`.
- `Descripcion`: texto con `Trim`, `Clean` y mayúsculas.
- Normalización de los demás nombres técnicos a `PascalCase` sin espacios ni caracteres especiales.
- Eliminación de `JEFE`, `ESPECIALISTA` y `ASESOR` antes de continuar con la limpieza.
- Conservación de todos los periodos disponibles, incluido agosto parcial.

## Controles

- `MesCoincideFecha`: valida la correspondencia entre `Mes` y `FechaAlta`.
- `EstadoPeriodo`: clasifica `Cerrado`, `En curso` o periodos posteriores no validados.
- `EsPeriodoComparable`: identifica periodos hasta el corte oficial `202607`.
- `MotivoInvalidez`: registra de forma no nominal las reglas incumplidas.
- `EstadoFila`: clasifica cada fila como `Valida` o `Invalida`.
- La consulta genera error si cambia el número de filas o la suma interpretable de altas.

## Conciliación y gate manual

- Filas antes y después: **26.496**.
- Altas antes y después: **33.854**.
- Filas inválidas: **0**.
- Discrepancias entre `Mes` y `FechaAlta`: **0**.
- Filas en periodos `Cerrado`: **26.044**.
- Filas de agosto de 2026 `En curso`: **452**.
- El gate manual confirmó tipos, columnas de control, estado de agosto, ausencia de errores en la vista previa y carga deshabilitada.

## Exclusiones confirmadas

No se crearon `Dim_Aliado`, `Fact_AltasTeResuelve`, `_Medidas_Altas`, relaciones, medidas DAX, páginas ni navegación. No se modificaron `Data`, el Excel fuente ni Satisfacción. No se reprodujeron datos personales ni registros individuales.

## Estado final

**GC-2 cerrado. GC-3 no iniciado.**
