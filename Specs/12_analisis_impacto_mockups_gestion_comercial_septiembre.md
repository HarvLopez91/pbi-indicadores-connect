# Análisis de impacto — Mockups de gestión comercial septiembre

| Campo | Valor |
|---|---|
| Estado | R0 funcional cerrado; análisis aprobado para planificación |
| Alcance | Migración de fuente, asignación PUSHER, metas, incentivos y tres páginas comerciales |
| Fuente de verdad | Repositorio actual y contratos funcionales R0 |
| Implementación | No iniciada |
| Privacidad | Repositorio público y distribución mediante Publicar en la Web |

## 1. Objetivo

Consolidar el impacto técnico y funcional de los tres mockups de septiembre sin modificar todavía Power Query, TMDL, PBIR, Excel ni el modelo publicado. El diseño debe preservar intacta la implementación aprobada de `GestionComercialAltas` mientras se construyen y validan tres páginas nuevas.

Este documento usa las siguientes etiquetas:

- **OBSERVADO:** confirmado directamente en archivos, modelo o datos inspeccionados.
- **INFERIDO:** relación sugerida por Graphify o por estructura y nombres.
- **CONFIRMADO:** inferencia contrastada con TMDL, PBIR, Power Query o fuente real.
- **PROPUESTO:** diseño pendiente de implementación y validación.

## 2. Alcance y límites

Incluye:

- migración de la fuente oficial acumulada de ventas;
- actualización de la asignación comercial de PUSHER;
- incorporación gobernada de metas;
- incorporación mínima de incentivos y legalización;
- creación de tres páginas nuevas, una por mockup, sin rediseñar ni sustituir `GestionComercialAltas`;
- medidas, validaciones, privacidad, documentación y publicación.

No incluye:

- combinar snapshots históricos;
- conservar artificialmente cifras sustituidas por la fuente oficial;
- fuzzy matching;
- inferir la semántica de niveles de incentivo o de `Valor recibido`;
- versionar libros, nombres personales, mockups sin sanitizar o `graphify-out/`;
- implementar cambios durante R0.

## 3. Fuentes y contratos R0

### 3.1 Ventas

**CONFIRMADO.** `Consolidado informe de Altas.xlsx`, tabla formal `Insumo2`, será la única fuente oficial acumulada desde enero de 2026.

Flujo aprobado:

```text
Consolidado informe de Altas.xlsx
  → Insumo2
  → Base_AltasTeResuelve
  → AltasTeResuelve_Limpio
  → Fact_AltasTeResuelve
```

Los libros anteriores son snapshots superpuestos y no se anexarán. La regla comercial permanece:

```DAX
Ventas = SUM ( Fact_AltasTeResuelve[Altas] )
```

El conteo de filas no representa ventas.

### 3.2 Autoridad temporal

**CONFIRMADO.** `FECHA_ALTA` determina año y mes. `MES` queda como control de calidad. Una discrepancia no elimina la venta: el periodo se deriva de `FECHA_ALTA` y la diferencia se registra para trazabilidad.

La fecha de corte será el máximo de `FECHA_ALTA`, nunca el nombre o la fecha de modificación del archivo.

Agosto de 2026 está cerrado; septiembre continúa en curso. La implementación deberá actualizar `Periodo_Corte_Comercial` de `202607` a `202608` sin convertir automáticamente el último mes disponible en periodo cerrado.

### 3.3 Asignación PUSHER

**CONFIRMADO.** La fuente autorizada es `BI - NUEVO.xlsx`, hoja `Asignacion_PUSHER`.

Orden de cruce:

1. `DESCRIPCION + DESCRIPCION2` para una regla específica.
2. `DESCRIPCION` para una regla general.
3. Sin coincidencia inequívoca: `Sin asignar`.

Se aplicarán `Trim`, `Clean` y normalización consistente. `ESPECIALISTA` no será llave obligatoria. No se usará fuzzy matching. Toda homologación deberá ser explícita y auditable.

Las etiquetas públicas serán exclusivamente `PUSHER 1`, `PUSHER 2` y `PUSHER 3`. Las equivalencias nominales internas no se versionan.

