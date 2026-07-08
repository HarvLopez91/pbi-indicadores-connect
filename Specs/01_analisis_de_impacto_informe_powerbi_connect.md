# Análisis de Impacto — Informe Power BI Connect Assistance S.A.S.

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Cliente / Contexto | Connect Assistance S.A.S. — call centers asociados a Claro |
| Rol funcional | PUSHER (auditoría de calidad + formación + seguimiento comercial) |
| Tipo de documento | Análisis de impacto (diagnóstico, sin cambios en artefactos PBIP) |
| Fecha | 2026-07-08 |
| Autor | Análisis asistido (Claude Code) |
| Estado | Diagnóstico inicial — pendiente de validación de negocio |

---

## 1. Resumen ejecutivo

El proyecto `PBI_Indicadores` contiene un **PBIP inicializado pero vacío**: no existen tablas, relaciones, medidas DAX ni visuales. Es un lienzo en blanco listo para construir el modelo desde cero, lo cual es positivo porque no hay deuda técnica previa que migrar.

La carpeta `Data` contiene **3 archivos Excel exportados de Google Forms** ("Form Responses 1"), cada uno representando una encuesta/matriz distinta:

1. **Matriz de calidad (Responses).xlsx** — auditoría de calidad de llamadas (checklist por el rol PUSHER).
2. **Satisfacción capacitación (Responses).xlsx** — encuesta de satisfacción post-capacitación.
3. **Encuesta satisfacción (Responses).xlsx** — encuesta de satisfacción/motivación de actividades comerciales.

**Hallazgo crítico:** el volumen de datos actual es de piloto/prueba (3, 32 y 5 filas respectivamente), muy por debajo de lo necesario para indicadores estadísticamente representativos. El modelo y el informe deben diseñarse para escalar automáticamente cuando el volumen crezca (sin rediseño), pero los KPIs no deben presentarse aún como definitivos.

Se identificaron problemas de calidad de datos típicos de formularios sin validación: encabezados con espacios al inicio/fin, tildes, mayúsculas inconsistentes, nombres de personas con variantes (typos, capitalización), una columna de escala mezclada con texto libre (`"N/A"` junto a números), y una columna de identificación 100% vacía en una de las encuestas.

Se propone un modelo estrella con 3 tablas de hechos (una por encuesta), 3 dimensiones compartidas (Calendario, Call Center, Jornada) y transformaciones de limpieza en Power Query antes de la carga. No se requieren cambios en el PBIP en este paso; este documento es el insumo para la siguiente fase de construcción.

---

## 2. Archivos analizados

### 2.1 Proyecto PBIP (`PBI/`)

| Elemento | Estado encontrado |
|---|---|
| [PBI_Indicadores.pbip](../PBI/PBI_Indicadores.pbip) | Existe. Apunta a `PBI_Indicadores.Report`. `enableAutoRecovery: true`. |
| [PBI_Indicadores.SemanticModel/definition/model.tmdl](../PBI/PBI_Indicadores.SemanticModel/definition/model.tmdl) | Modelo **vacío**: sin tablas, sin relaciones, sin medidas, sin Power Query. Cultura `es-ES`, `sourceQueryCulture: es-CO`. Time Intelligence habilitado. Formato TMDL (moderno, correcto). |
| `PBI_Indicadores.SemanticModel/definition/cultures/es-ES.tmdl` | Presente (metadatos de cultura por defecto). |
| [PBI_Indicadores.Report/definition/report.json](../PBI/PBI_Indicadores.Report/definition/report.json) | Tema base `CY25SU11` (tema estándar de Power BI, no personalizado a marca Connect). Ajustes de exportación/tooltips por defecto. |
| `PBI_Indicadores.Report/definition/pages/` | **1 sola página** ("Página 1"), 1280×720, sin visuales. |
| [PBI/.gitignore](../PBI/.gitignore) | Ya excluye `**/.pbi/localSettings.json` y `**/.pbi/cache.abf` — buena práctica ya aplicada. |
| Repositorio Git | **No inicializado** a nivel de `PBI_Indicadores` (carpeta raíz no es repo git). Se recomienda `git init` antes de iterar, para poder versionar el TMDL/JSON del PBIP correctamente. |
| Carpeta `Specs/` | No existía — creada en este paso para alojar este documento. |

