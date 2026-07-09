# Plan de Implementación — Informe Power BI Connect Assistance S.A.S.

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Documento base | [`Specs/01_analisis_de_impacto_informe_powerbi_connect.md`](01_analisis_de_impacto_informe_powerbi_connect.md) |
| Cliente / Contexto | Connect Assistance S.A.S. — call centers asociados a Claro |
| Rol funcional | PUSHER (auditoría de calidad + formación + seguimiento comercial) |
| Tipo de documento | Plan de implementación (fases + prompts ejecutables) |
| Fecha | 2026-07-08 |
| Estado | **Cerrado** — las 18 fases fueron ejecutadas y validadas. Ver el cierre formal en [Specs/03_documentacion_final_informe_powerbi_connect.md](03_documentacion_final_informe_powerbi_connect.md) y la bitácora completa en `Outputs/`. |

> Este documento **no modifica** el PBIP, el modelo semántico, las consultas Power Query, las medidas DAX ni las páginas del reporte. Es la hoja de ruta que se ejecutó fase por fase; el estado real y final de cada decisión está en [Specs/03_documentacion_final_informe_powerbi_connect.md](03_documentacion_final_informe_powerbi_connect.md), no en este documento — este plan se conserva como referencia histórica de cómo se ejecutó cada fase.

---

## Índice

