---
title: Roadmap de Producto
tipo: planificacion
estado: borrador
fecha: 2026-07-24
tags:
  - roadmap
  - sprints
  - user-stories
---

# Roadmap de Producto: Sistema de Agendamiento Nexus

Este documento organiza todo el ciclo funcional del sistema de agendamiento "Nexus", desglosando la lógica de negocio en **Épicas** e **Historias de Usuario (US)**. Finalmente, se propone un plan estructurado en **Sprints** priorizados por valor de negocio y dependencias técnicas.

---

## 🏗️ 1. Definición de Épicas e Historias de Usuario

### Épica 1: Foundation & Multi-Tenancy (Administración Base)
**Objetivo:** Permitir que clínicas y laboratorios (Tenants) puedan registrarse, configurar sus sucursales, recursos y catálogos globales.

- **US-1.1:** Como SuperAdmin, quiero **registrar una nueva Organización (Tenant)** para que puedan usar el sistema.
  - *Validaciones:* Tax ID único, nombre legal requerido.
- **US-1.2:** Como Admin de Clínica, quiero **crear Sucursales (Locations)** para separar la operación física de mi red.
  - *Dependencias:* Organización creada.
- **US-1.3:** Como Admin de Clínica, quiero **gestionar los Recursos y Salas (Facilities)** de mis sucursales para asignar citas a espacios físicos.
- **US-1.4:** Como SuperAdmin, quiero **gestionar Catálogos (Especialidades, Servicios, Países)** para estandarizar la información en todo el sistema.

### Épica 2: Gestión de Personal Médico (Practitioners & Schedules)
**Objetivo:** Registrar profesionales, sus especialidades y definir sus horarios de disponibilidad laboral.

- **US-2.1:** Como Admin de Clínica, quiero **registrar Profesionales de la Salud** (Médicos/Tecnólogos) con su licencia médica y especialidades.
  - *Validaciones:* Documento de identidad y licencia médica únicos por Tenant.
- **US-2.2:** Como Profesional, quiero **definir mi Horario de Atención Recurrente (Schedules)** por cada sucursal donde trabajo, para que el sistema genere automáticamente los espacios (Slots) disponibles.
  - *Reglas:* Validar cruce de horarios entre sucursales.
- **US-2.3:** Como Profesional, quiero **registrar Excepciones de Horario (Vacaciones/Permisos)** para cerrar agenda en fechas específicas.

### Épica 3: Portal del Paciente (Pacientes & Seguros)
**Objetivo:** Registrar pacientes, tutores legales, aseguradoras y recopilar firmas legales (HIPAA/LOPDP).

- **US-3.1:** Como Paciente, quiero **registrar mis datos personales** en el portal para poder agendar una cita.
- **US-3.2:** Como Paciente, quiero **agregar Dependientes (Hijos o Adultos Mayores)** a mi cuenta para agendar citas a su nombre.
  - *Reglas:* Relación vinculante (Tutor-Dependiente).
- **US-3.3:** Como Paciente, quiero **firmar electrónicamente los Consentimientos de Privacidad (PHI)** para cumplir con las regulaciones de salud.
- **US-3.4:** Como Paciente o Admin, quiero **asociar Pólizas de Seguro (Insurances)** a mi perfil para que apliquen coberturas al momento de pagar.

### Épica 4: Motor de Agendamiento (Core Scheduling)
**Objetivo:** Buscar espacios, reservar citas, telemedicina y listas de espera.

- **US-4.1:** Como Paciente, quiero **buscar Citas Disponibles** por Especialidad, Sucursal o Médico en una fecha específica, para ver los horarios libres (Slots).
- **US-4.2:** Como Paciente, quiero **Agendar una Cita** seleccionando un horario libre, y que el Slot quede bloqueado temporalmente hasta confirmar el proceso.
  - *Flujo Alternativo:* Si el paciente tarda > 10 mins, el Slot se libera (Held Until).
- **US-4.3:** Como Paciente, quiero **gestionar mi Cita (Cancelar / Reprogramar)** siguiendo las reglas de negocio de la clínica.
  - *Excepciones:* No cancelar si falta menos de 24h sin penalidad.
- **US-4.4:** Como Recepcionista, quiero **marcar el "Check-In" / "No-Show"** del paciente para actualizar el estado de la cita el día del evento.
- **US-4.5:** Como Paciente, quiero **unirme a la Lista de Espera (Waitlist)** de un médico lleno, para que me notifiquen si se libera un espacio.

### Épica 5: Atención Clínica y Telemedicina
**Objetivo:** Registrar la atención del médico y proveer canales virtuales.

- **US-5.1:** Como Profesional, quiero **registrar Notas Clínicas (Detalles)** durante la cita para mantener el historial (PHI) del paciente.
- **US-5.2:** Como Paciente/Profesional, quiero **generar y entrar a un enlace de Videollamada seguro (Telehealth)** para las consultas virtuales.
- **US-5.3:** Como Sistema, quiero **archivar de forma inmutable las citas antiguas (Archive)** por normativas de auditoría.