**Conclusión:** no hay nada que migrar ni romper. Todo el trabajo de modelado, DAX, páginas y tema visual se construye desde cero.

### 2.2 Archivos fuente (`Data/`)

| Archivo | Hoja | Filas de datos | Columnas | Naturaleza |
|---|---|---|---|---|
| `Matriz de calidad (Responses).xlsx` | `Form Responses 1` | **3** | 16 | Auditoría de calidad de llamada (checklist PUSHER) |
| `Satisfacción capacitación (Responses).xlsx` | `Form Responses 1` | **32** | 12 | Encuesta de satisfacción de capacitaciones |
| `Encuesta satisfacción (Responses).xlsx` | `Form Responses 1` | **5** | 10 | Encuesta de satisfacción/motivación de actividades comerciales |

Nota operativa: el archivo `Encuesta satisfacción (Responses).xlsx` estaba bloqueado por otro proceso (Excel/OneDrive) durante el análisis; se accedió mediante una copia de solo lectura. Esto no representa un problema del archivo en sí, pero **al conectar Power BI a estos archivos, deben estar cerrados en Excel** para evitar errores de actualización (`The process cannot access the file`).

---

## 3. Diagnóstico de datos

### 3.1 Matriz de calidad (Responses).xlsx

| # | Columna original | Problema detectado | Tipo de dato observado |
|---|---|---|---|
| 0 | `Timestamp` | OK | datetime |
| 1 | `CALL CENTER` | OK (mayúsculas consistentes) | texto — valores vistos: `ONE CONTACT`, `CAPITALS` |
| 2 | `  Nombre del asesor  ` | **Espacios al inicio/fin**; capitalización inconsistente (`Andres stitch` vs `Daniel Martinez`); sin ID único | texto |
| 3 | `Líder / supervisor  ` | Espacio final; nombre corto (solo nombre de pila) | texto |
| 4 | `  Auditor / PUSHER  ` | Espacios; único valor visto `Jeisy Martinez` (coincide con el `Nombre formador` de la encuesta de capacitación → **misma persona, rol PUSHER**) | texto |
| 5–12 | 7 preguntas de checklist (`¿El asesor inició…?`, `¿Usó una frase de impacto…?`, etc.) | **Escala de puntaje no uniforme entre preguntas** y **mezcla de tipos**: algunas celdas son numéricas (`0`, `1`, `2`, `3`) y otras son el texto `"N/A"` en la misma columna. Esto impide sumar directamente con `SUM()` sin tratar `"N/A"` como no aplica (no como 0). Además, cada pregunta parece tener un puntaje máximo distinto (p. ej. la pregunta de cierre llega a 3, la de tono llega a 2) — **se requiere confirmar con negocio la rúbrica de puntaje máximo por pregunta** para calcular `% calidad` correctamente. | float / str mixto |
| 13 | `Observaciones` | Texto libre largo, con saltos de línea; sin estandarizar | texto |
| 14 | `Objeción principal del cliente  ` | Espacio final; texto libre pero con pocos valores repetidos (`No me interesa`, `Muy caro`) → candidato a lista controlada | texto |
| 15 | `  ¿La llamada terminó en venta?  ` | Espacios; valores Sí/No (en la muestra solo se observó `No`) | texto |

**Riesgos específicos:**
- Con solo 3 registros no se puede validar el rango real de valores por pregunta ni la existencia de otros call centers/asesores.
- No existe un ID de llamada ni de auditoría — el `Timestamp` es el único campo con valores únicos garantizados; se usará como llave técnica temporal.
- Sin columna `Jornada`, no es posible cruzar calidad de llamada por turno (Mañana/Tarde), a diferencia de las otras dos encuestas.

### 3.2 Satisfacción capacitación (Responses).xlsx

