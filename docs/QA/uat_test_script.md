# Guion de Pruebas UAT: Flujo Principal (Paciente Privado)

Este documento detalla el paso a paso funcional para validar la plataforma *Nexus* desde el inicio (onboarding) hasta el cobro de una cita de un paciente particular (sin seguro médico).

## Precondiciones
1. Base de datos operativa (`NexusDbContext` migrado).
2. Endpoints ejecutándose en el entorno de pruebas local/staging.
3. Se recomienda el uso de Postman para gestionar y encadenar las variables (IDs generados).

---

## FASE 1: Onboarding de la Clínica (Tenant) y Sedes

**Objetivo:** Crear el tenant base, su primera sucursal (sede) y los recursos físicos.

1. **Crear Organización (Tenant)**
   - **Endpoint:** `POST /api/organizations`
   - **Data:** RUC, Nombre Comercial.
   - **Validación:** Retorna `organization_id`. Guardar como variable `{{organization_id}}`.

2. **Crear Sede (Location)**
   - **Endpoint:** `POST /api/locations`
   - **Data:** `organization_id`, Zona Horaria, Nombre de Sucursal.
   - **Validación:** Retorna `location_id`. Guardar como variable `{{location_id}}`.

3. **Crear Consultorio (Room)**
   - **Endpoint:** `POST /api/facilities` (con query `locationId`)
   - **Data:** Tipo de Cuarto, Capacidad.
   - **Validación:** Retorna HTTP 201.

---

## FASE 2: Configuración del Médico y Disponibilidad

**Objetivo:** Contratar a un médico, asignarle una sede y generar los horarios (slots) disponibles.

1. **Crear Médico (Practitioner)**
   - **Endpoint:** `POST /api/practitioners`
   - **Data:** Documento de Identidad, Licencia Médica, Nombres.
   - **Validación:** Retorna `practitioner_id`. Guardar como variable `{{practitioner_id}}`.

2. **Crear Regla de Horario (Schedule)**
   - **Endpoint:** `POST /api/schedules`
   - **Data:** `practitioner_id`, `location_id`, fecha de inicio, fecha de fin, patrón (Ej. Lunes a Viernes 08:00 a 14:00).
   - **Validación:** Retorna HTTP 201.

3. **Generar Slots (Bloques de Tiempo)**
   - **Endpoint:** `POST /api/schedules/generate-slots`
   - **Data:** Rango de fechas a generar, `practitioner_id`.
   - **Validación:** Retorna HTTP 200.
   - **Paso Extra:** Verificar en DB o vía endpoint GET que la tabla `slots` ahora tiene bloques de 30 mins (o según configuración) para ese médico. Escoger un `slot_id` y guardarlo como variable.

---

## FASE 3: Registro del Paciente

**Objetivo:** Ingresar al sistema los datos de un paciente nuevo.

1. **Crear Paciente**
   - **Endpoint:** `POST /api/patients`
   - **Data:** Nombres, Apellidos, Fecha de Nacimiento, Documento de Identidad.
   - **Validación:** Retorna `patient_id`. Guardar como variable `{{patient_id}}`.

---

## FASE 4: Agendamiento y Cita (Core)

**Objetivo:** Reservar el slot previamente generado a nombre del paciente registrado.

1. **Reservar la Cita (Appointment)**
   - **Endpoint:** `POST /api/bookings/appointments`
   - **Data:** `patient_id`, `practitioner_id`, `slot_id`.
   - **Validación:** Retorna `appointment_id`. La tabla `slots` debe cambiar de estado para que no pueda ser doblemente reservada. Guardar `{{appointment_id}}`.

2. **Crear Nota Clínica (Historia)**
   - **Endpoint:** `POST /api/bookings/appointments/{id}/clinical-notes`
   - **Data:** `appointment_id`, Notas Médicas ("Paciente presenta dolor de cabeza, se prescribe paracetamol").
   - **Validación:** Retorna HTTP 201.

---

## FASE 5: Facturación (Billing)

**Objetivo:** Generar el cobro por los servicios prestados y registrar el pago.

1. **Generar Factura (Invoice)**
   - **Endpoint:** `POST /api/billing/invoices`
   - **Data:** `appointment_id`, `patient_id`. (Actualmente puede usar un precio hardcoded o default ya que falta el módulo de Pricing).
   - **Validación:** Retorna `invoice_id`. Guardar como variable `{{invoice_id}}`.

2. **Pagar Factura**
   - **Endpoint:** `POST /api/billing/invoices/{id}/pay`
   - **Data:** `invoice_id`, monto, método de pago.
   - **Validación:** Retorna HTTP 200. La factura debe quedar en estado PAGADO.