La clasificación histórica del portafolio se conserva separada de la atribución de gestión: `PUSHER 2` inicia en `202607` y `PUSHER 3` en `202608`. Los periodos anteriores son línea base histórica, permanecen clasificados y no deben atribuirse como resultado, cumplimiento, crecimiento o impacto de la gestión. No se define una nueva fecha para `PUSHER 1`.

### 3.4 Metas

**CONFIRMADO.** La fuente autorizada es `BI - NUEVO.xlsx`, hoja `Metas_Bonos`. `Tipo_Meta` contiene `META PARA BONO`; el nivel funcional se identifica mediante `Asesor_Equipo`: `CALL`, `ESPECIALISTA` o `ASESOR`.

- Página 1: `CALL` es la meta canónica por periodo, PUSHER y aliado.
- `ESPECIALISTA`: control de calidad; debe coincidir con `CALL`. No se suma ni se usa como fallback silencioso.
- Página 2 (actualizado en R8): cada fila `ASESOR` es un nivel de premio (`Ranking_Valor_Incentivo` + `Meta_Ventas` + `Valor_Incentivo`); la meta mínima del contexto es la referencia de avance y los premios se asignan por ranking entre quienes alcanzan la meta de cada nivel (Output 77).
- Las filas `ASESOR` son niveles de premio; cada nivel puede tener su propia meta. Un nivel puede reservarse a un asesor dedicado mediante `Asesor_Dedicado` (Output 77).
- Por clave lógica debe existir un único valor distinto de `Meta_Ventas`; más de uno es error de calidad.

No se utilizarán como resultados oficiales `Ventas_Cumplimiento`, `Cumplimiento_Meta` ni `Ganador_Bono`. Ventas y cumplimiento se recalcularán desde las fuentes gobernadas.

### 3.5 Incentivos y legalización

**CONFIRMADO.** La fuente es `BI - NUEVO.xlsx`, hoja `Legalización_Bonos`. El campo `Aliado / Call` no está informado y no respalda una relación con aliado o asesor.

Se podrá modelar solamente lo demostrable: valor gastado, valor legalizado, pendiente por legalizar, fecha de corte, PUSHER, concepto, tipo de incentivo y estado cuando sea necesario.

Mientras no exista contrato para `Valor recibido`, los indicadores de valor recibido, saldo derivado y ejecución del recurso serán `BLANK`/N/A o se omitirán. No se denominarán ROI ni ejecución presupuestal formal.

## 4. Conciliación histórica aceptada

**CONFIRMADO.** La reexpresión de la fuente consolidada sustituye el baseline anterior.

| Mes 2026 | Baseline anterior | Consolidado | Diferencia |
|---|---:|---:|---:|
| Enero | 6.297 | 6.296 | -1 |
| Febrero | 5.310 | 5.309 | -1 |
| Marzo | 5.109 | 5.110 | +1 |
| Abril | 4.190 | 4.190 | 0 |
| Mayo | 4.171 | 4.171 | 0 |
| Junio | 3.700 | 3.700 | 0 |
| Julio | 4.518 | 4.519 | +1 |
| Agosto | 559 parcial | 5.715 cierre | +5.156 frente al corte parcial |
| Septiembre | Sin baseline | 5.628 parcial | No aplica |

El consolidado inspeccionado contiene 35.073 registros, `SUM(ALTAS) = 44.638` y corte hasta el 23 de septiembre de 2026. Existe una discrepancia `MES` frente a `FECHA_ALTA` que afecta una alta; con la regla R0, la venta se conserva y se asigna al periodo derivado de la fecha.

Impactos de control ya identificados:

- julio cambia de 4.518 a 4.519;
- con la asignación actual, julio cambia en `PUSHER 1` de 1.581 a 1.582; `PUSHER 2` y `Sin asignar` permanecen en 2.429 y 508;
- los dos principales drivers positivos y la principal caída mantienen valor y orden; un driver secundario cambia en una unidad sin alterar el orden;
- la meta canónica de julio cambia de 6.317 a 6.337 por migración de fuente, no por error de cálculo.

## 5. Contexto estructural e impacto Graphify

### 5.1 Resultado del grafo

