# Resultado R8 — Asesores

| Campo | Valor |
|---|---|
| Fase | R8 — Página 2 / Asesores (histórica) |
| Baseline | `12251993dbe6df7f5366973937b8cd533cb9916d` (cierre R7) |
| Página | `Asesores` (“Asesores”), 1280 × 720, FitToPage |
| Gate técnico (refresh + DAX) | PASS |
| Gate visual | PASS (§8) |
| R9 | No iniciado |

## 1. Línea base de datos

El consolidado de altas se actualizó durante R8 (nueva carga al corte, aprobada por negocio):

- 35.733 filas y **45.538 altas**; `MAX(FECHA_ALTA)` = 27/09/2026; 0 filas duplicadas.
- Enero-agosto sin cambios (julio 4.519, agosto 5.715).
- Septiembre **6.528** (antes 5.628): +908 altas del 24 al 27/09 y −8 altas revisadas por la fuente en días ya cerrados (del 1 al 23/09 pasa de 5.628 a 5.620).

## 2. Contrato aprobado

- **Asesor:** `Insumo2[ASESOR]` (no `ESPECIALISTA`). Ventas por asesor desde enero de 2026.
- **Niveles de premio:** cada fila `ASESOR` de `Metas_Bonos` es un nivel = `Ranking_Valor_Incentivo` + `Meta_Ventas` + `Valor_Incentivo` por periodo + aliado. Un nivel puede tener meta propia (por ejemplo, R1 = 750 y R2/R3 = 120 en un aliado).
- **Meta de referencia** = menor `Meta_Ventas` de los niveles del contexto (meta mínima), salvo para un asesor dedicado, cuya referencia es la meta de su nivel. Faltan = MAX(meta − altas, 0); % Avance = altas / meta.
- **Asesor dedicado:** `Metas_Bonos[Asesor_Dedicado]` (Excel privado) reserva un nivel a un asesor concreto, identificado por coincidencia exacta con `Insumo2[ASESOR]` (Trim/Clean, sin fuzzy matching). Si alcanza la meta de ese nivel, lo gana y queda fuera de los demás. Si no la alcanza, queda excluido de todos los incentivos del mes y ese nivel se compite entre los demás asesores con la meta general del contexto (menor meta de los niveles sin dedicado). El nombre solo existe en el Excel; no está en el código ni en la documentación. **Control fail-fast:** cada `Asesor_Dedicado` informado debe resolver a exactamente una identidad de `Insumo2[ASESOR]` en su mismo periodo y aliado (Trim/Clean/mayúsculas); sin coincidencia o con ambigüedad, el refresh se detiene con error y no hay fallback a la lógica general. Validado: las 3 configuraciones actuales resuelven a una identidad; un nombre alterado o un aliado distinto producen error. Caso actual: un aliado, nivel R1 con meta 750 y meta general 120.
- **Estados:** Cumplió ≥ 100 %; Cerca de cumplir 75-100 %; En progreso 50-75 %; Rezago < 50 %; Sin meta cuando el contexto no tiene niveles configurados.
- **Asignación secuencial (incentivo potencial / bono objetivo):** por contexto, niveles en ranking ascendente; en cada nivel gana el asesor con más altas que alcanza la meta aplicada de ese nivel y aún no ganó otro (con la regla de asesor dedicado de arriba). Un asesor recibe como máximo un incentivo. Nivel sin candidato = desierto. Empate exacto en el primer lugar = QA sin ganador (no hay desempate arbitrario).
- **Pago real:** `Cumplimiento_Meta = SI` indica que el nivel fue pagado. **Bonos entregados** = suma de `Valor_Incentivo` de esos niveles en los contextos visibles.
- **Limitación de `Ganador_Bono`:** ningún valor coincide exactamente con `Insumo2[ASESOR]` (son nombres cortos). Sin fuzzy matching no se atribuye ningún pago a un asesor: no existen “Valor entregado” ni “Saldo” por asesor.
- **Potencial pendiente** = suma de `Valor_Incentivo` de los niveles con ganador potencial y sin evidencia de pago en ese mismo nivel (periodo + aliado + ranking). No es Potencial − Pagado (un pago puede corresponder a un nivel sin ganador potencial) y **no representa deuda**.

## 3. Arquitectura

