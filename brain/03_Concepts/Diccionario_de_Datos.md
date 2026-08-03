---
title: Diccionario de Datos
tipo: arquitectura
estado: activo
fecha: 2026-07-31
tags:
  - database
  - schema
---

# Data Dictionary Completo: Core Nexus

Este documento contiene la especificación completa de todas las entidades, esquemas y tablas definidas en la base de datos `Nexus` (en su formato lógico DBML). Este diccionario es vital para la integración, el mantenimiento y la validación de cumplimiento de datos (HIPAA/GDPR/LOPDP).

## Resumen de Esquemas (Módulos)
1. **`system`**: Identidad, seguridad, catálogos base y logs de auditoría general (Event Sourcing).
2. **`geographic`**: División política estandarizada y tipos de documento.
3. **`organization`**: Prestadores de salud (Hospitales/Clínicas) como Tenants, sus sucursales físicas (Locations) y contactos.
4. **`facility`**: Recursos físicos y lógicos de una sucursal (Salas y Equipos).
5. **`practitioner`**: Personal de salud (médicos, tecnólogos), licencias y especializaciones (incluye catálogos de especialidades).
6. **`patient`**: Datos demográficos del paciente, familiares (dependientes) y consentimientos legales.
7. **`insurance`**: Catálogo de aseguradoras, planes y pólizas de pacientes.
8. **`scheduling`**: Horarios recurrentes de atención, excepciones y bloques reservados (Slots).
9. **`appointment`**: Transacciones de citas (Check-in, consulta, telemedicina, historial y detalles clínicos PHI), incluyendo estados y cancelaciones.
10. **`pricing`**: Tarifarios, asignación de precios por servicio y cobertura, servicios base y monedas.
11. **`billing`**: Facturación, cobros, impuestos, reembolsos financieros y sus estados.
12. **`archive`**: Almacenamiento en frío para citas históricas (>2 años).

---

## 1. Esquema: `system`
Manejo de acceso, configuraciones globales y bitácora de operaciones CRUD. Incluye catálogos transversales genéricos.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `users` | Entidad central de autenticación y acceso al sistema. | `user_id` (UUID), `email`, `password_hash`, `user_type`. | Alto (PII / Sec) |
| `audit_logs` | Motor de *Event Sourcing*. Captura todos los cambios de todas las tablas. | `record_id`, `table_name`, `old_values` (JSONB), `new_values`. | Variable |
| `configurations` | Flags y variables de entorno del negocio. | `config_key`, `config_value`, `data_type`. | Bajo |
| `contact_types` | Tipos de contacto (Ej. EMAIL, PHONE, WHATSAPP). | `code`, `validation_regex`. | Ninguno |

---

## 2. Esquema: `geographic`
División política estructurada y documentos. Ayuda a evitar textos libres en direcciones e identificaciones.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `countries` | Catálogo de países basado en ISO 3166. | `country_id` (PK), `country_code`, `iso3_code`, `phone_code`. | Ninguno |
| `states` | Provincias, estados o departamentos de un país. | `state_id`, `country_id`, `code`. | Ninguno |
| `cities` | Ciudades, municipios o cantones. | `city_id`, `state_id`, `name`. | Ninguno |
| `document_types` | Catálogo de tipos de documento de identidad (RUC, Cédula, Pasaporte). | `code`, `validation_regex`. | Ninguno |

---

### Descripción de Tablas

#### Tabla `geographic.countries`
| Columna | Tipo | Nulo | Descripción |
| :--- | :--- | :--- | :--- |
| **`country_id`** | `INTEGER` | No | PK. ID autonumérico del país (ej. 1=EC). |
| `country_code` | `VARCHAR(2)` | No | Código ISO-2 (ej. EC). |
| `iso3code` | `VARCHAR(3)` | No | Código ISO-3 (ej. ECU). |
| `name` | `VARCHAR(100)` | No | Nombre oficial del país. |
| `phone_code` | `VARCHAR(20)` | Sí | Código telefónico (ej. +593). |
| `is_active` | `BOOLEAN` | No | Estado de activación. |

#### Tabla `geographic.states` (Estados / Provincias)
| Columna | Tipo | Nulo | Descripción |
| :--- | :--- | :--- | :--- |
| **`state_id`** | `INTEGER` | No | Identificador único del estado. |
| `country_id` | `INTEGER` | No | FK hacia `geographic.countries(country_id)`. |
| `code` | `VARCHAR(10)` | No | Código estatal (ej. PCH para Pichincha). |
| `name` | `VARCHAR(100)` | No | Nombre del estado o provincia. |
| `is_active` | `BOOLEAN` | No | Estado de activación. |

---

