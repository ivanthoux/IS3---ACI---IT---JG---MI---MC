# ADR-008 Escalado de Base de Datos mediante Read Replica para el Módulo de Informes

**Estado:** Propuesto  
**Fecha:** 2026-06-09  
**Decisores:** Thoux, Ivan Ezequiel; Equipo de Desarrollo  
**Relacionado:** Spec 07, Issue #28  

---

## Contexto

- **Qué problema se está resolviendo:** El módulo de "Informes y Agenda" realiza consultas altamente pesadas (funciones de agregación como COUNT, AVG, SUM, agrupamientos y cruces de múltiples tablas mediante JOINs para obtener tasas de asistencia, análisis estadístico de encuestas y distribución demográfica). Cuando el Organizador accede a su Dashboard o genera el reporte PDF, produce caídas de rendimiento transaccional.
- **Qué restricciones aplican:** Los reportes analíticos leen grandes volúmenes de datos históricos, secuestrando la CPU de MySQL y afectando las operaciones críticas (como escanear un código QR en la puerta o confirmar una inscripción).
- **Qué datos de proyecto sustentan la decisión:** El crecimiento orgánico de la base de datos post-evento impacta en operaciones del evento en curso (OLAP interfiriendo con OLTP).

---

## Decisión

Implementar el patrón de arquitectura **Master-Slave (Read Replica)** a nivel de base de datos MySQL.

- **Alcance:** Todas las consultas GET de los controladores analíticos (`reportController.js`, `surveyController.js` para promedios) se enrutarán exclusivamente a la base de datos de réplica de lectura.

---

## Alternativas consideradas

- **Opción A - Añadir Caché:**
  - *Pros:* Muy rápido y barato de implementar.
  - *Contras:* Si el organizador necesita los datos "en tiempo real" de quién está acreditándose ahora mismo (como lo exige HU-06), el caché estático provee información desactualizada.

- **Opción B - Data Warehouse separado (Elasticsearch):**
  - *Pros:* Rendimiento extremo en analítica y búsqueda de texto.
  - *Contras:* Extrema complejidad de sincronización, desproporcionado para el volumen actual del proyecto.

---

## Consecuencias

- **Beneficios esperados:** Aislamiento de cargas de trabajo. Las pesadas consultas estadísticas de los Informes ya no bloquean las operaciones transaccionales críticas (Inscripciones / Escaneos QR).
- **Costos o riesgos que se aceptan:** La réplica puede tener un replication lag de algunos milisegundos. Mayor costo de infraestructura (dos instancias de BD en lugar de una).
- **Impacto en operación y equipo:** Configurar el ORM Sequelize para manejar múltiples conexiones (Replication configuration: write host vs read host).

---

## Plan de implementación

1. Levantar un nodo Slave MySQL configurado con replicación asíncrona desde el nodo Master.
2. Modificar la configuración en `src/config/database.js` para usar la propiedad `replication` de Sequelize.
3. Garantizar que las escrituras (crear evento, inscribir, acreditar) vayan al Master, y lecturas analíticas vayan al pool del Réplica.

**Dependencias:** Infraestructura Cloud o Docker que soporte replicación nativa de bases de datos.  
**Métrica de éxito:** Descenso de la latencia en las escrituras transaccionales (acreditación con QR) por debajo de 50ms, incluso cuando concurrentemente se están generando múltiples reportes estadísticos complejos en el sistema.

---

## Triggers de revisión

- **Condiciones:** Si el retraso de replicación (lag) es tan alto que un organizador acredita a alguien por QR y no lo ve reflejado en su Dashboard de asistencia varios minutos después.
