# Resultado - Fase 13: Home landing page Connect

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Fase | Fase 13 - Diseno del Home principal tipo landing page |
| Fecha | 2026-07-08 |
| Pagina objetivo | `Home` |
| Resultado tecnico | Pagina existente renombrada a `Home` y wireframe ejecutivo documentado para construccion manual segura en Power BI Desktop. |

## Estado inicial de `git status`

Estado inicial observado antes de modificar archivos de reporte:

```text
?? AGENTS.md
```

El unico cambio pendiente era `AGENTS.md` sin seguimiento, no relacionado con la Fase 13. Se mantuvo fuera del commit de esta fase. Por instruccion del usuario, el contenido de `AGENTS.md` quedo en espanol, pero no se incluyo en el commit.

## Contexto revisado

Documentos fuente revisados:

- `Specs/01_analisis_de_impacto_informe_powerbi_connect.md`
- `Specs/02_plan_implementacion_informe_powerbi_connect.md`
- `Outputs/16_cierre_fase_11_validacion_visual_powerbi.md`
- `Outputs/17_resultado_fase_12_tema_visual_connect.md`
- `Outputs/18_correccion_fase_12_tema_json_bold.md`

La Fase 11 esta cerrada con medidas validadas visualmente, la Fase 12 fue corregida y el tema Connect fue importado correctamente en Power BI Desktop.

## Estructura del reporte encontrada

- `PBI/PBI_Indicadores.Report/definition/pages/pages.json` contiene una sola pagina activa: `67eff42d82e1c9c15b84`.
- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/page.json` estaba vacia, sin contenedores de visuales persistidos.
- `PBI/PBI_Indicadores.Report/definition/report.json` mantiene el tema base compartido `CY25SU11`; no se modifico en esta fase.

La pagina vacia actual fue identificada como el punto seguro para el Home.

## Recursos encontrados en `Assets`

### Logos

- `Assets/logos/64b580afe0544a5492bb389f_logo-completo.svg`: variante blanca.
- `Assets/logos/64b580afe0544a5492bb38a3_logo-completo.svg`: variante completa a color, fuente de los colores `#F15B2B` y `#002733`.
- `Assets/logos/6973ca8b4e3df02ed6efdaa7_logo_connect_naranja.png`: logo candidato para el Home.

### Imagenes

- `Assets/imagenes/64b580afe0544a5492bb382c_Cambio_de_llanta Image@3x.png`: ilustracion lateral de servicio.
- `Assets/imagenes/64b580afe0544a5492bb382d_Moto_Sr_Connect%403x-p-800.png`: ilustracion lateral sobria de operador/moto.
- `Assets/imagenes/64b580afe0544a5492bb38dc_Group 8950 (1).png`
- `Assets/imagenes/64b580afe0544a5492bb38dd_Group 8949.png`
- `Assets/imagenes/64b580afe0544a5492bb38f5_Group 8951.png`
- `Assets/imagenes/64b580afe0544a5492bb38f6_Group 8955.png`

## Logo utilizado

Logo recomendado para el Home:

`Assets/logos/6973ca8b4e3df02ed6efdaa7_logo_connect_naranja.png`

El archivo es liviano y legible, aunque corresponde al wordmark `CONNECT` en naranja, no al logo completo con `Assistance`. Por eso debe acompanar el titulo y subtitulo del informe.

## Imagenes utilizadas

No se inserto ninguna imagen en `page.json` desde este entorno.

Imagen recomendada para uso manual opcional:

`Assets/imagenes/64b580afe0544a5492bb382d_Moto_Sr_Connect%403x-p-800.png`

Motivo: tiene estilo de linea sobrio, color oscuro compatible con `#002733` y funciona como apoyo lateral sin competir con KPIs. Si al insertarla recarga demasiado el lienzo, omitirla y priorizar limpieza visual.

## Decision tecnica: wireframe en lugar de visuales JSON manuales

No se crearon visuales directamente en el JSON del reporte porque la pagina actual no contiene ningun ejemplo persistido de `visualContainers`, `singleVisual`, imagenes, textos, tarjetas KPI ni botones que sirva como patron validado por esta version de Power BI Desktop.

Forzar contenedores visuales desde cero en PBIR seria especulativo y podria romper la apertura del proyecto. Se aplico solo el cambio seguro:

- Renombrar la pagina existente de `Pagina 1` a `Home`.

La construccion visual debe hacerse manualmente en Power BI Desktop siguiendo el wireframe exacto de abajo.

## Estructura del Home

Formato: pagina 16:9, `1280 x 720`, fondo blanco o gris muy claro.

### 1. Header / Hero

Ubicacion sugerida: `x=48`, `y=32`, `w=1184`, `h=150`.

- Logo: `Assets/logos/6973ca8b4e3df02ed6efdaa7_logo_connect_naranja.png`, alineado a la izquierda, ancho aproximado `190-230 px`.
- Titulo: `Dashboard Comercial y Formativo`, color `#002733`, tamano grande, peso visual alto.
- Subtitulo: `Seguimiento de calidad, capacitacion y motivacion en call centers asociados a Claro`, color gris oscuro.
- Nota pequena: `Datos piloto sujetos a validacion`, color gris medio, con linea/acento naranja `#F15B2B`.