## 3. Esquema: `organization`
Información de las entidades prestadoras de salud (Tenants) y sus sedes o sucursales (Locations).

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `organization_types` | Tipos de organizaciones (Clínica, Laboratorio, Consultorio). | `code`, `name`. | Ninguno |
| `organizations` | Entidad jurídica base. **Representa físicamente el Tenant** del sistema. | `organization_id` (UUID v7 PK), `document_type_id` (FK), `tax_id` (RUC/NIT). | Bajo |
| `locations` | Sucursales o sedes físicas del Tenant. | `location_id` (PK), `organization_id` (FK), `tenant_id` (FK RLS), `timezone`. | Bajo |
| `location_addresses` | Direcciones físicas de las sedes con coordenadas GPS. | `address_id` (PK), `location_id` (FK), `latitude`, `longitude`. | Bajo |
| `location_contacts` | Teléfonos o canales de atención de cada sede. | `contact_id` (PK), `location_id` (FK), `contact_value`, `purpose`. | Bajo |
| `location_specialties`| Especialidades ofertadas activamente en cada sede. | `location_specialty_id` (PK), `location_id` (FK), `specialty_id`. | Ninguno |

---

## 4. Esquema: `facility`
Recursos físicos asignables dentro de una sucursal para operar una cita médica.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `rooms` | Consultorios, quirófanos, boxes de extracción, etc. | `room_id` (PK), `location_id` (FK), `room_type`, `capacity`. | Ninguno |
| `equipments` | Equipamiento médico que condiciona o restringe agendamientos. | `equipment_id` (PK), `location_id` (FK), `room_id` (FK), `serial_number`. | Ninguno |

---

## 5. Esquema: `practitioner`
Administración del personal médico o técnico, licencias, especialidades médicas y sedes donde atiende.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `practitioners` | Perfil principal del profesional de la salud. | `practitioner_id` (UUID PK), `tenant_id` (FK RLS), `document_type_id` (FK), `document_number`, `medical_license`. | Alto (PII) |
| `practitioner_contacts` | Contactos de uso profesional del médico. | `practitioner_id` (FK), `contact_value`, `purpose`. | Medio (PII) |
| `practitioner_specialties`| Credenciales de especialidad asignadas al profesional. | `practitioner_id` (FK), `specialty_id`, `certification_date`. | Bajo |
| `practitioner_locations` | Relación de sucursales en las que atiende el profesional. | `practitioner_id` (FK), `location_id` (FK), `start_date`. | Bajo |
| `specialty_categories` | Clasificación o agrupación de especialidades médicas (Ej. Medicina, Cirugía). | `code`, `name_es`, `name_en`. | Ninguno |
| `specialties` | Especialidades médicas codificadas bajo el estándar oficial SNOMED CT. | `snomed_code`, `category_id` (FK), `name_es`, `name_en`. | Ninguno |

---

## 6. Esquema: `patient`
Datos sensibles de pacientes, dependientes (tutores/familiares) e historial de firmas legales (consentimientos). **PHI / PII estricto**.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `patients` | Maestro de pacientes y demografía básica. | `patient_id` (UUID PK), `tenant_id` (FK RLS), `document_number`, `birth_date`. | Crítico (PII/PHI)|
| `patient_relations` | Manejo de dependientes (Hijos, adultos mayores, tutores, garantes). | `primary_patient_id` (FK), `dependent_patient_id` (FK), `relationship_type`. | Crítico (PHI) |
| `patient_contacts` | Modos de contactar directamente al paciente. | `patient_id` (FK), `contact_value`, `contact_type_id`. | Crítico (PII) |
| `patient_addresses` | Residencia física para telemedicina o facturación. | `patient_id` (FK), `city_id`, `address_line_1`. | Alto (PII) |
| `patient_consents` | Firmas de políticas de privacidad o consentimiento informado. | `patient_id` (FK), `consent_type`, `document_url`, `agreed_at`. | Crítico (PHI) |

---

## 7. Esquema: `insurance`
Mapeo de coberturas y redes de financiadores de servicios de salud.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `coverage_types` | Tipo de aseguramiento (Público, Privado). | `code`, `name`. | Ninguno |
| `payers` | Aseguradoras corporativas o entes públicos (IESS, etc.).| `payer_id`, `tax_id`. | Bajo |
| `plans` | Planes de salud específicos de las aseguradoras. | `plan_id` (PK), `payer_id` (FK), `coverage_percentage`.| Bajo |
| `payer_provider_networks`| Convenio de red prestadora (Clínicas en convenio con seguros).| `payer_id` (FK), `organization_id` (FK), `contract_code`. | Bajo |
| `patient_insurances`| La póliza o cobertura específica de un paciente. | `patient_id` (FK), `payer_id` (FK), `plan_id` (FK), `member_number`. | Alto (PHI/PII) |

