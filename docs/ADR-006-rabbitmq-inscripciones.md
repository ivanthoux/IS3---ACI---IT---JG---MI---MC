# ADR-006 Inclusión de cola de mensajes (RabbitMQ) para alta concurrencia en inscripciones

**Estado:** Propuesto  
**Fecha:** 2026-06-09  
**Decisores:** Giménez, Joaquin Elian; Equipo de Desarrollo  
**Relacionado:** Spec 04 (OWASP Race Conditions), Issue #26  
---

## Contexto

- **Qué problema se está resolviendo:** Cuando se abre la inscripción a un evento muy demandado (ej. 1000 usuarios intentan inscribirse en el mismo segundo para un cupo de 100), los bloqueos transaccionales de MySQL (Row-level locking) causan deadlocks, timeouts y alta latencia.
- **Qué restricciones aplican:** Se debe asegurar que nunca se sobrepase el cupo máximo exacto y que la lista de espera se asigne de forma estrictamente FIFO.
- **Qué datos de proyecto sustentan la decisión:** Actualmente la API procesa la inscripción de forma síncrona en el mismo hilo HTTP, lo cual bloquea el Event Loop de Node.js durante los picos.

---

## Decisión

Implementar **RabbitMQ** como intermediario para asincronizar el procesamiento de inscripciones en eventos de alta demanda.

- **Alcance:** La ruta `POST /events/:id/register` solo colocará el mensaje de intención en la cola y devolverá un estado "Procesando". Un Worker consumirá la cola 1 a 1 para asegurar secuencialidad y consistencia en el cupo.

---

## Alternativas consideradas

- **Opción A - Bloqueo pesimista en DB:**
  - *Pros:* Arquitectura simple, sin componentes adicionales.
  - *Contras:* No escala frente a picos masivos de peticiones simultáneas.

- **Opción B - Redis Distributed Lock:**
  - *Pros:* Mantiene la respuesta síncrona al usuario y evita sobre-inscripción.
  - *Contras:* Node.js y la base de datos siguen recibiendo la carga de peticiones completas al unísono, pudiendo agotar conexiones.

---

## Consecuencias

- **Beneficios esperados:** Eliminación de deadlocks en la DB, el servidor web no se cae ante picos extremos, y garantía absoluta del orden de llegada para la lista de espera (FIFO inherente de la cola).
- **Costos o riesgos que se aceptan:** Complejidad en la experiencia de usuario (UX), ya que la interfaz deberá usar Polling o WebSockets para notificar si finalmente obtuvo el cupo o quedó en lista de espera.
- **Impacto en operación y equipo:** Necesidad de desplegar y mantener un nuevo servicio (RabbitMQ) y un script separado (Worker).

---

## Plan de implementación

1. Levantar RabbitMQ en Docker.
2. Instalar y configurar `amqplib` en Node.js.
3. Refactorizar `registrationController.js` para enviar el payload a la cola.
4. Crear un `worker.js` que consuma la cola, inicie la transacción en DB, reste el cupo y actualice el estado a `"confirmed"` o `"waitlisted"`.

**Dependencias:** Servidor RabbitMQ, WebSockets o Server-Sent Events (SSE) para notificar al frontend.  
**Métrica de éxito:** Capacidad de recibir 5000 peticiones de inscripción por segundo sin caídas ni timeouts (HTTP 202 Accepted inmediato), resolviendo el 100% de los estados correctamente en base al cupo.

---

## Triggers de revisión

- **Condiciones:** Quejas de usuarios por la asincronía (ej. "Me dijo procesando y al final no entré").
