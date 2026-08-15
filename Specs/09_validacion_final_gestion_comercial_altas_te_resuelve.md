# Validación final — Gestión comercial de altas Te Resuelve

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Iniciativa | Gestión comercial de altas Te Resuelve (GC-1 a GC-10) |
| Fase de este documento | GC-10 — cierre y publicación |
| Estado | Validación estática cerrada; publicación manual ejecutada y validada por el usuario |

## 1. Objetivo

Cerrar formalmente la iniciativa GC-1–GC-10 dejando registro consolidado de: fuente y reglas de negocio, arquitectura del modelo, cifras de control del corte validado, resultado de las pruebas funcionales/visuales (GC-8), la documentación entregada (GC-9) y el estado final de Git antes y después de la publicación manual en Power BI Service.

## 2. Fases GC-1 a GC-10

| Fase | Objetivo | Estado |
|---|---|---|
| GC-1 a GC-5 | Ingesta, limpieza, mapeo PUSHER, modelo semántico, medidas DAX iniciales | Cerradas |
| GC-6 | Conciliación técnica fuente ↔ modelo | Cerrada — commit `1584401` |
| GC-7 | Construcción de la página `GestionComercialAltas` | Cerrada — técnico `6d4097a`, cierre `02981b3` |
| GC-8 | Validación funcional y visual integral | Cerrada — técnico `846c58f`, cierre `8372214` |
| GC-9 | Documentación (`Docs/GESTION_COMERCIAL_ALTAS_TE_RESUELVE.md`) | Cerrada — doc `21adf7b`, cierre `0d93173` |
| GC-10 | Cierre final y publicación | En curso — este documento |

## 3. SHAs finales previos a GC-10

- `1584401` — cierre técnico GC-6 (conciliación).
- `6d4097a` — construcción de página (GC-7, técnico).
- `02981b3` — cierre GC-7.
- `846c58f` — corrección de navegación Home + ajuste de eje Y (GC-8, técnico).
- `8372214` — cierre GC-8.
- `21adf7b` — guía funcional/técnica (GC-9).
- `0d93173` — cierre GC-9. **Baseline confirmado al iniciar GC-10.**

## 4. Fuente

Tabla formal `Insumo2` del archivo vigente de altas, validada en cada refresh (archivo único, tabla única, columnas obligatorias `ALTAS`/`DESCRIPCION`/`FECHA_ALTA`/`MES`, y existencia del periodo de corte configurado). Regla de cálculo: `Altas = SUM(Fact_AltasTeResuelve[Altas])`. Grano agregado — una fila puede representar varias altas; no existe identificador de venta ni cliente individual.

## 5. Arquitectura

`Fact_AltasTeResuelve` (`FechaAlta`, `AliadoKey`, `Altas`) relacionado 1:* con `Dim_Calendario` y con `Dim_Aliado` (`AliadoKey`, `Descripcion`, `Pusher`, `Estado_Clasificacion`), ambas relaciones unidireccionales. Medidas en `_Medidas_Altas` (36 medidas). Página `GestionComercialAltas`, navegación bidireccional con Home, sin alterar las 7 páginas previas del reporte.

## 6. Reglas PUSHER

Clasificación por **coincidencia exacta** (tras `Trim`/`Clean`/mayúsculas) contra una lista fija (`Map_PusherAliado`) embebida en Power Query — sin coincidencia aproximada. Un aliado no listado queda como `Sin asignar`. Cobertura de clasificación dinámica (julio: 83,80 %; agosto parcial: 85,15 %), no es un indicador de desempeño.

**Inicio de gestión PUSHER 2: 01/07/2026** (parámetro `Fecha_Inicio_Gestion_Pusher_2`). Antes de esa fecha, las altas de los aliados hoy asignados a PUSHER 2 se presentan como "Línea base histórica del portafolio P2"; desde esa fecha, como "PUSHER 2 · desde inicio de gestión". No se afirma causalidad — solo contribución y asociación temporal observadas.

## 7. Agosto parcial

`Periodo_Corte_Comercial = 202607`. Agosto 2026 es un periodo "En curso" (`Es_Periodo_Comparable = FALSE`): sus altas se muestran, pero las medidas de cambio/variación mensual quedan en blanco automáticamente para no compararlo como si fuera un cierre.

## 8. Privacidad

`JEFE`, `ESPECIALISTA` y `ASESOR` se eliminan explícitamente en `AltasTeResuelve_Limpio`, antes de llegar al modelo. El modelo comercial expuesto se limita a `FechaAlta`/`AliadoKey`/`Altas` (hecho) y `AliadoKey`/`Descripcion`/`Pusher`/`Estado_Clasificacion` (dimensión de aliado). Verificado en GC-10: cero coincidencias de estos términos dentro de `GestionComercialAltas`; las únicas coincidencias del repositorio están en páginas ajenas y preexistentes de otro dominio (Calidad de llamadas, Motivación comercial), fuera del alcance de esta iniciativa. Sin rutas locales expuestas. `Data/` permanece sin versionar.

## 9. Cifras de control del corte validado

