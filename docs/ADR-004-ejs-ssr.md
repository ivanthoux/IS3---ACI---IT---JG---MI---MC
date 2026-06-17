# ADR-004 Motor de Templates Server-Side (EJS) para vistas y paneles

**Estado:** Aceptado  
**Fecha:** 2026-06-09  
**Decisores:** Thoux, Ivan Ezequiel; Equipo de Desarrollo  
**Relacionado:** Spec 07, Spec 08, Contracts.md  

---

## Contexto

- **Qué problema se está resolviendo:** Elegir la tecnología para el desarrollo del Frontend de la aplicación (vistas de eventos, escáner de QR, dashboard de organizador).
- **Qué restricciones aplican:** Necesidad de entregar un producto completo rápido, tener buen SEO (Search Engine Optimization) nativo para que los eventos sean indexables por Google y evitar el exceso de complejidad manteniendo un solo repositorio monolítico.
- **Qué datos de proyecto sustentan la decisión:** La aplicación requiere vistas estáticas robustas y secciones con control de sesión fuerte basado en cookies HTTP-Only generadas por el backend.

---

## Decisión

Utilizar **EJS (Embedded JavaScript templating)** para el Server-Side Rendering (SSR) junto con Bootstrap 5 como framework de CSS.

- **Alcance:** Toda la UI del sistema web será despachada ya renderizada (HTML) desde los controladores de Express.

---

## Alternativas consideradas

- **Opción A - SPA separada (React / Vue):**
  - *Pros:* Mayor interactividad, fluidez, experiencia similar a una app nativa.
  - *Contras:* Mayor costo de desarrollo, requiere mantener dos proyectos separados (API + Frontend).

- **Opción B - Handlebars / Pug:**
  - *Pros:* Sintaxis limpia y estructurada.
  - *Contras:* EJS permite ejecutar JavaScript puro entre etiquetas `<% %>`, lo cual es más natural y de curva de aprendizaje casi nula comparado con las reglas estrictas de Pug o los helpers limitados de Handlebars.

---

## Consecuencias

- **Beneficios esperados:** Simplicidad arquitectónica, velocidad de desarrollo superior (Backend y Frontend fuertemente acoplados), excelente SEO en el listado público de eventos.
- **Costos o riesgos que se aceptan:** Cargas de página completas en cada navegación (excepto partes donde explícitamente se integre AJAX/Fetch como el escáner de `html5-qrcode`).
- **Impacto en operación y equipo:** Los desarrolladores necesitan trabajar simultáneamente con la capa de controladores y las vistas asociadas.

---

## Plan de implementación

1. Configurar EJS mediante `app.set('view engine', 'ejs')`.
2. Estructurar el directorio `src/views` definiendo un layout principal y partials (navbar, footer, comments).
3. Utilizar Bootstrap vía CDN para agilizar.

**Dependencias:** Librería `ejs`, Bootstrap 5.  
**Métrica de éxito:** Capacidad de renderizar las páginas de detalle de evento en menos de 300ms de TTFB (Time to First Byte).

---

## Triggers de revisión

- **Condiciones:** Si se planifica desarrollar una aplicación móvil (App) en el futuro, requiriendo obligatoriamente separar el backend en una API 100% JSON.