**INFERIDO.** El grafo local identifica la cadena principal `Base_AltasTeResuelve → AltasTeResuelve_Limpio → Fact_AltasTeResuelve → Dim_Aliado/_Medidas_Altas` y la página `GestionComercialAltas` como consumidora final.

**CONFIRMADO.** La inspección directa amplía esa cadena:

```text
Base_AltasTeResuelve
  → AltasTeResuelve_Limpio
  → Fact_AltasTeResuelve
  → Dim_Calendario
  → Dim_Aliado
  → Dim_AsignacionPusherPeriodo
  → Fact_MetasComerciales
  → _Medidas_Altas
  → GestionComercialAltas
```

Las relaciones vigentes entre dimensiones y hechos son 1:* y unidireccionales. La página usa `Dim_Calendario`, `Dim_AsignacionPusherPeriodo`, `Dim_Aliado`, `Fact_AltasTeResuelve`, `Fact_MetasComerciales` y `_Medidas_Altas`.

`graphify-out/` es un artefacto local no versionable. Su exclusión se incorporará en R1.

### 5.2 Impacto por componente

| Componente | Estado | Impacto previsto |
|---|---|---|
| `Base_AltasTeResuelve` | Confirmado | Cambiar selección del archivo, conservar `Insumo2` y validaciones |
| `AltasTeResuelve_Limpio` | Confirmado | Derivar periodo de `FECHA_ALTA`; mantener discrepancia como control |
| `Fact_AltasTeResuelve` | Confirmado | Conservar un solo hecho de ventas; añadir soporte de asesor solo si Página 2 lo exige |
| `Dim_Calendario` | Confirmado | Ampliarse hasta el corte consolidado; agosto cerrado, septiembre en curso |
| `Dim_Aliado` | Confirmado | Reutilizar normalización y clave actual |
| `Dim_AsignacionPusherPeriodo` | Confirmado | Reutilizar y alimentar desde asignación autorizada, incluida la tercera categoría |
| `Fact_MetasComerciales` | Confirmado | Sustituir configuración gobernada por metas `CALL`; incorporar una columna contextual de meta individual si pasa validaciones |
| `_Medidas_Altas` | Confirmado | Reutilizar medidas actuales y agregar solo cálculos no existentes |
| `GestionComercialAltas` | Confirmado | Mantener intacta durante la construcción y validación de las páginas nuevas |
| Páginas 1, 2 y 3 | Propuesto | Crear tres páginas nuevas, una por mockup |
| Legalización | Propuesto | Crear un hecho mínimo independiente por tener proceso y grano propios |

## 6. Diferencias técnicas relevantes de las fuentes

### 6.1 Consolidado de ventas

**OBSERVADO.** `Insumo2` mantiene las columnas comerciales obligatorias, pero no contiene `UNIDAD_NEGOCIO2`. La consulta actual tolera su ausencia durante el renombrado mediante `MissingField.Ignore`, pero posteriormente intenta tiparla de forma fija; ese paso podría romper la carga.

**PROPUESTO.** Ajustar únicamente el tipado para operar sobre columnas existentes o retirar la dependencia opcional si se confirma que no tiene consumidor. No crear una columna ficticia si no aporta al modelo.

### 6.2 Metas

**OBSERVADO.** La hoja tiene 245 filas efectivas y no es tabla formal. Hay 49 contextos funcionales por cada categoría. `CALL` y `ESPECIALISTA` coinciden en los 49 controles. Cada contexto `ASESOR` aparece tres veces con una sola meta distinta y tres valores de incentivo.

Totales `CALL` observados:

| Periodo | Meta total |
|---|---:|
| Julio 2026 | 6.337 |
| Agosto 2026 | 7.155 |
| Septiembre 2026 | 11.080 |

La meta `ASESOR` se deduplicará por la clave lógica aprobada; no se sumarán sus tres filas.

### 6.3 Legalización

**OBSERVADO.** La hoja contiene fecha, mes, PUSHER, concepto, tipo de incentivo y valores de gasto/legalización. No existe una clave demostrable hacia aliado o asesor. El contrato de valor recibido sigue abierto.

## 7. Matriz de reutilización

