# PBI_Indicadores

**Cliente / contexto:** Connect Assistance S.A.S. — call centers asociados a Claro.

**Propósito:** informe Power BI para seguimiento comercial, formativo y de calidad en los call centers asociados a Claro, a partir de tres encuestas piloto: auditoría de calidad de llamadas, satisfacción de capacitaciones y motivación de actividades comerciales.

**Informe publicado:**
```
https://app.powerbi.com/view?r=eyJrIjoiZGI2ZjNiYmItODQ0Yy00M2Y1LThkNTYtZGQ5NDIxYWExNjk3IiwidCI6Ijc1NDEyNGJlLTM2NGItNDg1MS1hYzA3LTc0ZjljZGJhYzM0ZiIsImMiOjR9&pageName=67eff42d82e1c9c15b84
```
Ver [Docs/06_publicacion_powerbi.md](Docs/06_publicacion_powerbi.md) para detalles, vigencia y consideraciones de acceso.

**Repositorio:** [github.com/HarvLopez91/pbi-indicadores-connect](https://github.com/HarvLopez91/pbi-indicadores-connect) (rama `main`).

## Contenido del proyecto

| Carpeta / archivo | Contenido |
|---|---|
| `PBI/` | Carpeta raíz del proyecto Power BI en formato PBIP. |
| `PBI/PBI_Indicadores.pbip` | Punto de entrada del proyecto — se abre con Power BI Desktop como proyecto (no como `.pbix`). |
| `PBI/PBI_Indicadores.SemanticModel/definition/` | Modelo semántico en TMDL: parámetros y consultas de Power Query (`expressions.tmdl`), tablas (`tables/*.tmdl`), relaciones (`relationships.tmdl`), configuración raíz (`model.tmdl`). |
| `PBI/PBI_Indicadores.Report/definition/` | Definición del reporte: páginas (`pages/`), tema visual (`report.json` + `StaticResources/`). |
| `Data/` | Exportaciones Excel de Google Forms que alimentan el modelo. **Excluida de git** (contiene nombres reales de personas) — se respalda solo vía OneDrive. |
| `Assets/` | Logos, imágenes y el tema visual de marca Connect (`Assets/theme/connect_assistance_theme.json`). |
| `Specs/` | Documentos de planeación y cierre: análisis de impacto, plan de implementación de 18 fases, y documentación final de cierre. |
| `Outputs/` | Bitácora cronológica de cada fase/corrección ejecutada — el changelog operativo del proyecto. |
| `Docs/` | Documentación técnica y funcional de referencia (modelo, medidas DAX, mapa de páginas, actualización de datos, decisiones/pendientes, publicación). Empezar por [Docs/00_indice_documentacion.md](Docs/00_indice_documentacion.md). |

## Cómo abrir el proyecto

1. Abrir `PBI/PBI_Indicadores.pbip` con Power BI Desktop (requiere la característica **Power BI Project (.pbip)** habilitada).
2. Antes de que Power BI Desktop actualice los datos, confirmar que los 3 archivos Excel de `Data/` están **cerrados** — si alguno está abierto, la actualización falla con un error de archivo bloqueado.
3. Después de cualquier sesión en Power BI Desktop (edición o solo apertura), ejecutar `git status` y `git diff` — Desktop reescribe archivos automáticamente (metadatos, orden de página activa, etc.) incluso sin cambios de negocio intencionales. Ver la guía completa en [AGENTS.md](AGENTS.md).

## Fuentes de datos

El modelo se alimenta de **3 archivos Excel exportados desde Google Forms**, ubicados en `Data/`:

- **Matriz de calidad** — auditoría de calidad de llamadas.
- **Satisfacción capacitación** — encuesta de satisfacción posterior a capacitaciones.
- **Encuesta satisfacción / motivación** — encuesta de motivación de actividades comerciales.

Guía completa de actualización (cómo reexportar, dónde colocar los archivos, cómo refrescar y qué validar) en [Docs/04_fuentes_y_actualizacion_datos.md](Docs/04_fuentes_y_actualizacion_datos.md). Este README no incluye datos personales ni ejemplos con nombres reales de asesores/líderes.

## Páginas del reporte

1. **Home** — landing page con KPIs resumen y navegación visual.
2. **Resumen ejecutivo**
3. **Calidad de llamadas**
4. **Satisfacción de capacitaciones**
5. **Motivación comercial**
6. **Detalle por call center**
7. **Notas metodológicas**

Detalle de objetivo, indicadores, visuales y navegación de cada página en [Docs/03_mapa_reporte_paginas_visuales.md](Docs/03_mapa_reporte_paginas_visuales.md).

## Modelo semántico

Modelo en estrella:

- **3 tablas de hechos** (`Fact_CalidadLlamadas`, `Fact_SatisfaccionCapacitacion`, `Fact_MotivacionActividad`), una por encuesta, con grano y periodicidad distintos — intencionalmente no unificadas.
- **3 dimensiones** compartidas: `Dim_Calendario`, `Dim_CallCenter`, `Dim_Jornada` — las dos últimas construidas dinámicamente por unión de valores observados, nunca con listas fijas.
- **4 tablas de medidas** DAX (`_Medidas Generales`, `_Medidas Calidad`, `_Medidas Capacitacion`, `_Medidas Motivacion`).
- **Relaciones** por calendario (Fecha), call center y jornada, todas `1:*` de dirección de filtro única, sin ambigüedad.

Detalle completo de columnas, orígenes y relaciones en [Docs/01_modelo_datos.md](Docs/01_modelo_datos.md); catálogo completo de medidas en [Docs/02_catalogo_medidas_dax.md](Docs/02_catalogo_medidas_dax.md).

## Publicación

El informe está publicado mediante un enlace de "Publicar en la Web" de Power BI Service (ver el enlace en el encabezado de este documento). Este tipo de enlace **no requiere autenticación** — cualquier persona con el enlace puede verlo. El link puede depender de permisos vigentes en el tenant de Power BI, de que la publicación no haya sido revocada, y de la configuración de "Publicar en la Web" de la organización. Ver [Docs/06_publicacion_powerbi.md](Docs/06_publicacion_powerbi.md) para el detalle y una consideración de gobierno de datos sobre el acceso público.

## Limitaciones conocidas

- **Datos piloto**: el volumen actual es de fase piloto y cambia con cada actualización de `Data/`; los indicadores deben interpretarse junto al `n` visible, no como resultados definitivos.
- **Encuesta de motivación anónima**: no permite desglose por asesor individual (la columna de identificación llega vacía desde el origen).
- **`% Calidad Promedio Provisional`**: pendiente de la rúbrica oficial de puntaje máximo por pregunta; la medida existe en el modelo pero retorna en blanco.
- **`% Llamadas con Venta`**: puede aparecer en blanco en el piloto en vez de `0,0%`; observación pendiente de ajuste menor.
- **Catálogo oficial de call centers y alias de líderes**: pendientes de confirmación de negocio; se usa un catálogo dinámico y una tabla de alias parcial mientras tanto.
- **Los datos se actualizarán constantemente**: cualquier conteo mostrado en el informe (o en esta documentación) es dinámico y puede cambiar con la próxima actualización de `Data/`.

Detalle completo, incluyendo el estado de las dependencias D1–D9 del plan de implementación, en [Docs/05_decisiones_limitaciones_pendientes.md](Docs/05_decisiones_limitaciones_pendientes.md).

## Mantenimiento

- **No versionar `Data/**/*.xlsx`** — contiene nombres reales de personas; se excluye deliberadamente del repositorio, incluidas subcarpetas.
- **No escribir manualmente `lineageTag`, `description` ni `queryGroup`** en archivos `.tmdl` nuevos o editados — Power BI Desktop los genera automáticamente al guardar; escribirlos a mano rompe el analizador de la vista previa de Desktop.
- **Documentar cambios relevantes en `Outputs/`** — cada fase o corrección se registra en un archivo `Outputs/NN_...md` nuevo, sin sobrescribir el historial existente.
- **Actualizar `Docs/`** cuando cambien medidas DAX, páginas del reporte o fuentes de datos, para que la documentación siga reflejando el estado real del proyecto.