No usar naranja para texto largo sobre blanco.

### 2. KPIs principales

Ubicacion sugerida: fila central `y=210`, tarjetas de `180 x 105 px`, separacion `16 px`.

Crear maximo 6 tarjetas:

1. `Total Registros Piloto` -> medida `[Total Registros Piloto]`
2. `Total Evaluaciones Calidad` -> medida `[Total Evaluaciones Calidad]`
3. `Total Respuestas Capacitacion` -> medida `[Total Respuestas Capacitacion]`
4. `Total Respuestas Motivacion` -> medida `[Total Respuestas Motivacion]`
5. `Indice Global Capacitacion` -> medida `[Indice Global Capacitacion]`
6. `Indice Global Motivacion` -> medida `[Indice Global Motivacion]`

Estilo:

- Numero grande en `#002733`.
- Etiqueta secundaria pequena en gris oscuro.
- Borde o linea superior naranja `#F15B2B`.
- Fondo blanco y borde gris claro.
- No agregar graficos ni tablas en el Home.

## Tarjetas de navegacion preparadas

Ubicacion sugerida: grilla 3 x 2 desde `y=370`, tarjetas de `360 x 88 px`.

Tarjetas visuales:

1. `Resumen ejecutivo`
2. `Calidad de llamadas`
3. `Satisfaccion de capacitaciones`
4. `Motivacion comercial`
5. `Detalle por call center`
6. `Notas metodologicas`

Como las paginas destino aun no existen en esta fase, dejarlas como tarjetas visuales sin navegacion funcional. La navegacion se completara en la fase correspondiente, cuando existan las paginas internas.

## Nota metodologica visible

Ubicacion sugerida: franja inferior `x=48`, `y=660`, `w=1184`, `h=32`.

Texto:

`Los datos actuales corresponden a una muestra piloto. Interpretar los indicadores considerando el n de respuestas.`

Estilo: texto pequeno, gris medio, sin competir con los KPIs.

## Validaciones realizadas

- Se ejecuto `git status` antes de modificar: solo existia `AGENTS.md` sin seguimiento.
- Se inspecciono `pages.json`, `page.json` y `report.json`.
- Se identifico la pagina vacia actual.
- Se renombro la pagina a `Home`.
- Se valido que `page.json`, `pages.json` y `report.json` siguen siendo JSON valido.
- Se verifico que las medidas requeridas para los KPIs existen en las tablas de medidas.
- Se verifico que no se modificaron medidas DAX.
- Se verifico que no se modificaron relaciones.
- Se verifico que no se modifico Power Query.
- Se verifico que no se modificaron archivos `Data/*.xlsx`.
- Se verifico que no se agregaron manualmente `description`, `lineageTag` ni `queryGroup` en TMDL.

Validaciones que requieren Power BI Desktop:

- Confirmar que el PBIP abre sin errores despues del renombre.
- Construir manualmente los visuales del Home segun este wireframe.
- Confirmar que el logo se visualiza correctamente.
- Confirmar que los KPIs muestran valores.

## Errores encontrados y solucion aplicada

No se encontraron errores de JSON ni de estructura del reporte.

Riesgo identificado: ausencia total de visuales persistidos como patron PBIR en este repo. Solucion aplicada: no serializar visuales especulativos; renombrar la pagina de forma segura y documentar wireframe exacto para construccion manual en Desktop.

## Archivos modificados

- `PBI/PBI_Indicadores.Report/definition/pages/67eff42d82e1c9c15b84/page.json`
- `Outputs/19_resultado_fase_13_home_landing_page.md`

No se modificaron:

- `PBI/PBI_Indicadores.SemanticModel/definition/expressions.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/relationships.tmdl`
- `PBI/PBI_Indicadores.SemanticModel/definition/tables/*.tmdl`
- `Data/*.xlsx`
- `PBI/PBI_Indicadores.Report/definition/report.json`

## Resultado del commit

Commit previsto:

`feat(report): crear home landing page connect`

El commit debe incluir solo `page.json` y este documento. `AGENTS.md` queda fuera por ser un cambio no relacionado con Fase 13.

## Estado final de `git status`

Estado esperado tras el commit:

```text
?? AGENTS.md
```

## Recomendacion para avanzar a Fase 14

Avanzar a Fase 14 solo despues de:

1. Abrir el PBIP en Power BI Desktop y confirmar que la pagina `Home` existe.
2. Construir manualmente el Home siguiendo este wireframe.
3. Guardar desde Power BI Desktop.
4. Revisar el diff generado por Desktop para confirmar que los visuales se serializaron correctamente.
5. Comitear cualquier visual del Home guardado por Desktop en una correccion o cierre complementario de Fase 13 antes de iniciar paginas internas.

No avanzar a Fase 14 mientras el Home visual no haya sido confirmado en Power BI Desktop.