| Elemento | Clasificación | Decisión mínima |
|---|---|---|
| Pipeline Base → Limpio → Fact | Reutilizable con ajuste | Cambiar fuente y autoridad temporal; no crear otro pipeline |
| `Fact_AltasTeResuelve` | Reutilizable con ajuste | Mantener un solo hecho; conservar `SUM(Altas)` |
| `Dim_Calendario` | Reutilizable con ajuste | Actualizar corte y extensión por fecha real |
| `Dim_Aliado` | Reutilizable con ajuste | Mantener claves y homologación explícita |
| `Dim_AsignacionPusherPeriodo` | Reutilizable con ajuste | Reemplazar/alimentar reglas desde fuente autorizada |
| `Fact_MetasComerciales` | Reutilizable con ajuste | Mantener grano periodo-aliado; incorporar meta `CALL` y meta individual contextual |
| `_Medidas_Altas` | Reutilizable con ajuste | Evitar medidas duplicadas |
| `GestionComercialAltas` | Reutilizable sin cambio | Mantenerla intacta; reutilizar únicamente sus patrones y componentes |
| Tema, navegación y componentes | Reutilizable sin cambio conceptual | Replicar patrones existentes |
| Dimensión de asesor | Realmente nueva si Página 2 la requiere | Crear solo para navegación nominal y relación 1:* con ventas |
| Hecho de legalización | Realmente nuevo | Grano propio; sin relación inventada con aliado/asesor |
| Página de resumen comercial | Realmente nueva | No existe equivalente con el contrato del mockup 1 |
| Página de asesores | Realmente nueva | No existe equivalente |
| Página de legalización | Realmente nueva | No existe equivalente |
| Dimensión PUSHER paralela | No requerida | Reutilizar asignación actual; no crearla |
| Segundo hecho de ventas | No requerido | Prohibido por riesgo de duplicación |

## 8. Gate Ponytail por objeto propuesto

| Objeto | ¿Debe existir? | Equivalente actual | Solución nativa/reutilización | Cambio mínimo |
|---|---|---|---|---|
| Cambio de fuente | Sí | Pipeline actual | Reutilizar Power Query existente | Sustituir archivo y corregir tipado opcional |
| Nueva fact de ventas | No | `Fact_AltasTeResuelve` | Reutilizar | Ninguno |
| Asignación PUSHER | Sí | `Dim_AsignacionPusherPeriodo` | Reutilizar dimensión y claves | Cambiar origen/reglas explícitas |
| Metas Página 1 | Sí | `Fact_MetasComerciales` | Reutilizar fact | Alimentar `MetaAltas` desde `CALL` |
| Meta individual | Sí | Mismo contexto periodo-aliado | Añadir columna `MetaAsesor` a la fact actual | No crear otra fact de metas |
| Asesor | Sí para Página 2 | No existe en el modelo público | Implementado en R8 como columna `Fact_AltasTeResuelve[Asesor]`, sin `Dim_Asesor` (no hay identificador ni atributos adicionales) | Nombre desde la fuente |
| Incentivos/legalización | Sí para Página 3 | No existe hecho equivalente | Nueva fact mínima | Solo columnas con contrato confirmado |
| Bono objetivo | No por ahora | Semántica no confirmada | Omitir/BLANK | Cero lógica arbitraria |
| Valor recibido/saldo/ejecución | No por ahora | Contrato no confirmado | Omitir/BLANK | Cero derivación |
| Página 1 nueva | Sí | No existe | Reusar tema, navegación y componentes | Crear una página nueva sin modificar `GestionComercialAltas` |
| Páginas 2 y 3 | Sí | No existen | Reusar tema y navegación | Dos páginas nuevas |

## 9. Impacto funcional de los mockups

### 9.1 Página 1 — Resumen comercial

**PROPUESTO.** Crear una página nueva de resumen comercial según el mockup 1. Reutilizar los patrones de filtros, navegación, tema, volumen, variación, histórico, drivers y ranking sin modificar `GestionComercialAltas`. Incorporar Meta, Cumplimiento, Brecha, crecimiento comparable desde julio y valor gastado solo cuando la relación de incentivos sea demostrable.