| # | Columna original | Problema detectado |
|---|---|---|
| 0 | `Timestamp` | OK, 32 valores únicos |
| 1 | `  ¿En qué call center trabajas?  ` | Espacios; 1 nulo; valores: `ONE CONTACT`, `INTERACTIVO` |
| 2 | `  ¿En qué jornada participaste?  ` | Espacios; 1 nulo; valores: `Mañana`, `Tarde` |
| 3 | `Nombre completo (mayúscula)  ` | Espacio final; 1 nulo; **31 valores casi-únicos pero con inconsistencia de capitalización** (`PAULA VALENTINA BOLIVAR GARCIA` vs `Angela Andrea Ramírez mora` — el formulario pide mayúscula pero no se valida) |
| 4 | `Nombre del líder (Mayúscula) ` | **Hallazgo crítico de calidad**: el mismo líder aparece con al menos 4 grafías distintas: `JUAN ESTEBAN PEREZ CAMARGO `, `Juan Esteban Pérez caramargo `, `JUAN ESTEBAN CAMARGO `, `JUAN ESTEBAN CÁMARGO`, `JUAN ESTEBAN PEREZ CAMARGO` (con/sin tilde, con/sin apellido materno, con typo "caramargo"). **Sin normalización, cualquier visual "por líder" fragmentará resultados de una misma persona en varias filas.** |
| 5 | `Nombre formador` | Único valor: `Jeisy Martinez` (mismo PUSHER de la matriz de calidad) |
| 6–9 | 4 preguntas Likert (`satisfecho/a`, `clara y fácil`, `útil`, `dinámica`) | Escala 1–5 consistente, sin `N/A` mezclado — más limpia que la matriz de calidad |
| 10 | `Duración` | Categórica (`1 hora`, `30 minutos`); 2 nulos |
| 11 | `  ¿Qué mejorarías…?  ` | Texto libre abierto; 10 nulos de 32; valores tipo `"Na"`, `"N/A"`, `"Nada"`, `"Ninguna"` que deberían tratarse como "sin comentario" y no como texto analizable; incluye emojis |

**Riesgo específico:** la columna de líder requiere normalización obligatoria (Trim + Clean + Proper/mapeo manual) antes de poder agrupar resultados por líder de forma confiable.

### 3.3 Encuesta satisfacción / motivación de actividades comerciales (Responses).xlsx

| # | Columna original | Problema detectado |
|---|---|---|
| 0 | `Timestamp` | OK, 5 valores únicos |
| 1 | `  ¿En qué call center trabajas?  ` | Espacios; valores: `MILLENIUM`, `CAPITALS` (call centers **nuevos**, no vistos en los otros 2 archivos → el universo de call centers debe construirse por unión de las 3 fuentes, no como lista fija) |
| 2 | `  ¿En qué jornada participaste?  ` | Espacios; en la muestra solo `Mañana` |
| 3 | `  En general, ¿Qué tan satisfecho/a…?  ` | Likert 1–5 |
| 4 | ` La actividad fue clara, dinámica y útil…  ` | Likert 1–5 |
| 5 | `Después de la actividad, ¿te sentiste más motivado/a…?  ` | Likert 1–5 |
| 6 | `  ¿Cómo describirías el ambiente de tu equipo?  ` | Categórica: `Motivado`, `Colaborativo`, `Presionado` — candidata a lista fija controlada |
| 7 | `  ¿Qué tipo de actividades…funcionan mejor…?  ` | Categórica de texto libre con alta variedad potencial (`Juegos rápidos`, `Actividades con premios`, `Retos por ventas`, `Preguntas y respuestas`, `Dinámicas sentados`) |
| 8 | `  ¿Qué mejorarías…?  ` | Texto libre abierto |
| 9 | `Nombre completo (Mayúscula) ` | **100% vacía (5/5 nulos)**. La encuesta es efectivamente anónima — no se puede vincular esta encuesta a un asesor específico. |

**Riesgo específico:** al no existir nombre de encuestado, esta tabla solo permite análisis agregado por call center/jornada, **no** por asesor individual. Debe documentarse esta limitación en el informe (ej. nota o tooltip) para que el usuario final no espere poder filtrar por asesor en esta página.

