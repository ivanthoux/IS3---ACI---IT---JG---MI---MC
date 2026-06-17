# ADR-001 Implementación de express-session en base de datos para autenticación

**Estado:** Aceptado  
**Fecha:** 2026-06-09  
**Decisores:** Caruk, Maria Eugenia; Equipo de Desarrollo  
**Relacionado:** Spec 01, Contracts.md  

---

## Contexto

- **Qué problema se está resolviendo:** Se necesitaba definir el mecanismo para mantener el estado de autenticación de los usuarios (login) a través de las distintas peticiones HTTP.
- **Qué restricciones aplican:** Requerimiento de invalidar sesiones globalmente si se cambia la contraseña, facilidad de implementación en el ecosistema Node.js y control estricto sobre el tiempo de inactividad (24hs).
- **Qué datos de proyecto sustentan la decisión:** El sistema prevé roles (Admin, Organizador, etc.) que exigen un control estricto de acceso. Si un administrador quita un rol, la sesión debe poder verificarse en tiempo real.

---

## Decisión

Implementar autenticación basada en estado (Stateful) utilizando `express-session` respaldada por la base de datos MySQL mediante `connect-session-sequelize`.

- **Alcance:** Cubre la persistencia de cookies de sesión cifradas en el cliente y el almacenamiento del estado en el servidor. No cubre autenticación por terceros (OAuth).

---

## Alternativas consideradas

- **Opción A - JSON Web Tokens (JWT):**
  - *Pros:* Stateless, ideal para APIs y escalabilidad horizontal.
  - *Contras:* Difícil invalidar tokens antes de su expiración (requiere listas negras), más complejo para revocar accesos en tiempo real tras un cambio de contraseña o rol.

- **Opción B - express-session en memoria:**
  - *Pros:* Muy rápido y fácil de configurar.
  - *Contras:* Pérdida de todas las sesiones activas al reiniciar el servidor, no escala en entornos multi-instancia.

---

## Consecuencias

- **Beneficios esperados:** Facilidad para destruir sesiones específicas desde el backend (ej. al recuperar contraseña) y protección contra Session Fixation.
- **Costos o riesgos que se aceptan:** Mayor carga en la base de datos transaccional, ya que cada petición protegida requiere consultar la tabla de sesiones.
- **Impacto en operación y equipo:** Requiere limpiar periódicamente las sesiones expiradas en la base de datos para no saturar el almacenamiento.

---

## Plan de implementación

1. Instalar `express-session` y `connect-session-sequelize`.
2. Configurar el middleware en `app.js` pasando el store de Sequelize.
3. Implementar la rotación del Session ID en el login exitoso.

**Dependencias:** MySQL, Sequelize, express-session.  
**Métrica de éxito:** Capacidad de iniciar sesión, navegar módulos restringidos y cerrar sesión invalidando inmediatamente el acceso.

---

## Triggers de revisión

- **Condiciones:** Si el tráfico de usuarios concurrentes supera los 5000 y el acceso a la base de datos relacional para leer sesiones se convierte en un cuello de botella.
- **Fecha sugerida de revisión:** 6 meses luego de la salida a producción.
