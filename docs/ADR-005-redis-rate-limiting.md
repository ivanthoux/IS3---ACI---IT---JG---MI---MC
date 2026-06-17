# ADR-005 Implementación de Redis para Rate Limiting y control de fuerza bruta

**Estado:** Propuesto  
**Fecha:** 2026-06-09  
**Decisores:** Caruk, Maria Eugenia; Equipo de Desarrollo  
**Relacionado:** Spec 01 (OWASP Controles), Issue #27  

---.

## Contexto

- **Qué problema se está resolviendo:** El endpoint de login y reenvío de correos está vulnerable a ataques de fuerza bruta distribuida y Credential Stuffing. Si bien se mitigó controlando fallos en la base de datos, las peticiones masivas agotan las conexiones concurrentes de MySQL causando alta latencia en toda la aplicación.
- **Qué restricciones aplican:** Necesidad de responder con bloqueo en milisegundos sin tocar la base de datos transaccional.
- **Qué datos de proyecto sustentan la decisión:** Pruebas de estrés mostraron que 500 intentos de login falsos por segundo degradan la velocidad del sistema para usuarios legítimos.

---

## Decisión

Incorporar un clúster de **Redis** en la infraestructura de Docker como capa en memoria para llevar el conteo de intentos fallidos por IP y por Email, aplicando Rate Limiting.

- **Alcance:** Protección de endpoints públicos de autenticación (Login, Registro, Reset Password). No reemplaza a MySQL como base de datos principal.

---

## Alternativas consideradas

- **Opción A - Bloqueo solo por Base de Datos (estado actual):**
  - *Pros:* No requiere infraestructura adicional.
  - *Contras:* Alta latencia y consumo de I/O en la DB ante ataques masivos.

- **Opción B - Web Application Firewall externo (como Cloudflare):**
  - *Pros:* Bloquea en el borde de la red, cero impacto en el servidor.
  - *Contras:* Dependencia de un proveedor externo, costo adicional si se requieren reglas personalizadas complejas.

---

## Consecuencias

- **Beneficios esperados:** Reducción drástica del impacto de ataques automatizados en el rendimiento de la DB. Tiempos de respuesta de rechazo menores a 10ms.
- **Costos o riesgos que se aceptan:** Adición de un nuevo componente a la infraestructura (Redis), lo que implica mayor consumo de RAM en el servidor y complejidad en el despliegue (docker-compose).
- **Impacto en operación y equipo:** El equipo de DevOps debe monitorear la salud y uso de memoria del contenedor de Redis.

---

## Plan de implementación

1. Agregar la imagen de Redis al `docker-compose.yml`.
2. Instalar la librería `rate-limiter-flexible` o `express-rate-limit` con `rate-limit-redis` en Node.js.
3. Configurar las reglas de bloqueo (ej. 5 fallos / 15 minutos).

**Dependencias:** Contenedor Docker de Redis, paquete npm `rate-limit-redis`.  
**Métrica de éxito:** Mantener el tiempo de respuesta del servidor web estable bajo un ataque de 1000 requests/segundo al endpoint de login, bloqueando el 100% del tráfico abusivo sin caer.

---

## Triggers de revisión

- **Condiciones:** Consumo de memoria RAM de Redis supera el 80% del límite asignado.
