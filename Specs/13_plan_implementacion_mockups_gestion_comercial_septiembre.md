# Plan de implementación — Mockups de gestión comercial septiembre

| Campo | Valor |
|---|---|
| Estado | Propuesto; pendiente de aprobación |
| Baseline | Implementación comercial publicada y R0 funcional cerrado |
| Estrategia | Cambios mínimos, secuenciales, verificables y reversibles |
| Alcance | R1 a R10 |

## 1. Objetivo

Implementar la nueva fuente acumulada, la asignación PUSHER, las metas y las páginas comerciales sin duplicar ventas, sin relaciones ambiguas y sin sustituir la versión publicada antes de superar sus gates.

El análisis de impacto asociado es [Specs/12_analisis_impacto_mockups_gestion_comercial_septiembre.md](12_analisis_impacto_mockups_gestion_comercial_septiembre.md).

## 2. Principios de ejecución

1. Un solo hecho de ventas y una sola fuente acumulada.
2. `FECHA_ALTA` es autoridad temporal; `MES` es control.
3. Reutilizar antes de crear.
4. No fuzzy matching ni homologaciones implícitas.
5. Relaciones 1:* y unidireccionales; evitar many-to-many.
6. No publicar datos personales o fuentes privadas en Git.
7. Los tres mockups se implementan como páginas nuevas; `GestionComercialAltas` permanece intacta durante R7-R9.
8. Cada fase termina con validación y aprobación antes de la siguiente.
9. El rollback será selectivo por archivos/commit de fase; nunca masivo sobre el working tree.
10. La publicación mediante Publicar en la Web exige un gate de privacidad reforzado.

## 3. Arquitectura objetivo mínima

```text
Consolidado/Insumo2
  → Base_AltasTeResuelve
  → AltasTeResuelve_Limpio
  → Fact_AltasTeResuelve
      ↘ Dim_Calendario
      ↘ Dim_Aliado
      ↘ Dim_AsignacionPusherPeriodo
      ↘ Dim_Asesor (solo si Página 2 pasa el gate de privacidad)

Metas_Bonos
  → staging validado
  → Fact_MetasComerciales (MetaAltas CALL + MetaAsesor contextual)

Legalización_Bonos
  → staging validado
  → Fact_LegalizacionBonos (grano real, sin aliado/asesor inventado)

Modelo
  → _Medidas_Altas
  → GestionComercialAltas intacta + Página 1 nueva + Página 2 nueva + Página 3 nueva
```

No se crearán otro hecho de ventas, otra dimensión PUSHER ni otra fact de metas salvo evidencia técnica que invalide este diseño.

## 4. R1 — Protección y baseline

### Objetivo

Congelar el estado publicado y proteger artefactos locales antes de modificar el modelo.

### Archivos y objetos esperados

- `.gitignore`: agregar `graphify-out/` sin eliminar reglas existentes.
- Output de baseline R1.
- Sin cambios en PBIP/TMDL/PBIR.

### Reutilización y cambio mínimo

- Reutilizar reglas actuales para `Data/**/*.xlsx`.
- Añadir una sola exclusión para Graphify.
- Registrar hashes, cifras publicadas y archivos sensibles.

### Validaciones automáticas

- rama, HEAD, remoto, staging y working tree;
- `git check-ignore` para Excel, mockups no autorizados y `graphify-out/`;
- inventario de objetos y medidas actuales;
- controles de julio/agosto y navegación vigente;
- Desktop y Excel cerrados.

### Gate manual

No requerido si el baseline coincide con Git y los Outputs cerrados.

### PASS

Baseline reproducible, datos privados ignorados y cero cambios técnicos.

### Rollback

Revertir únicamente la regla nueva de `.gitignore` y el Output si la exclusión afecta archivos legítimos.

## 5. R2 — Migración del consolidado

### Objetivo

Sustituir el snapshot anterior por `Consolidado informe de Altas.xlsx` como única fuente de ventas y derivar el periodo de `FECHA_ALTA`.

### Archivos y objetos esperados