### Épica 6: Facturación y Pagos (Billing)
**Objetivo:** Configurar tarifas, generar facturas y procesar transacciones.

- **US-6.1:** Como Admin Financiero, quiero **crear Listas de Precios** por Servicio, Especialidad o Seguro.
- **US-6.2:** Como Sistema, quiero **generar la Factura (Invoice)** automáticamente cuando el paciente confirma una cita paga.
- **US-6.3:** Como Paciente, quiero **pagar mi factura online** para asegurar mi turno.
- **US-6.4:** Como Admin Financiero, quiero **procesar Devoluciones (Refunds)** automáticas cuando una cita pagada es cancelada bajo las políticas permitidas.

---

## 🗓️ 2. Planificación de Sprints (Roadmap)

Cada Sprint tiene una duración ideal de 2 semanas, agrupando historias por dependencias técnicas.

### Sprint 1: Foundation & Base Domain (✅ COMPLETADO)
**Foco:** Creación del Core de datos, Catálogos y Tenants (Estructura Base).
- **US-1.1** Registro de Organizaciones (Tenants).
- **US-1.2** Registro de Sucursales (Locations).
- **US-1.4** Catálogos (Especialidades, Servicios, Países).
- *Entregable:* Las clínicas pueden ser dadas de alta en el sistema con sus catálogos base.

### Sprint 2: Medical Staff & Onboarding de Pacientes (✅ COMPLETADO)
**Foco:** Registro de los actores principales (Médicos y Pacientes).
- **US-2.1** Registro de Profesionales y asignación de Especialidades.
- **US-1.3** Gestión de Salas y Consultorios (Facilities).
- **US-3.1** Registro de Pacientes.
- **US-3.2** Gestión de Dependientes Familiares.
- *Entregable:* Médicos y pacientes existen en la plataforma; se puede establecer qué médico atiende en qué sucursal.

### Sprint 3: The Scheduling Engine & Identity (✅ COMPLETADO)
**Foco:** Lógica compleja de horarios, disponibilidad de turnos y autenticación.
- **US-Security (IAM):** Implementación de Login y generación de Token JWT para acceso seguro a la API.
- **US-2.2** Configuración de Horarios (Schedules).
- **US-2.3** Excepciones y Vacaciones.
- *Backend Chore:* Job automático que genera `Slots` (turnos físicos de 30 mins) a partir de los `Schedules` y `Exceptions`.
- *Entregable:* Los usuarios pueden iniciar sesión con seguridad. El sistema ya sabe exactamente en qué bloques de tiempo un médico está disponible.

### Sprint 4: Bookings (✅ COMPLETADO)
**Foco:** Búsqueda, reserva y ciclo de vida de la Cita.
- **US-4.1** Búsqueda de disponibilidad de Citas.
- **US-4.2** Reserva (Booking) de la Cita (cambio de estado de Slot a Reserved).
- **US-4.3** Reprogramaciones y Cancelaciones.
- **US-4.4** Check-In / No-Show manual en recepción.
- *Entregable:* Flujo principal operativo. Un paciente ya puede agendar y asistir a la clínica.

### Sprint 5: Telehealth & Clinical Notes (✅ COMPLETADO)
**Foco:** Atención virtual y resguardo del historial clínico.
- **US-5.2** Generación de Links para Telemedicina.
- **US-5.1** Notas clínicas y confidencialidad.
- **US-3.3** Firmas de Privacidad y Consentimientos (PHI).
- *Entregable:* Capacidad para atender consultas online y guardar el reporte médico con seguridad HIPAA.

### Sprint 6: Reporting & Analytics (✅ COMPLETADO)
**Foco:** Extracción de valor y métricas de rendimiento.
- **US-6.1** Estadísticas Diarias (Agendadas, Canceladas, Completadas).
- **US-6.2** Utilización de Médicos (Horas de oferta vs demanda).
- *Entregable:* Inteligencia de negocio básica operativa para la clínica.

### Sprint 7: Billing & Pricing (Finanzas) (✅ COMPLETADO)
**Foco:** Monetización y gestión de seguros médicos.
- **US-7.1** Listas de Precios y Motor de Cotización (Pricing).
- **US-7.2** Seguros (Insurances) y Coberturas.
- **US-7.3** Generación de Facturas (Invoices).
- **US-7.4** Pagos y Devoluciones.
- *Entregable:* Sistema completo End-to-End monetizado.

### Sprint 8: Optimizaciones y Waitlists (✅ COMPLETADO)
**Foco:** Funciones avanzadas y automatizaciones.
- **US-8.1** Listas de Espera (Waitlists).
- **US-8.2** Archivo de datos inmutables (Cold Storage).
- **Infraestructura:** Notificaciones por Email/WhatsApp.
- *Entregable:* Refinamiento y experiencia de usuario premium.
