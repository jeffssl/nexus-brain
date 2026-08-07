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
2. **Listar Organizaciones**
3. **Obtener Organización por ID**
4. **Actualizar Datos de la Organización**
5. **Desactivar/Eliminar Organización**

---

## 1. Crear Organización (Tenant Onboarding)

*   **Nombre de la funcionalidad:** Registro de Clínica o Laboratorio (Tenant Onboarding).
*   **Objetivo del proceso:** Dar de alta a un nuevo cliente corporativo en el sistema, generando su identificador único (`TenantId`) que se usará para aislar sus datos (RLS).
*   **Dominio o módulo al que pertenece:** Organization / Core / Identity.
*   **Método HTTP y ruta del endpoint:** `POST /api/v1/organizations`
*   **Descripción de la operación:** Recibe los datos comerciales de la clínica, la registra en la base de datos y devuelve el `organization_id` (UUID v7) generado.
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
      "id": "019fd9fe-89f0-7f99-82b1-111302661e39"
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

## 2. Listar Organizaciones

*   **Nombre de la funcionalidad:** Consulta del Listado de Organizaciones.
*   **Objetivo del proceso:** Recuperar la información de las diferentes clínicas o tenants, con paginación y filtros.
*   **Dominio o módulo al que pertenece:** Organization.
*   **Método HTTP y ruta del endpoint:** `GET /api/v1/organizations`
*   **Descripción de la operación:** Consulta y devuelve el listado de organizaciones basado en los filtros proporcionados.
*   **Headers requeridos:**
    *   `Authorization`: Bearer token
*   **Parámetros de ruta (Path Parameters):** Ninguno.
*   **Parámetros de consulta (Query Parameters):**
    *   `page`: Número de página (ej. 1)
    *   `pageSize`: Tamaño de la página (ej. 10)
    *   `isActive`: Filtro por estado booleano.
    *   `search`: Búsqueda por `tradeName` o `legalName`.
*   **Body completo del request con ejemplos:** N/A (GET request).
*   **Response esperada con ejemplos:**
    ```json
    {
      "items": [
        {
          "id": "019fd9fe-89f0-7f99-82b1-111302661e39",
          "legalName": "Clínica San Juan S.A.",
          "tradeName": "Clínica San Juan",
          "taxId": "0992384750001",
          "isActive": true
        }
      ],
      "totalCount": 1,
      "page": 1,
      "pageSize": 10
    }
    ```
*   **Códigos HTTP posibles:**
    *   `200 OK`: Listado recuperado exitosamente.
*   **Validaciones y reglas de negocio:** Paginación válida requerida.
*   **Precondiciones necesarias para ejecutar el endpoint:** Haber creado organizaciones (Paso 1).
*   **Tablas de la base de datos involucradas:** `organization.organizations`.
*   **Operaciones realizadas sobre cada tabla:** `SELECT` con `LIMIT` y `OFFSET`.

---

## 3. Obtener Organización por ID

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
      "id": "019fd9fe-89f0-7f99-82b1-111302661e39",
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
      "updatedAt": null,
      "deletedAt": null
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
*   **Tablas de la base de datos involucradas:** `organization.organizations`.
*   **Operaciones realizadas sobre cada tabla:**
    *   `SELECT` en `organizations` filtrando por `organization_id`.
*   **Transacciones ejecutadas y su impacto sobre la información:** Ninguna (solo lectura).

---

## 4. Actualizar Datos de la Organización

*   **Nombre de la funcionalidad:** Actualización de Datos de la Organización.
*   **Objetivo del proceso:** Modificar información de una clínica existente.
*   **Dominio o módulo al que pertenece:** Organization.
*   **Método HTTP y ruta del endpoint:** `PUT /api/v1/organizations/{id}`
*   **Descripción de la operación:** Reemplaza los datos actuales de la organización.
*   **Headers requeridos:**
    *   `Authorization`: Bearer token
    *   `Content-Type`: application/json
*   **Parámetros de ruta (Path Parameters):**
    *   `id` (UUID): El identificador único de la organización.
*   **Parámetros de consulta (Query Parameters):** Ninguno.
*   **Body completo del request con ejemplos:**
    ```json
    {
      "legalName": "Clínica San Juan S.A. Editada",
      "tradeName": "Clínica San Juan Especialidades",
      "documentTypeId": 1,
      "taxId": "0992384750001",
      "organizationTypeId": 2,
      "countryId": 1,
      "website": "https://clinicasanjuan.com.ec",
      "logoUrl": "https://clinicasanjuan.com.ec/logo-new.png"
    }
    ```
*   **Response esperada con ejemplos:**
    No Content o devolver objeto.
    ```json
    { }
    ```
*   **Códigos HTTP posibles:**
    *   `204 No Content` / `200 OK`: Actualización exitosa.
    *   `404 Not Found`: No existe la organización.
    *   `400 Bad Request`: Formato de datos incorrecto.
*   **Validaciones y reglas de negocio:**
    *   `legalName`, `organizationTypeId` y `countryId` no pueden ser nulos si se hace actualización completa.
*   **Precondiciones necesarias para ejecutar el endpoint:** La organización debe existir.
*   **Tablas de la base de datos involucradas:** `organization.organizations`.
*   **Operaciones realizadas sobre cada tabla:**
    *   `UPDATE` en `organizations` seteando `updated_at = now()`.

---

## 5. Desactivar/Eliminar Organización

*   **Nombre de la funcionalidad:** Eliminación Lógica de Organización (Soft Delete).
*   **Objetivo del proceso:** Inactivar un Tenant (clínica o laboratorio).
*   **Dominio o módulo al que pertenece:** Organization.
*   **Método HTTP y ruta del endpoint:** `DELETE /api/v1/organizations/{id}`
*   **Descripción de la operación:** Marca a la organización como eliminada llenando el campo `deleted_at`.
*   **Headers requeridos:**
    *   `Authorization`: Bearer token
*   **Parámetros de ruta (Path Parameters):**
    *   `id` (UUID): El identificador único de la organización.
*   **Parámetros de consulta (Query Parameters):** Ninguno.
*   **Body completo del request con ejemplos:** N/A
*   **Response esperada con ejemplos:**
    ```json
    { }
    ```
*   **Códigos HTTP posibles:**
    *   `204 No Content` / `200 OK`: Eliminación exitosa.
    *   `404 Not Found`: No existe la organización.
*   **Validaciones y reglas de negocio:**
    *   Se debe aplicar **Soft Delete** (`is_active = false` o `deleted_at = now()`).
*   **Tablas de la base de datos involucradas:** `organization.organizations`.
*   **Operaciones realizadas sobre cada tabla:**
    *   `UPDATE` en `organizations` seteando `deleted_at = now()` y/o `is_active = false`.
