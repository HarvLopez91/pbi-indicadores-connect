# Corrección — Error `QueryGroup` en consultas de staging (Fase 3)

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Origen del error | Reportado por el usuario al abrir `PBI_Indicadores.pbip` en Power BI Desktop, tras la Fase 3 |
| Documentos de referencia | `Specs/01...`, `Specs/02...`, `Outputs/03_resultado_fase_3_ingesta_archivos_data.md` |
| Fecha | 2026-07-08 |
| Archivo corregido | `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` |

---

## Error encontrado

Al abrir el PBIP en Power BI Desktop, aparecieron 3 errores de validación del modelo:

```
La propiedad QueryGroup del objeto Expression Base_MatrizCalidad hace referencia a un objeto que no se puede encontrar.
La propiedad QueryGroup del objeto Expression Base_SatisfaccionCapacitacion hace referencia a un objeto que no se puede encontrar.
La propiedad QueryGroup del objeto Expression Base_EncuestaMotivacion hace referencia a un objeto que no se puede encontrar.
```

## Causa probable

En TOM/TMDL, un **grupo de consultas** (`queryGroup`) no es una simple etiqueta de texto: es un objeto propio del modelo (una carpeta de organización para el panel de consultas de Power Query) que debe existir declarado como tal. La propiedad `queryGroup: <nombre>` de una expresión es una **referencia** a ese objeto, no una definición.

En la Fase 3, `expressions.tmdl` se escribió directamente como texto (sin pasar por Power BI Desktop, según quedó documentado explícitamente en `Outputs/03`), y se agregó `queryGroup: Staging` a las 3 consultas `Base_*` para organizarlas visualmente, **sin declarar el objeto `queryGroup 'Staging'`** correspondiente en el modelo. Al abrir el archivo, Power BI Desktop validó las referencias del modelo y encontró que las 3 expresiones apuntaban a un grupo inexistente — de ahí el error, uno por cada expresión afectada. `RutaCarpetaData` no se vio afectado porque nunca tuvo esta propiedad.

## Archivo corregido

`PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`

## Cambio realizado

Se aplicó la solución preferida indicada: **se eliminó la línea `queryGroup: Staging`** de las 3 expresiones `Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion` y `Base_EncuestaMotivacion` (1 línea por consulta, 3 líneas en total). No se declaró un objeto `queryGroup 'Staging'` alternativo (esa habría sido la otra solución posible, pero no la solicitada).

**No se modificó nada más:**
- El código M de conexión (`Excel.Workbook`, `File.Contents`, navegación a `Form Responses 1`) quedó intacto en las 3 consultas.
- Los `lineageTag` de las 3 expresiones se conservaron sin cambios.
- La anotación `annotation PBI_ResultType = Table` se mantuvo.
- El parámetro `RutaCarpetaData` no se tocó.
- No se crearon tablas `Fact_`, dimensiones, relaciones, medidas ni visuales.

Verificación posterior al cambio: se buscó (`grep`) la palabra `Staging` y la propiedad `queryGroup` en todo el árbol `PBI/` — **cero coincidencias remanentes**, confirmando que no quedó ninguna referencia huérfana adicional.

## Validación en Power BI Desktop

**Limitación ya documentada en `Outputs/03`, vigente también aquí:** no tengo forma de abrir o refrescar Power BI Desktop desde este entorno (no controlo su interfaz gráfica). Por lo tanto, **no puedo confirmar yo mismo** que el error desapareció al reabrir la aplicación.

Lo que sí valida esta corrección, de forma independiente al motor de Power BI:
- El texto exacto que Power BI Desktop señaló como inválido (`queryGroup: Staging`) ya no existe en el archivo.
- La sintaxis TMDL restante sigue las reglas oficiales de indentación y declaración de expresiones (sin cambios respecto a lo ya validado en la Fase 3, salvo la eliminación de esa única propiedad por consulta).
- Los 3 archivos de `Data` siguen accesibles y sin bloqueo.

**Acción requerida de tu parte para completar la validación:** dado que el modelo ya estaba cargado (con error) en una sesión previa de Power BI Desktop, y que las ediciones externas a los archivos TMDL requieren reiniciar la aplicación para recargarse (no basta con "Actualizar"), te recomiendo **cerrar completamente Power BI Desktop y volver a abrir el `.pbip`** para confirmar que los 3 errores de `QueryGroup` ya no aparecen.

## Estado de las consultas `Base_*`

Sin cambios de fondo respecto a la Fase 3: las 3 siguen existiendo como expresiones (`expression`) de Power Query, **no cargadas al modelo** (staging), apuntando cada una a su archivo correcto en `Data/` y navegando a la hoja `Form Responses 1`:

| Consulta | Archivo origen | Hoja | `queryGroup` |
|---|---|---|---|
| `Base_MatrizCalidad` | `Matriz de calidad (Responses).xlsx` | `Form Responses 1` | *(eliminado)* |
| `Base_SatisfaccionCapacitacion` | `Satisfacción capacitación (Responses).xlsx` | `Form Responses 1` | *(eliminado)* |
| `Base_EncuestaMotivacion` | `Encuesta satisfacción (Responses).xlsx` | `Form Responses 1` | *(eliminado)* |

## Confirmación de vista previa o errores encontrados

- **Errores de sintaxis/referencia:** ninguno detectado tras la corrección (la búsqueda exhaustiva de `Staging`/`queryGroup` en `PBI/` no arrojó coincidencias).
- **Vista previa real del motor de Power Query:** sigue sin poder validarse desde este entorno (misma limitación de la Fase 3). Pendiente de confirmación manual al reabrir Power BI Desktop.
- Sin las líneas `queryGroup`, las 3 consultas quedarán sin carpeta de agrupación en el panel de Power Query (aparecerán sueltas, no bajo una carpeta "Staging"); esto es una consecuencia visual esperada y aceptable de la solución preferida, no un error.

## Estado final de `git status`

Antes del commit, `git status` mostró exactamente los 2 archivos esperados: `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl` (modificado) y `Outputs/04_correccion_fase_3_querygroup.md` (nuevo). El `git diff` del archivo TMDL confirmó que el único cambio fueron las 3 líneas `queryGroup: Staging` eliminadas (una por consulta), sin ninguna otra alteración. Tras el commit, el working tree queda limpio.

## Recomendación para avanzar o no a Fase 4

**No avanzar todavía a la Fase 4.** Antes de continuar:
1. Cierra y vuelve a abrir Power BI Desktop con este `.pbip` para confirmar que los 3 errores de `QueryGroup` ya no aparecen.
2. En el Editor de Power Query, confirma visualmente que `Base_MatrizCalidad`, `Base_SatisfaccionCapacitacion` y `Base_EncuestaMotivacion` muestran vista previa de datos sin error (esta es la validación pendiente que también quedó abierta desde la Fase 3).
3. Si todo carga correctamente, se puede proceder con la Fase 4 (limpieza y transformación) tal como está definida en `Specs/02`.
4. Si aparece un error distinto al de `QueryGroup`, repórtalo para diagnosticarlo antes de continuar — no debería aparecer ninguno nuevo, ya que el único cambio fue la eliminación de una propiedad no declarada.

---

*Documento generado como registro operativo de la corrección de la Fase 3, según la regla documental vigente: los resultados de ejecución/corrección de fases se documentan en `Outputs/`.*
