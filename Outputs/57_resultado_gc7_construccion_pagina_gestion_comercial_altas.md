# Resultado GC-7 - Construcción de página Gestión comercial de altas

## Objetivo

Construir una página ejecutiva para analizar históricamente las altas de Te Resuelve por periodo, PUSHER y aliado, sin alterar la fuente, Power Query, las relaciones ni el grano agregado del modelo.

## Implementación

- Commit técnico: `6d4097a4b3ed019736caaf31cbbba8cef3a65533`.
- Página nueva: `GestionComercialAltas`, con nombre visible **Gestión comercial de altas**.
- Navegación bidireccional entre Home y la nueva página; Home permanece como página inicial.
- Filtros: año, mes, PUSHER y aliado.
- KPI: altas del periodo, cambio y variación frente al mes anterior, promedio histórico previo, meta de altas +30 % y cobertura de clasificación PUSHER.
- Gráfico histórico apilado con PUSHER 1, línea base del portafolio P2, PUSHER 2 desde el inicio de gestión y universo sin asignar.
- La serie P2 de enero a junio se presenta como línea base histórica; desde julio de 2026 se identifica como periodo desde el inicio de gestión, sin atribuir causalidad.
- Drivers dinámicos de contribuciones positivas y caídas, además de ranking de aliados con volumen y cambio mensual.
- Modo de enfoque habilitado para el gráfico histórico, drivers y ranking.

## Medidas y controles

Se incorporaron las medidas necesarias para contribución por clasificación, separación temporal del portafolio P2, altas sin asignar, participación sin asignar, promedio histórico previo y meta +30 %.

Controles aprobados para julio de 2026:

- Altas: 4.518.
- Cambio frente a junio: +818.
- Variación mensual: +22,11 %.
- Promedio histórico enero-junio: 4.796.
- Meta +30 %: 6.235.
- Altas en aliados asignados a PUSHER: 83,80 %.
- Junio: 1.193 + 1.857 + 650 = 3.700.
- Julio: 1.357 + 2.429 + 732 = 4.518.
- Drivers visibles: ATENTO +381, ONE CONTACT +201 y GNP -70.

## Validación y privacidad

- El gate visual manual en Power BI Desktop fue aprobado sobre la versión finalmente serializada.
- Se validaron etiquetas por segmento, totales mensuales, unidades completas, colores diferenciados, filtros, drivers, ranking y navegación.
- Las referencias PBIR son válidas, Home continúa activa y no existen campos sensibles en la página.
- No se incluyeron nombres de personas, cargos sensibles, rutas locales, huellas ni registros individuales.
- No hubo cambios en Data, archivos Excel, Power Query, relaciones ni páginas ajenas a la navegación autorizada de Home.

## Observación no bloqueante

El título vertical del eje Y del gráfico histórico aparece parcialmente truncado. Se registra como mejora visual menor para evaluar en GC-8, sin modificar el diseño aprobado durante este cierre.

## Estado final

- GC-7: cerrado.
- GC-8: no iniciado.