- `AltasTeResuelve_Limpio` conserva `ASESOR` (Trim/Clean) como `Asesor`; `JEFE` y `ESPECIALISTA` siguen fuera. `Fact_AltasTeResuelve[Asesor]`; sin segunda fact de ventas.
- `Config_MetasComerciales` (R4): CALL y ESPECIALISTA conservan su llave; en ASESOR la llave incluye `Ranking_Valor_Incentivo` (una meta por nivel) y `MetaAsesor` pasa a ser la meta mínima del contexto. `Fact_MetasComerciales` mantiene el grano periodo + aliado.
- `Config_IncentivoAsesor` (oculta, sin relaciones): periodo + aliado + ranking + meta + valor, ganador potencial (asignación secuencial en Power Query con `List.Accumulate` sobre las ventas válidas del modelo), estado de asignación, pago y estado de conciliación. Validaciones: ranking entero positivo, una meta y un valor por nivel, claves únicas. Ventas y niveles se cargan con `Table.Buffer` para no reevaluar la limpieza por contexto.
- Sin `Dim_Asesor`, sin relaciones nuevas, 0 many-to-many.

## 4. Conciliación (refresh completo)

| Mes | Altas | Asesores | Con meta | Sin meta | Cumplieron | Niveles asignados / configurados | Desiertos | Incentivo potencial | Bonos entregados | Potencial pendiente |
|---|---|---|---|---|---|---|---|---|---|---|
| 2026-01 a 2026-06 | 6.296 / 5.309 / 5.110 / 4.190 / 4.171 / 3.700 | 1.173 / 959 / 775 / 622 / 690 / 632 | 0 | todos | — | — | — | — | — | — |
| 2026-07 | 4.519 | 687 | 457 | 230 | 2 | 2 / 42 | 40 | 600.000 | 600.000 | 0 |
| 2026-08 | 5.715 | 670 | 473 | 197 | 6 | 6 / 51 | 45 | 1.700.000 | 1.200.000 | 500.000 |
| 2026-09 | 6.528 | 862 | 574 | 288 | 5 | 5 / 54 | 49 | 1.400.000 | 0 | 1.400.000 |

- 147 niveles; 0 empates que afecten premios. Enero-junio: ninguna meta ni incentivo; todos “Sin meta”.
- Asesor dedicado (anonimizado): altas 331 / 66 / 472 frente a meta 750 → no cumple ningún mes; avance 44 % / 9 % / 63 %, estados Rezago / Rezago / En progreso, sin ranking ni bono. En su aliado, R1 pasa cada mes al mejor asesor restante con ≥ 120 altas (218 / 131 / 250); R2 y R3 desiertos.
- Resultados idénticos a la simulación independiente desde la fuente.

## 5. QA pago frente a potencial (sin nombres)

| Estado | Casos |
|---|---|
| Potencial y pagado | ATENTO R1 (julio, agosto); INTELIGENCE R1 (julio, agosto); BRM R1 y COS R1 (agosto) |
| Potencial no pagado | Agosto: ATENTO R2, MILLENIUM R1. Septiembre: AIB R1, ATENTO R1 y R2, BRM R1, INTELIGENCE R1 |
| Pagado sin potencial (QA) | Ninguno (INTELIGENCE R1 de julio y agosto dejó de serlo con la regla de asesor dedicado) |
| Desierto | 40 / 45 / 49 niveles |
| Empate (QA) | 0 casos actualmente; la regla queda activa |

No se modifican ni los pagos ni la simulación para hacerlos coincidir. `Ganador_Bono` no se usa para atribuir pagos (sin coincidencia exacta con `ASESOR`).

## 6. Página

- **KPI:** Asesores con meta; Cumplieron meta y Cerca de cumplir (con % sobre asesores con meta); Altas del periodo; Bonos entregados (pagos reales); Potencial pendiente (incentivo potencial sin pago registrado, no es deuda).
- **Detalle por asesor:** Asesor, PUSHER, Aliado, Altas, Meta mínima, Faltan, % Avance, Ranking, Bono objetivo, Estado. Sin totales, sin Valor entregado ni Saldo por asesor.
- **Top asesores por altas** (Top 5 por medida), **Estado de cumplimiento** (dona, % sobre asesores visibles) y **Asesores más cerca de cumplir** (Top 5 con meta y avance < 100 %).
- **Filtros:** Mes (selección única, septiembre 2026 como selección inicial nativa), PUSHER (`PusherNombre`), Aliado.
- **Navegación:** Resumen Comercial ↔ Asesores; “Incentivos y Legalización” sigue inactiva.

## 7. Regresión

