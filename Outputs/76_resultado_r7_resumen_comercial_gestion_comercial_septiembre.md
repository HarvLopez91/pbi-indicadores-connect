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
- `Dim_AsignacionPusherPeriodo.tmdl`: columna `PusherNombre` (la prioridad de clasificación se actualizó después; ver §14).
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

**Prioridad de clasificación (vigente desde la corrección posterior, §14):** override temporal (`Map_AsignacionPusherPeriodo`, por `AnioMes + AliadoKey`) > `Asignacion_PUSHER` > `Sin asignar`. Se aplica por igual a `PusherPeriodo`, `PusherAsignacion`, `PusherNombre`, `TipoReglaAsignacion` y `EsGestionAtribuible`.

**UNO 27:** julio de 2026 → PUSHER 1 (override temporal); desde agosto → PUSHER 3 (`Asignacion_PUSHER`). La versión inicial de R7 aplicaba el override solo a `PusherPeriodo` (ver §11); quedó sustituida en §14.

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

Solo la corrección de §14: restaurar en `Dim_AsignacionPusherPeriodo` las versiones anteriores de `PusherAsignacion`, `TipoReglaAsignacion` y `PusherNombre` y retirar el catálogo `CatalogoPusher`. Las correcciones de la fuente privada (§14) se revierten desde su respaldo local, fuera de Git.

## 14. Corrección posterior — homologación de metas y prioridad del override

**Fuente privada (fuera de Git):**
- `Metas_Bonos`: `Aliado_BI` `ABAI` → `UNO 27` en julio, agosto y septiembre de 2026 (15 filas CALL/ESPECIALISTA/ASESOR; ningún otro campo cambia). La meta CALL de UNO 27 quedaba en `Sin asignar` porque su clave no coincidía con `Asignacion_PUSHER`.
- `Asignacion_PUSHER`: regla general `LEONARDO | MILLENIUM` (activo hasta agosto; sin filas en septiembre).

**Modelo:** `Dim_AsignacionPusherPeriodo` aplica override temporal > `Asignacion_PUSHER` > `Sin asignar` también a `PusherAsignacion`, `PusherNombre` (resuelto con un catálogo código → nombre derivado de `Asignacion_PUSHER`, que falla si un código tiene cero o varios nombres) y `TipoReglaAsignacion` (“Override temporal”). `EsGestionAtribuible` se calcula sobre la clasificación efectiva. Sin nombres en TMDL. `GestionComercialAltas` usa solo `PusherPeriodo`, cuya lógica no cambia.

**Referencia de cumplimiento:** la meta de un PUSHER es la suma de la meta CALL de sus aliados. ESPECIALISTA es control y ASESOR es incentivo individual; no se suman.

| PUSHER | Julio (meta / altas / %) | Agosto | Septiembre |
|---|---|---|---|
| LEONARDO | 2.959 / 1.582 / 53,46 % | 2.740 / 1.115 / 40,69 % | 4.200 / 1.641 / 39,07 % |
| JEISY | 3.378 / 2.427 / 71,85 % | 3.475 / 3.641 / 104,78 % | 4.780 / 3.705 / 77,51 % |
| ERIKA | — / 2 | 940 / 635 / 67,55 % | 2.100 / 766 / 36,48 % |
| Sin asignar | meta 0 / 508 altas | meta 0 / 324 | meta 0 / 416 |

- Julio LEONARDO incluye UNO 27 (225 / 224) y MILLENIUM (100 / 59). ERIKA septiembre: UNO 27 1.370 / 763, ATENTO TRASLADOS PEREIRA 550 / 3, INTERACTIVO MANIZALES 90 / 0, EMERGIA 90 / 0. El 104,93 % anterior (766 / 730) desaparece.
- `Sin asignar` ya no tiene meta y no aparece en el gráfico de cumplimiento; conserva altas de CAV, tiendas y otros canales sin meta.
- Totales sin cambios: altas 45.538 (julio 4.519, agosto 5.715, septiembre 6.528); meta CALL 6.337 / 7.155 / 11.080; crecimiento global septiembre +2.868 / +78,36 %; valor legalizado 3.300.000 / 2.668.800 / 1.603.570.
- Crecimiento atribuible desde julio: agosto LEONARDO −467, JEISY +1.214, ERIKA +633; septiembre LEONARDO +349, JEISY +1.790, ERIKA +764. En la matriz, UNO 27 aparece bajo LEONARDO en agosto y septiembre solo con crecimiento negativo (−224 / −183): es su base de julio, que cambió de PUSHER. Top 10 de aliados sin cambios (ATENTO +1.007, UNO 27 +580, INTELIGENCE +469, GNP +463…).
- Valor legalizado por PUSHER en julio: PUSHER 1 1.500.000, PUSHER 2 1.500.000, PUSHER 3 300.000 (total 3.300.000).
- Gate: refresh completo PASS, 0 medidas con error, 18 relaciones, 0 many-to-many; código y nombre de PUSHER coherentes en todas las filas; `GestionComercialAltas` sin cambios (julio PUSHER 1 / 2 / Sin asignar: 1.582 / 2.429 / 508).
