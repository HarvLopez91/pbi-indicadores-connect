# Resultado — Migración de orígenes de datos a rutas web de OneDrive

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de tarea | Cambio controlado de Power Query (orígenes de datos únicamente) |
| Archivo modificado | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` |
| Fecha | 2026-07-22 |
| Alcance | Solo orígenes de datos (Power Query). No se modificaron medidas DAX, relaciones, visuales, páginas, tema ni `Data/*.xlsx`. |

---

## 1. Estado inicial de `git status`

```
?? Data/
?? PBI/PBI_Indicadores.SemanticModel/DAXQueries/
```

Working tree limpio salvo los 2 elementos ya conocidos y sin relación con esta tarea (`Data/` excluida por `.gitignore`; `DAXQueries/.pbi/daxQueries.json`, metadato local de Power BI Desktop dejado intencionalmente sin versionar desde la sesión de validación de `SC-4`). Rama `main`.

## 2. Revisión de las consultas Power Query actuales

Todo el acceso a los 3 archivos Excel de origen vive en un único archivo: `expressions.tmdl`. Antes del cambio, la cadena era:

- **Parámetro** `RutaCarpetaData` (tipo texto) = `C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores\Data` — ruta local absoluta, con el nombre de usuario del equipo original del proyecto (`edwin.clavijo`), no el actual (`eclavijo`). Ya era una dependencia frágil: solo funciona si el proyecto se abre en una máquina donde esa ruta exacta existe (o se reconfigura manualmente el parámetro en Power BI Desktop).
- **3 consultas de staging** (`Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion`, `Base_EncuestaMotivacion`), cada una con el patrón:
  ```
  Source = Excel.Workbook(File.Contents(RutaCarpetaData & "\<nombre archivo>.xlsx"), null, true)
  ```
  `File.Contents` solo puede leer un archivo accesible por ruta de sistema de archivos local — no funciona contra una URL, y no es compatible con actualización programada en Power BI Service sin una puerta de enlace (gateway) de datos locales.
- Las consultas `*_Limpio` (`MatrizCalidad_Limpio`, `SatisfaccionCapacitacion_Limpio`, `EncuestaMotivacion_Limpio`) y las particiones `Fact_*` **no referencian `RutaCarpetaData` directamente** — solo consumen `Base_*` como entrada. Confirmado con `grep` que `RutaCarpetaData`/`File.Contents`/`Web.Contents` solo aparecen en `expressions.tmdl` (y una mención de solo lectura en `model.tmdl`, la lista de orden de consultas del panel, que no requiere edición manual).

**Conclusión de la revisión:** el único punto de cambio necesario son las 3 consultas `Base_*` y el parámetro de ruta — nada más en el modelo depende de la ubicación física del archivo.

## 3. Propuesta de la forma más segura de reemplazo

Se evaluaron 2 enfoques:

1. **Reescribir `RutaCarpetaData` como una ruta base web** y seguir concatenando el nombre de archivo (`RutaCarpetaData & "\<archivo>.xlsx"`), igual que antes. **Descartado**: los 3 nombres de archivo requieren una codificación URL distinta por carácter especial (`%20` para espacios, `%C3%B3` para "ó"), y concatenar en M con `&` no aplica esa codificación automáticamente — reproducir esa lógica a mano introduce un riesgo de URL mal formada específico por archivo, sin ganar nada frente a usar las URLs completas ya entregadas.
2. **Un parámetro de texto dedicado por archivo, con la URL completa ya codificada, y `Web.Contents` en vez de `File.Contents`.** **Elegido.** Es el patrón recomendado para conectar Power Query a un archivo alojado en OneDrive/SharePoint sin depender de una ruta de disco local, y es el único de los 2 enfoques que no requiere replicar lógica de codificación de URL a mano.

**Decisión adicional (reversibilidad):** se **conservó** el parámetro `RutaCarpetaData` sin usar (ya no lo referencia ninguna consulta), en vez de eliminarlo. Esto permite revertir a la fuente local con un cambio de una sola línea por consulta si la fuente web presenta problemas, sin tener que reconstruir el parámetro desde cero. Un parámetro sin referencias no genera error en Power BI Desktop — solo queda visible en el panel de consultas sin uso activo.

## 4. Cambio aplicado

En `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`:

- **3 parámetros nuevos** (mismo patrón que `RutaCarpetaData`: `meta [IsParameterQuery=true, Type="Text", IsParameterQueryRequired=true]`, sin `lineageTag` escrito a mano — Power BI Desktop lo generará al guardar):
  - `RutaWebMatrizCalidad`
  - `RutaWebSatisfaccionCapacitacion`
  - `RutaWebEncuestaMotivacion`

  Cada uno con el valor exacto de la URL entregada por el usuario (incluyendo la codificación `%20`/`%C3%B3` ya presente en el texto proporcionado, sin modificarla).

- **3 consultas `Base_*` actualizadas** (se conservó el `lineageTag` original de cada una — es la misma consulta lógica, solo cambia su origen):
  ```dax
  Source = Excel.Workbook(Web.Contents(RutaWebMatrizCalidad), null, true)
  Source = Excel.Workbook(Web.Contents(RutaWebSatisfaccionCapacitacion), null, true)
  Source = Excel.Workbook(Web.Contents(RutaWebEncuestaMotivacion), null, true)
  ```
  El resto de cada consulta (`#"Form Responses 1_Sheet" = Source{[Item="Form Responses 1",Kind="Sheet"]}[Data]`) no cambió — la hoja de origen sigue siendo la misma.

- **`RutaCarpetaData` se conservó sin cambios**, ahora sin ninguna consulta que la referencie (ver §3).

No se tocó ninguna otra expresión (`*_Limpio`), ninguna tabla `Fact_*`, ninguna medida, relación, visual, página ni el tema.

## 5. Validaciones realizadas

- **Sintaxis TMDL**: balance de paréntesis (42/42), corchetes (19/19) y llaves (18/18) verificado en todo el archivo tras el cambio.
- **Alcance del diff**: `git diff --stat` confirma que el único archivo modificado es `expressions.tmdl` (15 inserciones, 3 eliminaciones).
- **Sin `lineageTag`/`description`/`queryGroup` escritos a mano** en los 3 parámetros nuevos, conforme a la convención del proyecto (`CLAUDE.md`, `Outputs/10`).
- **`lineageTag` de las 3 consultas `Base_*` preservado** (no se regeneró ni se inventó uno nuevo — son las mismas consultas, solo con su origen editado).
- **Rastreo de referencias**: `grep` confirmó que `RutaCarpetaData`, `File.Contents` y `Web.Contents` solo aparecen en `expressions.tmdl` (y la lista de orden de consultas de `model.tmdl`, que no requiere edición manual — Power BI Desktop la regenera).
- **Confirmado por `git status`/`git diff`**: ningún archivo de `Fact_*.tmdl`, `_Medidas *.tmdl`, `relationships.tmdl`, `model.tmdl`, `PBI_Indicadores.Report/` ni `Data/*.xlsx` cambió.

### Validación pendiente (requiere Power BI Desktop, no ejecutable en este entorno)

Este entorno no puede abrir Power BI Desktop ni resolver URLs de OneDrive en vivo. **No se pudo confirmar que las 3 consultas carguen correctamente contra las URLs reales.** Antes de dar por cerrado este cambio, el usuario debe:

1. Abrir el `.pbip` en Power BI Desktop y ejecutar "Actualizar".
2. Es esperable que Power BI Desktop solicite **credenciales** para el origen web `onedrive.live.com` (tipo "Cuenta profesional o educativa" / cuenta Microsoft) la primera vez que se use `Web.Contents` contra ese dominio — esto es normal para orígenes OneDrive/SharePoint y no indica un error de la fórmula.
3. Si la URL tal como fue entregada **no resuelve** (algunas URLs de navegación de OneDrive personal requieren en su lugar un enlace de "Compartir → Copiar vínculo", con parámetros `resid`/`authkey`, en vez de la ruta de carpeta visible en el navegador), la alternativa recomendada es reemplazar el valor del parámetro correspondiente por ese enlace de uso compartido, o evaluar la función `SharePoint.Files`/`SharePoint.Contents` como alternativa más robusta para OneDrive personal — sin volver a `File.Contents`.
4. Confirmar que las 3 tablas `Fact_*` siguen actualizando con el mismo número de filas que antes del cambio (los 3 archivos son los mismos, solo cambia el mecanismo de acceso).

## 6. Riesgos y consideraciones

- **Autenticación no verificable desde este entorno**: el resultado de `Web.Contents` contra `onedrive.live.com` depende de que la cuenta Microsoft usada en Power BI Desktop tenga acceso al archivo. Si el archivo no es accesible con las credenciales que Desktop use por defecto, la actualización fallará con un error de autenticación, no de sintaxis.
- **Formato de URL no garantizado por `Web.Contents`**: como se señaló en §5, un enlace de navegación de OneDrive (el que se ve en la barra de direcciones) no siempre es equivalente a un enlace de descarga directa. Esto solo se puede confirmar abriendo Power BI Desktop.
- **`Docs/04_fuentes_y_actualizacion_datos.md` queda desactualizado por este cambio** — describe el flujo anterior basado en `RutaCarpetaData` y una carpeta local. No se modificó en esta tarea (fuera del alcance explícito, que limitó el cambio a los orígenes de datos y a esta bitácora), pero debe actualizarse en una fase posterior una vez confirmada la fuente web en Power BI Desktop, para no dejar la documentación operativa desalineada del modelo real.
- **Los archivos físicos en `Data/` no cambiaron ni se eliminan**: siguen existiendo localmente (sincronizados por OneDrive). Lo que cambia es el mecanismo que Power Query usa para leerlos — de una ruta de disco local a la misma ubicación accedida vía su URL de OneDrive. Si el usuario sigue actualizando el archivo local en `Data/` y OneDrive lo sincroniza a la nube, la fuente web debería reflejar la misma versión más reciente (sujeto a la latencia de sincronización de OneDrive).
- **Rollback disponible**: si la fuente web falla, revertir es un cambio de 1 línea por consulta (volver a `File.Contents(RutaCarpetaData & "\<archivo>")`), ya que `RutaCarpetaData` se conservó intacto.

## 7. Confirmación de no modificación fuera de alcance

No se modificó:

- Ninguna medida DAX (`_Medidas *.tmdl`).
- `relationships.tmdl` ni ninguna tabla `Fact_*`/`Dim_*`.
- Ningún archivo de `PBI_Indicadores.Report/` (páginas, visuales, tema).
- Ningún archivo `Data/*.xlsx`.
- Ninguna documentación fuera de este archivo de `Outputs/` (incluyendo `Docs/04`, señalado como pendiente en §6, deliberadamente no tocado por estar fuera del alcance explícito de esta tarea).

## 8. Estado final de `git status`

```
 M PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl
?? Data/
?? PBI/PBI_Indicadores.SemanticModel/DAXQueries/
```

## 9. Commit

```
data(powerquery): migrar origenes de datos a rutas web de onedrive
```

No se hizo push remoto.

## 10. Recomendaciones futuras

1. Abrir el `.pbip` en Power BI Desktop y actualizar, resolviendo el prompt de credenciales de OneDrive — confirmar que las 3 tablas cargan con el mismo volumen de filas que antes.
2. Si alguna URL no resuelve, reemplazar ese parámetro por un enlace de "Compartir → Copiar vínculo" de OneDrive, o migrar esa consulta puntual a `SharePoint.Files`/`SharePoint.Contents` — no revertir a `File.Contents` salvo que se decida abandonar la fuente web por completo.
3. Una vez confirmada la fuente web en Desktop, actualizar `Docs/04_fuentes_y_actualizacion_datos.md` para reflejar el nuevo flujo (ya no depende de que el archivo esté en una carpeta local específica del equipo, sino de la sincronización de OneDrive y del acceso web).
4. Evaluar en una fase posterior si esta migración habilita la actualización programada en Power BI Service (sin gateway), dado que es la motivación típica para este tipo de cambio — documentarlo como una posible mejora si se confirma.
5. Si la fuente web queda estable durante varias actualizaciones, considerar eliminar `RutaCarpetaData` (hoy conservado sin uso por reversibilidad) en una limpieza posterior, una vez que ya no se necesite como plan de rollback.
