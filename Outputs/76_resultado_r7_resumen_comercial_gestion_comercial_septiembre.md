# Resultado R7 — Resumen Comercial

| Campo | Valor |
|---|---|
| Fase | R7 — Página 1 / Resumen Comercial |
| Baseline | `d926486a55d0ebf6de90feedcf63d636991fa892` (hotfix R5/R6) |
| Página | `ResumenComercial` (“Resumen Comercial”), 1280 × 720, FitToPage |
| Estado | Gate técnico y visual PASS, con observaciones |
| R8 | No iniciado |

## 1. Alcance

Página nueva basada en `mockup-pagina-1.jpeg`. `GestionComercialAltas` no se modificó. Los números del mockup son ilustrativos y no se copiaron.

Cambios:
- `pages/ResumenComercial/` (page.json y 32 visuales).
- `pages/pages.json`: la página se inserta después de `GestionComercialAltas`; la página activa no cambia.
- `_Medidas_Altas.tmdl`: medida auxiliar `Valor_Legalizado_Filtro_Pusher`.

Sin relaciones, dimensiones, facts ni many-to-many nuevos.

## 2. KPI

| KPI | Medida |
|---|---|
| Altas | `[Altas_Total]` |
| Meta | `[Meta_Asignada]` |
| % Cumplimiento | `[Cumplimiento_Meta_Pct]` |
| Faltante | `[Faltante_Meta]` |
| Crecimiento desde julio | `[Crecimiento_Desde_Julio_Pct]` |
| Valor legalizado | `[Valor_Legalizado_Filtro_Pusher]` (sobre `[Valor_Legalizado]`) |

No se muestran valor recibido, pendiente, valor gastado, ejecución ni ROI (recibido y pendiente quedan para R9).

Fecha de corte principal: `[Fecha_Corte_Altas]` (corte comercial de ALTAS). `[Fecha_Corte_Incentivos]` aparece solo en el pie, rotulada como fecha de información de incentivos.

## 3. Valor legalizado y filtros (diferencia de granularidad)

`Fact_LegalizacionBonos` tiene `Pusher` como atributo propio y no tiene aliado ni relación con `Dim_AsignacionPusherPeriodo`. Sin la medida auxiliar el KPI ignoraba el slicer PUSHER.

`Valor_Legalizado_Filtro_Pusher` aplica `TREATAS(VALUES(PusherAsignacion), Fact_LegalizacionBonos[Pusher])` solo cuando `ISFILTERED(Dim_AsignacionPusherPeriodo[PusherAsignacion])`; en otro caso devuelve `[Valor_Legalizado]`. Como `Dim_Aliado` no filtra a `Dim_AsignacionPusherPeriodo`, un filtro de Aliado no se transmite al KPI.

- Sin PUSHER: total del mes. PUSHER 1 / 2 / 3: su legalizado. Varios: suma. Sin asignar: BLANK.
- Aliado: no afecta el KPI. La tarjeta lo indica con la nota “Responde a Mes y PUSHER; no a Aliado”.
- No se simula ninguna relación Aliado → legalización.

## 4. Visuales e interacciones

| Visual | Contenido |
|---|---|
| Slicers | Mes (`Dim_Calendario[AnioMes]`, selección única, septiembre 2026 por defecto con filtro nativo del slicer), PUSHER (`PusherAsignacion`), Aliado (`Dim_Aliado[Descripcion]`) |
| Altas vs Meta por PUSHER | Columnas agrupadas `PusherAsignacion` × Altas, Meta |
| Evolución mensual de altas | Líneas Altas y Meta por mes; el slicer Mes no filtra este visual (interacción `NoFilter`), PUSHER y Aliado sí |
| Crecimiento por PUSHER desde julio | `[Crecimiento_Desde_Julio_Atribuible]`, con `%` en tooltip |
| Detalle por PUSHER > Aliado | Matriz Meta, Altas, % Cumpl., Faltante, Crec. julio (`PusherAsignacion`, no `PusherPeriodo`) |
| Aliados con mayor crecimiento desde julio | Barras Top 10 por `[Crecimiento_Desde_Julio]` (sustituye “Asesores cerca de cumplir”) |
| Navegación | Botón “Volver a Home” (mismo enlace que la página existente). “Asesores Septiembre” e “Incentivos y Legalización” aparecen como pestañas inactivas “· próximamente”, sin enlace |

## 5. Gate técnico (refresh real y DAX)

| Control | Julio | Agosto | Septiembre |
|---|---|---|---|
| Altas | 4.519 | 5.715 | 5.628 |
| Meta | 6.337 | 7.155 | 11.080 |
| % Cumplimiento | 71,31 % | 79,87 % | 50,79 % |
| Faltante | 1.818 | 1.440 | 5.452 |
| Crecimiento desde julio | BLANK | +1.196 / +26,47 % | +2.530 / +81,67 % |
| Corte altas | 31/07 | 31/08 | 23/09/2026 |
| Valor legalizado | 3.300.000 | 2.668.800 | 1.603.570 |