### 3.4 Problemas transversales (aplican a las 3 fuentes)

1. **Encabezados con espacios al inicio/fin y con tildes/signos** (`¿`, `?`) — no aptos como nombres técnicos de columna; requieren renombrado en Power Query.
2. **Sin identificador único de fila** — se usará `Timestamp` (más índice de fila como respaldo) como llave técnica de cada hecho.
3. **Sin identificador único de persona** (asesor/líder) — el nombre es la única llave disponible y ya se demostró inconsistente. Se requiere normalización + posible tabla de mapeo manual (alias → nombre estándar).
4. **Catálogo de Call Center no está centralizado**: se detectaron ya 4 call centers distintos entre los 3 archivos (`ONE CONTACT`, `CAPITALS`, `INTERACTIVO`, `MILLENIUM`) sin una fuente maestra común.
5. **Volumen de datos bajo (piloto)**: 3 + 32 + 5 = 40 registros en total. Cualquier KPI (%, promedios) debe mostrarse con el conteo base (`n=`) visible para evitar lecturas erróneas de indicadores calculados sobre muestras pequeñas.
6. **Campos de comentario abierto** con variantes de "sin respuesta" (`Na`, `N/A`, `Nada`, `Ninguna`, vacío) que deben unificarse si se van a analizar con nube de palabras o conteo de menciones.

---

## 4. Modelo de datos sugerido

### 4.1 Enfoque

Modelo en **estrella**, con 3 tablas de hechos (una por encuesta/instrumento, ya que tienen grano y periodicidad distintos) y dimensiones compartidas para permitir cruces transversales (call center, jornada, tiempo). Se recomienda **no fusionar las 3 encuestas en una sola tabla de hechos**, porque preguntas y escalas no son comparables entre sí y el grano de cada una es diferente (llamada vs. sesión de capacitación vs. actividad comercial).

### 4.2 Tablas de hechos

| Tabla | Grano (1 fila = ) | Origen |
|---|---|---|
| `Fact_CalidadLlamadas` | 1 llamada evaluada por el PUSHER | Matriz de calidad |
| `Fact_SatisfaccionCapacitacion` | 1 respuesta de encuesta de capacitación | Satisfacción capacitación |
| `Fact_MotivacionActividad` | 1 respuesta de encuesta de actividad comercial | Encuesta satisfacción/motivación |

Columnas técnicas recomendadas (post-limpieza) por tabla:

**`Fact_CalidadLlamadas`**
`FechaHora`, `Fecha` (calculada), `CallCenter`, `NombreAsesor`, `NombreLider`, `NombreAuditor`, `Preg_TonoSaludo`, `Preg_FraseImpacto`, `Preg_PreguntasNecesidad`, `Preg_ConexionBeneficio`, `Preg_ExplicacionProducto`, `Preg_ManejoObjeciones`, `Preg_CierreComercial`, `Preg_ConfirmacionCierre`, `Observaciones`, `ObjecionPrincipal`, `TerminoEnVenta` (booleano).

**`Fact_SatisfaccionCapacitacion`**
`FechaHora`, `Fecha`, `CallCenter`, `Jornada`, `NombreAsesor`, `NombreLider`, `NombreFormador`, `SatisfaccionGeneral`, `Claridad`, `Utilidad`, `Dinamismo`, `Duracion`, `Comentario`.

**`Fact_MotivacionActividad`**
`FechaHora`, `Fecha`, `CallCenter`, `Jornada`, `SatisfaccionGeneral`, `ClaridadUtilidad`, `MotivacionPostActividad`, `AmbienteEquipo`, `TipoActividadPreferida`, `Comentario`.

### 4.3 Tablas de dimensión

| Tabla | Contenido | Cómo se construye |
|---|---|---|
| `Dim_Calendario` | Fecha, Año, Mes, NombreMes, Trimestre, Semana ISO, DíaSemana, EsFinDeSemana | Tabla continua generada en Power Query o DAX (`CALENDAR`), desde `MIN(Fecha)` de las 3 fuentes hasta `HOY()`/`MAX(Fecha)` |
| `Dim_CallCenter` | Lista única de call centers | **Unión (Append) de los valores distintos de `CallCenter` de las 3 fact**, no lista manual, para que se auto-actualice si aparecen nuevos call centers |
| `Dim_Jornada` | `Mañana`, `Tarde` (y `Noche` si aparece a futuro) | Lista de valores distintos de las 2 fuentes que sí tienen jornada (capacitación y motivación) |

