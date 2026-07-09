# Resultado — Fase 3: Ingesta de archivos desde la carpeta `Data`

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase ejecutada | Fase 3 — Ingesta de archivos desde la carpeta `Data` (ver [Specs/02_plan_implementacion_informe_powerbi_connect.md](../Specs/02_plan_implementacion_informe_powerbi_connect.md)) |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/01...`, `Outputs/02...` |
| Fecha | 2026-07-08 |
| Archivos modificados | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (nuevo) |

---

## Nota metodológica importante (léase antes del resto del documento)

Esta ejecución **no se realizó abriendo Power BI Desktop de forma interactiva** (no dispongo de control sobre su interfaz gráfica). En su lugar, el parámetro y las 3 consultas de staging se crearon **editando directamente el archivo TMDL** del modelo semántico (`definition/expressions.tmdl`), siguiendo la sintaxis oficial de TMDL/Power Query documentada por Microsoft (verificada contra la documentación oficial de TMDL antes de escribir el archivo, para minimizar el riesgo de un TMDL inválido).

Esto tiene una consecuencia directa sobre el punto "vista previa validada" que pedía la tarea: **no pude generar una vista previa real ejecutada por el motor de Power Query** (eso requiere el motor de Power BI Desktop, que no puedo invocar desde este entorno). Lo que sí hice, como validación equivalente disponible:

- Verificación estructural byte-exacta de que los 3 nombres de archivo referenciados en el código M coinciden exactamente con los archivos reales en `Data/` (evita el error más común: nombre de archivo mal escrito).
- Verificación de que la hoja `Form Responses 1` existe en los 3 libros (con `openpyxl`, releyendo los archivos reales).
- Verificación de la sintaxis TMDL contra la documentación oficial (estructura de carpetas, reglas de indentación, forma correcta de declarar un parámetro M vía `meta [IsParameterQuery=true, ...]` y una expresión compartida no cargada al modelo).

**Queda pendiente, como validación complementaria recomendada antes de la Fase 4:** abrir el PBIP en Power BI Desktop una vez, dejar que cargue el modelo y confirmar en el editor de Power Query que las 3 consultas muestran vista previa sin error. Esto se detalla en la sección de riesgos al final de este documento.

---

## Estado inicial de `git status`

Se ejecutó `git status` antes de tocar cualquier archivo. Resultado: `On branch master / nothing to commit, working tree clean` — confirmando el cierre limpio de la Fase 2.

## Confirmación de accesibilidad de los 3 archivos Excel

Se repitió la prueba de apertura en modo lectura compartida (la misma usada en la Fase 1) sobre los 3 archivos justo antes de editar el modelo:

| Archivo | Estado |
|---|---|
| `Data/Matriz de calidad (Responses).xlsx` | Accesible, sin bloqueo |
| `Data/Satisfacción capacitación (Responses).xlsx` | Accesible, sin bloqueo |
| `Data/Encuesta satisfacción (Responses).xlsx` | Accesible, sin bloqueo |

## Parámetro creado en Power Query

- **Nombre:** `RutaCarpetaData`
- **Tipo:** Texto (`Type="Text"`), parámetro requerido (`IsParameterQueryRequired=true`)
- **Valor por defecto / actual:** `C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores\Data`
- **Representación TMDL** (objeto `expression` con metadato `IsParameterQuery=true`, forma estándar que usa Power BI Desktop para parámetros de Power Query):

```tmdl
expression RutaCarpetaData = "C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores\Data" meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]
	lineageTag: 53085232-a62f-43c5-997d-4ad091d4aacd
	annotation PBI_ResultType = Text
