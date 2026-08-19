# Resultado PC-4 - Sustitución visual de cumplimiento

## Estado

**PASS — PC-4 cerrado. PC-5 no iniciado.**

## Objetivo

Incorporar la clasificación PUSHER temporal y el porcentaje de cumplimiento en la página Gestión comercial de altas mediante una sustitución visual mínima, preservando el diseño aprobado.

## Commit técnico

- `f19c919d358928d26925ac4a580d8ba0f13228f1`
- Mensaje: `feat(report): actualiza visuales de cumplimiento comercial`

## Cambios realizados

- `gc_slicer_pusher`: el origen funcional cambió de `Dim_Aliado[Pusher]` a `Dim_AsignacionPusherPeriodo[PusherPeriodo]`; la etiqueta visible `PUSHER` y el formato permanecieron intactos.
- `gc_kpi_mes_anterior`: se conservó `Promedio_Altas_Hasta_Mes_Anterior` como primera proyección y se sustituyó `Meta_Altas_Promedio_Mas_30_Pct` por `Cumplimiento_Meta_Pct` como segunda proyección, con la etiqueta `% Cumplimiento`.
- `gc_matriz_aliados`: la columna visible `PUSHER` cambió a `Dim_AsignacionPusherPeriodo[PusherPeriodo]`; se conservaron seis columnas, el ancho `75D`, el ordenamiento y el formato.

No se modificaron geometría, estilos, layout, medidas DAX, modelo semántico, relaciones, Power Query, datos ni otras páginas o visuales.

## Validación automática

- Los tres archivos JSON parsean correctamente y conservan sus schemas.
- Geometría, estilos y ordenamiento no presentan diferencias no autorizadas.
- Referencias funcionales restantes a `Dim_Aliado.Pusher` en Gestión comercial de altas: 0.
- `git diff --check`: PASS.
- Revisión de privacidad: PASS; no se incorporaron rutas locales, usuarios, correos ni datos personales.

## Gate visual manual

Gate aprobado en Power BI Desktop para julio de 2026:

| Contexto | Altas | Cumplimiento | Estado |
|---|---:|---:|---|
| Todas | 4.518 | 71,52 % | PASS |
| PUSHER 1 | 1.581 | 53,43 % | PASS |
| PUSHER 2 | 2.429 | 72,33 % | PASS |

La matriz confirmó `UNO 27 | PUSHER 1 | 224`, demostrando que la clasificación temporal se representa de forma consistente entre slicer, medidas y detalle por aliado.

El layout visual existente fue preservado y aprobado.

## Cierre

PC-4 queda cerrado y publicado. PC-5 no fue iniciado.