**Dimensiones de persona — recomendación:** no crear `Dim_Asesor` / `Dim_Lider` como tablas separadas en esta primera fase, dado que (a) no hay ID único, (b) los nombres tienen inconsistencias que deben resolverse primero con negocio, y (c) el volumen es muy bajo para justificar la complejidad. Se recomienda:
- Mantener `NombreAsesor` / `NombreLider` / `NombreFormador` como columnas normalizadas (Trim + Clean + Proper Case) dentro de cada fact.
- Aplicar una **tabla de mapeo manual de alias** en Power Query (`alias original` → `nombre estándar`) para los casos ya detectados (ej. las 4 variantes de "Juan Esteban Pérez Camargo").
- Evaluar en una fase 2, cuando el volumen crezca y/o exista un maestro de asesores real (ej. desde nómina/WFM), la creación de `Dim_Colaborador` con `Rol` (Asesor/Líder/Formador/Auditor-PUSHER) para unificar `Jeisy Martinez` (Auditor y Formador) en una sola entidad.

### 4.4 Relaciones sugeridas

```
Dim_Calendario[Fecha]  (1) ── (*) Fact_CalidadLlamadas[Fecha]
Dim_Calendario[Fecha]  (1) ── (*) Fact_SatisfaccionCapacitacion[Fecha]
Dim_Calendario[Fecha]  (1) ── (*) Fact_MotivacionActividad[Fecha]

Dim_CallCenter[CallCenter]  (1) ── (*) Fact_CalidadLlamadas[CallCenter]
Dim_CallCenter[CallCenter]  (1) ── (*) Fact_SatisfaccionCapacitacion[CallCenter]
Dim_CallCenter[CallCenter]  (1) ── (*) Fact_MotivacionActividad[CallCenter]

Dim_Jornada[Jornada]  (1) ── (*) Fact_SatisfaccionCapacitacion[Jornada]
Dim_Jornada[Jornada]  (1) ── (*) Fact_MotivacionActividad[Jornada]
```

Todas de una sola dirección (filtro único, `Single`), sin relaciones activas entre las 3 tablas de hechos entre sí (evita ambigüedad; los cruces entre encuestas, si se necesitan, se hacen vía las dimensiones compartidas). Nota: `Fact_CalidadLlamadas` no se relaciona con `Dim_Jornada` porque la fuente no captura ese dato (ver §3.1 y §6).

### 4.5 Convenciones técnicas de nombres

- Prefijo `Fact_` para hechos, `Dim_` para dimensiones (formato TMDL, consistente con lo que ya usa el modelo — actualmente vacío pero configurado en `es-ES`).
- Columnas en `PascalCase` sin espacios ni tildes en el **nombre técnico**, reservando tildes/espacios solo para el `displayName` o etiquetas visuales si se desea mostrar en español natural (Power BI permite nombre técnico limpio + alias visual en visuales).
- Medidas DAX organizadas en tablas de medidas separadas por área: `_Medidas Calidad`, `_Medidas Capacitacion`, `_Medidas Motivacion`, `_Medidas Generales` (prefijo `_` para que aparezcan primero en el panel de campos).

---

## 5. Indicadores (DAX) propuestos

Se listan a nivel de especificación funcional (nombre, propósito y boceto de lógica). La sintaxis final se ajustará al momento de construir el modelo.

