# Resultado — Big Picture de arquitectura del sistema y reubicación de mockup

| Campo | Valor |
|---|---|
| Proyecto | `PBI_Indicadores` |
| Tipo de tarea | Documentación de arquitectura + organización de recursos (dos tareas controladas, sin análisis de impacto) |
| Documento de referencia | Diagnóstico previo entregado en conversación (ausencia de Big Picture de arquitectura), sin archivo `Outputs/` propio |
| Fecha | 2026-07-21 |
| Alcance | Documentación y reorganización de `Assets/` exclusivamente. No se modificó ningún archivo PBIR, TMDL, Power Query, DAX, relaciones, visuales ni `Data/*.xlsx`. |

---

## 1. Estado inicial de `git status`

Al iniciar, `git status --porcelain` mostraba:

```
?? "Dashboard de satisfacción de capacitaciones.png"
?? Data/
```

La imagen estaba sin trackear en la raíz del proyecto, con espacios y tilde en el nombre. `Data/` aparece sin trackear por diseño (excluida vía `.gitignore`, ver `CLAUDE.md`) y no se tocó.

## 2. Tarea 1 — Reubicación de la imagen de referencia

- **Archivo origen**: `Dashboard de satisfacción de capacitaciones.png` (raíz del proyecto).
- **Carpeta nueva creada**: `Assets/mockups/` — sigue el patrón ya existente de subcarpetas temáticas dentro de `Assets/` (`logos/`, `imagenes/`, `theme/`), reservada específicamente para recursos de referencia/mockups que aún no están cargados al PBIR.
- **Archivo destino**: `Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png` — nombre en `snake_case`, sin espacios, sin tildes ni caracteres especiales, siguiendo la convención de nombres de archivo ya usada en el proyecto (p. ej. `logo_connect_naranja_20260708.png`, `connect_assistance_theme.json`).
- **Confirmado**: la imagen **no** se referenció en ningún archivo `.json` de `PBI_Indicadores.Report/definition/` ni en `StaticResources/` — sigue siendo un recurso de referencia externo al PBIR, conforme a lo solicitado.

## 3. Tarea 2 — Documento de arquitectura (Big Picture)

Se creó [`Docs/07_arquitectura_sistema.md`](../Docs/07_arquitectura_sistema.md), con:

- Diagrama Mermaid (`flowchart TD`) que conecta las 10 capas del sistema: fuentes (Google Forms) → `Data/*.xlsx` → Power Query → modelo TMDL → medidas DAX → reporte PBIR → Power BI Service, con Git/GitHub y `Specs/`/`Docs/`/`Outputs/` como capas transversales de versionado y documentación.
- Flujo end-to-end en 10 pasos numerados, desde la captura en Google Forms hasta la publicación en Power BI Service, enlazando cada paso al documento `Docs/0N` que lo detalla.
- Tabla de capas (qué es, dónde vive el archivo, dónde se documenta).
- Sección de dependencias entre capas (qué se rompe aguas abajo si cambia una columna, una medida o un nombre de dimensión).
- Sección de riesgos de arquitectura: publicación pública sin autenticación, `Data/` como punto único de fallo no versionado, ausencia de catálogo maestro (D4/D5), ruido de Auto Date/Time, riesgo de desalineación de documentación ante crecimiento no coordinado, ausencia de build/prueba automática.
- Recomendaciones para futuras versiones, incluyendo tratar la integración de Altas T Resuelve (`Specs/04`) como extensión de este Big Picture en vez de un pipeline paralelo ad-hoc, y resolver el gobierno de la publicación pública antes de sumar fuentes con más datos personales.

El documento enlaza a `Docs/01`–`06` como fuente de verdad de cada capa, sin duplicar su contenido — sigue el mismo patrón de "enlazar, no repetir" ya usado en `Docs/00_indice_documentacion.md`.

## 4. Actualización del índice de documentación

Se agregó una fila a la tabla de `Docs/00_indice_documentacion.md` §1, referenciando `07_arquitectura_sistema.md` con su propósito (Big Picture, diagrama Mermaid, flujo end-to-end, dependencias y riesgos). No se modificó el resto del índice ni las rutas de lectura por rol (§2).

## 5. Validaciones realizadas

- **Sin cambios en PBIR/TMDL/DAX/relaciones/visuales**: confirmado por inspección — solo se tocaron `Docs/00_indice_documentacion.md` (edición), `Docs/07_arquitectura_sistema.md` (nuevo), `Assets/mockups/dashboard_satisfaccion_capacitaciones_mockup.png` (movido) y este archivo.
- **Sin cambios en `Data/`**: `git status --porcelain -- Data/` no cambió respecto al estado inicial (sigue sin trackear, sin modificaciones).
- **Nombre de archivo normalizado**: `dashboard_satisfaccion_capacitaciones_mockup.png` no contiene espacios, tildes ni caracteres fuera de `[a-z0-9_.]`.
- **Imagen no referenciada en el PBIR**: se verificó que ningún `page.json`/`visual.json`/`report.json` menciona el nombre del archivo, antes ni después del movimiento.
- **Diagrama Mermaid**: sintaxis revisada manualmente (nodos, subgrafos y aristas balanceados); no se dispone de un renderizador Mermaid en este entorno, por lo que se recomienda confirmar visualmente en GitHub o en un editor con soporte Mermaid.

## 6. Resultado de `git status` tras los cambios

```
?? "Data/"                                   (sin cambios, preexistente)
 M Docs/00_indice_documentacion.md
?? Docs/07_arquitectura_sistema.md
?? Assets/mockups/
```

(La imagen ya no aparece en la raíz — se movió, no se duplicó.)

## 7. Commit sugerido

`docs: agregar big picture de arquitectura y mockup de referencia`

No se hizo push remoto.

## 8. Recomendaciones futuras

1. Mantener `Docs/07_arquitectura_sistema.md` actualizado cada vez que se agregue una fuente, una capa o cambie el mecanismo de publicación — el mismo riesgo de desalineación que aplica a `Docs/01`–`06` aplica a este documento.
2. Si se decide avanzar con la integración de Altas T Resuelve (`Specs/04`), usar `Docs/07` §6 como checklist de arquitectura antes de escribir el plan de implementación `v1.1`.
3. `Assets/mockups/` queda disponible para futuras imágenes de referencia/diseño (wireframes, capturas de otras herramientas) — mantenerlas fuera del PBIR hasta que exista una decisión explícita de usarlas como recurso del reporte.