Valor legalizado en septiembre por slicer: sin PUSHER 1.603.570; PUSHER 1 595.000; PUSHER 2 1.008.570; PUSHER 3 BLANK (sin legalizado en la fuente); PUSHER 1+2 1.603.570; Sin asignar BLANK; con el aliado de mayor venta 1.603.570 (sin cambio); aliado + PUSHER 2 1.008.570. Coincide con la suma directa de la fact.

Matriz septiembre: 93 filas PUSHER > Aliado suman 5.628 altas y 11.080 de meta, igual al total. Histórico sin filtro de mes: enero-septiembre presentes. Medidas en error 0; relaciones 18; many-to-many 0.

## 6. Gate visual

Ejecutado en Power BI Desktop (instancia propia, cerrada sin guardar):

- Septiembre: KPI, fecha de corte 23/09/2026, gráficos, matriz, Top 10 y pie renderizados sin errores.
- PUSHER 2: Altas 3.187, Meta 4.780, 66,67 %, Faltante 1.593, +101,58 %, Valor legalizado $1.008.570.
- Aliado seleccionado: KPI comerciales cambian (98 / 250 / 39,20 %) y Valor legalizado se mantiene en $1.603.570.
- Evolución mensual mantiene enero-septiembre con el slicer Mes en septiembre.
- Etiquetas y ejes con unidades "Ninguna": valores completos con separador de miles (por ejemplo 5.628, 11.080, +1.606), sin abreviaturas "mil".
- `GestionComercialAltas` renderiza igual que antes (julio 4.519, 71,31 %).

Primer intento fallido documentado: `filterConfig` quedó dentro de `visual` y Desktop rechazó el informe; se movió a la raíz del contenedor.

## 7. Capturas

Locales, no versionadas y sin datos de cuenta: página en septiembre, con PUSHER 2, con un aliado seleccionado y `GestionComercialAltas`.

## 8. Diferencias frente al mockup

- “Brecha” pasa a “Faltante” y “Valor gastado” a “Valor legalizado” por decisión de negocio.
- “Asesores cerca de cumplir” se sustituye por Top 10 de aliados (asesor diferido a R8).
- Pestañas de R8/R9 inactivas; sin iconos decorativos ni lema manuscrito.
- El Mes muestra `2026-09` en lugar de “Septiembre 2026”.
- La matriz se abre colapsada por PUSHER; los aliados se expanden con +.

## 9. Privacidad

Solo etiquetas PUSHER 1/2/3 y Sin asignar; aliados son empresas. Sin nombres personales ni equivalencias en PBIR. Mockups, Excel y capturas no se versionan.

## 10. Limitaciones

- Crecimiento porcentual de PUSHER 3 extremo por su base mínima de julio; la página muestra el valor absoluto.
- La Home no tiene aún una tarjeta hacia esta página (no se modificó Home); se accede por la pestaña.
- Desktop añadirá `lineageTag` y ajustes de serialización la próxima vez que guarde.
- La navegación “Volver a Home” reutiliza la configuración existente; en Desktop requiere Ctrl+clic.

## 11. Rollback

Eliminar `pages/ResumenComercial/`, quitar `ResumenComercial` de `pages.json` y retirar `Valor_Legalizado_Filtro_Pusher` de `_Medidas_Altas.tmdl`. No revertir R2-R6.

## 12. Ajustes posteriores de R7

### Pantalla con errores de campos

Causa: la caché de datos local (`.pbi/cache.abf`, no versionada) se guardó sin refrescar. Al abrir, Desktop avisaba "Algunas de las tablas tienen datos incompletos o no tienen datos" y todos los visuales que usan objetos de R3-R7 (`PusherAsignacion`, medidas R6, `Fact_LegalizacionBonos`) fallaban, mientras los objetos anteriores funcionaban. El PBIR y las medidas eran correctos. Corrección: refresh completo y guardado desde Desktop; la página renderiza sin errores. Si el aviso reaparece tras un cambio de modelo, basta con Actualizar.

### UNO 27 en julio de 2026

`PusherAsignacion` deja de aplicar el override temporal y sigue solo a `Asignacion_PUSHER` (UNO 27 → PUSHER 3 en todos los periodos). `PusherPeriodo` conserva el override, por lo que `GestionComercialAltas` no cambia (julio: 1.582 / 2.429 / 508).

