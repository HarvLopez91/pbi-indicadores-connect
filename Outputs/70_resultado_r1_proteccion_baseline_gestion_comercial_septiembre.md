# Resultado R1 — Protección y baseline de gestión comercial septiembre

| Campo | Valor |
|---|---|
| Fase | R1 — Protección y baseline |
| Estado | PASS; pendiente de aprobación y versionamiento |
| SHA baseline | `5cc593bcd525ece852b11d0533fefdf6225e9e30` |
| Rama | `main` |
| Implementación Power BI | Sin cambios |
| Fase siguiente | R2 no iniciada |

## 1. Objetivo

Congelar el estado publicado anterior a la migración del consolidado y proteger del versionamiento accidental los artefactos locales de Graphify y los mockups de septiembre.

Este baseline aplica el contrato aprobado en:

- [Specs/12 — Análisis de impacto](../Specs/12_analisis_impacto_mockups_gestion_comercial_septiembre.md)
- [Specs/13 — Plan de implementación](../Specs/13_plan_implementacion_mockups_gestion_comercial_septiembre.md)

## 2. Baseline Git

| Control | Resultado |
|---|---|
| Rama | `main` |
| HEAD local inicial | `5cc593bcd525ece852b11d0533fefdf6225e9e30` |
| `origin/main` | `5cc593bcd525ece852b11d0533fefdf6225e9e30` |
| Commit R0 | `docs: planifica gestion comercial septiembre` |
| Staging inicial | Vacío |
| Cambios rastreados iniciales | Cero |
| No rastreados iniciales | Mockups de 2026 y `graphify-out/` |
| Power BI Desktop y Excel | Cerrados durante R1 |

No se eliminaron archivos locales ni se incorporaron artefactos privados al índice.

## 3. Protecciones agregadas

Se conservaron todas las reglas existentes de `.gitignore` y se añadieron exclusivamente:

```gitignore
graphify-out/
Assets/mockups/2026/09_Septiembre/
```

Validaciones realizadas:

| Recurso | Regla | Resultado |
|---|---|---|
| Excel bajo `Data/` | `Data/**/*.xlsx` | Ignorado |
| Artefactos Graphify | `graphify-out/` | Ignorados |
| Mockups de septiembre | `Assets/mockups/2026/09_Septiembre/` | Ignorados |

Ningún Excel, JPEG ni artefacto Graphify forma parte de los cambios rastreados de R1.

## 4. Baseline funcional publicado

Estas cifras corresponden al modelo publicado anterior a R2. No fueron recalculadas con el consolidado nuevo.

| Control | Valor publicado |
|---|---:|
| Histórico acumulado | 33.854 |
| Junio 2026 | 3.700 |
| Julio 2026 | 4.518 |
| Agosto 2026 parcial | 559 |
| Meta julio | 6.317 |
| Cumplimiento julio | 71,52 % |
| PUSHER 1 julio | 1.581 |
| PUSHER 2 julio | 2.429 |
| Sin asignar julio | 508 |

Drivers de control para julio:

| Aliado | Cambio mensual |
|---|---:|
| ATENTO | +381 |
| ONE CONTACT | +201 |
| GNP | -70 |

Los valores se confirmaron contra la documentación versionada de la implementación publicada y se usarán como referencia antes/después durante R2.

## 5. Baseline técnico

### Fuente y temporalidad

- Fuente implementada: `INFORME ALTAS TE RESUELVE Cierre Julio.xlsx`.
- Objeto seleccionado: tabla formal `Insumo2`.
- Flujo vigente: `Base_AltasTeResuelve → AltasTeResuelve_Limpio → Fact_AltasTeResuelve`.
- `Periodo_Corte_Comercial = 202607`.
- La limpieza vigente retira `JEFE`, `ESPECIALISTA` y `ASESOR` antes de cargar la fact pública.

### Tablas relevantes

