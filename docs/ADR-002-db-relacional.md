# ADR-002 Elección de Base de Datos Relacional (MySQL) para persistencia central

**Estado:** Aceptado  
**Fecha:** 2026-06-09  
**Decisores:** Giménez, Joaquin Elian; Equipo de Desarrollo  
**Relacionado:** Spec 03, Spec 04, Contracts.md  Issue #26  

---

## Contexto

- **Qué problema se está resolviendo:** Definir el sistema de almacenamiento persistente para la información de eventos, inscripciones, usuarios y certificados.
- **Qué restricciones aplican:** Necesidad de garantizar la integridad de los datos, evitar dobles inscripciones y mantener los cupos sincronizados.
- **Qué datos de proyecto sustentan la decisión:** El modelo de datos presenta múltiples relaciones estructuradas N:M (Usuarios-Eventos para inscripciones, Usuarios-Roles).

---

## Decisión

Utilizar MySQL (motor InnoDB) como base de datos relacional principal, gestionada a través del ORM Sequelize.

- **Alcance:** Toda la persistencia de datos transaccionales de la plataforma.

---

## Alternativas consideradas

- **Opción A - MongoDB / NoSQL:**
  - *Pros:* Flexibilidad en el esquema, fácil lectura de documentos anidados (ej. Evento con sus participantes).
  - *Contras:* Complejidad para garantizar transacciones ACID al restar cupos o manejar la lista de espera estricta (FIFO), riesgo de inconsistencia de datos relacionales.

- **Opción B - PostgreSQL:**
  - *Pros:* Excelente manejo de concurrencia y funciones avanzadas.
  - *Contras:* Curva de aprendizaje del equipo más orientada a MySQL en el stack planteado inicialmente.

---

## Consecuencias

- **Beneficios esperados:** Garantía de integridad referencial (Foreign Keys), control estricto de bloqueos en transacciones para el manejo de cupos.
- **Costos o riesgos que se aceptan:** Esquemas rígidos que requieren migraciones estructuradas ante cualquier cambio en los requerimientos.
- **Impacto en operación y equipo:** Obligación de crear y mantener scripts de migración (Sequelize CLI).

---

## Plan de implementación

1. Definición del modelo E-R.
2. Configuración de `database.js` y creación de migraciones para Tablas y Foreign Keys.

**Dependencias:** Driver `mysql2`, ORM `sequelize`.  
**Métrica de éxito:** Cero incidencias de datos huérfanos o inconsistencias en los cupos registrados versus los disponibles.

---

## Triggers de revisión

- **Condiciones:** Dificultad para escalar escrituras masivas en un solo nodo central.
