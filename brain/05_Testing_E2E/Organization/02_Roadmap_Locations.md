---
title: "Roadmap E2E: Módulo Locations (Sedes)"
tags:
  - testing
  - e2e
  - roadmap
  - organization
  - locations
  - postman
status: draft
---

# Roadmap Funcional End-to-End: Módulo Locations

Este documento define el flujo completo de extremo a extremo para las **Sedes (Locations)** pertenecientes a una organización. 
Las sedes están relacionadas directamente a un Tenant (`organization_id`).

## Flujo de Ejecución

1. **Crear Sede (Location)**
2. **Listar Sedes de una Organización**
3. **Obtener Sede por ID**
4. **Actualizar Datos de una Sede**
5. **Desactivar/Eliminar Sede**

---

## 1. Crear Sede (Location)

*   **Nombre de la funcionalidad:** Registro de una Sede o Sucursal.
*   **Objetivo del proceso:** Dar de alta una sede física perteneciente a una clínica o laboratorio (Organización).
*   **Dominio o módulo al que pertenece:** Organization.
*   **Método HTTP y ruta del endpoint:** `POST /api/v1/locations`
*   **Descripción de la operación:** Crea una sede, incluyendo su dirección principal, contactos y especialidades habilitadas.
*   **Headers requeridos:**
    *   `Content-Type`: application/json
    *   `Authorization`: Bearer token
    *   `x-tenant-id`: UUID de la organización
*   **Parámetros de ruta (Path Parameters):** Ninguno.
*   **Parámetros de consulta (Query Parameters):** Ninguno.
*   **Body completo del request con ejemplos:**
    ```json
    {
      "organizationId": "019fd9fe-89f0-7f99-82b1-111302661e39",
      "name": "Sede Norte Kennedy",
      "code": "SED-NOR-01",
      "isHeadOffice": false,
      "timezone": "America/Guayaquil",
      "address": {
        "cityId": 1,
        "addressLine1": "Av. Juan Tanca Marengo",
        "addressLine2": "Edificio Médico Kennedy, Piso 3",
        "neighborhood": "Kennedy Norte",
        "postalCode": "090501",
        "reference": "Frente al centro comercial",
        "latitude": -2.155462,
        "longitude": -79.897361
      },
      "contacts": [
        {
          "contactTypeId": 1,
          "contactValue": "+593987654321",
          "purpose": "Atención al cliente",
          "isPrimary": true
        }
      ],
      "specialtyIds": [1, 2, 5]
    }
    ```
*   **Response esperada con ejemplos:**
    ```json
    {
      "id": 1
    }
    ```
*   **Códigos HTTP posibles:**
    *   `201 Created`: Sede creada correctamente.
    *   `400 Bad Request`: Faltan campos obligatorios.
    *   `403 Forbidden`: No tiene permisos sobre el Tenant especificado.
    *   `409 Conflict`: Ya existe una sede con el mismo `code` en este Tenant.
*   **Validaciones y reglas de negocio:**
    *   `organizationId`, `name`, y `timezone` son requeridos.
    *   La dirección `address` requiere `cityId`, `addressLine1`, `latitude` y `longitude`.
    *   El código de la sede (`code`) debe ser único por Organización (`organization_id`).
*   **Precondiciones necesarias para ejecutar el endpoint:**
    *   La organización padre debe existir.
    *   Los catálogos (Cities, ContactTypes, Specialties) deben existir.
*   **Tablas de la base de datos involucradas:** 
    *   `organization.locations`
    *   `organization.location_addresses`
    *   `organization.location_contacts`
    *   `organization.location_specialties`
*   **Operaciones realizadas sobre cada tabla:**
    *   `INSERT` en `locations`.
    *   `INSERT` en `location_addresses`.
    *   `INSERT` en `location_contacts` (múltiples registros).
    *   `INSERT` en `location_specialties` (múltiples registros).
*   **Transacciones ejecutadas:** Debe envolver todos los INSERTS en una única transacción de Base de Datos.

---

## 2. Listar Sedes de una Organización