```

## Consultas base creadas

Las 3 consultas se crearon como **expresiones compartidas de M** (`expression`), no como tablas (`table`) — esta es la representación TMDL correcta para consultas con la carga al modelo deshabilitada (ver decisión más abajo). Se agruparon bajo `queryGroup: Staging` para que aparezcan organizadas en una carpeta "Staging" del panel de consultas.

| Consulta | Archivo origen | Hoja navegada |
|---|---|---|
| `Base_MatrizCalidad` | `Matriz de calidad (Responses).xlsx` | `Form Responses 1` |
| `Base_SatisfaccionCapacitacion` | `Satisfacción capacitación (Responses).xlsx` | `Form Responses 1` |
| `Base_EncuestaMotivacion` | `Encuesta satisfacción (Responses).xlsx` | `Form Responses 1` |

Código M usado (idéntico patrón en las 3, solo cambia el nombre de archivo), por ejemplo `Base_MatrizCalidad`:

```m
let
    Source = Excel.Workbook(File.Contents(RutaCarpetaData & "\Matriz de calidad (Responses).xlsx"), null, true),
    #"Form Responses 1_Sheet" = Source{[Item="Form Responses 1",Kind="Sheet"]}[Data]
in
    #"Form Responses 1_Sheet"
```

No se aplicó `Table.PromoteHeaders`, ni renombrado, ni cambio de tipos, ni ningún otro paso: la consulta se detiene justo después de navegar a la hoja, devolviendo la tabla cruda tal como la expone `Excel.Workbook`. La promoción de encabezados queda explícitamente reservada para la Fase 4 (así lo define `Specs/02` en las actividades de esa fase), para mantener trazabilidad de qué transformación se aplicó en qué fase/commit.

## Ruta lógica usada para la carpeta `Data`

Se usó el parámetro `RutaCarpetaData` (ver arriba) en vez de rutas absolutas repetidas en cada consulta. Las 3 consultas concatenan `RutaCarpetaData & "\<nombre de archivo>.xlsx"`. Esto centraliza la ruta en un solo lugar: si el proyecto se mueve de ubicación, basta con actualizar el valor del parámetro una vez.

**Limitación conocida (ya anticipada en `Specs/02`, dependencia de portabilidad):** el valor del parámetro es una ruta absoluta, no relativa al PBIP. Power Query M no tiene una forma nativa y confiable de obtener "la carpeta donde vive este PBIP" en un proyecto TMDL. Si otra persona clona este repositorio en otra máquina/ruta, deberá actualizar manualmente el valor de `RutaCarpetaData` en Power BI Desktop (Inicio > Editor de Power Query > Administrar parámetros) antes de poder actualizar el modelo.

## Hoja cargada en cada archivo

`Form Responses 1` en los 3 archivos — confirmado por doble vía: el diagnóstico original (`Specs/01`, sección 2.2) y una relectura fresca con `openpyxl` justo antes de escribir el TMDL (ver sección metodológica arriba).

## Vista previa validada o errores encontrados

- **No se pudo validar una vista previa ejecutada por el motor real de Power Query** (ver nota metodológica). No hay forma de invocar Power BI Desktop de forma headless desde este entorno.
- **Validación estructural realizada (sí ejecutada y con resultado exitoso):**
  - Los 3 nombres de archivo en el código M coinciden byte a byte con los archivos reales en `Data/`.
  - La hoja `Form Responses 1` existe en los 3 libros.
  - La sintaxis TMDL de `expressions.tmdl` sigue las reglas oficiales de indentación, delimitadores y declaración de expresiones documentadas por Microsoft (Tabular Model Definition Language).
- **No se encontraron errores** en ninguna de las validaciones realizadas.
- **Pendiente de confirmación manual:** abrir el PBIP en Power BI Desktop y verificar en el editor de Power Query que las 3 consultas (`Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion`, `Base_EncuestaMotivacion`, agrupadas en "Staging") muestran vista previa sin error, y que el parámetro `RutaCarpetaData` aparece correctamente en "Administrar parámetros".

## Decisión sobre habilitar o deshabilitar carga al modelo

**Deshabilitada (no se cargan al modelo).** Al representarse como `expression` en TMDL (en vez de `table`), estas 3 consultas **no existen como tablas del modelo semántico** — es la forma nativa en que TMDL/Power BI representa una consulta con "Habilitar carga" desactivado. No aparecerán en el panel de campos del informe ni podrán usarse en visuales hasta que, en una fase posterior, se conviertan en tablas cargadas (Fase 7 del plan). Esto cumple exactamente la instrucción de dejarlas como staging.

## Archivos modificados en el PBIP

- **Nuevo:** `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (parámetro `RutaCarpetaData` + 3 consultas `Base_*`).
- **Sin cambios:** `model.tmdl`, `database.tmdl`, `definition.pbism`, todo el árbol de `PBI_Indicadores.Report/` (no se tocaron páginas, visuales, tema ni medidas — confirmado, ver `git status` a continuación).

