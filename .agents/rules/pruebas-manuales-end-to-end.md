---
trigger: always_on
---

# Regla: Pruebas Manuales End-to-End

Toda funcionalidad deberá poder validarse mediante pruebas manuales utilizando **Postman**. Antes de desarrollar nuevas funcionalidades o realizar cambios importantes, se deberá construir un **roadmap funcional de extremo a extremo (End-to-End)** que documente el flujo completo del proceso de negocio.

El roadmap deberá seguir el orden real de ejecución del sistema, permitiendo que cualquier desarrollador pueda reproducir el flujo completo únicamente ejecutando las peticiones en Postman y comprendiendo qué sucede internamente en cada paso.

Para cada endpoint se deberá documentar, como mínimo, la siguiente información:

* Nombre de la funcionalidad.
* Objetivo del proceso.
* Dominio o módulo al que pertenece.
* Método HTTP y ruta del endpoint.
* Descripción de la operación.
* Headers requeridos.
* Parámetros de ruta (Path Parameters).
* Parámetros de consulta (Query Parameters).
* Body completo del request con ejemplos.
* Response esperada con ejemplos.
* Códigos HTTP posibles.
* Validaciones y reglas de negocio.
* Precondiciones necesarias para ejecutar el endpoint.
* Dependencias con otros endpoints o procesos.
* Tablas de la base de datos involucradas.
* Operaciones realizadas sobre cada tabla (SELECT, INSERT, UPDATE y DELETE).
* Transacciones ejecutadas y su impacto sobre la información.
* Eventos, colas o procesos asíncronos desencadenados.
* Integraciones con servicios externos, indicando las llamadas realizadas.
* Posibles errores, escenarios de fallo y comportamiento esperado en cada caso.

El roadmap deberá representar el recorrido completo del caso de uso, desde el primer endpoint hasta el último, incluyendo todas las decisiones, validaciones e interacciones entre servicios.

La documentación deberá mantenerse actualizada conforme evolucione el sistema y será considerada la fuente de verdad para comprender el comportamiento funcional y técnico del producto, facilitar las pruebas manuales, acelerar el onboarding de nuevos desarrolladores y servir como base para pruebas automatizadas e integración continua.

## Fidelidad de Contratos y Payloads
* **Prohibido Inventar Campos**: Al redactar roadmaps E2E, documentación de APIs, contratos de integración o crear colecciones de pruebas (Postman/xUnit), el agente tiene estrictamente prohibido inventar o asumir los payloads o campos (ej. `name`, `email`).
* **Validación Obligatoria**: Antes de generar cualquier payload (JSON) o definir reglas de validación, el agente **DEBE** leer primero el esquema de base de datos real (ej. `brain/01_Architecture/Nexus.dbml`) o las entidades de Dominio en C# (`src/Backend/src/Nexus.Domain/Entities/`) para asegurar que los nombres de las propiedades (ej. `legalName`, `taxId`) coinciden 100% con la arquitectura del sistema.
