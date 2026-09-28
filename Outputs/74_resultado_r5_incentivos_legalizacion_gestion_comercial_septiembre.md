# Resultado R5 — Incentivos y legalización

| Campo | Valor |
|---|---|
| Fase | R5 — Incentivos / Legalización |
| Baseline | `791c18484409ba1c9be67f4a4f413712e126d951` (cierre R4) |
| Fuente privada | `BI - NUEVO.xlsx`, hoja `Legalización_Bonos` (no versionada) |
| Estado | Validación estática y gate Desktop PASS |
| R6 | No iniciado |

## 1. Definiciones funcionales confirmadas por negocio

- `Valor recibido`: dinero que Connect entrega al PUSHER para realizar incentivos.
- `Valor gastado`: dinero efectivamente utilizado y reportado mediante facturas, recibos o soportes.
- `Pendiente por legalizar`: fuera del alcance de esta fase por decisión funcional.

## 2. Grano

Una fila = una entrega de incentivo por `Fecha + PUSHER + Concepto + Tipo de incentivo`. La fuente tiene 29 filas, sin duplicados exactos ni duplicados por esa clave. Una misma fecha admite varias entregas legítimas de tipos distintos; no se deduplica.

## 3. Campos

| Incluido en `Fact_LegalizacionBonos` | Origen |
|---|---|
| `Fecha` | `Fecha` (autoridad temporal; el periodo se obtiene de `Dim_Calendario`) |
| `Pusher` | `PUSHER`, traducido a `PUSHER 1/2/3` con las anclas públicas de R3 |
| `Concepto` | `Concepto` |
| `TipoIncentivo` | `Tipo de incentivo` |
| `ValorRecibido` | `Valor recibido` |
| `ValorGastado` | `Valor gastado` |

Excluidos:

- `Valor legalizado`: supera al gasto y coincide casi siempre con lo recibido; su comportamiento no corresponde a su nombre y no es necesario para el alcance aprobado.
- `Pendiente por legalizar`: fuera de alcance; en la fuente es `Gastado − Legalizado` con signo negativo. No se usa ni se deriva.
- `Saldo`, `Últimos 4 dígitos tarjeta`, `Observaciones`, `Aliado / Call`: vacíos en la fuente.
- `Estado legalización`: no es autoridad financiera (hay filas `CERRADO` con diferencias) y no aporta al alcance mínimo.
- `Medio de pago`, `Soporte`: no aportan al alcance mínimo.
- `Mes`: solo se usa para validar que coincide con `Fecha`.

No se modelan saldo, pendiente, ejecución, porcentaje de ejecución ni ROI. La diferencia entre recibido y gastado no se expone como métrica.

## 4. Reglas implementadas

- Los cuatro `Valor gastado` vacíos de la fuente permanecen `BLANK`/`null`; no se convierten en 0 porque no está definido si significan no gastado, no reportado o pendiente de soporte. R6 decidirá cómo presentarlos.
- La consulta falla el refresh si falta una columna obligatoria, si `Mes` no coincide con `Fecha`, si la equivalencia PUSHER no es unívoca o si algún PUSHER queda sin etiqueta pública.
- El identificador nominal de PUSHER se usa solo en memoria y no sale de la consulta.
- La equivalencia PUSHER repite las tres anclas de R3 dentro de la partición, en lugar de modificar `Map_AsignacionPusherFuente`, para no reabrir R3.

## 5. Relaciones

- `Rel_Calendario_Legalizacion`: `Fact_LegalizacionBonos[Fecha]` * → 1 `Dim_Calendario[Fecha]`, unidireccional.
- No hay relación con Aliado, Asesor ni `Dim_AsignacionPusherPeriodo`; `Pusher` es un atributo de la fact.
- Sin dimensión PUSHER nueva y sin many-to-many.

## 6. Conciliación (fuente = modelo tras refresh)

| Control | Resultado |
|---|---|
| Filas | 29 |
| `ValorRecibido` | 29 filas con valor; suma 8.698.800 |
| `ValorGastado` | 25 filas con valor; suma 7.089.800; 4 `BLANK`; 0 ceros |
| Fechas | 01/07/2026 – 23/09/2026 (corte observado); 0 filas sin calendario |
| Filas julio / agosto / septiembre | 11 / 10 / 8 |
| Recibido julio / agosto / septiembre | 3.300.000 / 2.398.800 / 3.000.000 |
| Gastado julio / agosto / septiembre | 3.300.000 / 2.668.800 / 1.121.000 |
| PUSHER 1 / 2 / 3 (filas) | 13 / 15 / 1 |
| Concepto | BONO DIARIO 26; ACTIVACION 3 |

Integridad del proyecto: ALTAS 44.638 (julio 4.519, agosto 5.715, septiembre 5.628); metas 6.337 / 7.155 / 11.080; `Fact_MetasComerciales` 49 filas; many-to-many totales en el modelo: 0. `GestionComercialAltas`, R2, R3 y R4 sin cambios.

## 7. Gate Desktop

PASS. Power BI Desktop abrió el PBIP; el refresh completo se ejecutó en su motor local mediante TOM y la conciliación se consultó con DAX. La instancia se cerró sin guardar y no dejó cambios en el repositorio. En la próxima sesión que guarde, Desktop añadirá `Fact_LegalizacionBonos` a `PBI_QueryOrder` y generará `lineageTag`; es ruido esperado.

## 8. Privacidad

No se versionan Excel, nombres de PUSHER, asesores, especialistas, ganadores, correos, datos de tarjeta, soportes ni rutas privadas. Este Output no contiene datos personales.

## 9. Limitaciones y decisiones pendientes

- Volumen bajo: 29 entregas en tres meses.
- Semántica de `ValorGastado` vacío pendiente para R6.
- `Valor legalizado` y `Pendiente por legalizar` quedan fuera hasta nueva definición funcional.
- Atribución por fecha de inicio de gestión (PUSHER 2 desde julio de 2026, PUSHER 3 desde agosto de 2026) corresponde a R6.

## 10. Rollback

Eliminar `tables/Fact_LegalizacionBonos.tmdl`, la línea `ref table Fact_LegalizacionBonos` de `model.tmdl` y la relación `Rel_Calendario_Legalizacion` de `relationships.tmdl`. No revertir R2, R3 ni R4.
