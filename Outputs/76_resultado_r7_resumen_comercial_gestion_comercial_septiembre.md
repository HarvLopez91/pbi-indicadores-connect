# Resultado R7 — Resumen Comercial

| Campo | Valor |
|---|---|
| Fase | R7 — Página 1 / Resumen Comercial |
| Baseline | `d926486a55d0ebf6de90feedcf63d636991fa892` (hotfix R5/R6) |
| Commit R7 | `8e98dffcbf608eef1ac8f3f4100c051f2cf27ee3` |
| Página | `ResumenComercial` (“Resumen Comercial”), 1280 × 720, FitToPage |
| Implementación local / Git | **PASS** |
| Power BI Desktop | **PASS** (técnico y visual) |
| Power BI Service — workspace | **PASS** |
| Publicar en web | **PASS** |
| Estado final | **R7 — CERRADO / PASS** |
| R8 | No iniciado |

## 1. Alcance

Página nueva basada en `mockup-pagina-1.jpeg`. Los números del mockup son ilustrativos y no se copiaron. `GestionComercialAltas` no se modificó.

Cambios:
- `pages/ResumenComercial/` (page.json y 32 visuales).
- `pages/pages.json`: la página se inserta después de `GestionComercialAltas`; la página activa no cambia.
- `expressions.tmdl`: `Map_AsignacionPusherFuente` conserva el nombre de `Asignacion_PUSHER` como `PusherNombre`.
- `Dim_AsignacionPusherPeriodo.tmdl`: columna `PusherNombre`; `PusherAsignacion` sin el override temporal.
- `_Medidas_Altas.tmdl`: medida auxiliar `Valor_Legalizado_Filtro_Pusher`.

Sin relaciones, dimensiones, facts ni many-to-many nuevos.

## 2. PUSHER: clasificación técnica y etiqueta visible

| Clasificación técnica (`PusherAsignacion`) | Etiqueta visible (`PusherNombre`) |
|---|---|
| PUSHER 1 | LEONARDO |
| PUSHER 2 | JEISY |
| PUSHER 3 | ERIKA |
| Sin asignar | Sin asignar |

- `PusherNombre` se deriva de `Asignacion_PUSHER` en cada refresh y se ordena por `PusherAsignacion` (`sortByColumn`).
- `PusherAsignacion` sigue siendo la clave técnica de medidas y cálculos.
- `ResumenComercial` muestra `PusherNombre` en el slicer PUSHER, en los gráficos de cumplimiento y crecimiento y en la matriz.

**UNO 27:**
- `PusherAsignacion`: UNO 27 pertenece a ERIKA / PUSHER 3 en todos los periodos, incluido julio de 2026, alineado con `Asignacion_PUSHER`.
- `PusherPeriodo`: conserva el override histórico de julio en PUSHER 1 solo para no alterar `GestionComercialAltas` (julio: 1.582 / 2.429 / 508).

## 3. KPI

| KPI | Medida |
|---|---|
| Altas | `[Altas_Total]` |
| Meta | `[Meta_Asignada]` |
| % Cumplimiento | `[Cumplimiento_Meta_Pct]` |
| Faltante | `[Faltante_Meta]` |
| Crecimiento desde julio | `[Crecimiento_Desde_Julio_Pct]` |
| Valor legalizado | `[Valor_Legalizado_Filtro_Pusher]` (sobre `[Valor_Legalizado]`) |

No se muestran valor recibido, pendiente, valor gastado, ejecución ni ROI (recibido y pendiente quedan para R9).

Fecha de corte principal: `[Fecha_Corte_Altas]`. `[Fecha_Corte_Incentivos]` aparece solo en el pie, rotulada como fecha de información de incentivos.

## 4. Valor legalizado y filtros

`Fact_LegalizacionBonos` tiene `Pusher` como atributo propio, sin aliado y sin relación con `Dim_AsignacionPusherPeriodo`.