El crecimiento contra julio debe comparar el mismo día de corte: un mes parcial al día N contra julio hasta el día N. Debe manejar meses más cortos, julio seleccionado, meses cerrados, ausencia de datos y filtros activos. Una selección multimes deberá devolver `BLANK` o una indicación no comparable, no una comparación ambigua.

### 9.2 Página 2 — Asesores

**PROPUESTO.** Página nueva con filtros PUSHER y Aliado, jerarquía PUSHER → Aliado → Asesor, ventas oficiales por `SUM(ALTAS)` y meta individual contextual deduplicada.

Estados:

- Cumplió: avance mayor o igual a 100 %.
- Cerca de cumplir: avance mayor o igual a 80 % y menor a 100 %.
- En progreso: avance menor a 80 %.
- Sin meta: no existe meta válida.

El bono objetivo y el valor entregado se omitirán mientras no exista una relación inequívoca. La selección inicial de septiembre se resolverá con capacidad nativa y estable de Power BI; no se hardcodeará una selección frágil en DAX.

### 9.3 Página 3 — Incentivos y legalización

**PROPUESTO.** Página nueva limitada a la granularidad real de `Legalización_Bonos`. Filtros: Mes, PUSHER y tipo de incentivo. Aliado se excluye mientras no tenga datos confiables.

Indicadores inicialmente demostrables: valor gastado, valor legalizado, pendiente por legalizar y fecha de corte. Los indicadores dependientes de valor recibido quedan omitidos o N/A.

## 10. Riesgos y mitigaciones

| Riesgo | Severidad | Mitigación prevista |
|---|---|---|
| Doble conteo al anexar snapshots | Bloqueante | Fuente única consolidada; no combinar archivos |
| Pérdida por discrepancia `MES`/fecha | Bloqueante | Periodo derivado de `FECHA_ALTA`; discrepancia solo como control |
| Ruptura por `UNIDAD_NEGOCIO2` ausente | Mayor | Tipado sobre columnas existentes o eliminación de dependencia no usada |
| Colisiones en asignación | Mayor | Prioridad regla específica/general y control de unicidad |
| Pérdida por usar `ESPECIALISTA` como llave | Mayor | No usarlo como condición obligatoria |
| Many-to-many o dimensión PUSHER redundante | Mayor | Reutilizar asignación periodo-aliado y relaciones 1:* |
| Meta duplicada por incentivos | Bloqueante | Un valor distinto por clave; tres filas no se suman |
| Divergencia `CALL`/`ESPECIALISTA` | Mayor | Error de calidad; no fallback silencioso |
| Mes parcial comparado con mes completo | Mayor | Corte comparable por día y estado de periodo |
| Reexpresión histórica inesperada | Mayor | Conciliación mensual y controles antes/después |
| Exposición nominal en Publicar en la Web | Crítico | Gate explícito de privacidad y aprobación funcional antes de publicar Página 2 |
| Relación inventada en legalización | Mayor | No relacionar con aliado/asesor sin evidencia |
| Regresión de página publicada | Mayor | Mantener `GestionComercialAltas` intacta; decidir conservarla, ocultarla o retirarla solo después de aprobar las tres páginas nuevas |
| Artefactos Graphify versionados | Menor | Agregar `graphify-out/` a `.gitignore` en R1 |

## 11. Deudas funcionales no bloqueantes

1. Semántica de los tres niveles de `Valor_Incentivo` y regla de bono objetivo.
2. Contrato de `Valor recibido` en legalización.
3. Relación de movimientos de legalización con aliado o asesor; hoy no está respaldada.
4. Sanitización de los mockups antes de considerar su versionado.

Estas deudas no bloquean la migración de ventas, la asignación, las metas, Página 1 ni el núcleo de Página 2. Sí limitan indicadores específicos de bonos y Página 3.

## 12. Decisiones consolidadas y estado

Las decisiones R0 están cerradas. No quedan preguntas de negocio bloqueantes para iniciar R1. La implementación deberá detenerse únicamente si aparece una contradicción técnica que pueda causar pérdida, duplicación, exposición o regresión.

El orden y los gates se definen en [Specs/13_plan_implementacion_mockups_gestion_comercial_septiembre.md](13_plan_implementacion_mockups_gestion_comercial_septiembre.md).