### 5.1 Calidad de llamadas
- **Total Evaluaciones** = `COUNTROWS(Fact_CalidadLlamadas)`
- **Puntaje Obtenido** = suma de las 8 preguntas, tratando `"N/A"` como blanco (no como 0), vía `SUMX` con `IF(ISNUMBER(...))`
- **Puntaje Máximo Aplicable** = suma del máximo posible **solo** de las preguntas que no fueron `N/A` en cada fila (requiere tabla de rúbrica de puntaje máximo por pregunta — pendiente de confirmar con negocio, ver §6)
- **% Calidad Promedio** = `DIVIDE([Puntaje Obtenido], [Puntaje Máximo Aplicable])`
- **% Llamadas con Venta** = `DIVIDE(COUNTROWS(FILTER(Fact_CalidadLlamadas, [TerminoEnVenta] = "Sí")), [Total Evaluaciones])`
- **Objeción Principal (Top)** = medida basada en `TOPN(1, VALUES(ObjecionPrincipal), [Total Evaluaciones])` para tarjeta destacada
- **% Calidad por Call Center / Líder / Asesor / Auditor** = la misma medida `% Calidad Promedio` evaluada por contexto de fila (matriz/tabla), sin necesidad de duplicar código

### 5.2 Satisfacción de capacitaciones
- **Total Respuestas Capacitación** = `COUNTROWS(Fact_SatisfaccionCapacitacion)`
- **Satisfacción Promedio** = `AVERAGE(SatisfaccionGeneral)`
- **Claridad Promedio** = `AVERAGE(Claridad)`
- **Utilidad Promedio** = `AVERAGE(Utilidad)`
- **Dinamismo Promedio** = `AVERAGE(Dinamismo)`
- **Índice Global Capacitación** = promedio de las 4 medidas anteriores (indicador compuesto para tarjeta ejecutiva)
- Desgloses por `CallCenter`, `Jornada`, `NombreLider`, `NombreFormador` reutilizando las mismas medidas por contexto de visual.

### 5.3 Motivación / actividades comerciales
- **Total Respuestas Motivación** = `COUNTROWS(Fact_MotivacionActividad)`
- **Satisfacción Actividad Promedio** = `AVERAGE(SatisfaccionGeneral)`
- **Claridad/Utilidad Actividad Promedio** = `AVERAGE(ClaridadUtilidad)`
- **Motivación Promedio** = `AVERAGE(MotivacionPostActividad)`
- **% Ambiente Motivado** = `DIVIDE(COUNTROWS(FILTER(..., AmbienteEquipo="Motivado")), [Total Respuestas Motivación])`
- Desgloses por `CallCenter` y `Jornada` (sin desglose por asesor, ver limitación §3.3).

### 5.4 Generales / transversales
- **Cobertura de Evaluación** (si a futuro se cruza con headcount real de WFM): evaluaciones vs. plantilla de asesores.
- Medidas de **conteo base (`n=`)** para acompañar cada promedio/porcentaje en tarjetas (buena práctica dado el bajo volumen actual).
- Medida de **variación período anterior** usando `Dim_Calendario` (`Time Intelligence`, ya habilitado en el modelo) una vez haya suficiente histórico.

---

## 6. Páginas sugeridas del informe

| # | Página | Contenido clave |
|---|---|---|
| 1 | **Resumen ejecutivo** | Tarjetas KPI top (Total evaluaciones, % Calidad promedio, Satisfacción capacitación promedio, Motivación promedio, % llamadas con venta), tendencia general en el tiempo, mapa/resumen por call center. Incluir nota de "n=" por el bajo volumen. |
| 2 | **Calidad de llamadas** | % Calidad por pregunta del checklist, ranking de asesores/líderes, objeciones principales (gráfico de barras), % llamadas con venta, tabla de observaciones filtrable. |
| 3 | **Satisfacción de capacitaciones** | Promedios de las 4 dimensiones Likert, evolución en el tiempo, desglose por formador/líder/call center/jornada, nube de comentarios (con "sin respuesta" filtrado). |
| 4 | **Motivación y actividades comerciales** | Satisfacción/motivación promedio, distribución de "ambiente de equipo", actividades preferidas (ranking), evolución por call center/jornada. Nota visible: encuesta anónima, sin desglose por asesor. |
| 5 | **Detalle por call center / asesor** | Tabla matriz cruzando las 3 fuentes por call center (y por asesor donde aplique: calidad y capacitación), con drill-through desde las páginas anteriores. |