`Valor_Legalizado_Filtro_Pusher` aplica `TREATAS(VALUES(PusherAsignacion), Fact_LegalizacionBonos[Pusher])` cuando `PusherAsignacion` o `PusherNombre` están filtrados directamente; en otro caso devuelve `[Valor_Legalizado]`. La transferencia se hace siempre contra la clasificación técnica. Como `Dim_Aliado` no filtra a `Dim_AsignacionPusherPeriodo`, un filtro de Aliado por sí solo no afecta al KPI. No se simula ninguna relación Aliado → legalización; la tarjeta lo indica con la nota “Responde a Mes y PUSHER; no a Aliado”.

| Septiembre | Valor legalizado |
|---|---|
| Sin filtro PUSHER | 1.603.570 |
| LEONARDO | 595.000 |
| JEISY | 1.008.570 |
| ERIKA | BLANK (sin legalizado en la fuente) |
| LEONARDO + JEISY | 1.603.570 |
| Sin asignar | BLANK |
| AIB seleccionado, sin PUSHER | 1.603.570 |

## 5. Visuales e interacciones

| Visual | Contenido |
|---|---|
| Slicers | Mes (`Dim_Calendario[AnioMes]`, selección única, septiembre 2026 por defecto con filtro nativo del slicer), PUSHER (`PusherNombre`), Aliado (`Dim_Aliado[Descripcion]`) |
| Cumplimiento de meta por PUSHER (%) | Una serie `[Cumplimiento_Meta_Pct]` por `PusherNombre`; sin línea de referencia al 100 % |
| Evolución mensual de altas | Líneas Altas y Meta por mes; el slicer Mes no filtra este visual (`NoFilter`), PUSHER y Aliado sí |
| Crecimiento por PUSHER desde julio | `[Crecimiento_Desde_Julio_Atribuible]` por `PusherNombre`, con `%` en tooltip |
| Detalle por PUSHER > Aliado | Matriz `PusherNombre` > Aliado con Meta, Altas, % Cumpl., Faltante, Crec. julio |
| Aliados con mayor crecimiento desde julio | Barras Top 10 por `[Crecimiento_Desde_Julio]` (sustituye “Asesores cerca de cumplir”) |
| Navegación | Botón “Volver a Home”; “Asesores Septiembre” e “Incentivos y Legalización” como pestañas inactivas “· próximamente”, sin enlace |

Los visuales por PUSHER muestran `PusherNombre`; los cálculos siguen basados en `PusherAsignacion`. Etiquetas y ejes con unidades “Ninguna”: valores completos con separador de miles, sin abreviaturas “mil”.

Cumplimiento septiembre: LEONARDO 34,10 %, JEISY 66,67 %, ERIKA 90,14 %, Sin asignar 25,62 %.

## 6. Gate técnico (refresh real y DAX)

| Control | Julio | Agosto | Septiembre |
|---|---|---|---|
| Altas | 4.519 | 5.715 | 5.628 |
| Meta | 6.337 | 7.155 | 11.080 |
| % Cumplimiento | 71,31 % | 79,87 % | 50,79 % |
| Faltante | 1.818 | 1.440 | 5.452 |
| Crecimiento desde julio | BLANK | +1.196 / +26,47 % | +2.530 / +81,67 % |
| Corte altas | 31/07 | 31/08 | 23/09/2026 |
| Valor legalizado | 3.300.000 | 2.668.800 | 1.603.570 |

Crecimiento atribuible septiembre: LEONARDO +529, JEISY +1.606, ERIKA +505. Matriz septiembre: suma 5.628 altas y 11.080 de meta, igual al total. Histórico enero-septiembre presente.

## 7. Gate Desktop — PASS técnico y visual

- Refresh completo PASS; 0 medidas con error; 18 relaciones; 0 many-to-many.
- Página sin visuales rotos; Mes septiembre por defecto; nombres visibles correctos.
- Vistas validadas: septiembre sin filtros; JEISY (3.187 / 4.780 / 66,67 %; valor legalizado 1.008.570); AIB (98 / 250 / 39,20 %; valor legalizado sin cambio, 1.603.570).
- `GestionComercialAltas` intacta.

## 8. Gate Power BI Service y Publicar en web — PASS

