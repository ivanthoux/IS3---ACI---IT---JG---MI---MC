# ADR-003 Uso de PDFKit para la generación de Certificados

**Estado:** Aceptado  
**Fecha:** 2026-06-09  
**Decisores:** Ivasiuta, Magdalena Ramona; Equipo de Desarrollo  
**Relacionado:** Spec 05, Contracts.md  

---

## Contexto

- **Qué problema se está resolviendo:** El sistema debe emitir certificados automáticos (en formato PDF) con códigos de verificación únicos cuando un evento finaliza o se completa una encuesta.
- **Qué restricciones aplican:** La generación debe hacerse del lado del servidor (backend) y almacenarse en disco (`public/certificates/`). El archivo debe ser inmutable.
- **Qué datos de proyecto sustentan la decisión:** Se necesitan generar certificados individualizados con datos dinámicos (nombre, evento, fecha, código QR).

---

## Decisión

Utilizar la librería `pdfkit` en Node.js para la generación programática de los documentos PDF de certificados.

- **Alcance:** Creación del diseño base (texto, líneas, inserción del logo e imagen del QR) y guardado en el File System del servidor.

---

## Alternativas consideradas

- **Opción A - Puppeteer / Headless Chrome:**
  - *Pros:* Permite diseñar el certificado usando HTML/CSS convencional y simplemente "imprimir" a PDF.
  - *Contras:* Extremadamente pesado, consume demasiada memoria RAM y CPU por cada instancia del navegador; no es óptimo para generar cientos de certificados de golpe.

- **Opción B - Generación en frontend:**
  - *Pros:* Carga de procesamiento derivada al cliente.
  - *Contras:* Inseguro, dificulta el almacenamiento de la copia original en el servidor para validación pública; rompe el flujo automatizado de "al cerrar evento, generar a todos".

---

## Consecuencias

- **Beneficios esperados:** Alta velocidad de generación de documentos, bajo consumo de recursos (comparado a usar un navegador headless) y control absoluto sobre el archivo generado.
- **Costos o riesgos que se aceptan:** El maquetado del documento es más complejo y tedioso, ya que requiere ubicar los elementos mediante coordenadas absolutas (X, Y) en lugar de usar CSS.
- **Impacto en operación y equipo:** Curva de aprendizaje inicial para la API de dibujado de PDFKit.

---

## Plan de implementación

1. Instalar `pdfkit`.
2. Crear el `pdfService.js` definiendo la clase base del certificado (tamaño A4, tipografías, ubicación de variables).

**Dependencias:** Librería `pdfkit`, acceso de escritura al directorio `public/certificates`.  
**Métrica de éxito:** Un certificado A4 con imágenes insertadas generado en menos de 200ms en el servidor.

---

## Triggers de revisión

- **Condiciones:** Si el equipo de diseño exige plantillas altamente gráficas y cambiantes que resultan imposibles de codificar mediante coordenadas estáticas en PDFKit.