Se recomienda un **botón/página de notas metodológicas** (o tooltip fijo) documentando limitaciones de datos (volumen bajo, encuesta anónima, rúbrica de puntaje pendiente) para que los usuarios de negocio interpreten los KPIs correctamente mientras el volumen de datos crece.

---

## 7. Lineamientos visuales Connect Assistance

- **Paleta principal:** naranja Connect como color de acento/foco (KPIs destacados, barras principales, elementos interactivos), blanco como fondo base, negro/gris oscuro para texto y elementos secundarios. Usar el naranja con moderación (regla 60/30/10: neutros dominan, naranja resalta lo importante).
- **Tema:** construir un **tema JSON personalizado** de Power BI (reemplazando el `CY25SU11` por defecto que trae el reporte) con la paleta Connect, tipografía consistente y estilos de tarjeta uniformes — esto es un cambio de fase de construcción, no de este diagnóstico.
- **Logo Connect:** incluir en el encabezado de cada página (esquina superior, tamaño moderado) como elemento de marca, sin saturar el espacio de visuales.
- **Estilo general:** moderno, ejecutivo, "clean" — evitar bordes/sombras excesivas, mantener espaciado generoso, tipografía legible (Segoe UI o similar), tarjetas KPI con jerarquía visual clara (número grande, etiqueta pequeña, variación si aplica).
- **Accesibilidad:** contraste suficiente entre naranja y blanco/gris para texto (el naranja puro suele fallar contraste AA sobre blanco si se usa como texto; reservarlo para fondos de tarjeta con texto oscuro/blanco según luminancia, o para barras/iconos, no como color de fuente sobre fondo claro).
- **Evitar saturación:** máximo 4-6 visuales por página, uso de tooltips/drillthrough para detalle en vez de saturar la página principal con tablas extensas.

---

## 8. Riesgos y consideraciones

| Riesgo | Impacto | Mitigación sugerida |
|---|---|---|
| Volumen de datos muy bajo (3 / 32 / 5 filas) | KPIs no representativos; promedios volátiles con cada nueva respuesta | Mostrar `n=` junto a cada indicador; comunicar que el informe está en fase piloto |
| Nombres de líder/asesor con variantes (typos, tildes, mayúsculas) | Fragmentación de resultados por persona en visuales agrupados | Normalización en Power Query (Trim/Clean/Proper) + tabla de mapeo manual de alias; a futuro, maestro de personas con ID único |
| Escala de puntaje no uniforme y mezclada con `"N/A"` en Matriz de calidad | Cálculo de `% Calidad` incorrecto si `N/A` se trata como 0 | Confirmar rúbrica de puntaje máximo por pregunta con el área de calidad/PUSHER antes de programar la medida DAX |
| Encuesta de motivación 100% anónima (columna nombre vacía) | No se puede analizar motivación a nivel de asesor individual | Documentar la limitación en el informe; evaluar con negocio si se debe volver obligatoria en el formulario a futuro |
| Catálogo de Call Center no centralizado (ya 4 valores distintos entre 3 fuentes) | Riesgo de que aparezcan variantes de escritura del mismo call center a futuro (ej. "One Contact" vs "ONE CONTACT") | Normalizar a mayúsculas + Trim en Power Query; construir `Dim_CallCenter` por unión dinámica, no lista fija |
| Archivos Excel pueden estar abiertos/bloqueados por el usuario u OneDrive al momento de refrescar | Falla de actualización del modelo (`file in use`) | Cerrar los archivos antes de refrescar; considerar migrar la fuente a SharePoint/Forms conectado directo o a un flujo con Power Automate si el volumen crece |
| Archivos son exportaciones manuales de Google Forms (no hay conexión automática) | Actualización depende de reexportar manualmente | Evaluar a futuro conexión directa (Google Sheets vía conector, o exportación programada) para reducir fricción operativa |
| Respuestas abiertas con variantes de "sin comentario" (`Na`, `N/A`, `Nada`, vacío, emojis) | Ruido en análisis de texto/nube de palabras | Normalizar valores equivalentes a "Sin comentario" en Power Query antes de cualquier visual de texto |
| Ausencia de columna `Jornada` en Matriz de calidad | No se puede cruzar calidad de llamada por turno, a diferencia de las otras 2 encuestas | Confirmar con negocio si el formulario de calidad debe incluir jornada a futuro; documentar como gap conocido en el informe |
| `Jeisy Martinez` aparece como "Auditor/PUSHER" y como "Nombre formador" — probable misma persona en dos roles | Doble conteo o inconsistencia si se trata como 2 entidades distintas al consolidar | Confirmar con negocio; tratar como un solo colaborador con múltiples roles si se crea `Dim_Colaborador` en fase futura |