**Workspace de Power BI Service: PASS.** Tras la publicación manual desde Desktop, el usuario abrió el informe dentro del workspace y confirmó que funcionan el slicer PUSHER, Cumplimiento por PUSHER, Crecimiento por PUSHER y la matriz PUSHER > Aliado, que LEONARDO / JEISY / ERIKA cargan correctamente y que el resto de visuales también carga.

**Publicar en web: PASS.** Justo después de publicar, el enlace público mostraba errores en los visuales por PUSHER. Pasado el tiempo de actualización del enlace, el usuario confirmó que el informe carga correctamente.

- El fallo del enlace público fue transitorio.
- El modelo publicado en el workspace estaba correcto.
- No se requirió ningún cambio técnico adicional.
- El comportamiento es compatible con la actualización diferida o caché de “Publicar en web”; la evidencia no permite afirmar una causa interna más específica.

Los cambios automáticos que Desktop dejó tras publicar (fin de línea, `lineageTag`, anotaciones, cultura Q&A, `PBI_QueryOrder`) se diagnosticaron como no necesarios para el funcionamiento y se descartaron; no se versionaron.

## 9. Privacidad

- LEONARDO, JEISY y ERIKA están expresamente autorizados para mostrarse en el informe público.
- Los nombres visibles se derivan de `Asignacion_PUSHER` durante el refresh; no se escriben en PBIR ni TMDL.
- No se exponen apellidos, correos, asesores, especialistas ni otra información personal. Los aliados son empresas.
- Excel, mockups y capturas permanecen fuera de Git.

## 10. Diferencias frente al mockup

- “Brecha” pasa a “Faltante” y “Valor gastado” a “Valor legalizado” por decisión de negocio.
- “Altas vs Meta por PUSHER” se sustituye por “Cumplimiento de meta por PUSHER (%)”.
- “Asesores cerca de cumplir” se sustituye por Top 10 de aliados (asesor diferido a R8).
- Pestañas de R8/R9 inactivas; sin iconos decorativos ni lema manuscrito.
- El Mes muestra `2026-09` en lugar de “Septiembre 2026”.
- La matriz se abre colapsada por PUSHER; los aliados se expanden con +.

## 11. Historial de ajustes durante R7

- **Caché local:** una sesión de Desktop guardó la caché de datos (`.pbi/cache.abf`, no versionada) sin refrescar y los visuales con objetos R3-R7 fallaban localmente. Se resolvió con refresh completo; tras cambios de modelo hay que Actualizar antes de guardar o publicar.
- **UNO 27:** se retiró el override de `PusherAsignacion` (ver §2). Impacto en `PusherAsignacion`: julio PUSHER 1 / PUSHER 3 1.523 / 2 → 1.299 / 226; cumplimiento julio PUSHER 1 57,82 % → 49,32 %; crecimiento agosto −504 / +633 → −280 / +409 y septiembre +377 / +657 → +529 / +505; altas atribuibles julio 3.950 → 3.726. Totales sin cambios.
- **Gráfico:** “Altas vs Meta por PUSHER” se reemplazó por “Cumplimiento de meta por PUSHER (%)”, sin línea de referencia al 100 % por decisión de negocio.
- **`PusherNombre`:** se incorporó como etiqueta de presentación derivada de la fuente.
- **PBIR:** un primer intento dejó `filterConfig` dentro de `visual`; se movió a la raíz del contenedor.

## 12. Limitaciones y riesgos

- Crecimiento porcentual de ERIKA extremo por su base mínima de julio; la página muestra el valor absoluto.
- Si la fuente cambia el nombre de un PUSHER o de un ancla, la etiqueta cambia o el refresh falla por las validaciones existentes.
- La Home no tiene aún una tarjeta hacia esta página; se accede por la pestaña. “Volver a Home” requiere Ctrl+clic en Desktop.
- Desktop añadirá `lineageTag` y ajustes de serialización la próxima vez que guarde.

## 13. Rollback

Contra `d926486`: eliminar `pages/ResumenComercial/` y su entrada en `pages.json`; retirar `Valor_Legalizado_Filtro_Pusher`; revertir `PusherNombre` en `Map_AsignacionPusherFuente` y `Dim_AsignacionPusherPeriodo`, y restaurar el override en `PusherAsignacion`. No revertir R2-R6.