*   **Nombre de la funcionalidad:** Listado de Sedes.
*   **Objetivo del proceso:** Recuperar todas las sedes configuradas para una organización.
*   **Dominio o módulo al que pertenece:** Organization.
*   **Método HTTP y ruta del endpoint:** `GET /api/v1/locations`
*   **Descripción de la operación:** Consulta y devuelve el listado de sedes con base en el Tenant actual.
*   **Headers requeridos:**
    *   `Authorization`: Bearer token
    *   `x-tenant-id`: UUID
*   **Parámetros de consulta (Query Parameters):**
    *   `organizationId`: (Opcional si se toma del header, requerido si se solicita desde admin general).
    *   `page`: Número de página (ej. 1)
    *   `pageSize`: Tamaño de la página (ej. 10)
    *   `isActive`: boolean
    *   `search`: Búsqueda por `name` o `code`.
*   **Response esperada con ejemplos:**
    ```json
    {
      "items": [
        {
          "id": 1,
          "name": "Sede Norte Kennedy",
          "code": "SED-NOR-01",
          "isHeadOffice": false,
          "isActive": true
        }
      ],
      "totalCount": 1,
      "page": 1,
      "pageSize": 10
    }
    ```
*   **Códigos HTTP posibles:** `200 OK`.
*   **Operaciones SQL:** `SELECT` en `locations` filtrando por `tenant_id` (o `organization_id`).

---

## 3. Obtener Sede por ID

*   **Nombre de la funcionalidad:** Detalle de Sede.
*   **Método HTTP y ruta del endpoint:** `GET /api/v1/locations/{id}`
*   **Descripción de la operación:** Retorna la información completa de una sede, incluyendo su dirección primaria y lista de contactos y especialidades.
*   **Headers requeridos:**
    *   `Authorization`: Bearer token
    *   `x-tenant-id`: UUID
*   **Parámetros de ruta (Path Parameters):**
    *   `id` (Integer): Identificador de la sede.
*   **Response esperada con ejemplos:**
    ```json
    {
      "id": 1,
      "organizationId": "019fd9fe-89f0-7f99-82b1-111302661e39",
      "name": "Sede Norte Kennedy",
      "code": "SED-NOR-01",
      "isHeadOffice": false,
      "timezone": "America/Guayaquil",
      "isActive": true,
      "address": {
        "id": 1,
        "cityId": 1,
        "addressLine1": "Av. Juan Tanca Marengo",
        "addressLine2": "Edificio Médico Kennedy, Piso 3",
        "neighborhood": "Kennedy Norte",
        "postalCode": "090501",
        "reference": "Frente al centro comercial",
        "latitude": -2.155462,
        "longitude": -79.897361
      },
      "contacts": [
        {
          "id": 1,
          "contactTypeId": 1,
          "contactValue": "+593987654321",
          "purpose": "Atención al cliente",
          "isPrimary": true
        }
      ],
      "specialties": [
        {
          "specialtyId": 1
        }
      ]
    }
    ```
*   **Tablas de la base de datos involucradas:** `locations` LEFT JOIN `location_addresses`, `location_contacts`, `location_specialties`.

---

## 4. Actualizar Datos de una Sede

*   **Nombre de la funcionalidad:** Actualización de Sede.
*   **Método HTTP y ruta del endpoint:** `PUT /api/v1/locations/{id}`
*   **Descripción de la operación:** Reemplaza los datos actuales de la sede.
*   **Parámetros de ruta:** `id` (Integer).
*   **Body completo del request:** Similar al POST. Se debe manejar el reemplazo de contactos y especialidades cuidadosamente (borrar los antiguos y crear nuevos, o manejarlos mediante diff).
*   **Response esperada:** HTTP `204 No Content` o `200 OK`.
*   **Tablas de BD:** `UPDATE` en `locations`, `location_addresses`. `DELETE/INSERT` en `location_contacts` y `location_specialties`.

---

## 5. Desactivar/Eliminar Sede

*   **Nombre de la funcionalidad:** Eliminación Lógica de Sede.
*   **Método HTTP y ruta del endpoint:** `DELETE /api/v1/locations/{id}`
*   **Descripción de la operación:** Soft delete de la sede para que ya no esté disponible.
*   **Headers:** `x-tenant-id`.
*   **Response esperada:** HTTP `204 No Content` o `200 OK`.
*   **Operaciones SQL:** `UPDATE` en `locations` seteando `deleted_at = now()` y `is_active = false`.
