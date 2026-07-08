# Correccion - Fase 12: tema JSON sin `bold` en `textClasses`

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 12 - Tema visual Connect Assistance |
| Fecha | 2026-07-08 |
| Archivo corregido | `Assets/theme/connect_assistance_theme.json` |
| Motivo | Power BI Desktop no importa el tema porque `textClasses.title` y `textClasses.callout` contienen la propiedad no soportada `bold`. |

## Diagnostico

El tema JSON era sintacticamente valido, pero Power BI Desktop rechaza propiedades no permitidas dentro de `textClasses`. El problema estaba acotado a:

- `textClasses.title.bold`
- `textClasses.callout.bold`

## Correccion aplicada

Se eliminaron unicamente las propiedades `bold` dentro de `textClasses.title` y `textClasses.callout`.

No se modificaron los `bold` definidos en `visualStyles`, porque pertenecen a configuraciones de visuales y no eran parte del error reportado.

## Validacion

Se valido que `Assets/theme/connect_assistance_theme.json` sigue siendo JSON valido despues del cambio.

## Alcance

No se avanzo a Home ni se modificaron paginas, visuales, modelo semantico, medidas DAX, relaciones, Power Query ni archivos de `Data/`.