- `Fact_AltasTeResuelve`
- `Dim_Calendario`
- `Dim_Aliado`
- `Dim_AsignacionPusherPeriodo`
- `Fact_MetasComerciales`
- `_Medidas_Altas`

### Relaciones

El modelo contiene 17 relaciones activas y cero relaciones declaradas como inactivas. Para el dominio comercial están confirmadas:

- `Dim_Calendario[Fecha]` 1:* `Fact_AltasTeResuelve[FechaAlta]`;
- `Dim_Aliado[AliadoKey]` 1:* `Fact_AltasTeResuelve[AliadoKey]`;
- `Dim_Calendario[Fecha]` 1:* `Fact_MetasComerciales[FechaPeriodo]`;
- `Dim_Aliado[AliadoKey]` 1:* `Fact_MetasComerciales[AliadoKey]`;
- `Dim_AsignacionPusherPeriodo[PeriodoAliadoKey]` 1:* `Fact_AltasTeResuelve[PeriodoAliadoKey]`;
- `Dim_AsignacionPusherPeriodo[PeriodoAliadoKey]` 1:* `Fact_MetasComerciales[PeriodoAliadoKey]`.

No se modificó ninguna relación durante R1.

### Medidas comerciales críticas

- `Altas_Total`
- `Altas_Mes_Anterior`
- `Diferencia_Altas_Mes`
- `Variacion_Altas_Mes_Pct`
- `Promedio_Altas_Hasta_Mes_Anterior`
- `Altas_Pusher_1`
- `Altas_Pusher_2`
- `Altas_Sin_Asignar`
- `Cobertura_Clasificacion_Pusher_Pct`
- `Delta_Aliado_Mes`
- `Ranking_Driver_Positivo`
- `Ranking_Driver_Negativo`
- `Meta_Asignada`
- `Cumplimiento_Meta_Pct`

No se modificó ninguna medida durante R1.

### Página y navegación

- Página técnica: `GestionComercialAltas`.
- Home continúa como página activa del informe.
- Los objetos navegables de Home apuntan a `GestionComercialAltas`.
- Los objetos de retorno de la página comercial apuntan a Home.

No se modificaron páginas, visuales, filtros ni navegación.

## 6. Dependencias e impacto de R2

El grafo local identifica la cadena Base → Limpio → Fact → dimensiones/asignación/metas → medidas → página comercial. La dependencia se confirmó directamente contra Power Query, TMDL y PBIR. El grafo fue construido sobre el último baseline técnico previo a los commits documentales; estos no alteraron el modelo.

R2 podría modificar únicamente, según necesidad comprobada:

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`;
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/Fact_AltasTeResuelve.tmdl` si cambia la selección final de columnas;
- metadatos automáticos generados por Desktop, solo después de clasificarlos;
- el Output de conciliación de R2.

R2 no deberá crear un segundo hecho de ventas ni combinar snapshots.

## 7. Controles de rollback

El rollback de R2 deberá recuperar selectivamente:

1. `Ruta_Informe_Altas` apuntando al archivo de cierre de julio;
2. `Base_AltasTeResuelve` seleccionando ese único archivo y `Insumo2`;
3. `Periodo_Corte_Comercial = 202607`;
4. los controles funcionales de la sección 4;
5. las relaciones, medidas, página y navegación registradas en este baseline.

No se usarán restauraciones masivas ni se combinará la fuente anterior con el consolidado.

## 8. Privacidad

- Cero nombres personales añadidos.
- Cero datos de asesores añadidos.
- Cero rutas locales o sensibles añadidas.
- Cero Excel, mockups o artefactos Graphify versionados.
- Power Query, TMDL, PBIR, relaciones, medidas y páginas permanecen sin cambios.

## 9. Resultado

**PASS.** El baseline anterior a R2 quedó registrado y las fuentes privadas locales quedaron protegidas contra inclusión accidental. Los únicos cambios rastreados de R1 son `.gitignore` y este Output.

R2 permanece **NO INICIADA** y requiere autorización independiente.