- `expressions.tmdl`: `Ruta_Informe_Altas`, `Base_AltasTeResuelve`, `AltasTeResuelve_Limpio` y `Periodo_Corte_Comercial`.
- `Fact_AltasTeResuelve.tmdl` solo si la selección final de columnas necesita ajuste.
- Output de conciliación R2.

### Reutilización y cambio mínimo

- Mantener `Insumo2` y el pipeline Base → Limpio → Fact.
- Cambiar únicamente la selección del archivo.
- Derivar año/mes de `FECHA_ALTA`.
- Conservar discrepancia `MES` como indicador de calidad.
- Hacer el tipado tolerante a la columna opcional ausente; no inventarla si no tiene consumidor.
- Actualizar `Periodo_Corte_Comercial` a `202608`.

### Validaciones automáticas

- una sola fuente y una sola tabla `Insumo2`;
- 35.073 registros y `SUM(ALTAS) = 44.638` para el corte diagnosticado;
- conciliación mensual enero-septiembre;
- cero pérdida de filas por discrepancia temporal;
- corte `MAX(FECHA_ALTA)`;
- agosto cerrado y septiembre en curso;
- esquema final de la fact sin duplicados técnicos;
- privacidad de columnas cargadas.

### Gate manual

Refresh en Desktop y verificación agregada de enero-septiembre. No revisar diseño todavía.

### PASS

La fact reproduce el consolidado por fecha, julio es 4.519, agosto 5.715 y septiembre permanece en curso.

### Rollback

Revertir el commit de R2 para recuperar el snapshot publicado; no combinar ambas fuentes.

## 6. R3 — Asignación PUSHER

### Objetivo

Alimentar la clasificación temporal desde `Asignacion_PUSHER` conservando reglas explícitas y la semántica histórica aplicable.

### Archivos y objetos esperados

- `expressions.tmdl`: staging/mapeo de asignación.
- `Dim_Aliado.tmdl` y `Dim_AsignacionPusherPeriodo.tmdl` si requieren adaptar columnas o fuente.
- Relaciones solo si una clave existente cambia; no crear rutas paralelas.
- Output de conciliación R3.

### Reutilización y cambio mínimo

- Reutilizar `Dim_Aliado`, `Dim_AsignacionPusherPeriodo`, claves periodo-aliado y overrides aprobados que sigan vigentes.
- Prioridad: regla específica `DESCRIPCION + DESCRIPCION2`, luego general `DESCRIPCION`.
- Normalización explícita `Trim/Clean/uppercase`.
- No usar `ESPECIALISTA` ni fuzzy matching.

### Validaciones automáticas

- unicidad de reglas específicas y generales;
- cero colisiones con asignaciones diferentes;
- cobertura y lista agregada de `Sin asignar` sin datos personales;
- tres etiquetas públicas de PUSHER;
- una clasificación por periodo-aliado;
- cero many-to-many y cero rutas ambiguas;
- conciliación julio por PUSHER y drivers/ranking agregados.

### Gate manual

Solo si la conciliación muestra cambios materiales no explicados por la nueva fuente o asignación.

### PASS

Toda venta tiene una clasificación única o `Sin asignar`; la semántica temporal se conserva y no hay pérdida de altas.

### Rollback

Revertir el commit R3 y mantener el modelo R2; no corregir colisiones con reglas ad hoc.

## 7. R4 — Metas

### Objetivo

Migrar las metas a `Metas_Bonos`, usando `CALL` para Página 1 y una meta individual contextual deduplicada para Página 2.

### Archivos y objetos esperados

- `expressions.tmdl`: staging y controles de metas.
- `Fact_MetasComerciales.tmdl`: conservar grano periodo-aliado y añadir `MetaAsesor` si la validación confirma unicidad.
- `_Medidas_Altas.tmdl`: ajustes mínimos a medidas de meta existentes.
- Output de conciliación R4.

### Reutilización y cambio mínimo

