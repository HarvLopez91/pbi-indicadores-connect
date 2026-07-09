# Publicación del informe en Power BI

## 1. Link publicado

```
https://app.powerbi.com/view?r=eyJrIjoiZGI2ZjNiYmItODQ0Yy00M2Y1LThkNTYtZGQ5NDIxYWExNjk3IiwidCI6Ijc1NDEyNGJlLTM2NGItNDg1MS1hYzA3LTc0ZjljZGJhYzM0ZiIsImMiOjR9&pageName=67eff42d82e1c9c15b84
```

- **Formato del enlace:** `app.powerbi.com/view?r=...` corresponde a un enlace de **"Publicar en la Web"** de Power BI Service. Este tipo de enlace **no requiere inicio de sesión**: cualquier persona que lo reciba puede ver el informe, sin controles de acceso ni auditoría de quién lo consultó.
- **Página inicial publicada:** el parámetro `pageName=67eff42d82e1c9c15b84` apunta a la página **Home**, consistente con el diseño de landing page del informe.
- El link puede depender de la vigencia de la publicación, de la configuración del tenant de Power BI (si la organización permite "Publicar en la Web") y de que el informe no haya sido despublicado o reemplazado desde Power BI Service — nada de esto se controla desde este repositorio.

## 2. Consideración de gobierno de datos (importante)

Dos visuales del informe muestran **nombres reales de personas**:

- `cl_tabla_asesor` (página Calidad de llamadas) — columna `NombreAsesor`.
- `sc_tabla_formador` (página Satisfacción de capacitaciones) — columnas `NombreFormador` y `NombreLider`.

Como el enlace publicado es de acceso público sin autenticación, **cualquier persona con el enlace puede ver esos nombres**, sin importar si pertenece o no a Connect Assistance. Esto ya estaba señalado como riesgo pendiente en `Docs/05_decisiones_limitaciones_pendientes.md` §4. Se recomienda que negocio confirme explícitamente si ese nivel de exposición es aceptable para el piloto, o si conviene:

- Ocultar las columnas de nombre en esas 2 tablas antes de publicar de nuevo, o
- Migrar la publicación de "Publicar en la Web" a un **espacio de trabajo de Power BI Service con control de acceso** (compartir solo con usuarios/grupos autorizados de la organización), que sí permite auditoría y revocación de acceso.

## 3. Recomendaciones al publicar (o republicar)

Antes de generar o actualizar un enlace de publicación, validar en Power BI Desktop:

- [ ] **Filtros**: los segmentadores de Fecha, Call Center y Jornada responden correctamente en las 5 páginas de detalle.
- [ ] **Navegación**: clic en cualquier punto de las 6 tarjetas de Home y de los 6 botones "Volver a Home" navega correctamente (ver `Outputs/28`).
- [ ] **Etiquetas de datos**: los 8 gráficos muestran sus etiquetas con el estilo Connect (`#002733`, tamaño moderado) — no deben quedar desactivadas por un cambio accidental.
- [ ] **Actualización**: los datos reflejan la versión más reciente de `Data/*.xlsx` (ejecutar Actualizar antes de publicar, ver `Docs/04_fuentes_y_actualizacion_datos.md`).
- [ ] **Permisos**: revisar quién tiene acceso de edición al informe en Power BI Service y si el enlace de "Publicar en la Web" sigue siendo el mecanismo de distribución deseado (ver §2).
- [ ] **Vigencia del enlace**: confirmar en Power BI Service (Archivo → Publicar en la Web → Administrar enlaces) que el enlace sigue activo y corresponde a la versión actual del informe.

## 4. Checklist posterior a cada publicación

Después de publicar o republicar:

1. Abrir el enlace publicado en una ventana de navegación privada (sin sesión iniciada) para confirmar que carga sin pedir credenciales inesperadas y que muestra la página Home.
2. Verificar visualmente que el tema Connect (naranja `#F15B2B`, oscuro `#002733`) se ve igual que en Power BI Desktop — la vista publicada a veces difiere de la vista de edición.
3. Confirmar que la fecha de los datos mostrados corresponde a la última actualización esperada.
4. Registrar la fecha de publicación y cualquier cambio relevante en un nuevo archivo `Outputs/NN_...md`, siguiendo el patrón de documentación ya establecido en este proyecto.
5. Si se detecta algún problema visual o de datos en la versión publicada que no existía en Power BI Desktop, no intentar corregirlo editando la versión publicada directamente — corregir en el PBIP, validar en Desktop, y volver a publicar.