Metas CALL 6.337 / 7.155 / 11.080 sin cambios. Enero-agosto sin cambios. R7 funciona con los datos nuevos (septiembre: 6.528 altas, corte 27/09, crecimiento +2.868). `GestionComercialAltas` intacta. 18 relaciones, 0 many-to-many, 0 medidas con error.

## 8. Gate Desktop

- Técnico PASS: refresh por tabla (Config_IncentivoAsesor 56 s; demás 3-77 s, todas Ready) y refresh completo en 113 s.
- Incidencia resuelta: una versión previa sin `Table.Buffer` reevaluaba las ventas por contexto (549 s y “Canalización interrumpida”); corregido.
- Visual PASS (capturas locales, fuera de Git): septiembre sin filtros (574 con meta / 5 cumplieron / 1 cerca / 6.528 altas, Bonos entregados $0, Potencial pendiente $1.400.000); aliado INTELIGENCE (asesor dedicado con meta 750 y 62,9 % de avance, En progreso, sin ranking; R1 $300.000 al mejor asesor restante con meta 120); PUSHER JEISY; enero (6.296 altas, 1.173 asesores, todos Sin meta, sin ranking ni incentivo); R7 con la pestaña Asesores funcional (Ctrl+clic en Desktop). Sin visuales rotos ni solapamientos.
- Ajustes durante el gate: “Bonos entregados” muestra $0 (no BLANK) cuando hay niveles sin pago; la tabla “Asesores más cerca de cumplir” se ordena por Faltan ascendente; el slicer Mes queda en selección única.
- Observación: en la tabla “más cerca de cumplir” algunos nombres y encabezados se ajustan en dos líneas.
- Nota: tras cambios de modelo, al abrir el PBIP hay que pulsar Actualizar en Desktop antes de revisar, guardar o publicar.

## 9. Privacidad

Exposición nominal de asesores autorizada. Los nombres salen de la fuente al refrescar; no hay nombres en TMDL, PBIR ni filtros (Top N por medida). No se cargan `JEFE` ni `ESPECIALISTA`. Este documento no contiene nombres de asesores. Excel y capturas fuera de Git.

## 10. Limitaciones

- La mayoría de asesores queda en Rezago frente a la meta mínima (30-140 frente a medianas de 1-17 altas).
- El nombre es la única identidad del asesor; homónimos en el mismo aliado y mes se sumarían.
- Pagos no atribuibles por asesor mientras `Ganador_Bono` no coincida exactamente con `ASESOR`.

## 11. Actualización posterior — UNO 27 y clasificación efectiva

Tras homologar `ABAI` → `UNO 27` en `Metas_Bonos` (fuente privada) y aplicar la prioridad override temporal > `Asignacion_PUSHER` > `Sin asignar` a `PusherAsignacion`/`PusherNombre` (ver Output 76 §14), los niveles ASESOR de UNO 27 se cruzan con sus ventas reales (antes su clave `ABAI` no tenía ventas). Resultados recalculados con refresh completo:

| Mes | Asesores con meta | Cumplieron | Cerca | Niveles asignados / configurados | Incentivo potencial | Bonos entregados | Potencial pendiente |
|---|---|---|---|---|---|---|---|
| 2026-07 | 524 (antes 457) | 2 | 1 | 2 / 42 | 600.000 | 600.000 | 0 |
| 2026-08 | 563 (antes 473) | 6 | 4 | 6 / 51 | 1.700.000 | 1.200.000 | 500.000 |
| 2026-09 | 673 (antes 574) | 9 (antes 5) | 7 (antes 1) | 8 / 54 (antes 5) | 2.000.000 (antes 1.400.000) | 0 | 2.000.000 |

- Septiembre suma UNO 27 R1 / R2 / R3 (meta 30; 300.000 / 200.000 / 100.000), “Potencial no pagado”. Julio y agosto no tienen ganadores nuevos.
- Septiembre por PUSHER (con meta / cumplieron / potencial): JEISY 387 / 3 / 800.000; LEONARDO 184 / 2 / 600.000; ERIKA 102 / 4 / 600.000.
- La regla del asesor dedicado y su control fail-fast no cambian (INTELIGENCE R1 sigue asignado al mejor asesor restante con meta 120).
- `Sin asignar` no tiene meta en ningún mes.

## 12. Rollback

Contra `1225199`: eliminar `pages/Asesores/`, su entrada en `pages.json` y `rc_tab_asesores_hitzone`; restaurar `rc_tab_asesores_label`; retirar `Config_IncentivoAsesor` y su `ref table`, las medidas R8, la columna `Asesor` y los cambios de `Config_MetasComerciales`. No revertir R2-R7.