- Reutilizar `Fact_MetasComerciales`, sus relaciones y `Meta_Asignada`.
- `MetaAltas` procede de `CALL`.
- `MetaAsesor` procede de `ASESOR`, deduplicada por clave lógica.
- `ESPECIALISTA` se conserva solo como control `CALL = ESPECIALISTA`.
- No crear otra fact de metas.
- No cargar resultados almacenados ni nombres internos.

### Validaciones automáticas

- un único `Meta_Ventas` distinto por clave lógica;
- tres filas de incentivo no multiplican la meta `ASESOR`;
- igualdad `CALL`/`ESPECIALISTA` en todos los contextos;
- metas `CALL`: julio 6.337, agosto 7.155, septiembre 11.080;
- julio por PUSHER: 2.959 y 3.378 para las categorías aplicables;
- cero duplicados en `PeriodoAliadoKey` de `Fact_MetasComerciales`;
- relaciones 1:* existentes conservadas.

### Gate manual

Requerido si aparece más de un valor de meta por clave o divergencia `CALL`/`ESPECIALISTA`.

### PASS

Metas reconciliadas sin suma duplicada, Página 1 obtiene meta canónica y Página 2 obtiene una meta individual contextual inequívoca.

### Rollback

Revertir R4 y mantener la configuración gobernada anterior hasta resolver el dato; nunca escoger una meta arbitraria.

## 8. R5 — Incentivos y legalización

### Objetivo

Modelar únicamente los campos demostrables de `Legalización_Bonos` y dejar en blanco los indicadores sin contrato.

### Archivos y objetos esperados

- `expressions.tmdl`: staging/limpieza de legalización.
- Nueva `Fact_LegalizacionBonos.tmdl` solo si supera validación de grano y tipos.
- `model.tmdl` y `relationships.tmdl` para registrar la fact y una relación de fecha válida.
- Output de perfil y conciliación R5.

### Reutilización y cambio mínimo

- Reutilizar `Dim_Calendario` cuando exista fecha válida.
- Usar PUSHER directamente en la fact para el filtro de Página 3 mientras no sea indispensable una dimensión común.
- No relacionar con aliado o asesor.
- No crear medidas de valor recibido, saldo o ejecución.

### Validaciones automáticas

- grano y clave técnica definidos sin duplicación;
- tipos de fecha e importes válidos;
- suma de gasto, legalizado y pendiente conciliada con la fuente;
- fecha de corte calculada con el máximo válido;
- cero relaciones inventadas;
- cero nombres personales o soportes/rutas en el modelo público.

### Gate manual

Requerido si el grano no permite distinguir movimientos o si las sumas no concilian.

### PASS

Fact mínima conciliada y segura; indicadores sin contrato permanecen omitidos/BLANK.

### Rollback

Eliminar selectivamente la nueva fact, registro y relación de R5; conservar R1-R4.

## 9. R6 — Medidas DAX

### Objetivo

Agregar únicamente las medidas necesarias para los tres mockups, reutilizando las existentes.

### Archivos y objetos esperados

- `_Medidas_Altas.tmdl` como ubicación principal.
- Tabla de medidas de legalización solo si el patrón actual exige separación funcional.
- Output de pruebas DAX R6.

### Reutilización y cambio mínimo

Reutilizar, entre otras, `Altas_Total`, `Meta_Asignada`, `Cumplimiento_Meta_Pct`, cambios mensuales, promedio, cobertura, drivers y ranking.

Medidas realmente nuevas previstas:

- fecha de corte de altas;
- brecha y faltante;
- crecimiento desde julio con día comparable;
- meta individual, faltante, avance y estado de asesor;
- conteos de asesores con meta/cumplimiento/cercanía;
- valor gastado, legalizado, pendiente y corte de legalización.

El bono objetivo y los cálculos dependientes de valor recibido se omiten.

### Validaciones automáticas

- pruebas por mes cerrado, septiembre parcial, julio, selección multimes y ausencia de datos;
- corte comparable respetando filtros PUSHER/Aliado;
- meses más cortos sin fechas inválidas;
- julio seleccionado sin comparación engañosa;
- metas individuales no sumadas por asesor ni por incentivo;
- medidas sin referencias rotas y formatos correctos;
- cifras reconciliadas con R2-R5.

