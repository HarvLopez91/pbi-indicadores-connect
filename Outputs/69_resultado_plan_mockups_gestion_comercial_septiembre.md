# Resultado — Plan de mockups de gestión comercial septiembre

| Campo | Valor |
|---|---|
| Fase | R0 — Diagnóstico funcional y planificación |
| Estado | Documentación creada; pendiente de aprobación |
| Implementación PBIP | No iniciada |
| Commit | No creado |
| Push | No realizado |

## 1. Objetivo

Consolidar las decisiones R0, el impacto estructural y un plan definitivo R1-R10 para los tres mockups comerciales de septiembre, sin modificar el artefacto Power BI ni las fuentes.

## 2. Documentos creados

- [Análisis de impacto](../Specs/12_analisis_impacto_mockups_gestion_comercial_septiembre.md)
- [Plan de implementación](../Specs/13_plan_implementacion_mockups_gestion_comercial_septiembre.md)
- Este resultado de planificación.

## 3. Decisiones consolidadas

- Fuente única de ventas: `Consolidado informe de Altas.xlsx`, tabla `Insumo2`.
- Ventas: `SUM(ALTAS)`; no conteo de filas.
- `FECHA_ALTA` es autoridad temporal y `MES` es control de calidad.
- Agosto de 2026 está cerrado; septiembre continúa en curso.
- Las reexpresiones históricas del consolidado sustituyen el baseline anterior.
- Asignación desde `Asignacion_PUSHER`, con reglas explícitas y sin fuzzy matching.
- Reutilización de `Dim_AsignacionPusherPeriodo`; sin dimensión PUSHER paralela.
- Meta Página 1: `CALL`; `ESPECIALISTA` es control de igualdad.
- Meta Página 2: `ASESOR`, una meta contextual deduplicada por aliado/PUSHER/mes.
- Bono objetivo queda omitido/N/A hasta confirmar la semántica de incentivos.
- Legalización se limita a gasto, legalizado, pendiente y corte; no se inventan relaciones ni valor recibido.
- Los tres mockups se implementarán como páginas nuevas; `GestionComercialAltas` permanecerá intacta durante R7-R9.
- Los mockups, Excel y `graphify-out/` no se versionan.

## 4. Conciliaciones registradas

La especificación incorpora la conciliación enero-septiembre, incluidos julio 4.519, agosto cerrado 5.715 y septiembre parcial 5.628. También registra la migración de meta de julio de 6.317 a 6.337 y los controles de asignación/drivers que podrían cambiar.

## 5. Contexto estructural

Graphify se utilizó como mapa local de dependencias y se contrastó con TMDL/PBIR. Se confirmó la cadena Base → Limpio → Fact → dimensiones/asignación/metas → medidas → página comercial.

La solución mínima conserva:

- un solo hecho de ventas;
- la fact de metas existente, ampliada con meta individual contextual;
- la asignación temporal existente;
- los patrones y componentes de la página comercial existente, sin modificarla.

Solo se consideran realmente nuevos una dimensión mínima de asesor si supera el gate de privacidad, una fact de legalización por su grano distinto y las tres páginas de los mockups.

## 6. Fases propuestas

| Fase | Alcance |
|---|---|
| R1 | Protección y baseline |
| R2 | Migración del consolidado |
| R3 | Asignación PUSHER |
| R4 | Metas |
| R5 | Incentivos y legalización |
| R6 | Medidas DAX |
| R7 | Nueva Página 1 de resumen comercial |
| R8 | Página 2 de asesores |
| R9 | Página 3 de incentivos/legalización |
| R10 | QA, documentación, publicación y cierre |

Cada fase incluye objetivo, objetos esperados, reutilización, cambio mínimo, validaciones, gate manual, PASS y rollback. Ninguna aprobación autoriza automáticamente la fase siguiente.

## 7. Riesgos y deudas

Riesgos principales:

- doble conteo por combinar snapshots;
- pérdida de ventas por confiar en `MES`;
- ruptura por columna opcional ausente;
- colisiones de asignación;
- metas repetidas por niveles de incentivo;
- comparación de meses parciales con meses completos;
- exposición nominal mediante Publicar en la Web;
- relaciones no demostradas en legalización;
- regresión de la página publicada.

Deudas no bloqueantes:

- semántica del bono objetivo;
- contrato de `Valor recibido`;
- relación de legalización con aliado/asesor;
- sanitización de mockups.

## 8. Validaciones de esta ejecución

- Numeración disponible confirmada: `Specs/12`, `Specs/13`, `Outputs/69`.
- Dependencias Graphify contrastadas con archivos reales.
- Referencias técnicas verificadas contra Power Query, TMDL y PBIR.
- Documentación sin equivalencias nominales internas ni ejemplos personales.
- Alcance limitado a los tres documentos autorizados.
- PBIP, Power Query, TMDL, PBIR, Excel, mockups y `graphify-out/`: sin modificaciones.

## 9. Estado

R0 funcional permanece cerrado. El plan R1-R10 queda **pendiente de aprobación**. No se implementó ninguna fase, no se creó commit y no se realizó push.
