# Resultado — Restauración de orígenes de datos a rutas locales (OneDrive personal)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de tarea | Corrección de Power Query (orígenes de datos únicamente), revirtiendo `Outputs/41` |
| Archivo modificado (cambio funcional) | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` |
| Fecha | 2026-07-22 |
| Alcance | Solo orígenes de datos. No se modificaron medidas DAX, relaciones, visuales, páginas PBIR, tema, navegación ni `Data/*.xlsx`. |

---

## 1. Contexto

`Outputs/41` migró los 3 orígenes de datos de `File.Contents` (ruta local) a `Web.Contents` contra URLs de `onedrive.live.com`, pensando en habilitar actualización sin depender de una ruta de disco fija. Al usarse desde un computador distinto con una cuenta de **OneDrive personal**, esas URLs solicitaban autenticación y no cargaban correctamente — el enfoque no era compatible con este entorno. Esta tarea revierte ese cambio, restaurando la conexión local vía el parámetro `RutaCarpetaData` (ya existente en el proyecto desde su diseño original, pensado precisamente para portabilidad entre equipos), mimicronizado a la ruta real de este computador.

## 2. Validación previa (antes de modificar)

- **Power BI Desktop cerrado**: verificado con `tasklist` antes de tocar cualquier archivo (primer intento detectó el proceso `PBIDesktop.exe` activo; se detuvo y se solicitó cerrarlo; la segunda verificación confirmó `NO_PBIDESKTOP_PROCESS_FOUND`).
- **Raíz real del repositorio**: `git rev-parse --show-toplevel` → `C:/Users/eclavijo/OneDrive/PBI_Indicadores`.
- **`git status --porcelain` inicial**: limpio salvo `Data/` y `PBI/.../DAXQueries/` (metadatos ya conocidos, sin relación con esta tarea) — confirmado que el usuario no había hecho cambios manuales.
- **Detección de la carpeta `Data` real**: se verificó físicamente (no se asumió) que los 3 archivos existen en `C:\Users\eclavijo\OneDrive\PBI_Indicadores\Data`.
- **Hallazgo importante**: `$env:OneDrive` apuntaba a `C:\Users\eclavijo\OneDrive - CHALLENGER S.A.S` — una **cuenta/carpeta de OneDrive distinta** a donde vive el proyecto (`OneDrive`, sin sufijo). No se usó esa variable de entorno como fuente de la ruta; se usó la ruta real confirmada por `git rev-parse` + la existencia física de los 3 archivos en `Data/`.

## 3. Cambio aplicado

En `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`:

- **`RutaCarpetaData`**: se actualizó únicamente su **valor** (mismo parámetro, mismo `lineageTag: 53085232-a62f-43c5-997d-4ad091d4aacd`), de `C:\Users\edwin.clavijo\OneDrive\PBI_Indicadores\Data` (ruta del computador anterior) a `C:\Users\eclavijo\OneDrive\PBI_Indicadores\Data` (ruta real de este equipo). No se creó un segundo parámetro.
- **3 parámetros web eliminados por completo**: `RutaWebMatrizCalidad`, `RutaWebSatisfaccionCapacitacion`, `RutaWebEncuestaMotivacion` — no se dejaron sin usar, se removieron íntegramente, ya que su sola presencia con el texto `onedrive.live.com` habría dejado una referencia residual no deseada.
- **3 consultas `Base_*` revertidas** a `Excel.Workbook(File.Contents(RutaCarpetaData & "\<archivo>.xlsx"), null, true)`, con el mismo `lineageTag` que tenían antes de `Outputs/41` (no se regeneraron).
- Se conservaron exactamente los nombres de hoja (`Form Responses 1`), consultas, y todos los pasos de transformación de `*_Limpio` — no se tocó nada más allá del origen.

## 4. Validaciones estructurales realizadas (antes de la sesión en Desktop)

- **Sin referencias residuales**: `grep` confirmó 0 coincidencias de `Web.Contents`, `onedrive.live.com`, `RutaWeb` y `edwin.clavijo` en todo `expressions.tmdl`.
- **3 usos de `File.Contents`** confirmados en el archivo.
- **Sintaxis balanceada**: paréntesis 42/42, corchetes 16/16, llaves 18/18.
- **Alcance del diff**: `git diff --name-only` mostró únicamente `expressions.tmdl` como archivo funcionalmente modificado.

## 5. Validación en vivo (reportada por el usuario en Power BI Desktop)

- No se solicitaron credenciales web.
- Las 3 fuentes locales actualizaron correctamente.
- `Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion` y `Fact_MotivacionActividad` cargaron sin errores.
- El informe volvió a mostrar información correctamente.

Esta validación **no se pudo ejecutar desde este entorno** (sin interfaz gráfica de Power BI Desktop) — se registra aquí como reportada y confirmada por el usuario.

## 6. Cambios automáticos de Power BI Desktop detectados tras la sesión de validación

Al abrir y actualizar el `.pbip` para la validación de §5, Power BI Desktop reescribió, como es su comportamiento conocido (`CLAUDE.md`):

| Archivo | Cambio | Naturaleza |
|---|---|---|
| `PBI/PBI_Indicadores.SemanticModel/definition/database.tmdl` | `compatibilityLevel: 1600` → `1606` | Metadato de motor/versión de Desktop, no de negocio |
| `PBI/PBI_Indicadores.SemanticModel/diagramLayout.json` | +52 líneas (posiciones de las tablas en la vista de diagrama del modelo) | Metadato puramente visual/cosmético de la vista de modelo, sin efecto funcional |

Ninguno de los dos afecta medidas, relaciones, Power Query (más allá de lo ya descrito en §3) ni visuales. Se separan del commit funcional, conforme a lo solicitado.

La carpeta `PBI/PBI_Indicadores.SemanticModel/DAXQueries/` (presente en la sesión anterior) ya no existe tras esta sesión — no requiere ninguna acción.

## 7. Confirmación de no modificación fuera de alcance

No se modificó:

- Ninguna medida DAX.
- `relationships.tmdl` ni ninguna tabla `Fact_*`/`Dim_*`.
- Ningún archivo de `PBI_Indicadores.Report/` (páginas, visuales, tema, navegación) — ni la página original `p14_satisfaccion_capacitaciones` ni la copia `p14_satisfaccion_capacitaciones_v2`.
- Ningún documento de `SC-5` (`Specs/06`, `Outputs/` de fases `SC-*`).
- Ningún archivo `Data/*.xlsx`.
- Ningún otro parámetro de Power Query no relacionado.

## 8. Commits

**Commit 1 — Metadatos de Desktop** (separado, no funcional):
```
chore(modelo): sincronizar metadatos tras validacion en desktop
```
Incluye: `database.tmdl`, `diagramLayout.json`.

**Commit 2 — Corrección funcional** (el solicitado):
```
fix(data): restaurar rutas locales de fuentes OneDrive
```
Incluye: `expressions.tmdl`, este documento (`Outputs/42_resultado_restauracion_rutas_locales_onedrive.md`).

No se agregó `Data/`, ningún archivo `.pbi`, consultas DAX locales ni cachés a ninguno de los 2 commits. No se hizo push remoto.

## 9. Recomendaciones futuras

1. Si el proyecto vuelve a abrirse en otro computador, `RutaCarpetaData` debe actualizarse nuevamente a la ruta real de ese equipo — es un parámetro pensado para portabilidad, pero su valor es específico de cada máquina/usuario y no se autodetecta.
2. Si en el futuro se retoma la idea de una fuente web (p. ej. para actualización programada en Power BI Service), evaluar `SharePoint.Files`/`SharePoint.Contents` o un enlace de "Compartir → Copiar vínculo" en vez de la URL de navegación de OneDrive personal usada en `Outputs/41`, y probarla primero con la cuenta y el tenant reales antes de aplicarla al modelo.
3. `Docs/04_fuentes_y_actualizacion_datos.md` sigue describiendo el flujo basado en `RutaCarpetaData` — esta restauración lo vuelve a dejar consistente con la documentación existente (a diferencia de `Outputs/41`, que la habría dejado desalineada).
