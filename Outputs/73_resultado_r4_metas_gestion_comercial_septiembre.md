# Resultado R4 — Metas

| Campo | Valor |
|---|---|
| Fase | R4 — Metas |
| Baseline | `d5f831c08b2127f2882cf001e82085cbdf97476c` (cierre R3) |
| Fuente privada | `BI - NUEVO.xlsx`, hoja `Metas_Bonos` (no versionada) |
| Estado | Validación estática y gate Desktop PASS, con observaciones |
| R5 | No iniciado |

## 1. Decisión funcional aplicada

- Una meta válida con cero ventas en el periodo existe en el modelo (meta > 0, altas = 0).
- Sin homologaciones inferidas ni fuzzy matching.
- `PusherAsignacion` (R3) es la única autoridad de clasificación. El PUSHER informado en `Metas_Bonos` se usa solo como control de calidad y nunca como respaldo.
- Un aliado de metas sin regla en `Asignacion_PUSHER` queda `Sin asignar`; su asignación debe corregirse en la fuente gobernada.

## 2. Cambio mínimo

| Objeto | Cambio |
|---|---|
| `Config_MetasComerciales` | Deja de ser una lista fija de julio y lee `Metas_Bonos` con controles. |
| `Fact_MetasComerciales` | Se añade `MetaAsesor`; se conserva el grano periodo-aliado y `MetaAltas`. |
| `Dim_Aliado` | Une las claves de ventas con las claves de metas que no tienen ventas. |
| `Dim_AsignacionPusherPeriodo` | Sin cambio: ya unía contextos de ventas y metas. |

No se crearon tablas, expresiones, relaciones, medidas ni cambios PBIR. No cambió `PBI_QueryOrder`.

## 3. Reglas implementadas en `Config_MetasComerciales`

- Solo se leen las columnas de meta. `Ventas_Cumplimiento`, `Cumplimiento_Meta`, `Ganador_Bono`, `Valor_Incentivo` y fechas de bono no se cargan.
- Normalización `Trim`/`Clean`/mayúsculas y filtro `Tipo_Meta = META PARA BONO`.
- Llave lógica `Año + Mes + PUSHER + Aliado + Asesor_Equipo + Tipo_Meta`: exactamente un `Meta_Ventas` distinto; si no, error de refresh.
- Por periodo-aliado: un único PUSHER, `CALL` obligatorio y `CALL = ESPECIALISTA`; si no, error de refresh.
- `MetaAltas = CALL`; `MetaAsesor = ASESOR` (meta contextual, no se divide ni se multiplica por incentivos). `ESPECIALISTA` no se carga.
- Las suboperaciones de R3 se enlazan por coincidencia exacta entre `Aliado_BI` y `DESCRIPCION2` de una regla específica. No se reparte ni duplica la meta del aliado principal.
- El identificador nominal de PUSHER se usa solo en memoria para validar la llave y no se expone.

## 4. Conciliación estática

| Control | Resultado |
|---|---|
| ALTAS acumuladas | 44.638 (sin cambio) |
| ALTAS julio / agosto / septiembre | 4.519 / 5.715 / 5.628 |
| Filas `Fact_MetasComerciales` | 49, claves periodo-aliado únicas |
| Meta CALL julio / agosto / septiembre | 6.337 / 7.155 / 11.080 |
| Llaves con más de un `Meta_Ventas` | 0 |
| CALL = ESPECIALISTA | 49/49 contextos, sin contextos incompletos |
| ASESOR | 147 filas → 49 metas contextuales |
| `Dim_Aliado` | 173 → 176 claves (3 aliados solo con meta) |
| Metas sin miembro de dimensión | 0 (antes 9) |

Las ALTAS no cambian: el hecho de ventas no se modificó, y `Dim_Aliado` solo añade claves nuevas sin duplicar claves existentes.

### Contextos con meta y cero ventas

Nueve contextos permanecen en el modelo con su meta y altas = 0: tres periodos de un aliado solo con meta, dos de otro, uno de un tercero, dos aliados con ventas en otros meses y una suboperación de R3 en septiembre. Suman 225 de meta en julio, 480 en agosto y 2.350 en septiembre.

## 5. PUSHER de la fuente de metas frente a R3

