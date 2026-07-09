# Fuentes de datos y guía de actualización

Guía operativa para reexportar, colocar y actualizar las 3 fuentes del informe. Escrita para que la pueda ejecutar una persona sin conocimientos técnicos de Power BI (por ejemplo, el rol PUSHER responsable de exportar los formularios).

## 1. Las 3 fuentes

El modelo se alimenta de **3 archivos Excel exportados desde Google Forms**, ubicados en la carpeta `Data/` en la raíz del proyecto. `Data/` está excluida de git (`.gitignore`) porque contiene nombres de personas reales — solo se respalda vía sincronización de OneDrive, no por control de versiones.

| Archivo esperado en `Data/` | Encuesta que contiene | Hoja usada | Tabla de hechos que alimenta |
|---|---|---|---|
| `Matriz de calidad (Responses).xlsx` | Auditoría de calidad de llamadas (checklist del rol PUSHER) | `Form Responses 1` | `Fact_CalidadLlamadas` |
| `Satisfacción capacitación (Responses).xlsx` | Encuesta de satisfacción post-capacitación | `Form Responses 1` | `Fact_SatisfaccionCapacitacion` |
| `Encuesta satisfacción (Responses).xlsx` | Encuesta de satisfacción/motivación de actividades comerciales | `Form Responses 1` | `Fact_MotivacionActividad` |

El nombre exacto del archivo y de la hoja debe coincidir con lo anterior — Power Query los referencia por nombre exacto (ver §5 si cambian).

## 2. Cómo reexportar desde Google Forms

1. Abrir el Google Form correspondiente (calidad, capacitación o motivación) en Google Drive.
2. Ir a la pestaña **Respuestas** del formulario.
3. Usar la opción **Exportar a Hojas de cálculo** (o abrir la hoja de cálculo vinculada ya existente).
4. Desde Google Sheets: **Archivo → Descargar → Microsoft Excel (.xlsx)**.
5. Guardar el archivo descargado con el **mismo nombre exacto** listado en la tabla del §1, reemplazando la versión anterior dentro de `Data/`.

## 3. Mantener los archivos cerrados antes de actualizar

**Antes de refrescar el modelo en Power BI Desktop, los 3 archivos de `Data/` deben estar cerrados en Excel.** Si alguno permanece abierto (por ejemplo, si Excel o OneDrive lo tienen bloqueado), la actualización falla con un error del tipo *"The process cannot access the file because it is being used by another process"*. Cerrar el archivo y reintentar la actualización resuelve el error — no es necesario reiniciar Power BI Desktop.

## 4. Cómo refrescar en Power BI Desktop

1. Abrir `PBI/PBI_Indicadores.pbip` con Power BI Desktop (requiere la característica **Power BI Project (.pbip)** habilitada).
2. Confirmar que los 3 archivos de `Data/` están cerrados (§3).
3. En la cinta **Inicio**, usar **Actualizar** (actualiza las 3 fuentes y recalcula `Dim_Calendario`, `Dim_CallCenter` y `Dim_Jornada`, que se reconstruyen dinámicamente en cada actualización).
4. Guardar el archivo (`Ctrl+S`) para que Power BI Desktop reescriba los archivos TMDL/JSON del proyecto con cualquier metadato automático generado (ver §6 de este documento y [AGENTS.md](../AGENTS.md)).
5. Ejecutar `git status` y `git diff` para revisar qué cambió antes de comitear — Power BI Desktop reescribe archivos incluso cuando el cambio de negocio es solo un refresco de datos (ver [06_publicacion_powerbi.md](06_publicacion_powerbi.md) para el flujo posterior de republicación).

## 5. Qué hacer si cambian columnas o nombres de hoja

Si Google Forms agrega, quita o renombra una pregunta (columna) del formulario, o si el nombre de la hoja de respuestas cambia:

1. **No editar el archivo Excel manualmente** para forzar que coincida con el modelo — en su lugar, ajustar la transformación en Power Query.
2. Abrir el editor de Power Query en Power BI Desktop y ubicar la consulta `Base_<Fuente>` correspondiente (`Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion` o `Base_EncuestaMotivacion`, definidas en `expressions.tmdl`).
3. Si cambió el **nombre de la hoja**: actualizar el paso `Source{[Item="Form Responses 1",Kind="Sheet"]}` con el nombre nuevo.
4. Si cambió el **nombre de una columna/pregunta**: actualizar el paso `Table.RenameColumns` correspondiente en la consulta `Fact_<Nombre>` (ver el mapeo completo columna original → columna técnica en [Docs/01_modelo_datos.md](01_modelo_datos.md)).
5. Si se **agregó una pregunta nueva**: decidir si debe incorporarse al modelo (nueva columna técnica) y, si aplica, documentar la decisión en un nuevo archivo `Outputs/NN_...md`, siguiendo el patrón ya establecido en este proyecto.
6. No escribir `lineageTag`, `description` ni `queryGroup` a mano en ningún archivo `.tmdl` nuevo o editado — dejar que Power BI Desktop los genere al guardar.

## 6. Qué validar después de actualizar

Antes de dar por buena una actualización de datos (y antes de republicar, si aplica — ver [06_publicacion_powerbi.md](06_publicacion_powerbi.md)), revisar:

- [ ] **Conteos**: las tarjetas `Total Evaluaciones Calidad`, `Total Respuestas Capacitacion`, `Total Respuestas Motivacion` y `Total Registros Piloto` muestran un número mayor o igual al anterior (nunca deberían bajar salvo que se haya depurado una fila a propósito).
- [ ] **Segmentadores**: el segmentador de Call Center muestra todos los call centers esperados (incluyendo cualquiera nuevo que haya aparecido); el de Fecha cubre hasta la fecha más reciente cargada.
- [ ] **KPIs**: ningún KPI muestra `#ERROR` ni una referencia rota; los promedios/porcentajes cambian de forma razonable respecto a la actualización anterior.
- [ ] **Gráficos**: los gráficos de barras/columnas/líneas de las páginas de detalle muestran las etiquetas de datos y reflejan los nuevos valores.
- [ ] **Notas metodológicas**: las 3 leyendas dinámicas `n=` (calidad, capacitación, motivación) reflejan el nuevo tamaño de muestra — no se debe escribir el número a mano en ningún texto del informe.
- [ ] **Publicación**: si el informe está publicado ([Docs/06_publicacion_powerbi.md](06_publicacion_powerbi.md)), decidir si corresponde republicar para que la versión pública refleje los datos actualizados.

No se incluyen en este documento datos personales de asesores/líderes ni ejemplos con nombres reales — solo nombres de columnas y de archivos.
