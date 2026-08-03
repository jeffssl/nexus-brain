---
title: Reglas de Negocio Centrales (Nexus)
tags:
  - product
  - business-rules
  - nexus
status: active
---

# Reglas de Negocio Centrales (Nexus)

Este documento recopila las reglas inquebrantables del dominio (Business Invariants) del sistema Nexus.

1. **Aislamiento Multi-Tenant**: Ningún usuario de la Clínica A puede ver o alterar datos de la Clínica B. Esto se garantiza a nivel de base de datos (RLS) y aplicación.
2. **Prevención de Sobre-reservas (Double-Booking)**: Un Slot de tiempo específico solo puede ser reservado una vez. Si dos pacientes intentan reservar el mismo Slot al milisegundo, la base de datos debe rechazar la segunda petición mediante concurrencia optimista.
3. **Privacidad de Datos (HIPAA/LOPDP)**: Los detalles médicos o síntomas de una cita no pueden mezclarse con los datos demográficos básicos del usuario.
4. **Validación de Citas**: Una cita no puede crearse en el pasado ni con una duración menor a 15 minutos.