### Gate manual

No requerido si todas las consultas DAX de control pasan. Se reserva para resultados ambiguos en selección multimes.

### PASS

Todas las medidas devuelven resultados conciliados y `BLANK` en contextos no comparables.

### Rollback

Revertir exclusivamente las medidas nuevas de R6; no alterar objetos conciliados de R2-R5.

## 10. R7 — Página 1: resumen comercial

### Objetivo

Crear una página nueva de resumen comercial según el mockup 1, sin modificar `GestionComercialAltas`.

### Archivos y objetos esperados

- PBIR de la nueva Página 1.
- `pages.json` para registrar la página nueva.
- Output visual R7.

### Reutilización y cambio mínimo

- Reutilizar patrones de navegación, tema, filtros, histórico, drivers y ranking sin editar la página existente.
- Crear únicamente los KPI y visuales exigidos por el nuevo contrato.
- Añadir matriz PUSHER → Aliado con Meta, Altas, Cumplimiento y Crecimiento.
- No mostrar valor gastado por aliado si R5 no demuestra esa relación.

### Validaciones automáticas

- JSON válido, referencias existentes, IDs únicos y canvas;
- Mes no filtra el histórico cuando deba preservar contexto;
- cifras por periodo/PUSHER/aliado;
- cero texto causal;
- cero campos personales;
- navegación Home ↔ página válida;
- `GestionComercialAltas` sin cambios.

### Gate manual

Obligatorio: render, legibilidad, filtros, interacciones, navegación y comparación con mockup sin copiar datos estáticos.

### PASS

Página nueva funcional y visualmente aprobada, sin regresión de cifras, navegación ni cambios en `GestionComercialAltas`.

### Rollback

Revertir únicamente la página nueva y su registro; `GestionComercialAltas` debe permanecer intacta.

## 11. R8 — Página 2: asesores

### Objetivo

Crear la página de cumplimiento individual de septiembre con metas contextuales y privacidad explícitamente validada.

### Archivos y objetos esperados

- `Dim_Asesor.tmdl` y relación 1:* con ventas, solo si se confirma como mínimo necesario.
- Ajuste mínimo de `Fact_AltasTeResuelve` para `AsesorKey`.
- Nueva carpeta de página PBIR y registro en `pages.json`.
- Navegación mínima desde Home o la página comercial, sujeta a aprobación.
- Output R8.

### Reutilización y cambio mínimo

- Ventas desde la fact existente; no segundo hecho.
- Meta individual desde `Fact_MetasComerciales[MetaAsesor]` bajo periodo/aliado.
- Tema, tarjetas, tablas y navegación existentes.
- Omitir Bono objetivo y Valor entregado si no hay contrato inequívoco.
- Usar un filtro predeterminado nativo estable para septiembre; evitar DAX hardcodeado.

### Validaciones automáticas

- cada asesor recibe una meta, no una fracción ni suma de incentivos;
- estados y orden por faltante;
- filtros PUSHER/Aliado;
- ventas por asesor suman al total del contexto;
- cero nombres en archivos versionados fuera de referencias de campo;
- ningún valor nominal queda serializado en filtros, ejemplos o logs;
- Publicar en la Web reconocido como exposición pública.

### Gate manual

Obligatorio: render, selección inicial, filtros, totales y aceptación explícita de privacidad nominal antes de publicar.

### PASS

Página conciliada y aprobada; exposición nominal aceptada conscientemente y limitada a la vista autorizada.

### Rollback

Retirar página, navegación, relación y dimensión de asesor de R8 mediante su commit atómico; mantener R1-R7.

## 12. R9 — Página 3: incentivos y legalización

### Objetivo

Crear la página de legalización limitada a datos con contrato demostrado.

### Archivos y objetos esperados

- Nueva página PBIR y registro en `pages.json`.
- Navegación mínima autorizada.
- Output R9.

