# Resultado R9 — Incentivos y Legalización

| Campo | Valor |
|---|---|
| Fase | R9 — Página 3 / Incentivos y Legalización |
| Baseline | `d6c86abdfbe6a2fb1d6adb2a1482d39413fd2cd5` (corrección R7/R8) |
| Página | `IncentivosLegalizacion` (“Incentivos y Legalización”), 1280 × 720, FitToPage |
| Gate técnico (refresh + DAX) | PASS |
| Gate visual Desktop | PASS |

## 1. Contrato

- Fuente: `Fact_LegalizacionBonos` (hoja `Legalización_Bonos` del Excel privado). Grano: una entrega de incentivo por Fecha + PUSHER + Concepto + Tipo de incentivo.
- **Valor recibido** = dinero entregado por Connect al PUSHER para incentivos (`[Valor_Recibido]`).
- **Valor legalizado** = valor registrado como legalizado en la fuente (`[Valor_Legalizado]`).
- **Pendiente por legalizar** = recibido − legalizado (`[Pendiente_Legalizar]`; legalizado vacío cuenta como 0).
- Campos usados: Fecha, Pusher, Concepto, TipoIncentivo, ValorRecibido, ValorLegalizado.
- Excluidos: aliado, asesor, estado de legalización, soporte, medio de pago, observaciones, saldo, valor gastado, ROI, costo por alta. Sin relaciones con ventas, aliado, asesor ni `Dim_AsignacionPusherPeriodo`.
- Las notas de trabajo que se habían escrito debajo de la tabla de la fuente se movieron a una hoja independiente `Notas`; `Legalización_Bonos` contiene solo registros tabulares.

## 2. Conciliación (refresh completo)

| Mes | Filas | Recibido | Legalizado | Pendiente | Corte |
|---|---:|---:|---:|---:|---|
| Julio | 11 | 3.300.000 | 3.300.000 | 0 | 30/07/2026 |
| Agosto | 10 | 2.668.800 | 2.668.800 | 0 | 26/08/2026 |
| Septiembre | 9 | 3.800.000 | 1.603.570 | 2.196.430 | 26/09/2026 |
| **Total** | **30** | **9.768.800** | **7.572.370** | **2.196.430** | 26/09/2026 |

- 0 pendientes negativos, 0 filas fuera de calendario, 0 filas sin PUSHER.
- Septiembre por PUSHER (recibido / legalizado / pendiente): PUSHER 1 1.500.000 / 595.000 / 905.000; PUSHER 2 1.500.000 / 1.008.570 / 491.430; PUSHER 3 800.000 / — / 800.000. Julio: 1.500.000 / 1.500.000 / 300.000 (todo legalizado). Agosto: PUSHER 1 1.200.000, PUSHER 2 1.468.800 (todo legalizado).
- Tipos de incentivo (todas las fechas): BONO 27 filas; COMETA, AUDIFONOS INALAMBRICOS y BONO CREPES & WAFFLES 1 fila cada uno. Septiembre solo tiene BONO.

## 3. Página

- **Filtros:** Mes (`Dim_Calendario[AnioMes]`, selección única, septiembre 2026 inicial), PUSHER (`Fact_LegalizacionBonos[PusherNombre]`), Tipo de incentivo (`Fact_LegalizacionBonos[TipoIncentivo]`). Sin Aliado ni Asesor.
- **KPI:** Valor recibido, Valor legalizado, Pendiente por legalizar (nota “Recibido - Legalizado”), Fecha de corte (`[Fecha_Corte_Incentivos]`).
- **Visuales:** Valor recibido vs legalizado por PUSHER (columnas agrupadas); Valor legalizado por tipo de incentivo (dona); Pendiente por legalizar por PUSHER (barras); Evolución de incentivos y legalización (columnas por mes; el slicer Mes no filtra este visual, PUSHER y Tipo sí); Detalle de incentivos y legalización (Fecha, PUSHER, Concepto, Tipo de incentivo, Valor recibido, Valor legalizado, Pendiente por legalizar; Fecha descendente; con total).
- “Evolución de incentivos y legalización” sustituye a “Altas vs valor entregado por asesor” del mockup: no existe relación demostrable con asesor.
- No se construyen: Asesores bonificados, Costo por alta, Desempeño e incentivos por asesor, filtro Aliado, Estado de legalización. “Incentivo potencial vs pago registrado” queda para una fase posterior.
- **Modo de enfoque:** activo en los 5 visuales analíticos (3 gráficos, dona y tabla); no en tarjetas, slicers, textos, botones ni formas.
- **Navegación:** las pestañas Resumen Comercial ↔ Asesores ↔ Incentivos y Legalización quedan activas en las tres páginas (se retira “· próximamente” en R7 y R8). “Volver a Home” se mantiene. En Desktop los enlaces requieren Ctrl+clic.

