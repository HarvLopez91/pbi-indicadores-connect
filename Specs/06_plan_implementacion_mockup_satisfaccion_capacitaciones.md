# Plan de Implementación — Adaptar "Satisfacción de capacitaciones" al mockup

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Documento base | [`Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md`](05_analisis_impacto_mockup_satisfaccion_capacitaciones.md) |
| Mockup de referencia | [`Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png`](../Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png) |
| Tipo de documento | Plan de implementación por fases (fases + prompts ejecutables), aún **no ejecutado** |
| Fecha | 2026-07-21 |
| Estado | **Planeación** — ninguna fase de este plan se ha ejecutado todavía |

> Convención de numeración: para no confundir estas fases con las 18 fases históricas de [`Specs/02`](02_plan_implementacion_informe_powerbi_connect.md) (ya cerradas, ver [`Specs/03`](03_documentacion_final_informe_powerbi_connect.md)), las fases de este plan se identifican con el prefijo **`SC`** (Satisfacción de Capacitaciones): `SC-1` a `SC-9`. Ninguna fase `SC-*` se ejecuta como parte de la creación de este documento — este plan es solo la hoja de ruta.

---

## Índice

1. [Resumen ejecutivo](#1-resumen-ejecutivo)
2. [Alcance del plan](#2-alcance-del-plan)
3. [Supuestos](#3-supuestos)
4. [Decisiones previas requeridas](#4-decisiones-previas-requeridas)
5. [Fases de implementación](#5-fases-de-implementación)
6. [Índice de prompts por fase](#6-índice-de-prompts-por-fase)
7. [Orden recomendado de ejecución](#7-orden-recomendado-de-ejecución)
8. [Dependencias entre fases](#8-dependencias-entre-fases)
9. [Validaciones por fase (resumen)](#9-validaciones-por-fase-resumen)
10. [Riesgos consolidados](#10-riesgos-consolidados)
11. [Criterios para avanzar o detenerse](#11-criterios-para-avanzar-o-detenerse)
12. [Criterios de cierre](#12-criterios-de-cierre)

---

## 1. Resumen ejecutivo

Este plan traduce el diagnóstico de [`Specs/05`](05_analisis_impacto_mockup_satisfaccion_capacitaciones.md) en **9 fases secuenciales**, cada una con objetivo, actividades, restricciones, archivos potencialmente afectados, validaciones, riesgos, resultado esperado y un prompt de ejecución independiente.

El plan respeta las 3 conclusiones centrales del análisis de impacto: **(1)** la página `p14_satisfaccion_capacitaciones` no se edita en sitio — se trabaja sobre una copia; **(2)** 3 de los 6 indicadores del mockup dependen de una decisión de negocio aún no tomada (qué constituye "una capacitación única"), por lo que las medidas asociadas quedan condicionadas y no se crean si esa decisión sigue pendiente; **(3)** cualquier cambio de modelo (medidas nuevas) requiere autorización explícita del usuario antes de ejecutarse, conforme a la restricción vigente del proyecto de no crear medidas DAX sin aprobación.

Ninguna fase de este plan se ejecuta al crear este documento. Cada fase se activa solo cuando el usuario lo solicite explícitamente, en una sesión de trabajo separada, usando el prompt correspondiente de §5/§6.

## 2. Alcance del plan

**Incluido:**
- Confirmación de las 4 decisiones de negocio pendientes identificadas en `Specs/05`.
- Preparación técnica del entorno (Git, Power BI Desktop, protección de la página publicada).
- Creación condicionada de hasta 3 medidas DAX nuevas en `_Medidas Capacitacion` (2 sin bloqueo, 1 bloqueada por decisión de negocio, más sus posibles variantes derivadas si la decisión se resuelve).
- Duplicado de la página `p14_satisfaccion_capacitaciones` como copia de trabajo, sin tocar la original.
- Rediseño de la copia según el mockup: encabezado, filtros, KPI, gráficos, panel de selección cruzada, tabla de detalle, comentarios destacados, nota metodológica.
- Configuración de interacciones entre visuales de la copia.
- Validación técnica, funcional y visual de la copia.
- Actualización de `Docs/02`, `Docs/03`, `Docs/05` y bitácora en `Outputs/`.
- Decisión final de reemplazo/publicación, con su propia validación previa.

**Fuera de alcance de este plan** (requeriría un plan aparte si se decide abordarlo):
- Cualquier cambio a `Fact_CalidadLlamadas`, `Fact_MotivacionActividad` u otras páginas del reporte.
- Conexión automática a la fuente (Google Forms/Sheets).
- Cambios al catálogo de call centers/jornadas o a la tabla de alias de líderes/formadores (dependencias D4/D5, sin relación directa con este mockup).
- Migración del mecanismo de publicación (enlace público → workspace con control de acceso) — se documenta como consideración en `SC-9`, pero su decisión y ejecución exceden el alcance de esta página.

## 3. Supuestos

- Las 4 decisiones de negocio de §4 las resuelve el usuario/negocio, no la IA que ejecute las fases — ninguna fase puede "adivinar" estas respuestas.
- Power BI Desktop está disponible para las fases que requieren su interfaz gráfica (duplicar página, validar cálculo en vivo, interacciones, publicar). Las fases que solo editan TMDL/JSON a mano pueden ejecutarse sin abrir Desktop, pero **deben validarse abriéndolo después**, conforme a la limitación ya documentada en `CLAUDE.md` ("Claude no puede manejar la interfaz gráfica de Power BI Desktop directamente").
- El volumen de datos sigue siendo de fase piloto durante la ejecución de este plan — ninguna fase asume un crecimiento súbito de `Data/`.
- Cada fase requiere autorización explícita del usuario antes de ejecutarse; este documento no se autoejecuta ni se encadena automáticamente.
- Los nombres técnicos definitivos (medidas, página nueva) propuestos en este plan son una recomendación inicial; pueden ajustarse en el momento de ejecución sin invalidar el plan.

## 4. Decisiones previas requeridas

| # | Decisión | Opciones | Impacto si no se resuelve |
|---|---|---|---|
| DEC-1 | Definición de **"capacitación única"** (dependencia candidata D9 de `Specs/05` §7) | (a) Clave compuesta `Fecha + CallCenter + NombreFormador`; (b) otra combinación de campos; (c) se agrega un campo nuevo de identificación de sesión en el origen (Google Forms) para el futuro; (d) se descarta el concepto y el mockup se ajusta para mostrar solo "Respuestas" | Bloquea la medida `Capacitaciones Realizadas` y sus 2 variantes (por fecha, por call center) — 3 de los 6 indicadores solicitados originalmente. `SC-3` solo crea las medidas no bloqueadas si esta decisión sigue pendiente. |
| DEC-2 | ¿La tabla por call center **reemplaza** o **coexiste** con `sc_tabla_formador` (formador/líder)? | (a) Reemplaza — se pierde el desglose nominal por formador/líder en esta página; (b) coexiste — se agregan ambas tablas (requiere más espacio en el lienzo); (c) el desglose por formador se traslada a un tooltip/drillthrough | Sin esta decisión, `SC-5` no puede definir el layout final de la fila de tabla/comentarios ni saber si se reduce la exposición de nombres reales (ver `Docs/06` §2). |
| DEC-3 | ¿El gráfico "Índice global por jornada" (`sc_chart_jornada`) se **conserva, se elimina o se reubica**? | (a) Se elimina de esta página (el mockup no lo incluye); (b) se conserva en una fila adicional/scroll; (c) se reubica como tooltip o página de detalle | Sin esta decisión, `SC-5` no puede decidir si la copia de página conserva o descarta ese visual. |
| DEC-4 | ¿El segmentador de Fecha en formato de **rango con 2 casillas** aplica solo a esta página o se extiende a las 7? | (a) Solo a esta página (rompe la consistencia de `Docs/05` §1.11: "16 segmentadores en modo Dropdown"); (b) se extiende a las 7 páginas en una iniciativa aparte; (c) se mantiene el modo `Dropdown` actual y el mockup se ajusta | Sin esta decisión, `SC-5` no puede decidir el modo del segmentador de Fecha de la copia sin crear una inconsistencia de diseño no aprobada. |

Estas 4 decisiones se capturan formalmente en `SC-1` y se usan como entrada de `SC-3` (medidas) y `SC-5` (rediseño visual). No es obligatorio resolver las 4 para avanzar de `SC-1` a `SC-2` — ver criterios de avance en §11.

> **Corrección de DEC-4 (post `SC-4`):** en `SC-1` se confirmó inicialmente "mantener el modo `Dropdown` actual" (opción c), bajo el supuesto de que el segmentador de Fecha se renderizaba como lista desplegable, igual que las otras 6 páginas. La validación en Power BI Desktop durante `SC-4` (`Outputs/40` §10) mostró que el segmentador **ya se renderiza como rango con 2 casillas de calendario (`Between`)**, tanto en la copia como en la página original — la propiedad PBIR `mode: Dropdown` no controla el estilo de un segmentador sobre una columna de Fecha continua; Power BI Desktop aplica `Between` por defecto independientemente de ese valor. **DEC-4 queda resuelta como:** *"Se conserva el comportamiento actual del segmentador de Fecha en modo `Between`, únicamente en la página de Satisfacción de capacitaciones. No se requiere modificarlo durante `SC-5`."* El resultado práctico coincide con la intención original de la respuesta de `SC-1` (no introducir un cambio nuevo) y además coincide con el mockup de referencia, que muestra el mismo estilo de rango. Esta corrección se limita a esta página — no se investigó si las otras 6 páginas del reporte también renderizan su segmentador de Fecha como `Between`, lo cual quedaría fuera del alcance de este plan.

---

## 5. Fases de implementación

### Fase SC-1 — Confirmación de decisiones de negocio

**1. Objetivo:** Obtener del usuario/negocio una respuesta explícita a las 4 decisiones de §4, documentarlas, y determinar qué partes del plan quedan desbloqueadas o siguen condicionadas.

**2. Actividades:**
- Presentar las 4 decisiones de §4 al usuario/negocio en un formato claro (esta misma tabla sirve de base).
- Registrar la respuesta de cada una, incluyendo la fecha y quién la confirmó.
- Si DEC-1 no se resuelve, confirmar explícitamente con el usuario si el plan continúa solo con los indicadores no bloqueados (Última capacitación, Satisfacción por call center seleccionado, Comentarios destacados, Call centers capacitados) o si se pausa el plan completo.
- No proponer un valor "por defecto" para ninguna decisión sin marcarlo explícitamente como supuesto no confirmado.

**3. Restricciones:**
- No modificar ningún archivo del proyecto (`PBI/`, `Docs/`, `Data/`) en esta fase — es una fase de recolección de decisiones, no de documentación formal (eso ocurre en `SC-8`).
- No asumir respuestas en nombre del usuario/negocio.

**4. Archivos que podrían modificarse:** Ninguno. El resultado de esta fase es un registro de decisiones (puede vivir como una nota temporal de la conversación) que se formaliza en `Docs/05` durante `SC-8`.

**5. Validaciones:**
- [ ] Las 4 decisiones tienen una respuesta explícita, o una decisión explícita de "queda pendiente, se avanza solo con lo no bloqueado".
- [ ] Ninguna decisión quedó inferida o asumida sin confirmación directa del usuario.

**6. Riesgos:**
- Avanzar sin resolver DEC-1 y luego intentar forzar la medida `Capacitaciones Realizadas` de todos modos (violaría la restricción de no crear medidas basadas en supuestos no confirmados).
- Que la respuesta a DEC-2/DEC-3 cambie a mitad del rediseño (`SC-5`) — mitigar cerrando esta fase antes de iniciar `SC-4`.

**7. Resultado esperado:** Registro explícito de las 4 decisiones (o de su estado "pendiente, se avanza parcialmente"), listo para alimentar `SC-3` y `SC-5`.

**8. Prompt de ejecución:**
```
Actúa como especialista en planificación de producto de datos / Power BI.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md
(secciones 5 y 7) y Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
sección 4 (Decisiones previas requeridas).

Tarea: NO modifiques ningún archivo del proyecto. Preséntale al usuario, de
forma clara y una por una, las 4 decisiones pendientes documentadas en
Specs/06 sección 4 (DEC-1 a DEC-4), incluyendo sus opciones y el impacto de
no resolverlas. Registra la respuesta textual del usuario para cada una.

Si el usuario no puede resolver DEC-1 (definición de "capacitación única")
en esta sesión, pregúntale explícitamente si el plan debe continuar solo
con los indicadores no bloqueados (Última capacitación, Satisfacción por
call center seleccionado, Comentarios destacados, Call Centers Capacitados)
o si prefiere pausar el plan completo hasta resolverla.

Al finalizar, resume las 4 decisiones tomadas (o su estado pendiente) en un
formato que pueda copiarse directamente a Docs/05_decisiones_limitaciones_pendientes.md
en una fase posterior (SC-8). No edites Docs/05 todavía.
```

---

### Fase SC-2 — Preparación técnica

**1. Objetivo:** Verificar que el entorno de trabajo está en un estado limpio y seguro antes de tocar cualquier archivo del PBIP, y confirmar que la página publicada actual queda protegida durante todo el plan.

**2. Actividades:**
- Ejecutar `git status` y `git branch --show-current`; confirmar rama `main` (o la rama de trabajo que el usuario indique) y working tree limpio salvo `Data/` (excluida por diseño).
- Si `git status` muestra cambios pendientes de una sesión previa de Power BI Desktop (metadatos, `lineageTag`, orden de página activa), comitearlos por separado en un commit `chore(report):` **antes** de iniciar cualquier cambio intencional de este plan, siguiendo el patrón ya establecido (`CLAUDE.md`, precedente commit `97be1a4`).
- Confirmar que `p14_satisfaccion_capacitaciones` no se tocará directamente en ninguna fase posterior salvo `SC-9` (y solo si se decide reemplazo).
- Confirmar acceso a Power BI Desktop y que los 3 archivos de `Data/` están cerrados (mismo checklist operativo de `Docs/04`).

**3. Restricciones:**
- No crear la copia de página todavía (eso es `SC-4`).
- No crear medidas DAX todavía (eso es `SC-3`).
- No hacer push remoto.

**4. Archivos que podrían modificarse:** Ninguno intencional. Si existen cambios automáticos de Power BI Desktop pendientes de una sesión previa, podrían comitearse archivos ya modificados por Desktop (p. ej. `pages.json`, algún `.tmdl` con `lineageTag` regenerado) — siempre en un commit separado, nunca mezclado con cambios de este plan.

**5. Validaciones:**
- [ ] `git status` limpio (salvo `Data/`) antes de iniciar `SC-3`.
- [ ] Rama de trabajo confirmada con el usuario.
- [ ] Cualquier cambio automático de Desktop de una sesión previa quedó comiteado por separado, con diff revisado (no asumido).
- [ ] Confirmación explícita de que `p14_satisfaccion_capacitaciones` no se modificará hasta `SC-9`.

**6. Riesgos:**
- Mezclar cambios automáticos de Desktop con los cambios intencionales de `SC-3`/`SC-4` si no se revisa `git status` con cuidado antes de empezar.
- Iniciar el plan sobre una rama incorrecta o con cambios sin comitear de otra iniciativa en curso.

**7. Resultado esperado:** Entorno limpio, rama confirmada, página original protegida explícitamente, listo para iniciar cambios de modelo.

**8. Prompt de ejecución:**
```
Actúa como especialista en control de versiones para proyectos Power BI/PBIP.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores

Tarea: ejecuta una verificación de preparación, SIN modificar ningún archivo
de PBI/ ni Data/ todavía.

1. Ejecuta git status y git branch --show-current. Reporta el estado exacto.
2. Si hay cambios pendientes que correspondan a una reescritura automática
   de Power BI Desktop (metadatos, lineageTag, orden de página activa en
   pages.json) de una sesión previa NO relacionada con este plan, coméntalo
   al usuario y, si confirma, comitéalos en un commit separado con prefijo
   chore(report): antes de continuar. No mezcles esos cambios con nada de
   este plan.
3. Confirma con el usuario la rama de trabajo a usar para este plan.
4. Confirma explícitamente: la página p14_satisfaccion_capacitaciones NO se
   modifica en ninguna fase de este plan salvo SC-9, y solo si el usuario
   autoriza el reemplazo en ese momento.
5. Recuerda al usuario el checklist operativo de Docs/04_fuentes_y_actualizacion_datos.md
   si va a abrir Power BI Desktop (archivos de Data/ cerrados).

No crees la copia de página ni medidas DAX en esta fase.
```

---

### Fase SC-3 — Creación de medidas DAX nuevas (condicionada)

**1. Objetivo:** Crear en `_Medidas Capacitacion` únicamente las medidas DAX que no dependen de DEC-1 sin resolver, y las medidas dependientes de DEC-1 solo si esa decisión ya fue confirmada en `SC-1`.

**2. Actividades:**
- Verificar el resultado de `SC-1` para DEC-1 antes de escribir cualquier fórmula.
- Crear siempre (no bloqueadas):
  - `Call Centers Capacitados = DISTINCTCOUNT(Fact_SatisfaccionCapacitacion[CallCenter])`
  - `Ultima Capacitacion = MAX(Fact_SatisfaccionCapacitacion[Fecha])`
- Crear **solo si DEC-1 fue confirmada**, con la clave compuesta exacta que el negocio definió (ejemplo de referencia, a ajustar según DEC-1):
  - `Capacitaciones Realizadas = DISTINCTCOUNTX(Fact_SatisfaccionCapacitacion, Fact_SatisfaccionCapacitacion[Fecha] & "|" & Fact_SatisfaccionCapacitacion[CallCenter] & "|" & Fact_SatisfaccionCapacitacion[NombreFormador])` (o la combinación de campos que DEC-1 haya definido)
  - Evaluar si "Capacitaciones por fecha" y "Capacitaciones por call center" requieren medidas separadas o si basta reutilizar `Capacitaciones Realizadas` en el eje correspondiente (recomendado: reutilizar, ver `Specs/05` §6).
- Aplicar formato de número (entero `0` para conteos, fecha corta para `Ultima Capacitacion`).
- No escribir `lineageTag`, `description` ni `queryGroup` a mano — dejar que Power BI Desktop los genere al guardar (convención de `CLAUDE.md`).
- Validar cada medida nueva en una tarjeta temporal contra un cálculo manual antes de darla por terminada.

**3. Restricciones:**
- No crear `Capacitaciones Realizadas` (ni sus variantes) si DEC-1 sigue sin resolver — dejarlas explícitamente pendientes, documentadas como bloqueadas.
- No modificar ninguna medida existente de las 25 ya catalogadas.
- No tocar `Fact_CalidadLlamadas`, `Fact_MotivacionActividad` ni sus medidas.
- No modificar Power Query ni las relaciones del modelo.

**4. Archivos que podrían modificarse:**
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl` (única tabla afectada).

**5. Validaciones:**
- [ ] `Call Centers Capacitados` y `Ultima Capacitacion` calculan correctamente contra un cálculo manual sobre los datos actuales de `Data/`.
- [ ] Si se creó `Capacitaciones Realizadas`, su valor coincide con el conteo manual de combinaciones únicas según la clave definida en DEC-1.
- [ ] Ninguna medida nueva tiene `lineageTag`/`description`/`queryGroup` escrito a mano.
- [ ] El archivo `.tmdl` abre sin error de sintaxis en Power BI Desktop.
- [ ] Las 25 medidas preexistentes siguen calculando igual que antes (sin regresión).

**6. Riesgos:**
- Si `NombreFormador` u otro campo de la clave compuesta tiene variantes de escritura no normalizadas (ver dependencia D5), `Capacitaciones Realizadas` podría sobreestimar el conteo de sesiones — advertir esto explícitamente si se implementa.
- Romper el analizador TMDL de Power BI Desktop si se escribe `lineageTag` a mano (riesgo ya documentado 3 veces en el historial del proyecto, `Outputs/10`).

**7. Resultado esperado:** 2 medidas nuevas siempre creadas y validadas; 1 medida adicional (con sus posibles variantes) creada solo si DEC-1 se resolvió, también validada; ninguna medida bloqueada creada como supuesto no confirmado.

**8. Prompt de ejecución:**
```
Actúa como especialista en DAX para Power BI/PBIP.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md
sección 6, y Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
fase SC-3.

Tarea: sobre la tabla de medidas _Medidas Capacitacion (archivo
PBI/PBI_Indicadores.SemanticModel/definition/tables/_Medidas Capacitacion.tmdl),
crea las siguientes medidas SOLO si corresponde:

1. SIEMPRE crea:
   - Call Centers Capacitados = DISTINCTCOUNT(Fact_SatisfaccionCapacitacion[CallCenter])
   - Ultima Capacitacion = MAX(Fact_SatisfaccionCapacitacion[Fecha])

2. Verifica primero el resultado de la Fase SC-1 para la decisión DEC-1
   (definición de "capacitación única"). Si DEC-1 NO fue confirmada por el
   usuario, NO crees ninguna medida de conteo de capacitaciones/sesiones -
   repórtalo como bloqueado y detente ahí.

3. Si DEC-1 SÍ fue confirmada, crea Capacitaciones Realizadas usando
   DISTINCTCOUNTX sobre la clave compuesta exacta que el usuario definió en
   DEC-1 (no inventes la combinación de campos). Evalúa si "Capacitaciones
   por fecha" y "Capacitaciones por call center" pueden reutilizar esta
   misma medida en el eje correspondiente, en vez de crear 2 medidas
   adicionales.

Reglas obligatorias:
- No escribas lineageTag, description ni queryGroup a mano en el bloque TMDL
  nuevo - dejalos vacíos para que Power BI Desktop los genere al guardar.
- No modifiques ninguna de las 25 medidas ya existentes en el modelo.
- No toques Fact_CalidadLlamadas, Fact_MotivacionActividad, Power Query ni
  relaciones.
- Aplica formatString entero (0) a los conteos y fecha corta a Ultima Capacitacion.

Al terminar, reporta el valor de cada medida nueva calculado por el modelo y
compáralo contra un cálculo manual sobre los datos actuales, antes de dar la
fase por cerrada.
```

---

### Fase SC-4 — Creación de copia de página

**1. Objetivo:** Duplicar `p14_satisfaccion_capacitaciones` en una página de trabajo independiente, sin modificar ni enlazar todavía la original.

**2. Actividades:**
- Duplicar la página `p14_satisfaccion_capacitaciones` completa (todos sus 29 visuales) en Power BI Desktop (clic derecho → Duplicar página), o replicar la estructura equivalente a mano en PBIR si se hace por archivo.
- Asignar nombre técnico nuevo, por ejemplo `p14_satisfaccion_capacitaciones_v2`, y `displayName` de trabajo, por ejemplo `"Satisfacción de capacitaciones (v2 — borrador)"`, para que sea inequívoco que es un prototipo.
- **No** enlazar esta página nueva a la navegación de Home ni a ningún botón "Volver a Home" con destino distinto al ya existente — debe quedar como página huérfana de navegación mientras se prototipa (accesible solo desde el panel de páginas de Power BI Desktop).
- Confirmar que `p14_satisfaccion_capacitaciones` (original) sigue exactamente igual, byte a byte, tras la duplicación.

**3. Restricciones:**
- No modificar ningún visual de `p14_satisfaccion_capacitaciones` (original).
- No modificar la navegación de Home ni de ninguna otra página en esta fase.
- No aplicar todavía el rediseño del mockup (eso es `SC-5`) — la copia debe quedar idéntica a la original justo después de duplicar.

**4. Archivos que podrían modificarse:**
- `PBI/PBI_Indicadores.Report/definition/pages/pages.json` (nueva entrada de página en el orden de páginas).
- Nueva carpeta `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/` con su `page.json` y los 29 `visual.json` duplicados.
- Ningún archivo dentro de `pages/p14_satisfaccion_capacitaciones/` (original).

**5. Validaciones:**
- [ ] `p14_satisfaccion_capacitaciones` (original) no tiene ningún cambio en `git diff` tras esta fase.
- [ ] La copia nueva abre sin error en Power BI Desktop y muestra los mismos datos/visuales que la original en el momento de duplicar.
- [ ] La copia no está enlazada desde Home ni desde ningún botón de navegación existente.
- [ ] El nombre técnico de la copia no colisiona con ningún `name` de página ya existente en `pages.json`.

**6. Riesgos:**
- Que Power BI Desktop, al duplicar, regenere `lineageTag` en visuales de la copia (comportamiento esperado y aceptado — Desktop los genera, no se escriben a mano) — revisar que esto no se filtre como cambio en la página original.
- Dejar la copia accesible accidentalmente por un usuario final si por error queda enlazada desde algún visual existente.

**7. Resultado esperado:** Página de trabajo `p14_satisfaccion_capacitaciones_v2` creada, idéntica a la original en su contenido inicial, sin ningún cambio en la página publicada ni en su navegación.

**8. Prompt de ejecución:**
```
Actúa como especialista en PBIR/Power BI Desktop.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
fase SC-4.

Tarea: duplica la página p14_satisfaccion_capacitaciones (carpeta
PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones/)
en una página de trabajo nueva.

1. Nombre técnico sugerido: p14_satisfaccion_capacitaciones_v2. DisplayName
   sugerido: "Satisfacción de capacitaciones (v2 - borrador)".
2. La copia debe incluir los 29 visuales de la página original, sin cambios
   de contenido en este momento (el rediseño ocurre en la fase SC-5, no aquí).
3. NO enlaces esta página nueva desde Home ni desde ningún botón de
   navegación existente - debe quedar accesible solo desde el panel de
   páginas.
4. Verifica con git diff que la carpeta pages/p14_satisfaccion_capacitaciones/
   (original) no tiene NINGÚN cambio tras esta operación.
5. Verifica que pages.json registra la página nueva sin romper el orden ni
   la página activa actual.

No apliques todavía ningún cambio de diseño del mockup. No toques Home ni
ninguna otra página del reporte.
```

---

### Fase SC-5 — Rediseño visual según el mockup

**1. Objetivo:** Adaptar el contenido visual de `p14_satisfaccion_capacitaciones_v2` al mockup, usando las decisiones de `SC-1` y las medidas de `SC-3`.

**2. Actividades** (una por bloque del mockup, ver `Specs/05` §3–§4):
- **Encabezado**: conservar logo, título, subtítulo, insignia "Datos piloto sujetos a validación" y botón "Volver a Home" (apuntando a Home, igual que la original) — sin cambios de fondo.
- **Filtros**: no se requiere ninguna acción sobre el segmentador de Fecha — DEC-4 (corregida, ver §4) confirma que ya funciona en modo `Between` (rango de 2 casillas), igual que el mockup; conservar Call Center y Jornada en modo `Dropdown`.
- **Tarjetas KPI**: reconstruir la fila de 6 tarjetas del mockup (Capacitaciones realizadas [solo si DEC-1 resuelta], Respuestas recibidas, Call centers capacitados, Satisfacción promedio, Última capacitación, % con comentarios), retirando las tarjetas de Claridad/Utilidad/Dinamismo/Índice de la fila superior (se reubican en el panel de selección y en la tabla).
- **Gráfico "Capacitaciones por call center"**: si DEC-1 resuelta, retargetear `sc_chart_callcenter` (o su copia) para usar `Capacitaciones Realizadas` en vez de `Indice Global Capacitacion`; si DEC-1 no resuelta, mantener este gráfico fuera de alcance y documentarlo como pendiente.
- **Gráfico "Capacitaciones por fecha"**: nuevo gráfico de líneas, mismo criterio que el anterior (bloqueado por DEC-1 si no está resuelta).
- **Panel "Satisfacción del call center seleccionado"**: nuevo visual (tarjetas o barras horizontales) con `Satisfaccion Promedio Capacitacion`, `Claridad Promedio Capacitacion`, `Utilidad Promedio Capacitacion`, `Dinamismo Promedio Capacitacion` y `Total Respuestas Capacitacion` — no requiere medida nueva.
- **Tabla de detalle**: aplicar la decisión DEC-2 — reemplazar o coexistir con `sc_tabla_formador`; columnas Call Center, Capacitaciones (si aplica), Respuestas, Satisfacción, Claridad, Utilidad, Dinamismo, Última fecha.
- **Comentarios destacados**: nueva tabla/lista sobre `Comentario`, `CallCenter`, `Fecha`, con filtro de visual `Comentario <> "Sin comentario"`.
- **Gráfico "Índice global por jornada"**: aplicar la decisión DEC-3 (conservar, eliminar o reubicar).
- **Nota metodológica**: conservar el mensaje de muestra piloto, ajustando el texto si cambian los indicadores mostrados.

**3. Restricciones:**
- Todo el trabajo ocurre exclusivamente en `p14_satisfaccion_capacitaciones_v2` — cero cambios en la página original.
- No usar colores fuera del tema `Assets/theme/connect_assistance_theme.json` (naranja `#F15B2B`, oscuro `#002733`, grises de apoyo) — sin hardcodear HEX nuevos por visual.
- No crear medidas DAX adicionales a las ya definidas en `SC-3` — si el rediseño revela la necesidad de una medida no prevista, se documenta como pendiente para una fase de autorización separada, no se crea sobre la marcha.
- No implementar visuales para indicadores bloqueados por DEC-1 sin resolver (dejar el espacio vacío o con una nota "pendiente de definición de negocio", no un valor inventado).

**4. Archivos que podrían modificarse:**
- Todos los `visual.json` dentro de `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/visuals/` (edición y creación de nuevos visuales).
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json` si cambia el tamaño/layout general.
- Ningún archivo de `p14_satisfaccion_capacitaciones/` (original), `Home`, ni de otras páginas.

**5. Validaciones:**
- [ ] Cada visual nuevo/ajustado usa exclusivamente colores del tema Connect.
- [ ] Ningún visual de la copia referencia una medida que no existe en el modelo.
- [ ] Los indicadores bloqueados por DEC-1 (si sigue sin resolver) no aparecen con datos inventados ni con la medida `Indice Global Capacitacion` disfrazada de conteo.
- [ ] El layout de la copia cabe en el lienzo 1280×720 sin recortes, igual que las demás páginas.
- [ ] La tabla de comentarios destacados no muestra ninguna fila con `"Sin comentario"`.

**6. Riesgos:**
- Sobrecargar el lienzo si DEC-2 resulta en "coexisten ambas tablas" (formador/líder + call center) — puede requerir ajustar tamaños o mover la tabla de comentarios a una posición secundaria.
- ~~Inconsistencia visual entre esta página y las otras 6 si el segmentador de Fecha cambia de modo (DEC-4) y las demás páginas no~~ — **riesgo descartado**: DEC-4 corregida confirma que no se modifica el segmentador (ya funciona en `Between`); no hay cambio que introduzca inconsistencia nueva. Queda abierta, fuera de alcance de este plan, la pregunta de si las otras 6 páginas también renderizan su Fecha como `Between` (ver `Specs/06` §4, nota de corrección de DEC-4).
- Reutilizar `sc_tabla_formador` como base de la nueva tabla de call center puede arrastrar configuraciones de columna que ya no aplican (revisar `columnFormatting` heredado).

**7. Resultado esperado:** `p14_satisfaccion_capacitaciones_v2` visualmente alineada al mockup (en la medida en que las decisiones de negocio lo permitan), con los indicadores bloqueados claramente señalados como pendientes en vez de omitidos silenciosamente.

**8. Prompt de ejecución:**
```
Actúa como especialista en UX/UI y PBIR para Power BI.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png,
Specs/05_analisis_impacto_mockup_satisfaccion_capacitaciones.md (secciones 3
y 4), Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md fase
SC-5, y el resultado de las fases SC-1 y SC-3.

Tarea: rediseña ÚNICAMENTE la página p14_satisfaccion_capacitaciones_v2
(la copia creada en SC-4) para que se parezca al mockup, bloque por bloque:

1. Encabezado: conserva logo, título, subtítulo, insignia de datos piloto y
   botón Volver a Home (sin cambios de fondo).
2. Filtros: no toques el segmentador de Fecha (DEC-4 corregida: ya funciona
   en modo Between, coincide con el mockup, no requiere cambio); conserva
   Call Center y Jornada en modo Dropdown.
3. Fila de KPI: reconstruye las 6 tarjetas del mockup. Si DEC-1 NO fue
   resuelta en SC-1, omite la tarjeta "Capacitaciones realizadas" y dejala
   marcada visualmente como "Pendiente de definición de negocio" en vez de
   mostrar un valor.
4. Gráficos "Capacitaciones por call center" y "Capacitaciones por fecha":
   constrúyelos SOLO si DEC-1 fue resuelta y la medida Capacitaciones
   Realizadas existe (creada en SC-3). Si no, deja el espacio con una nota
   de texto "Pendiente de definición de negocio (ver Specs/05 seccion 7)".
5. Panel "Satisfacción del call center seleccionado": constrúyelo con las
   medidas Satisfaccion/Claridad/Utilidad/Dinamismo Promedio Capacitacion y
   Total Respuestas Capacitacion (ya existentes, sin DAX nuevo).
6. Tabla de detalle: aplica la decisión DEC-2 (reemplaza o coexiste con
   sc_tabla_formador).
7. Tabla "Comentarios destacados": constrúyela sobre las columnas Comentario/
   CallCenter/Fecha existentes, con un filtro de visual que excluya el valor
   "Sin comentario".
8. Gráfico de jornada: aplica la decisión DEC-3.
9. Nota metodológica: conserva el mensaje de muestra piloto, ajusta el texto
   si cambia el conjunto de indicadores mostrados.

Usa exclusivamente los colores del tema en Assets/theme/connect_assistance_theme.json
(no hardcodees HEX nuevos). NO modifiques p14_satisfaccion_capacitaciones
(original), Home, ni ninguna otra página. NO crees medidas DAX nuevas fuera
de las ya creadas en SC-3.
```

---

### Fase SC-6 — Configuración de interacciones

**1. Objetivo:** Configurar el comportamiento de cruce entre visuales de `p14_satisfaccion_capacitaciones_v2` para que coincida con el mockup, y confirmar que la navegación hacia Home sigue intacta.

**2. Actividades:**
- Configurar que un clic en una barra del gráfico "Capacitaciones por call center" (o el visual equivalente que haya quedado tras `SC-5`) filtre/resalte el panel "Satisfacción del call center seleccionado" — usando la interacción de visuales nativa de Power BI (`Filter`/`Highlight`), no una medida DAX.
- Revisar las interacciones cruzadas entre: gráfico por call center ↔ panel de selección, tabla de detalle ↔ comentarios destacados, segmentadores ↔ todos los visuales de la página.
- Confirmar que ningún visual nuevo interfiere con la interacción de los segmentadores existentes (Fecha, Call Center, Jornada).
- Confirmar que el botón "Volver a Home" y su zona clicable ("hitzone") siguen apuntando correctamente a Home, sin cambios respecto al patrón usado en el resto del reporte (`Outputs/28`).

**3. Restricciones:**
- Solo se configuran propiedades de interacción de visuales (`visualInteractions` en `page.json`, o edición vía el panel "Editar interacciones" de Power BI Desktop) — no se crean medidas DAX nuevas en esta fase.
- No modificar la navegación de Home ni de otras páginas.

**4. Archivos que podrían modificarse:**
- `PBI/PBI_Indicadores.Report/definition/pages/p14_satisfaccion_capacitaciones_v2/page.json` (bloque de interacciones entre visuales, si aplica).
- `visual.json` de los visuales involucrados en la interacción (selección única, comportamiento de resaltado).

**5. Validaciones:**
- [ ] Seleccionar un call center en el gráfico correspondiente actualiza el panel "Satisfacción del call center seleccionado" con los valores correctos de ese call center.
- [ ] Deseleccionar (clic en espacio vacío) regresa el panel al estado general (todos los call centers).
- [ ] Los 3 segmentadores siguen filtrando correctamente todos los visuales de la página, incluidos los nuevos.
- [ ] El botón "Volver a Home" navega correctamente a Home desde la copia.

**6. Riesgos:**
- Si el gráfico de origen permite selección múltiple, el panel podría mostrar un agregado combinado en vez de "el call center seleccionado" — puede requerir configurar selección única en ese visual.
- Interacciones no configuradas explícitamente pueden heredar comportamiento por defecto de Power BI (filtrado cruzado en todos los visuales), lo cual puede no coincidir con el mockup si el panel debe reaccionar solo al gráfico específico y no a la tabla.

**7. Resultado esperado:** Interacciones de la copia funcionando como en el mockup (selección → panel reactivo), sin romper los segmentadores ni la navegación hacia Home.

**8. Prompt de ejecución:**
```
Actúa como especialista en interacción de visuales Power BI (PBIR).
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
fase SC-6.

Tarea: sobre la página p14_satisfaccion_capacitaciones_v2 (ya rediseñada en
SC-5), configura las interacciones entre visuales:

1. Un clic en el gráfico "Capacitaciones por call center" (o el visual
   equivalente) debe actualizar el panel "Satisfacción del call center
   seleccionado" con los valores de ese call center. Si el gráfico permite
   selección múltiple y eso no es lo esperado, configura selección única.
2. Revisa que los 3 segmentadores (Fecha, Call Center, Jornada) sigan
   filtrando correctamente TODOS los visuales de la página, incluidos los
   nuevos (gráficos, panel, tabla de detalle, comentarios destacados).
3. Confirma que el botón "Volver a Home" y su zona clicable siguen apuntando
   a Home sin cambios, igual que en el resto del reporte.
4. No uses DAX para lograr el comportamiento de selección - usa el panel de
   interacción de visuales nativo de Power BI.

NO modifiques Home ni ninguna otra página. NO toques p14_satisfaccion_capacitaciones
(original).
```

---

### Fase SC-7 — Validación técnica, funcional y visual

**1. Objetivo:** Confirmar que `p14_satisfaccion_capacitaciones_v2` funciona correctamente antes de documentar (`SC-8`) o decidir su reemplazo (`SC-9`).

**2. Actividades:**
- **PBIR**: revisar que todos los `visual.json` de la copia son JSON válido, sin referencias a campos/medidas inexistentes.
- **TMDL** (si `SC-3` creó medidas): revisar que `_Medidas Capacitacion.tmdl` abre sin error de sintaxis y sin `lineageTag`/`description`/`queryGroup` escritos a mano.
- **Medidas en Power BI Desktop**: abrir el `.pbip`, confirmar que el modelo carga sin advertencias, y que cada medida nueva calcula en vivo (tarjeta de prueba) con el valor esperado.
- **Filtros**: probar los 3 segmentadores de la copia en distintas combinaciones y confirmar que todos los visuales (incluidos los nuevos) responden.
- **Navegación**: confirmar que "Volver a Home" funciona desde la copia y que Home sigue navegando correctamente hacia la página original (no hacia la copia).
- **Textos corruptos**: buscar el patrón típico de tildes corrompidas (mojibake) en los textos nuevos de la copia (títulos de visuales, notas), siguiendo el mismo método ya usado en `Outputs/32` §4.
- **No regresión**: confirmar que `p14_satisfaccion_capacitaciones` (original) y las demás 6 páginas no presentan ningún cambio no intencional.

**3. Restricciones:**
- Esta fase es de validación — no se corrigen errores "sobre la marcha" sin reportarlos primero; si se detecta un error, se documenta y se decide con el usuario si se corrige en `SC-5`/`SC-6` (retroceder) o se acepta como limitación conocida.
- No se publica ni se reemplaza la página original en esta fase (eso es `SC-9`).

**4. Archivos que podrían modificarse:** Ninguno directamente — esta fase es de lectura/verificación. Si se detectan errores y se autoriza corregirlos de inmediato, los archivos afectados serían los mismos de `SC-3`/`SC-5`/`SC-6` (medidas o visuales de la copia).

**5. Validaciones:**
- [ ] El `.pbip` abre sin errores de validación del modelo.
- [ ] Cada medida nueva calcula el valor esperado en una tarjeta de prueba.
- [ ] Los 3 segmentadores de la copia filtran correctamente todos sus visuales.
- [ ] La navegación Home ↔ copia ↔ Home (o Home ↔ original, según corresponda) funciona sin enlaces rotos.
- [ ] Cero coincidencias de mojibake en los textos nuevos.
- [ ] `git diff` confirma que ninguna página distinta a la copia (`p14_satisfaccion_capacitaciones_v2`) cambió durante `SC-3`–`SC-6`.

**6. Riesgos:**
- Detectar en esta fase que una decisión de `SC-1` (p. ej. DEC-2 o DEC-3) generó un layout no viable — puede forzar retroceder a `SC-5`, no solo corregir un detalle menor.
- Que Power BI Desktop reescriba archivos adicionales al abrir el PBIP (comportamiento conocido) — deben revisarse y separarse de los cambios intencionales antes de continuar a `SC-8`.

**7. Resultado esperado:** Reporte de validación con hallazgos (si los hay) y confirmación explícita de que la copia está lista para documentarse y, eventualmente, reemplazar o no la página original.

**8. Prompt de ejecución:**
```
Actúa como especialista en control de calidad de modelos y reportes Power BI.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
fase SC-7.

Tarea: valida p14_satisfaccion_capacitaciones_v2 (y las medidas nuevas de
_Medidas Capacitacion si se crearon en SC-3), SIN corregir nada todavía si
encuentras un error - repórtalo primero.

1. Revisa que todos los visual.json de la copia son JSON válido y que no
   referencian ninguna medida o columna inexistente.
2. Si se crearon medidas nuevas, revisa que _Medidas Capacitacion.tmdl no
   tiene lineageTag/description/queryGroup escritos a mano y que abre sin
   error de sintaxis.
3. Pide al usuario que abra el .pbip en Power BI Desktop y confirma junto a
   él: el modelo carga sin advertencias, cada medida nueva calcula el valor
   esperado en una tarjeta de prueba, los 3 segmentadores de la copia
   filtran correctamente todos los visuales, la navegación Home <-> copia
   funciona.
4. Busca el patrón de mojibake (tildes corrompidas) en los textos nuevos de
   la copia, igual que se hizo en Outputs/32 seccion 4.
5. Ejecuta git diff y confirma que NINGUNA página distinta a
   p14_satisfaccion_capacitaciones_v2 cambió desde el inicio de SC-3.

Reporta cada hallazgo (correcto o incorrecto) en una lista clara. Si hay
errores, NO los corrijas automáticamente - pregunta si se retrocede a SC-5/
SC-6 o si se documenta como limitación conocida.
```

---

### Fase SC-8 — Documentación

**1. Objetivo:** Dejar la documentación del proyecto alineada con el estado real de `p14_satisfaccion_capacitaciones_v2` y las medidas nuevas, siguiendo el mismo patrón de mantenimiento ya usado en el resto del proyecto.

**2. Actividades:**
- Actualizar `Docs/02_catalogo_medidas_dax.md`: agregar las medidas nuevas creadas en `SC-3` con su fórmula exacta, qué calculan, en qué visuales se usan y sus observaciones (incluida la limitación de `Capacitaciones Realizadas` si se implementó como proxy de clave compuesta).
- Actualizar `Docs/03_mapa_reporte_paginas_visuales.md`: documentar `p14_satisfaccion_capacitaciones_v2` como página de trabajo/prototipo (aún no reemplaza a la original), con su inventario de visuales.
- Actualizar `Docs/05_decisiones_limitaciones_pendientes.md`: formalizar las 4 decisiones de `SC-1` (DEC-1 a DEC-4) en el formato ya usado para D1–D8, incluyendo si DEC-1 se convierte en la dependencia D9.
- Crear `Outputs/NN_resultado_...md` (numeración consecutiva al momento de ejecutar esta fase) documentando el resultado consolidado de `SC-1` a `SC-7`.

**3. Restricciones:**
- No modificar `PBI/` en esta fase — es exclusivamente de documentación.
- No sobrescribir el historial de `Outputs/` existente — se agrega un archivo nuevo.

**4. Archivos que podrían modificarse:**
- `Docs/02_catalogo_medidas_dax.md`
- `Docs/03_mapa_reporte_paginas_visuales.md`
- `Docs/05_decisiones_limitaciones_pendientes.md`
- `Outputs/NN_resultado_....md` (nuevo)

**5. Validaciones:**
- [ ] Cada medida nueva documentada en `Docs/02` coincide exactamente con la fórmula real del `.tmdl`.
- [ ] `Docs/03` refleja el inventario real de visuales de la copia, no una aspiración.
- [ ] Las 4 decisiones de `SC-1` quedan formalizadas en `Docs/05`, incluida la fecha y quién las confirmó.
- [ ] El nuevo `Outputs/NN` sigue el mismo formato de encabezado (tabla de campos) que los documentos previos de la carpeta.

**6. Riesgos:**
- Documentar medidas o visuales "aspiracionales" (como se planeaban) en vez del estado real verificado en `SC-7` — mitigar releyendo el `.tmdl`/`visual.json` real antes de escribir cada sección, igual que se hizo en `Outputs/32`.

**7. Resultado esperado:** Documentación del proyecto actualizada y consistente con el estado real de la copia de página y las medidas nuevas.

**8. Prompt de ejecución:**
```
Actúa como especialista en documentación técnica de proyectos Power BI.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
fase SC-8, y el resultado real verificado en la fase SC-7.

Tarea: actualiza la documentación del proyecto para reflejar el estado real
(no aspiracional) de p14_satisfaccion_capacitaciones_v2 y las medidas nuevas:

1. Docs/02_catalogo_medidas_dax.md: agrega cada medida nueva leyendo su
   fórmula exacta del archivo .tmdl real (no la reescribas de memoria),
   siguiendo el mismo formato del resto del catálogo.
2. Docs/03_mapa_reporte_paginas_visuales.md: documenta
   p14_satisfaccion_capacitaciones_v2 como página de trabajo/prototipo,
   inventariando sus visuales reales (leídos de los visual.json, no del
   mockup).
3. Docs/05_decisiones_limitaciones_pendientes.md: formaliza las decisiones
   DEC-1 a DEC-4 tomadas en SC-1, en el mismo formato ya usado para D1-D8
   (incluye si DEC-1 pasa a ser la dependencia D9).
4. Crea un nuevo archivo Outputs/NN_resultado_....md (usa el siguiente
   número consecutivo disponible en la carpeta) documentando el resultado
   consolidado de las fases SC-1 a SC-7: qué se decidió, qué se creó, qué
   se validó, qué quedó pendiente.

No modifiques ningún archivo de PBI/. No sobrescribas ningún Outputs/
existente.
```

---

### Fase SC-9 — Reemplazo o publicación

**1. Objetivo:** Decidir, con el usuario, si `p14_satisfaccion_capacitaciones_v2` reemplaza a la página original, y ejecutar esa decisión con la misma disciplina de validación previa a cualquier publicación ya establecida en el proyecto.

**2. Actividades:**
- Presentar al usuario el resultado validado de `SC-7` y preguntar explícitamente: ¿reemplazar la página original, mantener ambas, o descartar el prototipo?
- **Si se reemplaza:** repuntar el botón/hitzone de navegación de Home (que hoy apunta a `p14_satisfaccion_capacitaciones`) hacia `p14_satisfaccion_capacitaciones_v2`; decidir si la página original se elimina, se oculta, o se conserva como referencia histórica fuera de la navegación.
- **Si se mantienen ambas:** decidir un nombre definitivo para la copia y si se agrega a la navegación de Home como una página adicional (fuera del alcance original de 7 páginas — requeriría su propia decisión de negocio).
- **Si se descarta:** documentar por qué, y decidir si la copia se elimina o se conserva como referencia no publicada.
- Antes de publicar cualquier cambio, ejecutar el checklist ya establecido en `Docs/06_publicacion_powerbi.md` §3 (filtros, navegación, etiquetas de datos, actualización, permisos, vigencia del enlace).
- Si se publica, seguir el checklist posterior de `Docs/06` §4 y registrar el cambio.

**3. Restricciones:**
- No publicar sin que el usuario confirme explícitamente la decisión de reemplazo/coexistencia/descarte.
- No eliminar la página original sin confirmación explícita — considerar primero ocultarla o conservarla como referencia, dado que es reversible, mientras que eliminar no lo es.
- No hacer push remoto ni republicar sin autorización explícita del usuario en esa sesión (una autorización previa no cubre esta fase).

**4. Archivos que podrían modificarse:**
- `PBI/PBI_Indicadores.Report/definition/pages/pages.json` (orden de páginas, página activa).
- Visuales de navegación en `Home` (`home_*` correspondientes al módulo "Satisfacción de capacitaciones") si se repunta el destino.
- Posiblemente eliminación/ocultamiento de `pages/p14_satisfaccion_capacitaciones/` (original) — solo con confirmación explícita.
- `Docs/06_publicacion_powerbi.md` y `README.md` si cambia el enlace publicado o la fecha de publicación.

**5. Validaciones:**
- [ ] La decisión de reemplazo/coexistencia/descarte quedó confirmada explícitamente por el usuario, no inferida.
- [ ] Si se repunta la navegación de Home, el botón correspondiente lleva a la página correcta y su hitzone sigue cubriendo el área completa (patrón de `Outputs/28`).
- [ ] El checklist de `Docs/06` §3 se ejecutó antes de publicar.
- [ ] El checklist de `Docs/06` §4 se ejecutó después de publicar (si se publicó).
- [ ] `README.md`/`Docs/06` reflejan el enlace y la fecha de publicación reales tras el cambio.

**6. Riesgos:**
- Publicar sin que `SC-7` haya cerrado sin errores pendientes.
- Romper la navegación de Home si el repunte del botón no se prueba en las 4 zonas del módulo (tarjeta + acento + etiqueta + hitzone).
- Si se decide "coexisten ambas páginas", el informe pasaría de 7 a 8 páginas — revisar si eso reabre la discusión ya cerrada en `Specs/03` §2 sobre el alcance final de páginas.

**7. Resultado esperado:** Decisión de reemplazo ejecutada (o explícitamente diferida) con la misma disciplina de validación que el resto del proyecto, y la publicación (si ocurre) documentada en `Outputs/` conforme al patrón establecido.

**8. Prompt de ejecución:**
```
Actúa como especialista en publicación y gobierno de reportes Power BI.
Proyecto: C:\Users\eclavijo\OneDrive\PBI_Indicadores
Referencia: Specs/06_plan_implementacion_mockup_satisfaccion_capacitaciones.md
fase SC-9, Docs/06_publicacion_powerbi.md secciones 3 y 4, y el resultado
validado de la fase SC-7.

Tarea: NO tomes la decisión de reemplazo por tu cuenta. Presenta al usuario
el resultado validado de p14_satisfaccion_capacitaciones_v2 y pregunta
explícitamente: ¿reemplaza a la página original, coexisten ambas, o se
descarta el prototipo?

Según la respuesta:
- Si reemplaza: repunta el botón de navegación de Home (módulo "Satisfacción
  de capacitaciones": tarjeta + acento + etiqueta + hitzone) hacia la nueva
  página. Pregunta si la original se oculta, se elimina o se conserva fuera
  de navegación (prefiere ocultar/conservar sobre eliminar, es reversible).
- Si coexisten: advierte que el informe pasaría de 7 a 8 páginas, revirtiendo
  una decisión ya cerrada en Specs/03 seccion 2 - confirma que el usuario
  quiere reabrir esa decisión antes de continuar.
- Si se descarta: documenta el motivo y pregunta si la copia se elimina o se
  conserva sin publicar.

Antes de publicar cualquier cambio, ejecuta el checklist de
Docs/06_publicacion_powerbi.md sección 3. NO publiques ni hagas push remoto
sin autorización explícita del usuario en esta misma sesión. Si se publica,
ejecuta el checklist de la sección 4 y registra el cambio en un nuevo
Outputs/NN_....md.
```

---

## 6. Índice de prompts por fase

Cada prompt completo vive en el punto 8 de su fase correspondiente en §5. Este índice es solo de navegación rápida:

| Fase | Nombre | Prompt en |
|---|---|---|
| SC-1 | Confirmación de decisiones de negocio | [§5 → SC-1, punto 8](#fase-sc-1--confirmación-de-decisiones-de-negocio) |
| SC-2 | Preparación técnica | [§5 → SC-2, punto 8](#fase-sc-2--preparación-técnica) |
| SC-3 | Creación de medidas DAX nuevas (condicionada) | [§5 → SC-3, punto 8](#fase-sc-3--creación-de-medidas-dax-nuevas-condicionada) |
| SC-4 | Creación de copia de página | [§5 → SC-4, punto 8](#fase-sc-4--creación-de-copia-de-página) |
| SC-5 | Rediseño visual según el mockup | [§5 → SC-5, punto 8](#fase-sc-5--rediseño-visual-según-el-mockup) |
| SC-6 | Configuración de interacciones | [§5 → SC-6, punto 8](#fase-sc-6--configuración-de-interacciones) |
| SC-7 | Validación técnica, funcional y visual | [§5 → SC-7, punto 8](#fase-sc-7--validación-técnica-funcional-y-visual) |
| SC-8 | Documentación | [§5 → SC-8, punto 8](#fase-sc-8--documentación) |
| SC-9 | Reemplazo o publicación | [§5 → SC-9, punto 8](#fase-sc-9--reemplazo-o-publicación) |

## 7. Orden recomendado de ejecución

```
SC-1 (decisiones) → SC-2 (preparación) → SC-3 (medidas) → SC-4 (copia de página)
      → SC-5 (rediseño) → SC-6 (interacciones) → SC-7 (validación)
      → SC-8 (documentación) → SC-9 (reemplazo/publicación)
```

`SC-3` y `SC-4` no dependen entre sí (ver §8) y podrían ejecutarse en cualquier orden relativo, pero ambas deben completarse antes de `SC-5`. El resto del orden es estrictamente secuencial.

## 8. Dependencias entre fases

| Fase | Depende de | Bloquea a |
|---|---|---|
| SC-1 | — | SC-3 (parcialmente), SC-5 (parcialmente), SC-9 |
| SC-2 | — | SC-3, SC-4 |
| SC-3 | SC-1 (para la parte condicionada), SC-2 | SC-5 |
| SC-4 | SC-2 | SC-5 |
| SC-5 | SC-1, SC-3, SC-4 | SC-6 |
| SC-6 | SC-5 | SC-7 |
| SC-7 | SC-6 | SC-8, SC-9 |
| SC-8 | SC-7 | SC-9 (recomendado, no estrictamente bloqueante) |
| SC-9 | SC-7 (obligatorio), SC-8 (recomendado) | — |

## 9. Validaciones por fase (resumen)

| Fase | Validación clave |
|---|---|
| SC-1 | Las 4 decisiones tienen respuesta explícita o estado "pendiente" confirmado |
| SC-2 | `git status` limpio y página original protegida explícitamente |
| SC-3 | Medidas nuevas calculan correctamente; ninguna medida bloqueada creada sin DEC-1 |
| SC-4 | Página original sin cambios; copia idéntica al momento de duplicar |
| SC-5 | Solo colores del tema; indicadores bloqueados marcados como pendientes, no inventados |
| SC-6 | Selección cruzada funciona; segmentadores y navegación siguen intactos |
| SC-7 | Modelo sin advertencias; medidas correctas; cero mojibake; cero cambios fuera de la copia |
| SC-8 | Documentación coincide con el estado real verificado, no con lo planeado |
| SC-9 | Decisión de reemplazo confirmada explícitamente; checklist de publicación ejecutado |

## 10. Riesgos consolidados

- **Riesgo de grano no resuelto (DEC-1)**: si se avanza sin resolverlo, 3 de los 6 indicadores originales del mockup quedan permanentemente fuera de esta iteración — no es un riesgo técnico, es una limitación de alcance que debe comunicarse con claridad en `SC-8`.
- **Riesgo de inconsistencia de diseño**: remover/reubicar el gráfico de jornada (DEC-3) solo en esta página puede generar una experiencia inconsistente frente a las otras 6 — mitigar con confirmación explícita en `SC-1`, no como efecto secundario del rediseño. (El riesgo equivalente para el segmentador de Fecha, DEC-4, quedó descartado tras `SC-4`: no se modifica nada, el segmentador ya funciona en modo `Between` desde antes — ver §4, nota de corrección de DEC-4.)
- **Riesgo de exposición de datos personales**: si DEC-2 resulta en "coexisten ambas tablas", la página seguiría exponiendo nombres reales de formador/líder además de la nueva vista por call center — revisar contra `Docs/06` §2 antes de `SC-9`.
- **Riesgo de regresión sobre la página publicada**: cualquier error en `SC-3`–`SC-6` que accidentalmente toque `p14_satisfaccion_capacitaciones` (original) afecta el informe ya publicado — mitigado por trabajar exclusivamente en la copia y verificar `git diff` en cada fase.
- **Riesgo de alcance de páginas**: si `SC-9` resulta en "coexisten ambas páginas", se reabre la decisión ya cerrada de 7 páginas (`Specs/03` §2) — debe tratarse como una decisión nueva, no una consecuencia automática.
- **Riesgo operativo conocido del proyecto**: bloqueo de archivos de `Data/` durante actualización, cambios automáticos de Power BI Desktop no revisados — mismos riesgos ya documentados en `Docs/05` §5, aplicables también durante la ejecución de este plan.

## 11. Criterios para avanzar o detenerse

**Avanzar de SC-1 a SC-2:** si al menos DEC-2, DEC-3 y DEC-4 tienen respuesta (DEC-1 puede quedar "pendiente, se avanza parcialmente" sin bloquear el resto del plan).

**Avanzar de SC-3 a SC-4:** si las medidas no bloqueadas (`Call Centers Capacitados`, `Ultima Capacitacion`) fueron creadas y validadas sin error; `Capacitaciones Realizadas` solo si DEC-1 se resolvió y también validó sin error.

**Detenerse antes de SC-5 (no avanzar el rediseño completo):** si DEC-1 sigue sin resolver y el usuario prefiere pausar hasta tenerla, en vez de avanzar con una versión parcial del mockup.

**Detenerse antes de SC-9 (no reemplazar ni publicar):** si `SC-7` encontró errores sin resolver, o si el usuario no confirma explícitamente la decisión de reemplazo/coexistencia/descarte.

**Retroceder de SC-7 a SC-5/SC-6:** si la validación encuentra un layout no viable o una interacción mal configurada — no se "parchea" en `SC-7`, se retrocede a la fase que corresponde.

## 12. Criterios de cierre

Esta iniciativa (adaptación de "Satisfacción de capacitaciones" al mockup) se considera cerrada cuando:

- [ ] Las 4 decisiones de negocio (§4) están resueltas o explícitamente diferidas con su impacto documentado.
- [ ] Las medidas DAX no bloqueadas están creadas, validadas y documentadas en `Docs/02`.
- [ ] `p14_satisfaccion_capacitaciones_v2` (o su nombre final) está validada técnica, funcional y visualmente (`SC-7`), sin errores pendientes.
- [ ] `Docs/02`, `Docs/03` y `Docs/05` reflejan el estado real del modelo y del reporte tras el cambio.
- [ ] La decisión de reemplazo/coexistencia/descarte (`SC-9`) está tomada, ejecutada y documentada.
- [ ] Si se publicó, el checklist de `Docs/06` se ejecutó antes y después, y el enlace/documentación quedaron actualizados.
- [ ] Existe un `Outputs/NN_...md` de cierre que resume las 9 fases ejecutadas y su resultado, siguiendo el patrón ya establecido en el proyecto.
