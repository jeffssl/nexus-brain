---
title: "Roadmap E2E: Módulo Organization"
tags:
  - testing
  - e2e
  - roadmap
  - organization
  - tenant
  - postman
status: draft
---

# Roadmap Funcional End-to-End: Módulo Organization

Este documento define el flujo completo de extremo a extremo para el módulo **Organization** (Tenants, Clínicas, Laboratorios). Sigue el orden real de ejecución del sistema, permitiendo su validación manual mediante Postman.

## Flujo de Ejecución

1. **Crear Organización (Tenant Onboarding)**
2. **Obtener Organización por ID**
3. **Actualizar Datos de la Organización**
4. **Desactivar/Eliminar Organización**

---

## 1. Crear Organización (Tenant Onboarding)

*   **Nombre de la funcionalidad:** Registro de Clínica o Laboratorio (Tenant Onboarding).
*   **Objetivo del proceso:** Dar de alta a un nuevo cliente corporativo en el sistema, generando su identificador único (`TenantId`) que se usará para aislar sus datos (RLS).
*   **Dominio o módulo al que pertenece:** Organization / Core / Identity.
*   **Método HTTP y ruta del endpoint:** `POST /api/v1/organizations`
*   **Descripción de la operación:** Recibe los datos comerciales de la clínica, la registra en la base de datos y devuelve el `TenantId` asignado.
*   **Headers requeridos:**
    *   `Content-Type`: application/json
    *   `Authorization`: Bearer token (Opcional en registro público, requerido si es desde un backoffice de administración).
*   **Parámetros de ruta (Path Parameters):** Ninguno.
*   **Parámetros de consulta (Query Parameters):** Ninguno.
*   **Body completo del request con ejemplos:**
    ```json
    {
      "legalName": "Clínica San Juan S.A.",
      "tradeName": "Clínica San Juan",
      "documentTypeId": 1,
      "taxId": "0992384750001",
      "organizationTypeId": 1,
      "countryId": 1,
      "website": "https://clinicasanjuan.com",
      "logoUrl": "https://clinicasanjuan.com/logo.png"
    }
    ```
*   **Response esperada con ejemplos:**
    ```json
    {
      "organizationId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "legalName": "Clínica San Juan S.A.",
      "tradeName": "Clínica San Juan",
      "taxId": "0992384750001",
      "isActive": true,
      "createdAt": "2026-08-03T01:00:00Z"
    }
    ```
*   **Códigos HTTP posibles:**
    *   `201 Created`: Organización creada correctamente.
    *   `400 Bad Request`: Faltan campos obligatorios o formato inválido.
    *   `409 Conflict`: Ya existe una organización con ese `taxId`.
*   **Validaciones y reglas de negocio:**
    *   `legalName`, `organizationTypeId` y `countryId` son obligatorios.
    *   El `taxId` (RUC/NIT) debe ser estrictamente único en el sistema.
*   **Precondiciones necesarias para ejecutar el endpoint:**
    *   La base de datos PostgreSQL debe estar activa.
    *   Los catálogos `geographic.countries`, `geographic.document_types` y `organization.organization_types` deben tener datos.
*   **Dependencias con otros endpoints o procesos:** Depende de la carga de catálogos inicial.
*   **Tablas de la base de datos involucradas:** `organization.organizations`, `geographic.countries`, `organization.organization_types`.
*   **Operaciones realizadas sobre cada tabla:**
    *   `SELECT` en `organizations` (Para validar si el `tax_id` ya existe).
    *   `SELECT` en tablas referenciales (`countries`, `organization_types`) para validar Foreign Keys.
    *   `INSERT` en `organizations` (Con el estado por defecto `is_active = true`).
*   **Transacciones ejecutadas y su impacto sobre la información:**
    *   Se abre una transacción simple para insertar la organización. Si la inserción tiene éxito, se hace commit.
*   **Eventos, colas o procesos asíncronos desencadenados:**
    *   Publicación de `OrganizationCreatedEvent` (para notificaciones por email de bienvenida o aprovisionamiento de configuración por defecto).
*   **Integraciones con servicios externos:** Ninguna en esta fase inicial.
*   **Posibles errores, escenarios de fallo y comportamiento esperado en cada caso:**
    *   *Timeout BD*: Si la base de datos no responde, retorna `500 Internal Server Error`.
    *   *Conflicto de concurrencia*: Si dos peticiones intentan registrar el mismo RUC al mismo tiempo, el Unique Constraint de la BD arrojará error, la capa de aplicación lo capturará y devolverá `409 Conflict`.

---

## 2. Obtener Organización por ID

*   **Nombre de la funcionalidad:** Consulta de Datos de la Organización.
*   **Objetivo del proceso:** Recuperar la información detallada de una clínica específica.
*   **Dominio o módulo al que pertenece:** Organization.
*   **Método HTTP y ruta del endpoint:** `GET /api/v1/organizations/{id}`
*   **Descripción de la operación:** Consulta y devuelve los datos de la organización.
*   **Headers requeridos:**
    *   `Authorization`: Bearer token
    *   `x-tenant-id`: UUID (Opcional, pero buena práctica si el usuario consultando pertenece a ese tenant).
*   **Parámetros de ruta (Path Parameters):**
    *   `id` (UUID): El identificador único de la organización.
*   **Parámetros de consulta (Query Parameters):** Ninguno.
*   **Body completo del request con ejemplos:** N/A (GET request).
*   **Response esperada con ejemplos:**
    ```json
    {
      "organizationId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "legalName": "Clínica San Juan S.A.",
      "tradeName": "Clínica San Juan",
      "documentTypeId": 1,
      "taxId": "0992384750001",
      "organizationTypeId": 1,
      "countryId": 1,
      "website": "https://clinicasanjuan.com",
      "logoUrl": "https://clinicasanjuan.com/logo.png",
      "isActive": true,
      "createdAt": "2026-08-03T01:00:00Z",
      "updatedAt": null
    }
    ```
*   **Códigos HTTP posibles:**
    *   `200 OK`: Datos recuperados exitosamente.
    *   `404 Not Found`: No existe la organización.
*   **Validaciones y reglas de negocio:**
    *   El ID debe ser un UUID válido.
*   **Precondiciones necesarias para ejecutar el endpoint:**
    *   La organización debe haber sido creada previamente (Paso 1).
*   **Dependencias con otros endpoints o procesos:** Depende de `POST /api/v1/organizations`.
*   **Tablas de la base de datos involucradas:** `organizations`.
*   **Operaciones realizadas sobre cada tabla:**
    *   `SELECT` en `organizations` filtrando por `id`.
*   **Transacciones ejecutadas y su impacto sobre la información:** Ninguna (solo lectura).
*   **Eventos, colas o procesos asíncronos desencadenados:** Ninguno.
*   **Integraciones con servicios externos:** Ninguna.
*   **Posibles errores, escenarios de fallo y comportamiento esperado en cada caso:**
    *   Si se provee un UUID mal formado, el framework retorna `400 Bad Request` por defecto.

---
*(Nota: Para no extender demasiado este archivo, puedes usar Postman para replicar los pasos 3 (Update) y 4 (Delete) siguiendo lógicas HTTP estándar `PUT` y `DELETE`).*
