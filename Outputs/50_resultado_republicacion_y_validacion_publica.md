# Resultado — republicación y validación pública

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fecha | 2026-07-22 |
| Tipo | Republicación manual posterior al hotfix de `Dim_Calendario` |
| Estado final | `Republicación validada` |

## Contexto

El plan de Satisfacción de capacitaciones ya estaba cerrado y sincronizado. Después de incorporar nuevos datos en `Data/Satisfacción capacitación (Responses).xlsx`, Power BI Desktop falló al actualizar por un valor no convertible en `Timestamp`.

El hotfix quedó documentado en [`Outputs/49`](49_resultado_hotfix_dim_calendario_actualizacion.md) y fue validado localmente antes de republicar.

## Confirmación de actualización previa

Power BI Desktop actualizó correctamente después del hotfix:

- Sin error de `Dim_Calendario`.
- 6 capacitaciones.
- 97 respuestas.
- 6 call centers.
- Nuevo call center `ALMA`.
- Última capacitación `15/07/2026`.
- Gráfico por fecha con `15/07`.
- Filtros, tablas, KPI y panel de satisfacción funcionando.

## Enlace utilizado

Se mantuvo el enlace publicado existente:

```text
https://app.powerbi.com/view?r=eyJrIjoiZGI2ZjNiYmItODQ0Yy00M2Y1LThkNTYtZGQ5NDIxYWExNjk3IiwidCI6Ijc1NDEyNGJlLTM2NGItNDg1MS1hYzA3LTc0ZjljZGJhYzM0ZiIsImMiOjR9&pageName=67eff42d82e1c9c15b84
```

No se modificó el vínculo desde el repositorio.

## Validación en navegación privada

El usuario confirmó la validación del enlace público en navegación privada:

- Home carga correctamente como página inicial.
- La navegación Home -> Satisfacción de capacitaciones funciona.
- La página de Satisfacción muestra los datos actualizados.
- Se ven 6 capacitaciones, 97 respuestas y 6 call centers.
- La última capacitación es `15/07/2026`.
- Los filtros e interacciones funcionan.
- `Volver a Home` funciona.
- No aparecen nombres de formadores ni líderes.

## Gobierno de datos

La página oficial de Satisfacción de capacitaciones ya no expone `NombreFormador` ni `NombreLider`.

Riesgo pendiente:

- `cl_tabla_asesor` en Calidad de llamadas conserva `NombreAsesor`.
- El enlace sigue siendo público sin autenticación (`Publicar en la Web`).

## Archivos/documentación relacionados

- [`Outputs/49`](49_resultado_hotfix_dim_calendario_actualizacion.md): hotfix de fechas y actualización local.
- [`Docs/06`](../Docs/06_publicacion_powerbi.md): enlace publicado, última republicación validada y riesgo de gobierno de datos.

## Estado final

`Republicación validada`.