| Métrica (`PusherAsignacion`) | Antes | Después |
|---|---|---|
| Altas julio PUSHER 1 / PUSHER 3 | 1.523 / 2 | 1.299 / 226 |
| Cumplimiento julio PUSHER 1 | 57,82 % | 49,32 % |
| Crecimiento agosto PUSHER 1 / PUSHER 3 | −504 / +633 | −280 / +409 |
| Crecimiento septiembre PUSHER 1 / PUSHER 3 | +377 / +657 | +529 / +505 |
| Altas atribuibles julio (total) | 3.950 | 3.726 |

Totales mensuales, metas y crecimiento total sin cambios. Julio de PUSHER 3 sigue sin ser atribuible.

### Visual de cumplimiento

`Altas vs Meta por PUSHER` se reemplazó por `Cumplimiento de meta por PUSHER (%)`: una serie `[Cumplimiento_Meta_Pct]` por `PusherAsignacion`. Septiembre: PUSHER 1 34,10 %, PUSHER 2 66,67 %, PUSHER 3 90,14 %, Sin asignar 25,62 %. Sin línea de referencia al 100 %.

### Nombres visibles de PUSHER

Correspondencia aprobada y autorizada para el informe publicado: PUSHER 1 = LEONARDO, PUSHER 2 = JEISY, PUSHER 3 = ERIKA.

- `Map_AsignacionPusherFuente` conserva el nombre de `Asignacion_PUSHER` como `PusherNombre` (antes se descartaba).
- `Dim_AsignacionPusherPeriodo[PusherNombre]`: etiqueta de presentación ordenada por `PusherAsignacion` (`sortByColumn`); "Sin asignar" cuando no hay regla. `PusherAsignacion` sigue siendo la clave técnica.
- Los nombres no se escriben en TMDL ni PBIR: salen de la fuente al refrescar.
- `ResumenComercial` usa `PusherNombre` en el slicer PUSHER, Cumplimiento por PUSHER, Crecimiento por PUSHER y la matriz. `GestionComercialAltas` no cambia.
- `Valor_Legalizado_Filtro_Pusher` transfiere el filtro si `PusherAsignacion` o `PusherNombre` están filtrados directamente. Septiembre: sin filtro 1.603.570; LEONARDO 595.000; JEISY 1.008.570; ERIKA BLANK; LEONARDO+JEISY 1.603.570; Sin asignar BLANK; aliado AIB 1.603.570 (sin cambio).

No se añadió línea de referencia al 100 % (decisión de negocio).

## 13. Gate Desktop y gate Power BI Service

**Gate Desktop (local): PASS técnico.** Refresh completo sin errores; 0 medidas en error; 18 relaciones; 0 many-to-many; equivalencia `PusherNombre` ↔ `PusherAsignacion` exacta; UNO 27 → ERIKA en julio-septiembre en `PusherAsignacion` y PUSHER 1 en `PusherPeriodo` (julio `GestionComercialAltas` PUSHER 1 = 1.582). Gate visual PASS con nombres: septiembre sin filtros (LEONARDO 34,10 %, JEISY 66,67 %, ERIKA 90,14 %, Sin asignar 25,62 %; Valor legalizado $1.603.570), JEISY (3.187 / 4.780 / 66,67 %; Valor legalizado $1.008.570) y AIB (98 / 250 / 39,20 %; Valor legalizado sin cambio, $1.603.570). Sin visuales rotos; Mes en septiembre por defecto.

**Gate Power BI Service: NO validado.** No hay acceso al servicio desde este entorno y no se publicó nada. La vista publicada con visuales rotos coincide con el patrón observado localmente: los visuales que dependen de objetos R3-R7 fallan y los anteriores funcionan. Causa probable: se publicó un modelo cuyos datos (caché importada) no estaban refrescados para esos objetos, o un modelo publicado anterior a esos cambios. El servicio no puede refrescar por sí mismo porque las fuentes son archivos locales sin gateway.

Acción manual para el enlace público: abrir el PBIP, Actualizar (refresh completo), confirmar que no aparece el aviso de datos incompletos y que `ResumenComercial` renderiza, publicar reemplazando el modelo semántico existente y revisar el informe en el servicio; el enlace de "Publicar en web" se actualiza con el mismo código, con un retraso de caché que puede ser de hasta una hora.

## 14. Riesgos

- Los nombres reales quedan visibles en el informe público (autorizado).
- Si la fuente cambia el nombre de un PUSHER o de un ancla, la etiqueta cambia o el refresh falla por las validaciones existentes.
- Cualquier cambio de modelo requiere refresh antes de guardar o publicar; si no, la caché deja visuales rotos.

## 15. Rollback (actualizado)

Eliminar `pages/ResumenComercial/` y su entrada en `pages.json`; retirar `Valor_Legalizado_Filtro_Pusher`; revertir `PusherNombre` en `Map_AsignacionPusherFuente` y `Dim_AsignacionPusherPeriodo`, y restaurar el override en `PusherAsignacion`, todo contra `d926486`. No revertir R2-R6.