| Indicador | Julio | Agosto (parcial) |
|---|---:|---:|
| Altas | 4.518 | 559 |
| Cambio vs. mes anterior | +818 | *(en blanco)* |
| Variación vs. mes anterior | +22,11 % | *(en blanco)* |
| Promedio histórico previo | 4.796 | 4.756 |
| Meta +30 % | 6.235 | 6.183 |
| Cobertura PUSHER | 83,80 % | 85,15 % |

Junio (referencia): 3.700. Drivers julio: ATENTO +381, ONE CONTACT +201, GNP -70. Estas cifras corresponden al corte vigente; no se recalcularon ni ajustaron durante GC-10.

## 10. Resultado GC-8 (validación funcional y visual)

Aprobado por el usuario en Power BI Desktop: navegación Home ↔ Gestión comercial (con corrección de `home_nav_07_accent`/`home_nav_07_card`), filtros Año/Mes/PUSHER/Aliado, exclusión intencional del segmentador Mes sobre el gráfico histórico, drivers y ranking, Focus mode en las 4 visuales principales, etiquetas por segmento y totales del gráfico apilado, eje Y en números completos, y ausencia de datos personales. Detalle completo en `Outputs/58`.

## 11. Resultado GC-9 (documentación)

`Docs/GESTION_COMERCIAL_ALTAS_TE_RESUELVE.md` publicado y validado línea por línea contra el modelo real (medidas, parámetros, columnas y textos visibles). Corrección menor de cierre aplicada en GC-10: `Outputs/59` corregido de "3 filtros" a "4 filtros" (Año, Mes, PUSHER, Aliado) — la guía principal ya era correcta.

## 12. Validación estática GC-10 (sin modificar el modelo)

Revalidado sin repetir las pruebas extensas de GC-8: 261 archivos PBIR/JSON válidos, 0 referencias rotas, Home activa, navegación de los 3 objetos de la tarjeta 07 hacia `GestionComercialAltas`, navegación inversa intacta, interacción `NoFilter` (Mes → gráfico histórico) intacta, Focus mode confirmado en las 4 visuales, `labels`/`totals` del gráfico conservados (6 y 2 entradas respectivamente), `MesNombre.sortByColumn = MesNumero` persiste, `git diff --check` PASS.

## 13. Limitación conocida

En Focus mode del gráfico "Evolución mensual y distribución por PUSHER", Power BI puede mostrar un título vertical truncado en el eje aunque `showAxisTitle` esté desactivado en ambos ejes. No afecta la vista normal del reporte ni ninguna cifra. Aceptada como limitación visual no bloqueante desde GC-8; no se reabre en GC-10.

## 14. Estado de Git al iniciar GC-10

Rama `main`, `HEAD` local = `origin/main` = `0d93173` (baseline confirmado), working tree limpio, staging vacío, un único worktree (el checkout principal, sin worktrees secundarios).

## 15. Publicación

**Ejecutada y validada por el usuario en Power BI Service.** Evidencia confirmada mediante captura del artefacto publicado:

**Julio 2026** (Año=2026, Mes=julio, PUSHER=Todas, Aliado=Todas): Altas = 4.518, Cambio = +818, Variación = +22,11 %, Promedio histórico previo = 4.796, Meta +30 % = 6.235, Cobertura = 83,80 %. Gráfico histórico con etiquetas por segmento, totales mensuales y colores correctos; drivers ATENTO +381 / ONE CONTACT +201; caída GNP -70; ranking visible; navegación disponible. PASS.

**Agosto 2026 parcial** (Mes=agosto): Altas = 559, Cambio y Variación en blanco (`--`), Promedio histórico previo = 4.756, Meta +30 % = 6.183, Cobertura = 85,15 %. Drivers comparativos vacíos, ranking muestra altas del periodo sin comparación mensual, histórico conserva enero-julio sin presentar agosto como cierre. PASS.

### Modalidad de publicación — decisión intencional

El artefacto se distribuye mediante **"Publicar en la Web"** (enlace público, sin autenticación). Esta es una **decisión funcional consciente**, no un descuido: los usuarios finales no cuentan con licencia de Power BI y el reporte debe poder consultarse mediante enlace público por un consumidor externo a Connect. No se sustituye por una modalidad autenticada.

Esta decisión hace que la restricción de privacidad del modelo comercial sea **permanente, no solo de esta fase**: cualquier actualización futura del modelo de Gestión comercial de altas debe seguir excluyendo `JEFE`, `ESPECIALISTA`, `ASESOR`, nombres individuales y cualquier dato no autorizado para exposición pública, porque el enlace público no tiene control de acceso. El URL completo no se reproduce en esta documentación por no ser necesario para el mantenimiento técnico.

### Privacidad del artefacto publicado

Confirmado sobre las capturas del reporte publicado: sin nombres individuales, sin cargos sensibles, datos agregados por aliado/PUSHER únicamente — consistente con lo validado en el modelo local (§8).

## 16. Resultado final

GC-1 a GC-9 cerrados y verificados sin regresiones. GC-10 con validación estática completa, gate manual de Power BI Desktop aprobado y publicación en Power BI Service ejecutada y validada por el usuario (julio y agosto parcial, ambos PASS). **Implementación de Gestión comercial de altas Te Resuelve cerrada.**