---

## 9. Plan de implementación recomendado

1. **Fase 0 — Validación con negocio** *(antes de tocar el PBIP)*
   - Confirmar rúbrica de puntaje máximo por pregunta en la Matriz de calidad.
   - Confirmar lista oficial de call centers y jornadas vigentes.
   - Confirmar si la encuesta de motivación debe capturar identificación del asesor a futuro.
   - Confirmar nombres estándar de líderes/formadores para la tabla de mapeo de alias.

2. **Fase 1 — Ingesta y limpieza (Power Query)**
   - Conectar las 3 fuentes desde `Data/` (parámetro de carpeta o rutas relativas al PBIP).
   - Renombrar columnas a nombres técnicos limpios (sin espacios/tildes en el nombre técnico).
   - Trim/Clean/Proper Case en campos de texto; normalizar `CallCenter` a mayúsculas.
   - Convertir columnas Likert y checklist a tipo numérico correcto; tratar `"N/A"` como `null`, no como texto ni como 0.
   - Extraer `Fecha` (date) desde `Timestamp` (datetime) en cada fact.
   - Aplicar tabla de mapeo de alias para líderes/asesores ya detectados.
   - Unificar variantes de "sin comentario" en campos abiertos.

3. **Fase 2 — Modelado**
   - Crear `Dim_Calendario`, `Dim_CallCenter` (por unión), `Dim_Jornada`.
   - Cargar las 3 `Fact_*` y establecer relaciones descritas en §4.4.
   - Definir formatos de columna (fecha, porcentaje, entero) y ocultar columnas técnicas no necesarias en visuales (IDs, columnas auxiliares).

4. **Fase 3 — Medidas DAX**
   - Construir las medidas de §5 en tablas de medidas separadas por área.
   - Validar `% Calidad Promedio` con casos de prueba manuales (los 3 registros actuales) antes de escalar.

5. **Fase 4 — Reporte y visual**
   - Crear tema JSON de marca Connect y aplicarlo al reporte.
   - Construir las 5 páginas propuestas en §6, con navegación consistente (botones/bookmarks).
   - Incorporar logo Connect y nota metodológica de limitaciones de datos.

6. **Fase 5 — Validación y entrega**
   - Revisar cálculos contra los datos fuente fila por fila (dado el bajo volumen, es viable una validación manual 100%).
   - Publicar y documentar el flujo de actualización (quién exporta los Forms, con qué frecuencia).

---

## 10. Próximos pasos

1. Validar este diagnóstico con el negocio (rúbrica de puntaje, catálogo de call centers, alias de líderes).
2. Autorizar el inicio de la **Fase 1 (Power Query)** sobre el PBIP existente.
3. Definir si se desea versionar el proyecto con Git (`git init` en `PBI_Indicadores/`) antes de empezar a construir, dado que actualmente no es un repositorio — recomendado para poder revisar diffs de TMDL/JSON a medida que se construya el modelo.
4. Solicitar (si existe) el logo oficial de Connect Assistance en formato vectorial/alta resolución y los códigos HEX exactos de la paleta de marca, para construir el tema JSON con precisión en la Fase 4.
5. Una vez validado lo anterior, proceder con la construcción del modelo semántico y del informe conforme al plan de la sección 9.

---

*Este documento es un diagnóstico. No se realizaron cambios en `PBI_Indicadores.pbip`, el modelo semántico, medidas, consultas ni visuales del reporte.*
