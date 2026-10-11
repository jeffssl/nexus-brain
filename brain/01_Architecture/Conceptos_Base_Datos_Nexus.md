# Conceptos Clave: Arquitectura de Base de Datos Nexus

Acabamos de revisar el archivo maestro de nuestra base de datos (`Nexus.dbml`). Antes de tirar una sola línea de código, es vital entender el **"por qué"** detrás de las decisiones de diseño. Si entiendes esto, el código C# que escribiremos más adelante te parecerá lógico e intuitivo.

Aquí están los 4 pilares fundamentales de nuestra base de datos:

## 1. Multi-Tenancy (El Inquilino)
El sistema está diseñado para ser **SaaS (Software as a Service)**. Esto significa que múltiples clínicas u hospitales (Tenants) usarán la misma base de datos física, pero **nunca** deben ver los datos del otro.

*   **¿Cómo lo logramos?** Si te fijas en el DBML, casi todas las tablas (Locations, Practitioners, Patients, Slots) tienen una columna llamada `tenant_id` (que es una llave foránea hacia la tabla principal `organizations`).
*   **¿Por qué importa?** A nivel de PostgreSQL aplicaremos una regla llamada **RLS (Row-Level Security)**. Esto hará que, si un usuario de la "Clínica A" hace un `SELECT * FROM patient.patients`, la base de datos automáticamente intercepte la consulta y le agregue un `WHERE tenant_id = 'ID-CLINICA-A'`. Así es imposible filtrar datos por accidente en el código.

## 2. Alineación con HL7 FHIR (Estándar Médico)
El mundo de la salud tiene un estándar global para el intercambio de datos llamado FHIR. No inventamos los nombres de las tablas; usamos el estándar:
*   **Organization:** La clínica o grupo médico (El Tenant).
*   **Location:** La sucursal física o edificio (Ej. Sede Norte).
*   **Practitioner:** El profesional de la salud (El médico).
*   **Patient:** El paciente.
*   **Slot / Schedule:** Los espacios de tiempo disponibles en la agenda.
*   **Appointment:** La cita médica concreta.

*   **¿Por qué importa?** Cuando en el futuro necesitemos integrar nuestro software con un sistema gubernamental o un seguro médico internacional, nuestros datos ya "hablan su mismo idioma".

## 3. Seguridad y HIPAA (Separación de PII y PHI)
En HealthTech, la privacidad de los datos médicos es sagrada y está regulada (leyes como HIPAA en USA o LOPDP).
*   **PII (Personally Identifiable Information):** Datos demográficos (Nombres, Cédula, Dirección). Están en las tablas `patients`.
*   **PHI (Protected Health Information):** Datos clínicos sensibles (Diagnósticos, síntomas).
*   **¿Qué hicimos?** Fíjate en la tabla `appointment.appointment_clinical_details`. Hemos separado los datos puramente médicos de la tabla principal de citas.
*   **¿Por qué importa?** Porque a nivel de base de datos podemos encriptar la tabla de `clinical_details` y darle permisos solo a los médicos, mientras que la recepcionista solo tiene permiso para ver `appointments` (la hora de la cita) sin saber el motivo clínico de la consulta.

## 4. El Modelo de IDs (UUID v7 vs Identity)
En bases de datos hay un debate clásico: ¿Usar números autoincrementales (1, 2, 3) o UUIDs (cadenas alfanuméricas largas)? Nosotros usamos un **Modelo Híbrido**:

*   **UUID v7 para Entidades Principales:** Tablas como `organizations`, `practitioners` y `patients` usan `UUID v7`. 
    *   *Razón:* Es imposible adivinar el número (si un paciente tiene ID 5, alguien malicioso podría intentar buscar el ID 6). El UUIDv7, además de ser seguro, se genera basado en el tiempo, lo que significa que la base de datos no se pone lenta al indexarlos (a diferencia de los UUID v4 antiguos).
*   **BigInt Identity para Tablas de Soporte:** Tablas internas como `locations`, `specialties`, `rooms` usan números (`GENERATED ALWAYS AS IDENTITY`).
    *   *Razón:* Son datos internos, ocupan menos memoria RAM y hacen que las relaciones (JOINs) sean ultrarrápidas.
