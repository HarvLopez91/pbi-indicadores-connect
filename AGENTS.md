# Repository Guidelines

## Estructura del Proyecto y Organización

Este repositorio es un proyecto Power BI (PBIP) para indicadores de Connect Assistance. El punto de entrada es `PBI/PBI_Indicadores.pbip`, que se abre con Power BI Desktop.

- `PBI/PBI_Indicadores.SemanticModel/definition/`: modelo semántico en TMDL, incluyendo `model.tmdl`, `expressions.tmdl`, `relationships.tmdl` y `tables/*.tmdl`.
- `PBI/PBI_Indicadores.Report/definition/`: definición del informe, páginas, metadatos y recursos estáticos.
- `Assets/`: logos, imágenes y tema visual, incluyendo `Assets/theme/connect_assistance_theme.json`.
- `Data/`: exportaciones Excel locales usadas como fuentes. Contienen datos privados y están excluidas de git.
- `Specs/`: diagnóstico, plan y decisiones de implementación. Revísalos antes de cambiar diseño de modelo o reporte.
- `Outputs/`: bitácora por fase y correcciones ejecutadas. Úsala como changelog operativo.

## Comandos de Desarrollo y Validación

No hay build, gestor de paquetes ni suite automática de pruebas. Desarrollar aquí significa editar archivos PBIP/TMDL/JSON y validar en Power BI Desktop.

- Abrir el informe: `PBI/PBI_Indicadores.pbip`
- Revisar cambios: `git status` y `git diff`
- Ver convenciones recientes: `git log --oneline -5`

Después de cualquier sesión en Power BI Desktop, revisa `git status` porque Desktop puede reescribir archivos del modelo o del reporte.

## Estilo y Convenciones de Nombres

Mantén la nomenclatura de negocio en español y los patrones TMDL existentes. Las tablas de hechos usan `Fact_*`, las dimensiones `Dim_*` y las tablas de medidas `_Medidas <Area>`. Los nombres técnicos de columnas deben usar `PascalCase`, sin espacios, tildes ni signos.

No agregues manualmente `lineageTag`, `description` ni `queryGroup` en bloques TMDL nuevos. Deja que Power BI Desktop genere metadatos. Conserva el pipeline de Power Query como `Base_<Fuente>` -> `<Fuente>_Limpio` -> tabla final `Fact_*`.

## Guía de Pruebas

Valida los cambios reabriendo el PBIP en Power BI Desktop, actualizando datos y revisando visuales o medidas afectadas. Antes de pedir validación, revisa sintaxis TMDL/JSON, nombres de columnas referenciadas, relaciones y medidas DAX. Documenta resultados relevantes en `Outputs/NN_resultado_*.md` o `Outputs/NN_correccion_*.md` cuando correspondan a una fase.

## Commits y Pull Requests

El historial usa prefijos tipo conventional commit con asuntos en español, por ejemplo `feat(modelo):`, `fix(modelo):`, `test(dax):`, `style(theme):` y `chore(report):`.

No comitees `Data/*.xlsx` ni exportaciones privadas. Mantén commits enfocados: separa cambios intencionales de reescrituras automáticas de Power BI Desktop cuando sea práctico. Un PR debe resumir el cambio de negocio, listar archivos PBIP/TMDL afectados, indicar validación en Power BI Desktop y adjuntar capturas si hay cambios visibles.

## Seguridad y Configuración

Trata los datos fuente como confidenciales. Mantén los Excel cerrados antes de refrescar Power BI para evitar errores por archivo bloqueado. Evita rutas personales o valores sensibles fuera de los patrones existentes de configuración y Power Query.