## 4. Modelo

- `Fact_LegalizacionBonos` conserva dos columnas de PUSHER del mismo contrato fuente: `Pusher` (código técnico PUSHER 1/2/3, clave usada por `Valor_Legalizado_Filtro_Pusher` en R7) y `PusherNombre` (nombre visible: `Legalización_Bonos[PUSHER]` normalizado con Trim/Clean/mayúsculas, ordenado por `Pusher`). La consulta ya validaba la equivalencia nombre ↔ código contra `Asignacion_PUSHER` y falla si deja de ser 1:1 o si una fila queda sin código; verificado 1:1 (13 / 15 / 2 filas). Los nombres salen de la fuente, igual que en R7/R8; no se escriben en PBIR ni DAX. R9 muestra solo `PusherNombre`.
- Sin tablas, relaciones, dimensiones ni medidas nuevas; 18 relaciones y 0 many-to-many.
- Formato monetario en `[Valor_Recibido]` y `[Valor_Legalizado]` (`$#,0`) y `[Pendiente_Legalizar]` (`$#,0;-$#,0;$0`). Ningún otro visual usaba estas medidas.

## 5. Gate

- Refresh completo PASS (99 s); 0 medidas con error.
- Filtros validados: julio 3.300.000 / 3.300.000 / 0; julio + PUSHER 1 1.500.000 / 1.500.000 / 0; Tipo de incentivo responde por DAX (septiembre solo BONO).
- Regresión sin cambios: R7 septiembre 6.528 altas, meta 11.080, ERIKA 2.100 / 766 = 36,48 %, valor legalizado 1.603.570; LEONARDO julio 2.959 / 1.582 = 53,46 %; R8 septiembre 673 con meta, 9 cumplieron, potencial y potencial pendiente 2.000.000; `GestionComercialAltas` julio 1.582 / 2.429 / 508.
- Visual (Desktop, capturas locales fuera de Git): septiembre completa con nombres visibles, slicer PUSHER abierto, septiembre + LEONARDO (1.500.000 / 595.000 / 905.000), julio, navegación R7 → R9 y R8 → R9. Ningún código técnico visible en R9.

## 6. Limitaciones

- Un nombre en `Legalización_Bonos[PUSHER]` que no exista en `Asignacion_PUSHER` detiene el refresh (control deliberado).
- Septiembre tiene un solo tipo de incentivo; la dona muestra una sola porción.
- Con una sola serie en cero (por ejemplo, pendiente de julio), el eje del gráfico de pendiente muestra decimales ($0,0 – $1,0).
- Algunas etiquetas de datos se superponen en columnas contiguas con valores cercanos.

## 7. Rollback

Contra `d6c86ab`: eliminar `pages/IncentivosLegalizacion/` y su entrada en `pages.json`; eliminar `rc_tab_incentivos_hitzone` y `as_tab_incentivos_hitzone`; restaurar las etiquetas `rc_tab_incentivos_label` y `as_tab_incentivos_label` (“· próximamente”); restaurar el formato `#,0` de las tres medidas; retirar `PusherNombre` de `Fact_LegalizacionBonos`. La hoja `Notas` del Excel privado no afecta al modelo.
