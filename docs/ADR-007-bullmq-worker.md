# ADR-007 Implementación de BullMQ para generación masiva asíncrona de certificados

**Estado:** Propuesto  
**Fecha:** 2026-06-09  
**Decisores:** Ivasiuta, Magdalena Ramona; Equipo de Desarrollo  
**Relacionado:** Spec 05, Issue #25  

---

## Contexto

- **Qué problema se está resolviendo:** Latencia inaceptable y bloqueo del Event Loop de Node.js al ejecutar la regla de negocio "generar certificados de disertante/autor automáticamente cuando el evento se marca como completed". En eventos con decenas de disertantes o congresos, generar y escribir decenas/cientos de PDFs simultáneamente frena la respuesta HTTP del Organizador.
- **Qué restricciones aplican:** El organizador no debe quedar esperando en un estado de carga mientras se procesan los archivos para poder ver el evento como "Finalizado".
- **Qué datos de proyecto sustentan la decisión:** Node.js es monohilo; operaciones masivas intensivas en CPU/Disco bloquean las peticiones de todos los demás usuarios de la aplicación.

---

## Decisión

Incluir **BullMQ** (respaldado por Redis) para gestionar trabajos en segundo plano (Background Jobs).

- **Alcance:** Generación de PDFs y el posterior envío de emails notificando la disponibilidad de los certificados.

---

## Alternativas consideradas

- **Opción A - Promises síncronas:**
  - *Pros:* Sin dependencias extra.
  - *Contras:* Bloquea completamente el servidor ante volúmenes grandes; si una falla, es difícil gestionar reintentos (Retries) de los archivos específicos que fallaron.

- **Opción B - Cron Jobs periódicos para batch processing:**
  - *Pros:* Saca la carga de la petición HTTP.
  - *Contras:* Introduce demora innecesaria; los certificados no se generan en cuanto el evento se cierra sino en el próximo ciclo del cron.

---

## Consecuencias

- **Beneficios esperados:** La solicitud HTTP de cerrar el evento responde en menos de 50ms. Los certificados se encolan y generan de manera controlada (ej. 5 por segundo) sin afectar al resto de usuarios de la web. Permite reintentos automáticos en caso de fallo (ej. fallo temporal del disco o de envío de email).
- **Costos o riesgos que se aceptan:** Complejidad de infraestructura (requiere Redis).
- **Impacto en operación y equipo:** Necesidad de un panel de monitoreo (como Bull-Board) para ver certificados estancados o fallidos.

---

## Plan de implementación

1. Instalar Redis y la librería `bullmq`.
2. Refactorizar `certificateController.js` para añadir las tareas (Jobs) a la cola `certificate-queue` en lugar de llamar directo al servicio.
3. Crear un archivo `worker.js` que escuche esta cola, procese el PDF usando `pdfkit` y envíe el correo vía `nodemailer`.

**Dependencias:** Redis, módulo `bullmq`.  
**Métrica de éxito:** Marcar un evento con 100 autores como "Completed" retorna un status 200 al instante, y los 100 PDFs se generan exitosamente en un lapso de 1 a 2 minutos en background sin bloquear otras rutas.

---

## Triggers de revisión

- **Condiciones:** Errores continuos en la generación en background sin notificación al organizador.
