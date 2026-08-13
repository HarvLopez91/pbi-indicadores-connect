# Resultado GC-8 — Validación funcional y visual de Gestión comercial de altas

## Objetivo

Probar el comportamiento integral de la página `GestionComercialAltas` (refresh, filtros cruzados, estados de periodo, medidas, drivers, navegación, narrativa y privacidad), ejecutar regresión de las siete páginas previas y obtener el gate visual manual del usuario, sin implementar funcionalidades nuevas.

## Validaciones automáticas

- PBIR/JSON válido en las 261 definiciones del reporte (8 páginas). PASS.
- 0 referencias rotas a campos/medidas entre las 8 páginas (60 referencias únicas verificadas contra el modelo real). PASS.
- 0 referencias `[Medida]`/`Tabla[Columna]` rotas en las fórmulas DAX de `_Medidas_Altas.tmdl`. PASS.
- 252 IDs de visual, 252 únicos. PASS.
- Home permanece como `activePageName`. PASS.
- `MesNombre.sortByColumn = MesNumero` persiste. PASS.
- `gc_participacion_periodo` confirmado ausente del modelo y del reporte. PASS.
- Focus mode confirmado habilitado en gráfico histórico, drivers positivos, drivers negativos y ranking. PASS.
- Columnas sensibles: `Fact_AltasTeResuelve` (`FechaAlta`, `AliadoKey`, `Altas`) y `Dim_Aliado` (`AliadoKey`, `Descripcion`, `Pusher`, `Estado_Clasificacion`) sin campos personales. Búsqueda de `JEFE`/`ESPECIALISTA`/`ASESOR` y de rutas locales en las 8 páginas: 0 coincidencias. PASS.

## Cero regresiones a las siete páginas previas

Diff real de Git entre el cierre de GC-6 (`1584401`) y el cierre de GC-7 (`02981b3`): 7.520 líneas insertadas, 0 eliminadas, 0 modificadas — el único cambio compartido fue 1 línea en `pages.json` (registro de la página nueva) y 1 línea en `Dim_Calendario.tmdl` (`sortByColumn` autorizado, usado exclusivamente por la página nueva). Ninguna de las 7 páginas anteriores sufrió modificación alguna durante GC-7 ni durante GC-8.

## Navegación — hallazgo y corrección

GC-8 detectó que `home_nav_07_accent` y `home_nav_07_card` (2 de los 4 objetos que componen la tarjeta "Gestión comercial de altas" en Home) navegaban incorrectamente a `p14_notas_metodologicas`, mientras `home_nav_07_hitzone` y `home_nav_07_label` ya apuntaban correctamente a `GestionComercialAltas`. El hitzone, con el z-index más alto, interceptaba el clic en la práctica, por lo que el defecto no era observable en uso normal, pero sí representaba una inconsistencia real en el PBIR.

**Corrección aplicada** (commit `846c58f`): `navigationSection` y `tooltip` de ambos objetos actualizados a `GestionComercialAltas` / *"Ir a Gestión comercial de altas"*. Verificado tras la corrección: los 3 objetos de navegación de la tarjeta apuntan al destino correcto, navegación inversa Gestión comercial → Home intacta, 0 referencias rotas, `git diff --check` PASS, diff acotado exactamente a los archivos autorizados.

## Gate manual — evidencia del usuario

### Julio 2026, contexto general
Altas = 4.518, Cambio = +818, Variación = +22,11 %, Promedio histórico previo = 4.796, Meta +30 % = 6.235, Altas en aliados con PUSHER = 83,80 %.

### Filtro PUSHER = PUSHER 1
Altas = 1.357, Cambio = +164, Variación = +13,75 %, Promedio = 1.811, Meta = 2.354.

### Filtro Aliado = ATENTO
Altas = 1.395, Cambio = +381, Variación = +37,57 %, Cobertura = 100,00 %.

### Drivers julio
ATENTO +381, ONE CONTACT +201, GNP -70 — ambos nombres visibles en "Mayores contribuciones positivas" tras el ajuste de densidad del eje de categoría.

### Agosto parcial (periodo en curso)
Altas = 559, Cambio = BLANK (`--`), Variación = BLANK (`--`), Promedio histórico previo = 4.756, Meta +30 % = 6.183, Cobertura = 85,15 %. Los comparativos mensuales no se presentan como cierre para un periodo en curso, conforme a la regla de `Es_Periodo_Comparable`.

### Otros controles manuales confirmados por el usuario
Navegación Home ↔ Gestión comercial funcional en ambos sentidos; filtros Año/Mes/PUSHER/Aliado responden correctamente; ranking de aliados correcto; Focus mode funcional en gráfico histórico y ranking; etiquetas por segmento y totales mensuales visibles; eje en números completos; sin datos personales observados; sin visuales rotos.

## Limitación conocida — no bloqueante

En **Focus mode** del gráfico "Evolución mensual y distribución por PUSHER" puede mostrarse un título vertical truncado generado por Power BI Desktop. La configuración PBIR ya contiene `categoryAxis.showAxisTitle = false` y `valueAxis.showAxisTitle = false`; en la vista normal del reporte el título no aparece y no afecta la lectura. Se aplicó un refuerzo manual aprobado por el usuario sobre `valueAxis` (commit `846c58f`) sin que exista una corrección PBIR adicional verificable; no se seguirá iterando sobre esta propiedad por no haber beneficio funcional demostrable frente al riesgo de nuevas rondas de prueba y error.

## Privacidad y exportación

`exportDataMode: AllowSummarized` a nivel de reporte (preexistente, no modificado por GC-7/GC-8) solo permite exportar la vista agregada visible; dado que el modelo de Altas no contiene campos personales, ningún export posible desde esta página puede exponer datos individuales. Sin páginas de tooltip personalizadas en la página nueva. `gc_matriz_aliados` expone exactamente las 6 columnas del diseño aprobado (Aliado, PUSHER, Altas, Mes anterior, Diferencia, Variación %).

## Ausencia de cambios fuera de alcance

Sin cambios en `Data/`, Excel, Power Query (`expressions.tmdl` sin alteración de lógica, verificado línea por línea), relaciones, medidas preexistentes ni otras páginas del reporte. Los archivos de metadatos automáticos generados por Power BI Desktop durante las sesiones de prueba (`lineageTag` faltantes, reordenamiento de bloques, sinónimos Q&A, estado de selección de segmentadores) fueron identificados, clasificados y descartados del commit por no aportar valor y no formar parte del alcance autorizado.

## Estado final

- GC-8: **cerrado**, aprobado por el usuario con la limitación conocida documentada.
- GC-9: **no iniciado**.