1. [Resumen ejecutivo del plan](#1-resumen-ejecutivo-del-plan)
2. [Objetivo de la implementación](#2-objetivo-de-la-implementación)
3. [Alcance del informe](#3-alcance-del-informe)
4. [Supuestos y dependencias](#4-supuestos-y-dependencias)
5. [Fases de implementación](#5-fases-de-implementación)
6. [Orden recomendado de ejecución](#6-orden-recomendado-de-ejecución)
7. [Recomendaciones UX/UI — Home](#7-recomendaciones-uxui--home-landing-page)
8. [Recomendaciones UX/UI — páginas internas](#8-recomendaciones-uxui--páginas-internas)
9. [Criterios de cierre](#9-criterios-de-cierre)
10. [Próximos pasos sugeridos](#10-próximos-pasos-sugeridos)

---

## 1. Resumen ejecutivo del plan

Este plan traduce el diagnóstico del documento 01 en **18 fases secuenciales y ejecutables**, cada una con un prompt independiente que puede correrse por separado (en una sesión nueva, sin depender del historial de conversación) siempre que las fases previas ya se hayan completado sobre el PBIP.

El plan cubre el ciclo completo: preparación del entorno → versionamiento → ingesta → limpieza → modelado (hechos + dimensiones) → medidas DAX → validación → identidad visual → construcción de páginas (Home tipo landing page + páginas internas) → navegación → notas metodológicas → validación final → documentación de cierre.

El resultado final es un informe Power BI **profesional, ejecutivo y con identidad Connect Assistance**, construido sobre datos piloto pero preparado para escalar sin rediseño cuando el volumen de respuestas crezca.

---

## 2. Objetivo de la implementación

Construir, desde el PBIP vacío ya existente, un informe Power BI funcional que permita al rol PUSHER y a la gerencia de Connect Assistance dar seguimiento a:

- **Calidad de llamadas** (matriz de auditoría).
- **Satisfacción de capacitaciones**.
- **Motivación en actividades comerciales**.

Con desglose por **call center, jornada, líder, asesor y formador** donde los datos lo permitan, bajo una identidad visual de marca Connect, siguiendo buenas prácticas de modelado (estrella), Power Query (transformaciones documentadas y reproducibles) y DAX (medidas reutilizables).

---

## 3. Alcance del informe

### Incluido en el alcance
- Modelo semántico con 3 tablas de hechos + 3 dimensiones compartidas (`Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada`), según lo definido en el documento 01, §4.
- Medidas DAX de calidad, satisfacción y motivación (documento 01, §5).
- 8 páginas de reporte: Home, Resumen ejecutivo, Calidad de llamadas, Satisfacción de capacitaciones, Motivación y actividades comerciales, Detalle por call center, Detalle por asesor/líder (parcial, solo donde los datos lo permitan), Notas metodológicas.
- Tema visual personalizado de marca Connect (naranja/blanco/negro-gris).
- Navegación entre páginas mediante botones visuales.
- Advertencias visibles de bajo volumen (`n=`) y de limitaciones de fuente (encuesta anónima de motivación).

### Fuera de alcance (fase 2 o posteriores, no en este plan)
- Conexión automática/en vivo a Google Forms o Google Sheets (por ahora se mantiene la ingesta desde los archivos Excel exportados manualmente en `Data`).
- Creación de `Dim_Colaborador` maestro con ID único (requiere fuente de nómina/WFM, ver documento 01 §8).
- Publicación en Power BI Service, configuración de gateway o RLS (seguridad a nivel de fila) — se puede añadir como fase futura si se requiere restringir por call center o líder.
- Diseño final de tema JSON con HEX exactos de marca (depende de que el negocio entregue manual de marca — ver §4).

---

## 4. Supuestos y dependencias

| # | Supuesto / Dependencia | Estado | Impacto si no se cumple |
|---|---|---|---|
| D1 | Power BI Desktop instalado tiene soporte de **Power BI Project (PBIP) + TMDL** habilitado (versión reciente; característica de vista previa en versiones anteriores a 2024, GA desde 2024) | Por confirmar en Fase 1 | Si no está habilitado, el PBIP no se abre correctamente como proyecto editable |
| D2 | Los 3 archivos de `Data` deben estar **cerrados** (no abiertos en Excel) al momento de conectar/actualizar en Power Query | Conocido riesgo (documento 01 §8) — ya se detectó bloqueo de archivo durante el diagnóstico | Error `The process cannot access the file` al actualizar |
| D3 | Negocio confirma la **rúbrica de puntaje máximo por pregunta** de la Matriz de calidad (documento 01, Fase 0 / §8) | **Pendiente** | Sin esto, la medida `% Calidad Promedio` no puede calcularse con exactitud; se construirá con un supuesto documentado y marcado como provisional |
| D4 | Negocio confirma **catálogo oficial de call centers y jornadas** vigentes | **Pendiente** | Se usará el catálogo dinámico (unión de valores distintos observados) hasta recibir la lista oficial |
| D5 | Negocio confirma **nombres estándar de líderes/formadores** para la tabla de alias | **Pendiente** | Se construirá con los alias ya detectados en el documento 01 §3.2; nuevas variantes futuras requerirán mantenimiento manual de la tabla |
| D6 | Negocio entrega **logo oficial de Connect Assistance** (vector o PNG alta resolución) y **códigos HEX exactos** de la paleta de marca | **Pendiente** | Se usará una paleta de marcador de posición (naranja aproximado) y espacio reservado para logo, reemplazable sin rediseño estructural |
| D7 | El proyecto se versiona con Git antes de iniciar cambios estructurales (Fase 2) | Pendiente de decisión del usuario | Sin versionamiento, no hay forma de revertir cambios en TMDL/JSON si algo falla a mitad de una fase |
| D8 | El volumen de datos actual (3/32/5 filas) es representativo solo de una prueba piloto | Confirmado en documento 01 | El informe debe comunicar esto explícitamente (Fase 16); los KPIs no deben presentarse como definitivos ante gerencia |

---

## 5. Fases de implementación

Cada fase incluye un **prompt de ejecución** listo para copiar y pegar en una sesión de trabajo (con Claude Code u otro asistente/especialista), diseñado para poder ejecutarse de forma independiente citando explícitamente los documentos de referencia y el alcance permitido.

---

### Fase 1 — Preparación del proyecto PBIP

**Objetivo:** Verificar que el entorno de trabajo (Power BI Desktop + estructura PBIP) está listo para iterar, sin realizar cambios de contenido todavía.

**Actividades principales:**
- Confirmar que Power BI Desktop tiene habilitada la característica **"Power BI Project (.pbip) save option"** y formato **TMDL**.
- Abrir `PBI_Indicadores.pbip` y verificar que carga sin errores como proyecto (no como `.pbix`).
- Confirmar que los 3 archivos de `Data/` están cerrados y accesibles (sin bloqueo de OneDrive/Excel).
- Revisar que `PBI_Indicadores.SemanticModel/definition/model.tmdl` sigue vacío (checkpoint antes de empezar).
- Verificar espacio de trabajo limpio: sin cambios sin guardar, sin procesos de Power BI Desktop colgados.

**Prompt de ejecución:**
```
Actúa como especialista en Power BI/PBIP. Proyecto ubicado en
C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores.

Tarea: realizar una verificación de preparación (checklist), SIN modificar
ningún archivo del PBIP todavía.

1. Confirma la estructura de PBI_Indicadores.pbip, PBI_Indicadores.SemanticModel
   y PBI_Indicadores.Report (rutas, archivos .platform, model.tmdl).
2. Confirma que model.tmdl sigue vacío (sin tablas ni medidas) - esto es el
   checkpoint de partida.
3. Verifica que los 3 archivos en Data/ (Matriz de calidad, Satisfacción
   capacitación, Encuesta satisfacción) no están bloqueados por otro proceso.
4. Reporta cualquier hallazgo que impida iniciar la Fase 3 (ingesta), por
   ejemplo archivos bloqueados, o inconsistencias entre .pbip y las carpetas
   .Report/.SemanticModel.

No crees, edites ni borres ningún archivo. Solo diagnostica y reporta.
```

**Resultado esperado:** Checklist de preparación aprobado; lista de bloqueos (si existen) antes de avanzar.

**Validaciones necesarias:**
- [ ] Power BI Desktop abre el `.pbip` sin errores.
- [ ] `model.tmdl` confirmado vacío (checkpoint).
- [ ] Los 3 archivos Excel están cerrados/accesibles.

**Riesgos / puntos de control:**
- Si Power BI Desktop es una versión antigua sin soporte TMDL nativo, se debe actualizar antes de continuar.
- Si algún archivo de `Data` permanece bloqueado, resolver manualmente (cerrar Excel) antes de la Fase 3.

---

### Fase 2 — Versionamiento recomendado del proyecto

**Objetivo:** Establecer control de versiones antes de iniciar cambios estructurales, para poder revertir o comparar el TMDL/JSON en cada fase.

**Actividades principales:**
- Inicializar repositorio Git en `PBI_Indicadores/` (actualmente no es repo).
- Revisar/ampliar `.gitignore` a nivel raíz (el existente en `PBI/.gitignore` ya cubre `.pbi/localSettings.json` y `.pbi/cache.abf`; evaluar si además se debe excluir `Data/*.xlsx` del repo por contener nombres de personas — dato semi-sensible — o versionarlos igualmente por ser insumo reproducible del piloto).
- Definir convención de commits (ej. `feat:`, `fix:`, `docs:`, en español o inglés, a elección del equipo) y de ramas (ej. `main` estable + rama de trabajo por fase, o commits directos a `main` si el equipo es una sola persona).
- Commit inicial ("estado base": PBIP vacío + Specs/01 + Specs/02).

**Prompt de ejecución:**
```
Actúa como especialista en control de versiones para proyectos Power BI/PBIP.
Proyecto ubicado en C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores.

Tarea:
1. Verifica si la carpeta raíz ya es un repositorio Git (git status). Si no lo
   es, propone inicializarlo (git init) y espera confirmación antes de
   ejecutar cualquier comando.
2. Revisa el .gitignore existente en PBI/.gitignore y recomienda si se debe
   añadir un .gitignore a nivel raíz, incluyendo si Data/*.xlsx debería
   versionarse o excluirse (contiene nombres de asesores y líderes reales).
3. Propon una convención simple de commits y de ramas adecuada para un
   proyecto Power BI de un equipo pequeño.
4. Si el usuario aprueba, crea el primer commit con el estado actual
   (PBIP vacío + carpeta Specs) usando un mensaje descriptivo.

No hagas push a ningún remoto ni configures credenciales. No elimines datos
existentes. Pide confirmación antes de cualquier operación irreversible.
```

**Resultado esperado:** Repositorio Git inicializado, `.gitignore` validado, primer commit del estado base creado.

**Validaciones necesarias:**
- [ ] `git status` limpio tras el primer commit.
- [ ] `.gitignore` excluye archivos de caché/local settings de Power BI.
- [ ] Decisión documentada sobre versionar o no los Excel de `Data`.

**Riesgos / puntos de control:**
- Si se decide excluir `Data/*.xlsx` de Git, documentar claramente cómo se distribuyen/respaldan esos archivos fuera del repositorio.
- Evitar commits que incluyan `cache.abf` (binario pesado y regenerable).

---

### Fase 3 — Ingesta de archivos desde la carpeta `Data`

**Objetivo:** Conectar Power Query a los 3 archivos Excel de `Data` como consultas base, sin transformar todavía.

**Actividades principales:**
- Crear consultas Power Query: `Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion`, `Base_EncuestaMotivacion` (nombres provisionales, "Base_" indica que son la carga cruda antes de limpieza).
- Usar `Excel.Workbook(File.Contents(...))` apuntando a la hoja `Form Responses 1` de cada archivo.
- Evaluar el uso de un **parámetro de Power Query** (`RutaCarpetaData`) para la ruta base, en vez de rutas absolutas embebidas, para facilitar portabilidad si el proyecto se mueve o lo abre otro usuario.
- Deshabilitar la carga al modelo de las consultas `Base_*` una vez se creen las consultas limpias (Fase 4), dejándolas como punto de partida ("staging").

**Prompt de ejecución:**
```
Actúa como especialista en Power Query dentro de un proyecto Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Documentos de referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md
(sección 2.2 y 3) y Specs/02_plan_implementacion_informe_powerbi_connect.md
(Fase 3).

Tarea: en PBI_Indicadores.SemanticModel, crear 3 consultas Power Query de
staging (sin transformar aún, solo carga cruda de la hoja "Form Responses 1"):
- Base_MatrizCalidad          <- Data/Matriz de calidad (Responses).xlsx
- Base_SatisfaccionCapacitacion <- Data/Satisfacción capacitación (Responses).xlsx
- Base_EncuestaMotivacion     <- Data/Encuesta satisfacción (Responses).xlsx

Usa un parámetro de Power Query "RutaCarpetaData" para la ruta base de la
carpeta Data, en vez de rutas absolutas fijas.

No apliques limpieza, renombrado ni tipado todavía (eso es la Fase 4). No
actives la carga al modelo de estas consultas de staging (deshabilitar
"Enable load"). No crees medidas ni visuales. Verifica que las 3 consultas
carguen sin error de vista previa.
```

**Resultado esperado:** 3 consultas de staging funcionando en el editor de Power Query, sin errores de carga, sin transformar.

**Validaciones necesarias:**
- [ ] Las 3 consultas muestran vista previa de datos sin error.
- [ ] El parámetro `RutaCarpetaData` resuelve correctamente a la carpeta `Data`.
- [ ] Ninguna consulta de staging está cargando al modelo todavía.

**Riesgos / puntos de control:**
- Archivo bloqueado (ver D2) — reintentar tras cerrar Excel.
- Verificar que la hoja se llama exactamente `Form Responses 1` en los 3 archivos (confirmado en el diagnóstico).

---

### Fase 4 — Limpieza y transformación de datos en Power Query

**Objetivo:** Aplicar limpieza estructural (encabezados, tipos, espacios) sobre las consultas de staging para producir consultas intermedias limpias.

**Actividades principales:**
- Trim + Clean sobre todas las columnas de texto.
- Promoción/corrección de encabezados (quitar espacios al inicio/fin y signos `¿?` del encabezado técnico, conservando el texto original como posible `displayName`/etiqueta si se desea mostrar en visuales).
- Tipado explícito por columna: fecha/hora, texto, número decimal (Likert), booleano donde aplique (`TerminoEnVenta`).
- Extracción de columna `Fecha` (date) desde `Timestamp` (datetime) en las 3 consultas.
- Normalización de `CallCenter` a mayúsculas + Trim.
- Documentar cada paso aplicado con nombres de paso descriptivos (no dejar los nombres genéricos `Cambiado tipo1`, `Cambiado tipo2`, etc.).

**Prompt de ejecución:**
```
Actúa como especialista en Power Query (M) dentro de un proyecto Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 3
(diagnóstico de calidad de datos por archivo) y Specs/02..., Fase 4.

Tarea: a partir de las consultas de staging Base_MatrizCalidad,
Base_SatisfaccionCapacitacion y Base_EncuestaMotivacion (creadas en la Fase 3),
crea 3 consultas intermedias limpias (sufijo "_Limpio"):

1. Trim + Clean en todas las columnas de texto.
2. Elimina espacios y símbolos ¿? de los encabezados técnicos (conserva el
   texto original de la pregunta como referencia en un comentario del paso
   o en documentación, no en el nombre técnico).
3. Tipado correcto: Timestamp como datetime, agrega columna Fecha (solo
   fecha) derivada de Timestamp, columnas Likert/checklist como número
   decimal, TerminoEnVenta como texto Sí/No (se convierte a booleano en
   Fase 6/7 si aplica).
4. CallCenter normalizado a MAYÚSCULAS + Trim.
5. Nombra cada paso aplicado de forma descriptiva (evita "Cambiado tipo1").

No renombres aún columnas a los nombres técnicos finales (eso es Fase 5).
No trates todavía los "N/A" ni las respuestas abiertas (eso es Fase 6). No
actives carga al modelo de estas consultas "_Limpio" si vas a encadenar más
pasos en fases siguientes; puedes dejarlas como intermedias.
```

**Resultado esperado:** 3 consultas `_Limpio` con encabezados sin espacios/símbolos, tipos correctos y columna `Fecha` derivada.

**Validaciones necesarias:**
- [ ] Sin errores de tipo en la vista previa de las 3 consultas.
- [ ] Columna `Fecha` (date) presente y correcta en las 3 consultas.
- [ ] `CallCenter` normalizado (mayúsculas, sin espacios) verificado contra los valores del diagnóstico (`ONE CONTACT`, `CAPITALS`, `INTERACTIVO`, `MILLENIUM`).

**Riesgos / puntos de control:**
- El cambio de tipo puede fallar si alguna celda contiene texto inesperado (ej. `"N/A"` en columnas numéricas de la Matriz de calidad) — si falla el tipado, documentar el error y resolverlo en conjunto con la Fase 6 (no forzar conversión que genere errores silenciosos).

---

### Fase 5 — Normalización de nombres técnicos de tablas y columnas

**Objetivo:** Aplicar la convención de nombres técnicos definida en el documento 01 (§4.5) de forma consistente en las 3 consultas.

**Actividades principales:**
- Renombrar consultas a su nombre final de tabla de hechos: `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`.
- Renombrar columnas a `PascalCase` sin espacios ni tildes, según el mapeo detallado en documento 01 §4.3 (ej. `Preg_TonoSaludo`, `SatisfaccionGeneral`, `AmbienteEquipo`, etc.).
- Producir y documentar una **tabla de mapeo** (columna original → columna técnica) como referencia para auditoría futura (puede vivir en este mismo documento o en un anexo).
- Verificar que no queden encabezados con `¿`, `?`, espacios dobles o tildes en el nombre técnico.

**Prompt de ejecución:**
```
Actúa como especialista en modelado de datos Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 4.3
(columnas técnicas recomendadas por tabla de hechos).

Tarea: sobre las consultas "_Limpio" de la Fase 4, aplica el renombrado final:

1. Renombra las consultas a: Fact_CalidadLlamadas, Fact_SatisfaccionCapacitacion,
   Fact_MotivacionActividad.
2. Renombra cada columna a nombre técnico PascalCase sin espacios/tildes,
   siguiendo el detalle de la sección 4.3 del documento 01 para cada tabla.
3. Genera una tabla de mapeo (columna original -> columna técnica) por cada
   Fact y guárdala como referencia en Specs/ (archivo nuevo o anexo al
   documento 02, según prefieras) para trazabilidad.
4. Verifica que ningún nombre técnico de columna contenga ¿, ?, espacios al
   inicio/fin o dobles espacios.

No cambies aún la lógica de negocio de los datos (nulos, N/A, respuestas
abiertas: eso es Fase 6). No actives todavía la carga final al modelo si
prefieres encadenar con la Fase 6 primero.
```

**Resultado esperado:** 3 consultas con nombre y columnas técnicas finales; tabla de mapeo original→técnico documentada.

**Validaciones necesarias:**
- [ ] Ningún nombre de columna contiene `¿`, `?`, tildes o espacios extremos.
- [ ] Los 3 nombres de tabla coinciden exactamente con `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`.
- [ ] Tabla de mapeo original→técnico revisada y legible para alguien no técnico (útil para negocio).

**Riesgos / puntos de control:**
- Evitar nombres técnicos ambiguos entre tablas (ej. `SatisfaccionGeneral` existe tanto en `Fact_SatisfaccionCapacitacion` como en `Fact_MotivacionActividad` — es aceptable porque están en tablas distintas, pero debe quedar claro en la documentación a cuál pertenece cada medida).

---

### Fase 6 — Tratamiento de valores nulos, `"N/A"` y respuestas abiertas

**Objetivo:** Resolver los problemas de calidad de datos específicos detectados en el documento 01 (§3.1–§3.4) antes de dar por cerrada la limpieza.

**Actividades principales:**
- **Matriz de calidad:** convertir el texto `"N/A"` en las 8 columnas de checklist a `null` (no a `0`), preservando el tipo numérico de la columna.
- **Satisfacción capacitación:** tratar la fila con `CallCenter`/`Jornada`/`Nombre` nulos (1 de 32) — decidir si se excluye del modelo o se marca como "Sin dato" visible; documentar la decisión.
- **Ambas encuestas de comentario abierto:** unificar variantes de "sin respuesta" (`Na`, `N/A`, `Nada`, `Ninguna`, vacío) a un valor estándar `"Sin comentario"`.
- **Nombres de líder (Satisfacción capacitación):** aplicar la tabla de mapeo de alias (documento 01 §3.2) para consolidar las 4 variantes de `Juan Esteban Pérez Camargo` en un solo valor estándar. Aplicar Trim/Clean/Proper Case general al resto de nombres.
- **Encuesta de motivación:** confirmar y documentar explícitamente que la columna de nombre está 100% vacía (no es un error de carga) para que quede trazable en el modelo.

**Prompt de ejecución:**
```
Actúa como especialista en calidad de datos y Power Query dentro de un
proyecto Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md
secciones 3.1, 3.2, 3.3 y 3.4 (diagnóstico detallado de calidad de datos).

Tarea: sobre Fact_CalidadLlamadas, Fact_SatisfaccionCapacitacion y
Fact_MotivacionActividad (ya renombradas en Fase 5), aplica:

1. En Fact_CalidadLlamadas: convierte el texto "N/A" en las columnas de
   checklist (Preg_*) a valor nulo real, sin alterar los valores numéricos
   existentes.
2. En Fact_SatisfaccionCapacitacion: identifica la fila con CallCenter/
   Jornada/Nombre nulos y aplica la decisión que se acuerde (excluir del
   modelo o mantener marcada como "Sin dato"); documenta la decisión tomada
   y por qué.
3. En las columnas de comentario abierto de ambas encuestas de satisfacción/
   motivación: unifica variantes de "sin respuesta" (Na, N/A, Nada, Ninguna,
   vacío) al valor estándar "Sin comentario".
4. En NombreLider de Fact_SatisfaccionCapacitacion: aplica una tabla de
   mapeo de alias (columna alias original -> nombre estándar) para
   consolidar las variantes ya detectadas de "Juan Esteban Pérez Camargo".
   Aplica Proper Case general al resto de nombres de asesor/líder/formador.
5. Confirma y deja documentado que Fact_MotivacionActividad no tiene
   columna de nombre utilizable (100% vacía) - no la elimines, pero no la
   uses como llave en ningún paso posterior.

Reporta al final un resumen de cuántas celdas fueron modificadas por cada
regla, para trazabilidad.
```

**Resultado esperado:** Datos limpios de inconsistencias conocidas; decisión documentada sobre la fila con nulos; comentarios abiertos unificados; nombres de líder consolidados.

**Validaciones necesarias:**
- [ ] `COUNTBLANK`/nulos en columnas de checklist de calidad ya no contienen el texto `"N/A"` como string.
- [ ] `NombreLider` en `Fact_SatisfaccionCapacitacion` muestra un solo valor para "Juan Esteban Pérez Camargo" (antes 4 variantes).
- [ ] Comentarios abiertos: conteo de valores únicos "equivalentes a sin comentario" se redujo a 1 valor estándar.
- [ ] Decisión sobre la fila nula documentada (excluida o marcada).

**Riesgos / puntos de control:**
- La tabla de mapeo de alias es manual y debe mantenerse a futuro si aparecen nuevas variantes — dejar el paso de Power Query fácilmente editable (tabla de referencia, no hardcode disperso).
- Si se decide excluir la fila con nulos, verificar que no se pierda información relevante (revisar manualmente esa fila antes de excluir, dado el bajo volumen).

---

### Fase 7 — Creación de tablas de hechos

**Objetivo:** Dejar las 3 tablas de hechos (`Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`) cargadas al modelo semántico, con carga habilitada y sin las consultas de staging intermedias visibles en el panel de campos.

**Actividades principales:**
- Habilitar "Enable load" solo en las 3 consultas finales; deshabilitar/organizar como auxiliares las de staging (`Base_*`) si aún existen como pasos separados, o consolidar todo en una sola consulta encadenada por tabla (Base → Limpio → Final) sin dejar consultas intermedias sueltas cargando al modelo.
- Agrupar las 3 consultas en una carpeta de consultas (`Hechos`) en el editor de Power Query para orden visual.
- Verificar tipos de datos finales en cada columna desde la vista de Power BI Desktop (no solo Power Query).
- Ocultar del panel de campos las columnas técnicas que no se usarán directamente en visuales (si aplica).

**Prompt de ejecución:**
```
Actúa como especialista en modelado de datos Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 4.2.

Tarea: finaliza la carga al modelo semántico de las 3 tablas de hechos:
Fact_CalidadLlamadas, Fact_SatisfaccionCapacitacion, Fact_MotivacionActividad.

1. Habilita "Enable load" únicamente en estas 3 consultas finales.
2. Si existen consultas intermedias de staging (Base_*, *_Limpio) que ya no
   se necesitan cargadas, deshabilita su carga al modelo (mantenlas como
   pasos de referencia/documentación, no las elimines sin revisar).
3. Agrupa las 3 consultas en una carpeta de Power Query llamada "Hechos".
4. Verifica en la vista de modelo de Power BI Desktop que las 3 tablas
   cargan correctamente y que los tipos de dato son correctos.

No crees todavía las dimensiones (Fase 8) ni relaciones (Fase 9). No
crees medidas DAX todavía (Fase 10).
```

**Resultado esperado:** 3 tablas de hechos visibles y cargadas en el modelo, sin ruido de consultas intermedias en el panel de campos.

**Validaciones necesarias:**
- [ ] Las 3 tablas aparecen en el panel de campos con el nombre técnico correcto.
- [ ] `COUNTROWS` de cada tabla coincide con el conteo esperado (3, 32 o 31 según decisión de Fase 6, y 5 filas respectivamente).
- [ ] No hay consultas de staging cargando innecesariamente al modelo.

**Riesgos / puntos de control:**
- Revisar el impacto en `refresh` (actualización) si se dejan demasiados pasos encadenados — validar tiempo de carga razonable (dato menor dado el bajo volumen actual).

---

### Fase 8 — Creación de dimensiones

**Objetivo:** Construir `Dim_Calendario`, `Dim_CallCenter` y `Dim_Jornada` según lo especificado en el documento 01 §4.3.

**Actividades principales:**
- `Dim_Calendario`: tabla continua generada con `CALENDAR`/`CALENDARAUTO` en DAX o vía Power Query, desde el mínimo `Fecha` observado en las 3 fact hasta `HOY()` (o el máximo `Fecha` + margen), con columnas `Año`, `Mes`, `NombreMes`, `Trimestre`, `Semana`, `DiaSemana`, `EsFinDeSemana`. Marcar como **tabla de fechas oficial** (`Mark as Date Table`) sobre la columna `Fecha`.
- `Dim_CallCenter`: consulta Power Query que anexa (`Append`) los valores distintos de `CallCenter` de las 3 Fact, sin duplicados, ordenados alfabéticamente.
- `Dim_Jornada`: consulta que anexa los valores distintos de `Jornada` de `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` (recordar que `Fact_CalidadLlamadas` no tiene esta columna).

**Prompt de ejecución:**
```
Actúa como especialista en modelado de datos Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 4.3.

Tarea: crea las 3 tablas de dimensión sobre el modelo semántico que ya
tiene cargadas Fact_CalidadLlamadas, Fact_SatisfaccionCapacitacion y
Fact_MotivacionActividad (Fase 7):

1. Dim_Calendario: tabla continua de fechas (CALENDAR/CALENDARAUTO) desde
   el mínimo de Fecha observado en las 3 Fact hasta HOY(). Incluye Año,
   Mes, NombreMes, Trimestre, Semana, DiaSemana, EsFinDeSemana. Márcala
   como tabla de fechas oficial sobre la columna Fecha.
2. Dim_CallCenter: consulta Power Query que une (Append) los valores
   distintos de CallCenter de las 3 Fact, sin duplicados, orden alfabético.
3. Dim_Jornada: consulta que une los valores distintos de Jornada de
   Fact_SatisfaccionCapacitacion y Fact_MotivacionActividad únicamente
   (Fact_CalidadLlamadas no tiene columna Jornada, no la incluyas como
   fuente de esta dimensión).

Agrupa estas 3 consultas/tablas en una carpeta "Dimensiones". No crees
todavía las relaciones (Fase 9) ni medidas DAX (Fase 10).
```

**Resultado esperado:** 3 dimensiones cargadas al modelo, `Dim_Calendario` marcada como tabla de fechas oficial.

**Validaciones necesarias:**
- [ ] `Dim_CallCenter` contiene exactamente los 4 call centers observados en el diagnóstico (`ONE CONTACT`, `CAPITALS`, `INTERACTIVO`, `MILLENIUM`) — o más, si el volumen creció.
- [ ] `Dim_Jornada` contiene `Mañana` y `Tarde` (sin duplicados por variantes de mayúsculas).
- [ ] `Dim_Calendario` marcada correctamente como tabla de fechas (ícono de calendario en el panel de campos).

**Riesgos / puntos de control:**
- Si `Dim_CallCenter`/`Dim_Jornada` se construyen como lista fija en vez de dinámica, se rompe la escalabilidad — confirmar que la lógica es de unión de valores distintos, no hardcode.

---

### Fase 9 — Diseño del modelo de datos y relaciones

**Objetivo:** Establecer las relaciones del modelo en estrella definidas en el documento 01 §4.4, sin ambigüedades ni relaciones circulares.

**Actividades principales:**
- Crear las 7 relaciones descritas en documento 01 §4.4 (Calendario→3 Fact, CallCenter→3 Fact, Jornada→2 Fact), todas de cardinalidad `1:*` y dirección de filtro única.
- Verificar en la vista de modelo que no aparecen advertencias de relación ambigua ni relaciones inactivas no intencionadas.
- Ocultar las dimensiones técnicas auxiliares si corresponde, y organizar la vista de modelo (diseño en capas: dimensiones arriba, hechos abajo, estilo estrella clásico).
- Confirmar formatos de columna finales (fecha, porcentaje, entero, texto) en todas las tablas.

**Prompt de ejecución:**
```
Actúa como especialista en modelado de datos Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 4.4
(diagrama de relaciones sugerido).

Tarea: sobre el modelo con las 3 Fact (Fase 7) y las 3 Dim (Fase 8) ya
cargadas, crea las relaciones:

- Dim_Calendario[Fecha] (1) -> Fact_CalidadLlamadas[Fecha] (*)
- Dim_Calendario[Fecha] (1) -> Fact_SatisfaccionCapacitacion[Fecha] (*)
- Dim_Calendario[Fecha] (1) -> Fact_MotivacionActividad[Fecha] (*)
- Dim_CallCenter[CallCenter] (1) -> Fact_CalidadLlamadas[CallCenter] (*)
- Dim_CallCenter[CallCenter] (1) -> Fact_SatisfaccionCapacitacion[CallCenter] (*)
- Dim_CallCenter[CallCenter] (1) -> Fact_MotivacionActividad[CallCenter] (*)
- Dim_Jornada[Jornada] (1) -> Fact_SatisfaccionCapacitacion[Jornada] (*)
- Dim_Jornada[Jornada] (1) -> Fact_MotivacionActividad[Jornada] (*)

Todas con dirección de filtro única (Single), sin relaciones activas entre
las 3 Fact entre sí. Organiza la vista de modelo en estilo estrella
(dimensiones arriba, hechos abajo). Confirma que no hay advertencias de
ambigüedad ni relaciones inactivas no intencionadas. Ajusta formatos de
columna finales (fecha, porcentaje, entero, texto) en todas las tablas.

No crees medidas DAX todavía (Fase 10).
```

**Resultado esperado:** Modelo en estrella completo, sin advertencias, con las 8 relaciones activas correctamente configuradas.

**Validaciones necesarias:**
- [ ] Vista de modelo sin íconos de advertencia (relación ambigua, tabla sin relación).
- [ ] Un slicer de `CallCenter` filtra correctamente a las 3 Fact simultáneamente (prueba manual con una tabla visual temporal de validación, eliminada después).
- [ ] `Dim_Calendario` filtra correctamente por fecha a las 3 Fact.

**Riesgos / puntos de control:**
- Verificar que `Fact_CalidadLlamadas` no queda huérfana de `Dim_Jornada` (comportamiento esperado y documentado, no un error).

---

### Fase 10 — Creación de medidas DAX

**Objetivo:** Implementar el catálogo de medidas del documento 01 §5, organizadas en tablas de medidas por área.

**Actividades principales:**
- Crear tablas de medidas vacías: `_Medidas Generales`, `_Medidas Calidad`, `_Medidas Capacitacion`, `_Medidas Motivacion`.
- Implementar las medidas de calidad (`Total Evaluaciones`, `Puntaje Obtenido`, `Puntaje Máximo Aplicable`, `% Calidad Promedio`, `% Llamadas con Venta`, `Objeción Principal`), documentando el supuesto de puntaje máximo por pregunta mientras no se reciba la rúbrica oficial (D3).
- Implementar medidas de satisfacción de capacitación y de motivación (documento 01 §5.2 y §5.3).
- Implementar medidas de conteo base (`n=`) para acompañar cada indicador visualmente.
- Aplicar formato de visualización (porcentaje, decimales, separador de miles) a cada medida.

**Prompt de ejecución:**
```
Actúa como especialista en DAX para Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 5
(catálogo completo de medidas propuestas).

Tarea: sobre el modelo ya relacionado (Fase 9), crea 4 tablas de medidas:
_Medidas Generales, _Medidas Calidad, _Medidas Capacitacion, _Medidas Motivacion.

Implementa las medidas descritas en la sección 5 del documento 01:
- Calidad: Total Evaluaciones, Puntaje Obtenido, Puntaje Máximo Aplicable
  (documenta como supuesto provisional el puntaje máximo por pregunta
  mientras no se confirme la rúbrica oficial - dependencia D3 del plan),
  % Calidad Promedio, % Llamadas con Venta, Objeción Principal (Top).
- Capacitación: Total Respuestas Capacitación, Satisfacción Promedio,
  Claridad Promedio, Utilidad Promedio, Dinamismo Promedio, Índice Global
  Capacitación.
- Motivación: Total Respuestas Motivación, Satisfacción Actividad Promedio,
  Claridad/Utilidad Actividad Promedio, Motivación Promedio, % Ambiente
  Motivado.
- Generales: medidas de conteo base "n=" para acompañar cada indicador en
  tarjetas.

Aplica formato de número adecuado a cada medida (porcentaje con 1 decimal,
promedios con 1-2 decimales, enteros sin decimales). No crees visuales
todavía (eso empieza en Fase 12-14). Documenta cada medida con una
descripción corta en las propiedades del objeto DAX (campo Description).
```

**Resultado esperado:** Catálogo completo de medidas DAX implementado, organizado y documentado con descripciones.

**Validaciones necesarias:**
- [ ] Cada medida definida en documento 01 §5 existe en el modelo.
- [ ] Las medidas de `% Calidad Promedio` y `% Ambiente Motivado` muestran formato porcentaje correcto.
- [ ] Ninguna medida retorna error al colocarla en una tarjeta de prueba.

**Riesgos / puntos de control:**
- La medida `Puntaje Máximo Aplicable` depende del supuesto de rúbrica (D3) — dejar comentario explícito en la medida y en la documentación final (Fase 18) indicando que es provisional.

---

### Fase 11 — Validación de indicadores con los datos actuales

**Objetivo:** Confirmar que las medidas DAX calculan correctamente contra los datos reales, aprovechando que el bajo volumen permite validación manual 100%.

**Actividades principales:**
- Para cada Fact, calcular manualmente (Excel/mental) los valores esperados de las medidas principales y compararlos con lo que muestra Power BI.
- Documentar la validación fila por fila para `Fact_CalidadLlamadas` (solo 3 registros, validación exhaustiva viable).
- Muestreo dirigido (no exhaustivo) para `Fact_SatisfaccionCapacitacion` (32 filas) — validar al menos 5 filas representativas + los totales agregados.
- Validación exhaustiva de `Fact_MotivacionActividad` (5 registros).
- Registrar resultados de la validación (coincide / no coincide / diferencia y causa).

**Prompt de ejecución:**
```
Actúa como especialista en control de calidad de modelos Power BI/DAX.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md secciones
3 y 5 (datos fuente y medidas propuestas).

Tarea: valida las medidas DAX creadas en la Fase 10 contra los datos reales:

1. Fact_CalidadLlamadas (3 filas): calcula manualmente Total Evaluaciones,
   % Calidad Promedio y % Llamadas con Venta a partir de los datos crudos,
   y compáralos contra lo que muestra el modelo. Documenta cualquier
   discrepancia y su causa.
2. Fact_SatisfaccionCapacitacion (32 filas): valida los totales agregados
   (Satisfacción Promedio, Claridad Promedio, Utilidad Promedio, Dinamismo
   Promedio) y realiza validación puntual de al menos 5 filas.
3. Fact_MotivacionActividad (5 filas): validación exhaustiva de las 5 filas
   contra Satisfacción Actividad Promedio, Motivación Promedio y % Ambiente
   Motivado.
4. Produce una tabla de resultados: medida | valor esperado | valor en
   modelo | coincide (sí/no) | observación.

Si encuentras discrepancias, NO corrijas medidas DAX de forma silenciosa:
repórtalas primero para decidir el ajuste correcto.
```

**Resultado esperado:** Tabla de validación con evidencia de que las medidas calculan correctamente (o lista de discrepancias a resolver).

**Validaciones necesarias:**
- [ ] 100% de `Fact_CalidadLlamadas` validado manualmente.
- [ ] 100% de `Fact_MotivacionActividad` validado manualmente.
- [ ] Muestra representativa de `Fact_SatisfaccionCapacitacion` validada.
- [ ] Cero discrepancias sin explicar antes de avanzar a la Fase 12.

**Riesgos / puntos de control:**
- Si aparecen discrepancias por el tratamiento de `"N/A"` (Fase 6), revisar la lógica de `Puntaje Máximo Aplicable` antes de continuar — no avanzar con medidas no validadas al diseño visual.

---

### Fase 12 — Creación de tema visual Connect Assistance

**Objetivo:** Construir el tema JSON de marca para reemplazar el tema por defecto (`CY25SU11`) del reporte.

**Actividades principales:**
- Definir paleta: naranja Connect (placeholder hasta recibir HEX oficial — D6), blanco, negro/gris oscuro, y 2-3 grises intermedios para fondos/bordes.
- Construir el archivo de tema JSON (Report Theme) con colores de datos, fondo, texto, tarjetas KPI y estados (positivo/negativo si aplica).
- Definir tipografía consistente (ej. Segoe UI) y tamaños jerárquicos (título de página, subtítulo, texto de tarjeta KPI, texto de detalle).
- Validar contraste de accesibilidad (naranja como fondo con texto oscuro/blanco, no como texto sobre blanco — ver documento 01 §7).
- Aplicar el tema al reporte reemplazando `CY25SU11`.

**Prompt de ejecución:**
```
Actúa como especialista en diseño visual/UX para Power BI.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 7
(lineamientos visuales Connect).

Tarea: crea un tema JSON de Power BI ("Report Theme") con identidad Connect
Assistance:
- Color primario: naranja Connect (usa un placeholder razonable, ej. #FF6A13,
  y déjalo marcado como PENDIENTE DE VALIDAR con el HEX oficial de marca -
  dependencia D6 del plan de implementación).
- Neutros: blanco (#FFFFFF), negro/gris oscuro (#1A1A1A o similar), 2-3
  grises intermedios para fondos y bordes sutiles.
- Tipografía consistente (Segoe UI o similar) con jerarquía de tamaños para
  título de página, tarjetas KPI y texto de detalle.
- Verifica contraste de accesibilidad AA: el naranja se usa como fondo de
  acento/barras/iconos, no como color de texto sobre fondo blanco.

Guarda el archivo de tema en la carpeta de StaticResources del reporte
(PBI_Indicadores.Report/StaticResources) y actualiza report.json para que
reemplace el tema base actual (CY25SU11) por el nuevo tema Connect.

No construyas todavía las páginas ni visuales (eso son las Fases 13-14).
```

**Resultado esperado:** Tema Connect Assistance aplicado al reporte, reemplazando el tema por defecto.

**Validaciones necesarias:**
- [ ] El tema se aplica sin errores al abrir el reporte en Power BI Desktop.
- [ ] Contraste de texto validado (herramienta de contraste WCAG AA).
- [ ] Colores marcados claramente como "placeholder pendiente de HEX oficial" en la documentación del tema.

**Riesgos / puntos de control:**
- No hardcodear el naranja placeholder en visuales individuales — todo debe heredar del tema para poder actualizar el HEX oficial en un solo lugar cuando el negocio lo confirme (D6).

---

### Fase 13 — Diseño del Home principal tipo landing page

**Objetivo:** Construir la página de inicio del informe como landing page ejecutiva, con navegación visual hacia el resto de páginas.

**Actividades principales:**
- Encabezado con espacio reservado para logo Connect (o logo real si ya se recibió — D6), título del informe y subtítulo de contexto (Connect Assistance — Seguimiento comercial, formativo y de calidad).
- Fila de tarjetas KPI resumen (máx. 4-6): Total evaluaciones de calidad, % Calidad promedio, Satisfacción capacitación promedio, Motivación promedio — cada una con su `n=` visible.
- Tarjetas/botones de navegación visual hacia cada página interna (Resumen ejecutivo, Calidad de llamadas, Satisfacción de capacitaciones, Motivación, Detalle por call center, Detalle por asesor/líder, Notas metodológicas).
- Pie de página con fecha de última actualización y nota breve de "informe en fase piloto".
- Ver recomendaciones detalladas de diseño en §7 de este documento.

**Prompt de ejecución:**
```
Actúa como especialista en UX/UI para dashboards ejecutivos en Power BI.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/02_plan_implementacion_informe_powerbi_connect.md secciones
7 (recomendaciones UX/UI Home) y Specs/01... sección 7 (lineamientos Connect).

Tarea: diseña y construye la página "Home" del reporte como landing page
ejecutiva, usando el tema Connect ya aplicado (Fase 12):

1. Encabezado: espacio reservado para logo Connect (si no existe el archivo
   de logo en el proyecto, deja un placeholder claramente marcado), título
   "Connect Assistance — Seguimiento Comercial, Formativo y de Calidad",
   subtítulo breve de contexto.
2. Fila de 4 a 6 tarjetas KPI resumen (Total evaluaciones de calidad,
   % Calidad promedio, Satisfacción capacitación promedio, Motivación
   promedio), cada una mostrando su conteo base "n=" en texto pequeño.
3. Tarjetas o botones de navegación visual hacia: Resumen ejecutivo, Calidad
   de llamadas, Satisfacción de capacitaciones, Motivación y actividades
   comerciales, Detalle por call center, Detalle por asesor/líder, Notas
   metodológicas. (Las páginas de destino se crean en la Fase 14 - si aún
   no existen, crea primero las páginas vacías con su nombre final para
   poder enlazar la navegación).
4. Pie de página: fecha de última actualización (medida DAX o campo de
   metadatos) y nota "Informe en fase piloto - volumen de datos limitado".

Máximo 6 elementos visuales principales en la página (sin contar botones de
navegación). Sigue el criterio de diseño moderno, limpio, ejecutivo definido
en el documento 01 sección 7.
```

**Resultado esperado:** Página Home funcional, visualmente alineada a Connect, con navegación hacia todas las páginas internas.

**Validaciones necesarias:**
- [ ] Máximo 6 visuales principales (sin contar botones de navegación) en Home.
- [ ] Todos los botones de navegación apuntan a una página válida (aunque esté vacía temporalmente si aún no se construyó en detalle).
- [ ] `n=` visible en cada tarjeta KPI.
- [ ] Espacio de logo presente (real o placeholder claramente identificado).

**Riesgos / puntos de control:**
- No sobrecargar el Home con demasiados KPIs — priorizar los 4-6 más relevantes para una vista ejecutiva de 10 segundos.

---

### Fase 14 — Diseño profesional de páginas internas

**Objetivo:** Construir las páginas de detalle (Resumen ejecutivo, Calidad de llamadas, Satisfacción de capacitaciones, Motivación y actividades comerciales, Detalle por call center, Detalle por asesor/líder) con consistencia visual y máximo 4-6 visuales por página.

**Actividades principales:**
- Aplicar la estructura de páginas definida en documento 01 §6.
- Mantener encabezado consistente en todas las páginas (título + navegación de regreso al Home).
- Ubicar segmentadores (`CallCenter`, `Jornada`, rango de fecha) de forma consistente (misma posición en todas las páginas, ej. barra superior o panel lateral izquierdo).
- Página "Detalle por asesor/líder": construir **solo** con las fuentes que sí permiten ese desglose (Calidad y Capacitación); incluir nota visible de que Motivación no permite desglose por asesor (ver Fase 16).
- Limitar cada página a 4-6 visuales principales, usando tooltips/drillthrough para detalle adicional en vez de saturar.

**Prompt de ejecución:**
```
Actúa como especialista en UX/UI para dashboards ejecutivos en Power BI.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 6
(páginas sugeridas) y Specs/02... sección 8 (recomendaciones páginas internas).

Tarea: construye las siguientes páginas del reporte, con el tema Connect ya
aplicado (Fase 12) y las medidas DAX ya validadas (Fase 11):

1. Resumen ejecutivo: tarjetas KPI top + tendencia en el tiempo + resumen
   por call center (máx. 6 visuales).
2. Calidad de llamadas: % calidad por pregunta del checklist, ranking de
   asesores/líderes, objeciones principales, % llamadas con venta, tabla de
   observaciones filtrable (máx. 6 visuales).
3. Satisfacción de capacitaciones: promedios de las 4 dimensiones Likert,
   evolución en el tiempo, desglose por formador/líder/call center/jornada
   (máx. 6 visuales).
4. Motivación y actividades comerciales: satisfacción/motivación promedio,
   distribución de ambiente de equipo, actividades preferidas, evolución
   por call center/jornada (máx. 6 visuales). Incluye nota visible: "Encuesta
   anónima - sin desglose por asesor" (ver Fase 16 para el texto exacto).
5. Detalle por call center: tabla matriz cruzando las 3 fuentes por call
   center.
6. Detalle por asesor/líder: SOLO con datos de Calidad y Capacitación
   (Motivación no tiene identificación de asesor). Incluye la misma nota
   de limitación donde aplique.

Mantén encabezado consistente (título + botón de regreso al Home) y
segmentadores de CallCenter/Jornada/Fecha en la misma posición en todas las
páginas. Máximo 4-6 visuales principales por página (sin contar
segmentadores ni botones de navegación).
```

**Resultado esperado:** 6 páginas internas construidas, consistentes en estructura y estilo, respetando el límite de visuales por página.

**Validaciones necesarias:**
- [ ] Cada página cumple el máximo de 4-6 visuales principales.
- [ ] Segmentadores en la misma posición/estilo en todas las páginas.
- [ ] Página "Detalle por asesor/líder" no incluye datos de Motivación como si tuviera desglose por asesor.
- [ ] Todos los visuales responden correctamente a los segmentadores (prueba de interacción cruzada).

**Riesgos / puntos de control:**
- Vigilar que las tablas de observaciones/comentarios abiertos no rompan el layout (texto largo) — usar altura de fila controlada o tooltip expandido.

---

### Fase 15 — Configuración de navegación entre páginas

**Objetivo:** Asegurar que la navegación entre Home y páginas internas es fluida, consistente y a prueba de usuarios no técnicos.

**Actividades principales:**
- Configurar acciones de botón (`Page navigation`) en las tarjetas de navegación del Home.
- Agregar botón de "Volver al Home" en el encabezado de cada página interna.
- Ocultar del panel de navegación lateral de Power BI (si se usará solo en pantalla completa) las páginas que no deban navegarse directamente, si aplica.
- Definir la página de inicio del reporte (`activePageName` en `pages.json`) como el Home.
- Probar la navegación completa (ida y vuelta) desde cada página.

**Prompt de ejecución:**
```
Actúa como especialista en Power BI (UX de navegación).
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores

Tarea: configura la navegación completa del reporte:

1. En el Home, configura cada tarjeta/botón de navegación con acción
   "Page navigation" hacia su página correspondiente.
2. En cada página interna, agrega un botón "Volver al Home" en el
   encabezado con la misma acción de navegación.
3. Define el Home como página de inicio del reporte (activePageName en
   definition/pages/pages.json).
4. Prueba manualmente la navegación completa: desde Home hacia cada página
   y de regreso, verificando que no queden páginas huérfanas sin acceso.

No modifiques el contenido de los visuales de cada página, solo la capa de
navegación (botones y configuración de página activa).
```

**Resultado esperado:** Navegación completa y funcional entre Home y las 6 páginas internas, con retorno consistente.

**Validaciones necesarias:**
- [ ] Todos los botones de navegación funcionan sin error.
- [ ] El reporte abre siempre en el Home.
- [ ] Ninguna página queda inaccesible desde la navegación visual.

**Riesgos / puntos de control:**
- Si se usa el panel de navegación lateral nativo de Power BI además de los botones, verificar que no generen confusión (decidir un solo patrón de navegación predominante).

---

### Fase 16 — Notas metodológicas y advertencias por bajo volumen de datos

**Objetivo:** Dejar explícitas, dentro del propio informe, las limitaciones de datos identificadas en el diagnóstico, para que ningún usuario de negocio interprete los KPIs como definitivos.

**Actividades principales:**
- Crear la página "Notas metodológicas" con: fecha de corte de datos, volumen total de respuestas por fuente (`n=`), explicación de por qué el volumen es piloto, y explicación de la limitación de la encuesta de motivación (sin identificación de asesor).
- Agregar textos/íconos de advertencia visibles (no solo en la página de notas) en las tarjetas KPI de bajo volumen y en la página de Motivación, ej. tooltip o texto fijo: *"Encuesta anónima — no permite análisis por asesor individual"*.
- Documentar el supuesto de rúbrica de puntaje (D3) en la página de notas metodológicas, si para ese momento el negocio aún no lo ha confirmado.
- Revisar consistencia de mensaje en todas las páginas afectadas.

**Prompt de ejecución:**
```
Actúa como especialista en documentación funcional dentro de dashboards
Power BI.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md sección 8
(riesgos y consideraciones).

Tarea: construye la página "Notas metodológicas" del reporte con:

1. Fecha de corte de los datos y volumen total de respuestas por fuente
   (n= de Fact_CalidadLlamadas, Fact_SatisfaccionCapacitacion,
   Fact_MotivacionActividad), usando las medidas de conteo base ya creadas
   (Fase 10).
2. Texto explicando que el informe está en fase piloto y los indicadores
   deben interpretarse con cautela dado el bajo volumen.
3. Texto explicando que la encuesta de motivación es anónima y no permite
   desglose por asesor individual.
4. Si para este momento la dependencia D3 (rúbrica de puntaje máximo por
   pregunta de calidad) sigue sin confirmar, documenta el supuesto usado.

Además, agrega un texto/ícono de advertencia visible (no solo en esta
página) en:
- La página "Motivación y actividades comerciales": nota fija "Encuesta
  anónima - no permite análisis por asesor individual".
- Las tarjetas KPI del Home y Resumen ejecutivo: texto pequeño "n=" con el
  conteo base de cada indicador.

Revisa que el mensaje sea consistente en tono y redacción en todas las
páginas donde aparece.
```

**Resultado esperado:** Página de notas metodológicas completa; advertencias visibles y consistentes en todo el informe.

**Validaciones necesarias:**
- [ ] Página de notas metodológicas construida y accesible desde el Home.
- [ ] Advertencia de encuesta anónima visible en la página de Motivación.
- [ ] `n=` visible en todas las tarjetas KPI principales (Home, Resumen ejecutivo, y páginas de detalle).

**Riesgos / puntos de control:**
- Evitar que las advertencias se vean como "ruido" visual — usar iconografía sutil (ej. ícono de información) con texto expandible en tooltip, en vez de bloques de texto grandes que rompan el diseño ejecutivo.

---

### Fase 17 — Validaciones técnicas, funcionales y visuales

**Objetivo:** Ejecutar una pasada de control de calidad integral antes de considerar el informe listo, cubriendo las 3 dimensiones: técnica, funcional y visual.

**Actividades principales:**
- **Técnica:** revisar el modelo (sin advertencias de relaciones), revisar consultas Power Query (sin errores, sin pasos huérfanos), revisar que el PBIP completo abre sin errores en una sesión limpia de Power BI Desktop.
- **Funcional:** verificar que cada medida definida en el documento 01 §5 está presente, correctamente calculada (reutilizando la validación de Fase 11) y responde a los segmentadores.
- **Visual:** verificar consistencia de tema, tipografía, colores, alineación de tarjetas, máximo de visuales por página, y comportamiento responsivo básico (redimensionar ventana de Power BI Desktop).
- Registrar hallazgos en una checklist de cierre y corregir antes de pasar a la Fase 18.

**Prompt de ejecución:**
```
Actúa como especialista en control de calidad (QA) de proyectos Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md y
Specs/02_plan_implementacion_informe_powerbi_connect.md (fases 1 a 16 ya
ejecutadas).

Tarea: ejecuta una validación integral de cierre en 3 dimensiones:

1. Técnica: revisa el modelo semántico completo (sin advertencias de
   relaciones ambiguas o tablas sin relación), revisa que todas las
   consultas Power Query cargan sin error y sin pasos huérfanos o
   duplicados, y confirma que el PBIP completo abre sin errores en una
   sesión limpia de Power BI Desktop.
2. Funcional: confirma que todas las medidas del catálogo (Specs/01 sección
   5) están presentes y responden correctamente a los segmentadores de
   CallCenter, Jornada y Fecha en todas las páginas.
3. Visual: confirma consistencia del tema Connect en todas las páginas
   (colores, tipografía), que ninguna página excede 6 visuales principales,
   que la navegación funciona en todas direcciones, y que las advertencias
   de bajo volumen/encuesta anónima son visibles donde corresponde.

Entrega una checklist de resultados (aprobado/pendiente) y una lista de
hallazgos a corregir antes de dar por cerrada la implementación.
```

**Resultado esperado:** Checklist de QA con resultado aprobado (o lista puntual de correcciones pendientes) en las 3 dimensiones.

**Validaciones necesarias:**
- [ ] Cero advertencias técnicas en el modelo.
- [ ] 100% de medidas del catálogo presentes y funcionales.
- [ ] 100% de páginas cumplen el estándar visual (tema, límite de visuales, navegación).

**Riesgos / puntos de control:**
- No pasar a la Fase 18 (documentación de cierre) si quedan hallazgos críticos sin resolver — esta fase es un punto de control duro (gate), no solo informativo.

---

### Fase 18 — Documentación final del proceso

**Objetivo:** Dejar registro trazable de todo lo construido, para negocio y para futura mantenibilidad del informe.

**Actividades principales:**
- Crear [Specs/03_documentacion_final_informe_powerbi_connect.md](03_documentacion_final_informe_powerbi_connect.md) con: inventario final de tablas/columnas, catálogo final de medidas DAX (con descripciones), mapa de páginas y navegación, decisiones tomadas durante la implementación (incluyendo las de las Fases 6, 10 y 12 marcadas como "provisional" mientras D3/D4/D5/D6 no se confirmen con negocio), y guía de actualización de datos (cómo y cuándo reexportar los Google Forms).
- Actualizar el estado de las dependencias del §4 de este plan (cuáles se resolvieron durante la ejecución, cuáles siguen pendientes).
- Commit final en Git (si se adoptó versionamiento en Fase 2) con mensaje de cierre de la implementación inicial.

**Prompt de ejecución:**
```
Actúa como especialista en documentación técnica de proyectos Power BI/PBIP.
Proyecto: C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores
Referencia: Specs/01_analisis_de_impacto_informe_powerbi_connect.md y
Specs/02_plan_implementacion_informe_powerbi_connect.md (todas las fases
1 a 17 ya ejecutadas).

Tarea: crea Specs/03_documentacion_final_informe_powerbi_connect.md con:

1. Inventario final del modelo: tablas de hechos, dimensiones, columnas
   técnicas por tabla y su origen (columna original del Excel).
2. Catálogo final de medidas DAX con su descripción y tabla de medidas a
   la que pertenece.
3. Mapa de páginas del reporte y su navegación (diagrama simple en texto o
   tabla).
4. Registro de decisiones tomadas durante la implementación, marcando
   explícitamente cuáles son provisionales por depender de confirmación de
   negocio (dependencias D3, D4, D5, D6 del plan de implementación) y cuál
   es el impacto de actualizarlas más adelante.
5. Guía operativa de actualización de datos: cómo reexportar los 3 Google
   Forms, dónde colocar los archivos, y cómo actualizar el modelo en Power
   BI Desktop.
6. Actualiza el estado de las dependencias D1-D8 del plan de implementación
   (Specs/02, sección 4): cuáles se resolvieron, cuáles siguen pendientes.

Si el proyecto está versionado en Git (Fase 2), propone un mensaje de commit
de cierre para esta primera implementación completa, y espera confirmación
antes de ejecutarlo.
```

**Resultado esperado:** Documentación de cierre completa (`Specs/03_...`), estado de dependencias actualizado, commit de cierre (si aplica).

**Validaciones necesarias:**
- [ ] [Specs/03_documentacion_final_informe_powerbi_connect.md](03_documentacion_final_informe_powerbi_connect.md) creado y completo.
- [ ] Todas las decisiones provisionales quedan identificadas explícitamente.
- [ ] Guía de actualización de datos es ejecutable por alguien no técnico (el PUSHER u otro usuario de negocio).

**Riesgos / puntos de control:**
- Esta documentación debe mantenerse viva — cualquier cambio futuro al modelo/DAX/páginas debería reflejarse aquí o en un documento posterior versionado (`04_...`, etc.), no sobrescribiendo el historial.

---

## 6. Orden recomendado de ejecución

Las fases están diseñadas para ejecutarse **secuencialmente** (1 → 18), ya que cada una depende del estado dejado por la anterior sobre el mismo PBIP. Excepciones donde hay margen de paralelismo:

- **Fase 2 (versionamiento)** puede ejecutarse en paralelo con la Fase 1, ya que no depende del contenido del modelo.
- **Fase 12 (tema visual)** puede prepararse (diseño del JSON) en paralelo con las Fases 4-9 (Power Query/modelado), pero su *aplicación* al reporte requiere que el reporte ya tenga al menos una página base.
- **Fases 4, 5 y 6** (limpieza, normalización, tratamiento de nulos) en la práctica suelen ejecutarse en una sola sesión de Power Query encadenada; se mantienen separadas en este plan por trazabilidad y para permitir puntos de control independientes, no porque deban ser sesiones distintas obligatoriamente.

No se recomienda saltar fases ni ejecutar 13-16 (diseño de páginas) antes de que la Fase 11 (validación de indicadores) esté aprobada — construir visuales sobre medidas no validadas obliga a rehacer trabajo.

---

## 7. Recomendaciones UX/UI — Home (landing page)

- **Estructura tipo hero + navegación:** franja superior con logo Connect (o placeholder) + título + subtítulo de contexto, seguida de una fila de tarjetas KPI, seguida de una cuadrícula de tarjetas de navegación hacia las demás páginas.
- **Jerarquía visual:** el número del KPI debe ser el elemento más grande y contrastante de cada tarjeta; la etiqueta descriptiva y el `n=` van en texto pequeño debajo, en gris.
- **Uso del naranja:** reservarlo para acentos (borde superior de tarjeta, ícono, botón activo), no como fondo completo de grandes áreas — mantiene la sensación "limpia" que pide el negocio.
- **Tarjetas de navegación:** usar íconos simples y consistentes por tema (calidad, capacitación, motivación, call center, asesor, notas) para que la navegación sea reconocible de un vistazo.
- **Espacio de logo:** si el logo real aún no está disponible (dependencia D6), dejar un rectángulo placeholder con texto "LOGO CONNECT" en gris claro, fácil de ubicar y reemplazar después.
- **Pie de página:** fecha de corte de datos + nota de "fase piloto" en texto pequeño, sin competir visualmente con los KPIs principales.
- **Densidad:** máximo 4-6 tarjetas KPI + máximo 6-7 tarjetas de navegación; evitar que el Home se sienta como un dashboard analítico — su función es orientar, no analizar.

---

## 8. Recomendaciones UX/UI — páginas internas

- **Encabezado consistente:** mismo alto, misma tipografía de título, mismo botón "Volver al Home" en la misma posición (ej. esquina superior izquierda) en las 6 páginas internas.
- **Zona de filtros fija:** segmentadores de `CallCenter`, `Jornada` y rango de `Fecha` siempre en la misma ubicación (recomendado: barra superior debajo del encabezado, o panel lateral izquierdo angosto) para que el usuario no tenga que reaprender la interfaz al cambiar de página.
- **Límite de 4-6 visuales:** priorizar 1-2 KPIs destacados + 1 gráfico de tendencia + 1-2 gráficos de desglose (categoría/ranking) + máximo 1 tabla de detalle.
- **Texto libre/observaciones:** presentarlo en tabla con altura de fila controlada y scroll interno, o mediante drillthrough a una página de detalle dedicada, para no romper el layout de 4-6 visuales.
- **Notas de limitación:** usar un ícono de información (ⓘ) pequeño junto al título de la página o del visual afectado, con tooltip expandible, en vez de bloques de texto permanentes.
- **Color por categoría:** si se usa el naranja para resaltar una categoría (ej. call center con menor desempeño), reservarlo para ese propósito de alerta/foco, no como color decorativo genérico de toda la paleta categórica (usar grises/naranjas tonales para el resto de categorías).

---

## 9. Criterios de cierre

La implementación se considera **finalizada** cuando se cumplen todos los siguientes criterios:

- [ ] Las 18 fases fueron ejecutadas y sus validaciones respectivas aprobadas.
- [ ] El modelo semántico carga sin advertencias ni errores en una sesión limpia de Power BI Desktop.
- [ ] Las medidas DAX del catálogo (documento 01 §5) están implementadas, documentadas y validadas contra los datos reales (Fase 11).
- [ ] Las 8 páginas del informe (Home + 6 internas + Notas metodológicas) están construidas, respetando el máximo de 4-6 visuales principales por página.
- [ ] El tema visual Connect está aplicado de forma consistente en todo el informe (con colores placeholder claramente marcados si D6 sigue pendiente).
- [ ] La navegación entre Home y páginas internas funciona en ambas direcciones sin páginas huérfanas.
- [ ] Las advertencias de bajo volumen (`n=`) y de limitación de la encuesta anónima de motivación son visibles donde corresponde.
- [ ] La documentación final (`Specs/03_...`) está completa y refleja el estado real del modelo, incluyendo decisiones provisionales pendientes de confirmación de negocio.
- [ ] El proyecto está versionado en Git con al menos un commit de cierre (si se adoptó la Fase 2).

---

## 10. Próximos pasos sugeridos

1. **Revisar y aprobar este plan** antes de iniciar la Fase 1.
2. **Resolver en paralelo las dependencias D3, D4, D5 y D6** (§4) con el negocio/PUSHER, ya que varias fases (10, 12, 16) avanzan con supuestos provisionales mientras estas no se confirmen — no bloquean el inicio, pero sí la versión "definitiva" del informe.
3. Decidir si se adopta versionamiento Git (Fase 2) antes de tocar el PBIP — recomendado dado que se realizarán múltiples cambios estructurales.
4. Ejecutar las fases en orden, usando el prompt de cada una como punto de partida de la sesión de trabajo correspondiente.
5. Al cerrar la Fase 18, evaluar como iniciativa futura (fuera de este plan): conexión automática a la fuente de datos, `Dim_Colaborador` maestro con ID único, RLS por call center/líder, y publicación en Power BI Service.

---

*Este documento es un plan de implementación. No se realizaron cambios en `PBI_Indicadores.pbip`, el modelo semántico, medidas, consultas ni visuales del reporte. La ejecución de cada fase debe hacerse de forma controlada, usando el prompt correspondiente y validando antes de avanzar a la siguiente.*