No fue necesario modificar `model.tmdl` para referenciar las nuevas expresiones: a diferencia de tablas/roles/culturas/perspectivas (que sí requieren una línea `ref` en `model.tmdl` por vivir cada una en su propio archivo), las expresiones de Power Query viven todas juntas en un único archivo raíz (`expressions.tmdl`) que Power BI Desktop descubre automáticamente — así lo confirma la documentación oficial de TMDL.

## Resultado del commit de la Fase 3

- Mensaje: `data(powerquery): conectar archivos base desde carpeta Data` (con cuerpo describiendo el alcance exacto: parámetro + 3 consultas de staging, sin limpieza/tablas/medidas/visuales).
- Archivos incluidos: `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (nuevo), `Outputs/03_resultado_fase_3_ingesta_archivos_data.md` (este documento).
- Hash del commit: `aa13cb202369d691180076ecee50743f85436099`. (Nota: el commit se generó en dos pasos —`git commit` seguido de un `git commit --amend` para incorporar este mismo documento ya finalizado— por lo que el hash cambió una vez respecto al registrado inicialmente; el historial sigue siendo un único commit, no publicado, sin riesgo de romper referencias de terceros.)
- Autor/committer: `HarvLopez91 <eclavijo29@gmail.com>` (identidad local del proyecto, ya configurada en la Fase 2).
- No se realizó `push` a ningún remoto. No se usaron banderas de bypass de hooks ni de firma.

## Estado final de `git status`

`On branch master / nothing to commit, working tree clean` — confirmado tras el commit.

## Riesgos o recomendaciones antes de avanzar a Fase 4

- **Validación pendiente en Power BI Desktop (la más importante):** antes de iniciar la Fase 4 (limpieza), abrir este PBIP en Power BI Desktop al menos una vez y confirmar en el editor de Power Query que las 3 consultas cargan vista previa sin error. Si Power BI Desktop reporta algún error de sintaxis M o de acceso a archivo que las validaciones estructurales de este documento no pudieron anticipar, corregirlo ahí antes de continuar.
- **Ruta absoluta del parámetro:** si el proyecto se clona/mueve a otra máquina o ruta, actualizar `RutaCarpetaData` manualmente antes de refrescar.
- **Riesgo operativo heredado (Fases 1 y 2):** mantener los 3 archivos de `Data` cerrados al momento de refrescar en Power BI Desktop.
- **Alcance respetado:** no se crearon tablas `Fact_`, dimensiones, relaciones, medidas DAX, páginas ni visuales — verificado que `PBI_Indicadores.Report/` no tiene cambios y que `expressions.tmdl` es el único archivo nuevo.
- Con esta salvedad de validación pendiente en Power BI Desktop documentada explícitamente, **no hay bloqueos estructurales para preparar la Fase 4** (limpieza y transformación), pero se recomienda no iniciarla hasta confirmar la vista previa real.

---

*Documento generado como registro operativo de la Fase 3, según la regla documental vigente: los resultados de ejecución de fases se documentan en `Outputs/`, mientras que el diagnóstico y el plan permanecen en `Specs/`.*