### Reutilización y cambio mínimo

- Reusar fact y medidas R5-R6, tema y componentes.
- Filtros Mes, PUSHER y tipo de incentivo.
- Excluir Aliado mientras la fuente no lo informe.
- Mostrar gasto, legalizado, pendiente y corte.
- Omitir/mostrar N/A en valor recibido, saldo y ejecución.

### Validaciones automáticas

- JSON y referencias válidas;
- totales conciliados con R5;
- filtros sin relaciones ambiguas;
- títulos sin ROI ni ejecución presupuestal formal;
- cero columnas de soporte, rutas, observaciones sensibles o nombres personales;
- canvas y navegación válidos.

### Gate manual

Obligatorio: legibilidad, interpretación, filtros, navegación y confirmación de que los N/A no inducen a error.

### PASS

Página muestra solo información demostrable y no inventa granularidad ni relaciones.

### Rollback

Revertir el commit PBIR de R9; conservar modelo conciliado si se aprueba independientemente.

## 13. R10 — QA, documentación, publicación y cierre

### Objetivo

Validar integralmente la iniciativa, actualizar documentación estable, publicar manualmente y cerrar el repositorio.

### Archivos y objetos esperados

- documentación de medidas, modelo, páginas, decisiones y operación;
- Outputs de QA y cierre;
- sin cambios técnicos salvo defecto crítico aprobado.

### Reutilización y cambio mínimo

- Reusar scripts/checklists de QA anteriores.
- No repetir gates ya demostrados salvo áreas afectadas.
- Publicación manual; no automatizar ni exponer el enlace en Git.

### Validaciones automáticas

- parseo PBIR/JSON y sintaxis TMDL/Power Query;
- relaciones, referencias, IDs, canvas y navegación;
- conciliación de ventas, metas, asignación y legalización;
- regresión de páginas existentes;
- privacidad del modelo público y archivos versionados;
- cero mockups/libros/Graphify en Git;
- `git diff --check`, staging, commits y remoto.

### Gate manual

Obligatorio en Desktop y Power BI Service/Publicar en la Web: refresh, tres páginas, periodos cerrado/parcial, navegación, privacidad y ausencia de visuales rotos.

### PASS

Artefacto publicado validado, documentación actualizada, `main` sincronizada y working tree limpio salvo archivos locales ignorados.

### Rollback

Retirar/republicar el último artefacto aprobado y revertir únicamente el commit de fase defectuoso. No eliminar fuentes ni worktrees sin auditoría.

## 14. Secuencia y gates

```text
R1 → R2 → R3 → R4 → R5 → R6 → R7 → R8 → R9 → R10
```

- R2 bloquea R3-R9 porque fija cifras y temporalidad.
- R3 y R4 deben estar conciliadas antes de DAX y páginas.
- R5 puede cerrarse con alcance reducido si los contratos pendientes siguen sin resolverse.
- R7 crea una página nueva y no sustituye `GestionComercialAltas`. Solo después de aprobar R7-R9 podrá decidirse si la página anterior se conserva, se oculta o se retira.
- R8 exige gate de privacidad nominal.
- R9 no depende de resolver valor recibido.
- Ninguna fase autoriza automáticamente la siguiente.

## 15. Criterios de aceptación globales

- una sola fuente y un solo hecho de ventas;
- `SUM(ALTAS)` conciliado con el consolidado;
- periodo derivado de `FECHA_ALTA` y septiembre en curso;
- asignación única o `Sin asignar`, sin fuzzy matching;
- metas `CALL` y `ASESOR` sin duplicación;
- cero many-to-many nuevas;
- indicadores no confirmados omitidos/BLANK;
- privacidad aprobada para Publicar en la Web;
- cero archivos privados o nombres personales versionados;
- páginas y navegación aprobadas;
- rollback probado por commits atómicos;
- documentación y Git consistentes.

## 16. Gate para iniciar R1

R1 solo puede comenzar después de aprobar expresamente este plan. La aprobación no autoriza R2 ni fases posteriores.