---

## 8. Esquema: `scheduling`
Gestión de horarios de médicos (Schedules), excepciones (vacaciones) y reservas confirmadas (Slots).

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `weekdays` | Días fijos de la semana. | `weekday_id`, `code`. | Ninguno |
| `practitioner_schedules` | Disponibilidad laboral periódica de un profesional en una sede. | `schedule_id` (PK), `tenant_id` (FK RLS), `practitioner_id` (FK), `start_time`.| Bajo |
| `schedule_exceptions`| Excepciones puntuales a la regla recurrente (Vacaciones, paros). | `schedule_id` (FK), `practitioner_id` (FK), `exception_date`, `is_closed`. | Bajo |
| `slots` | Bloque temporal físico e inmutable reservado para una cita. | `slot_id` (PK), `tenant_id` (FK RLS), `practitioner_id` (FK), `slot_start_at`, `version`.| Medio |
| `waitlists` | Pacientes esperando turno si un doctor libera espacio. | `preferred_date_from`, `patient_id` (FK), `specialty_id` (FK). | Alto (PHI) |

---

## 9. Esquema: `appointment`
Control transaccional del día de la cita, videollamadas de telemedicina, historial clínico PHI y estados de reserva.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `appointments` | La cita médica. Conecta paciente, médico, sede y slot. | `appointment_id` (UUID PK), `tenant_id` (FK RLS), `patient_id` (FK), `practitioner_id` (FK). | Crítico (PHI) |
| `appointment_clinical_details`| Datos clínicos de la cita (Cifrado a nivel de aplicación). | `clinical_detail_id` (PK), `appointment_id` (FK), `patient_notes`, `internal_notes`. | Crítico (PHI) |
| `telehealth_sessions`| Sesiones virtuales y links de videollamadas. | `appointment_id` (FK), `meeting_url`, `host_token`, `guest_token`. | Alto (Sec/PHI) |
| `appointment_histories`| Logs técnicos de auditoría de estados de la cita. | `history_id` (PK), `appointment_id` (FK), `old_value` (JSONB), `new_value` (JSONB). | Medio |
| `appointment_statuses` | Ciclo de vida estricto de una cita. | `status_code` (PK), `is_final`, `allowed_transitions`. | Ninguno |
| `cancellation_reasons` | Motivos de cancelación de un agendamiento. | `code`, `requires_refund`. | Ninguno |

---

## 10. Esquema: `pricing`
Estrategias, motores de precios parametrizados y catálogo de servicios base.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `price_lists` | Definición general de listados de precios (VIP, Convenio). | `code`, `is_active`. | Bajo |
| `service_prices` | Precios configurados por servicio, especialidad, sede (location) o médico. | `service_price_id` (PK), `price`, `currency_code` (FK), `valid_from`. | Bajo |
| `services` | Tipos de atención ofertados por la red médica. | `code`, `default_duration_minutes`, `requires_pre_auth`. | Ninguno |
| `currencies` | Monedas transaccionales soportadas. | `currency_code` (PK), `decimal_places`. | Ninguno |

---

## 11. Esquema: `billing`
Componentes financieros que resultan tras una cita médica, formas de pago y devoluciones.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `invoices` | Documentos fiscales de cobro a pacientes o aseguradoras. | `invoice_id` (UUID PK), `tenant_id` (FK RLS), `invoice_number`, `total`. | Alto (Finanzas)|
| `invoice_line_items`| Detalle granular de servicios cobrados. | `invoice_id` (FK), `service_id` (FK), `subtotal`. | Medio (Finanzas)|
| `payments` | Transacciones de dinero aplicadas a las facturas. | `payment_id` (UUID PK), `tenant_id` (FK RLS), `invoice_id` (FK), `amount`. | Alto (Finanzas)|
| `refunds` | Devoluciones de dinero por citas canceladas. | `refund_id` (PK), `payment_id` (FK), `amount`, `status_code` (FK). | Medio (Finanzas)|
| `payment_statuses` | Ciclo de vida de una transacción monetaria. | `status_code` (PK), `is_final`. | Ninguno |
| `payment_methods` | Formas de pago aceptadas. | `code`, `requires_gateway`. | Ninguno |
| `refund_statuses` | Estados para devoluciones de pagos. | `status_code` (PK), `is_final`. | Ninguno |

---

## 12. Esquema: `archive`
Almacenamiento histórico inmutable.

| Tabla | Descripción | Columnas Clave | Sensibilidad |
|---|---|---|---|
| `appointment_archives`| Resguardo completo de citas en formato JSONB para cumplimiento legal. | `appointment_id` (PK), `archived_at`, `original_data`. | Crítico (PHI) |