| Periodo | Fuente de metas | Clasificación gobernada R3 |
|---:|---|---|
| 202607 | P1 2.959 · P2 3.378 | P1 2.634 · P2 3.378 · Sin asignar 325 |
| 202608 | P1 2.740 · P2 3.475 · P3 940 | P1 2.640 · P2 3.475 · P3 540 · Sin asignar 500 |
| 202609 | P1 4.200 · P2 4.780 · P3 2.100 | P1 4.200 · P2 4.780 · P3 730 · Sin asignar 1.370 |

La diferencia corresponde a dos aliados con meta sin regla en `Asignacion_PUSHER`. Se reporta como QA y no se corrige: su clasificación debe incorporarse en la fuente gobernada si negocio lo aprueba.

## 6. Observaciones

1. `Cumplimiento_Meta_Pct` devuelve `BLANK` (no 0 %) cuando altas = 0, porque `Altas_Total` es `BLANK`. Mostrar 0 % requiere ajustar la medida, y eso corresponde a R6.
2. Los tres aliados solo con meta aparecen como opciones del segmentador Aliado de `GestionComercialAltas`, sin ventas. La tarjeta `% Cumplimiento` de esa página consume `Meta_Asignada`: en julio pasa de 71,54 % (meta fija anterior 6.317) a 71,31 % (meta oficial 6.337). Es el efecto esperado del cambio de fuente, no una regresión. El resto de cifras y visuales no cambia.
3. `Config_MetasComerciales` conserva su nombre para no alterar referencias ni el orden de consultas.
4. Gestión atribuible: PUSHER 2 desde julio de 2026 y PUSHER 3 desde agosto de 2026. Los meses anteriores son línea base histórica y no deben interpretarse como cumplimiento, crecimiento, impacto ni resultados atribuibles a su gestión. La implementación DAX de esta regla corresponde a R6; R4 no la implementa.

## 7. Privacidad

No se versionan Excel, nombres de PUSHER, especialistas, asesores, ganadores, correos ni rutas privadas. El Output no contiene datos personales.

## 8. Gate Desktop

PASS. Power BI Desktop 2.156 abrió el PBIP; el refresh completo se ejecutó contra su motor local mediante TOM y los controles se consultaron con DAX (MSOLAP). La instancia se cerró sin guardar y no dejó cambios en el repositorio.

- Refresh completo: PASS, sin errores; ningún control de `Config_MetasComerciales` se disparó.
- `Fact_MetasComerciales`: 49 filas, 49 claves únicas, 49 con `MetaAsesor`.
- Meta julio / agosto / septiembre: 6.337 / 7.155 / 11.080. ALTAS: 44.638; 4.519 / 5.715 / 5.628; la suma por aliado también da 44.638.
- `Dim_Aliado`: 176 filas y 176 claves. Metas sin miembro en `Dim_Aliado`, `Dim_AsignacionPusherPeriodo` o `Dim_Calendario`: 0. Meta bajo `PusherAsignacion` en blanco: 0.
- Por `PusherAsignacion` coincide con la sección 5 (columna R3). ALTAS por PUSHER idénticas a Output 72.
- Contextos con meta y cero ventas: 9 (225 / 480 / 2.350), todos bajo PUSHER 1, PUSHER 3 o Sin asignar.
- `GestionComercialAltas`: render PASS; histórico por `PusherPeriodo` sin cambios.

La consulta por PUSHER fue:

```DAX
EVALUATE CALCULATETABLE(
 SUMMARIZECOLUMNS(Dim_AsignacionPusherPeriodo[AnioMes], Dim_AsignacionPusherPeriodo[PusherAsignacion], "Meta", [Meta_Asignada], "Altas", [Altas_Total]),
 TREATAS({202607, 202608, 202609}, Dim_AsignacionPusherPeriodo[AnioMes]))
ORDER BY Dim_AsignacionPusherPeriodo[AnioMes], Dim_AsignacionPusherPeriodo[PusherAsignacion]
```

La separación entre línea base y gestión atribuible (PUSHER 2 desde 202607, PUSHER 3 desde 202608) se mantiene documentada y su DAX corresponde a R6.

## 9. Rollback

Restaurar exclusivamente `expressions.tmdl`, `Dim_Aliado.tmdl` y `Fact_MetasComerciales.tmdl` contra `d5f831c`. No revertir R2 ni R3.
