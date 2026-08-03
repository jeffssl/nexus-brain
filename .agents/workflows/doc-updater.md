---
name: doc-updater
description: Ejecuta el workflow de actualización de documentación y sincronización del cerebro (diario del agente) cuando el usuario escribe "/doc" o pide guardar el contexto.
---

# 🧠 Document Updater & Brain Sync Workflow

Cuando el usuario escriba `/doc` o solicite actualizar la documentación y su diario, **DEBES** detenerte y ejecutar estrictamente los siguientes pasos para preservar el contexto de las sesiones. 

Esta skill actúa como el mecanismo de memoria a largo plazo del agente.

## 1. Actualizar Documentación del Proyecto
Revisa y actualiza de ser necesario los siguientes archivos de negocio para reflejar el estado actual del desarrollo. Respeta siempre el formato YAML Frontmatter de Obsidian y mantén los enlaces (wikilinks).
- `docs/Producto_y_Ventas/Roadmap.md` (Avanza las tareas completadas).
- `docs/Producto_y_Ventas/Historias_de_Usuario.md`
- `brain/02_Business_Rules/Reglas_de_Negocio.md`
- `brain/03_Concepts/Diccionario_de_Datos.md` (Si se hicieron cambios en EF Core o la base de datos).

## 2. Actualizar el "Cerebro" (Diario del Agente)
El diario es vital para no perder contexto en el futuro. Sigue este procedimiento exacto:
1. Obtén la fecha y hora local actual del metadata que provee el sistema.
2. Identifica o crea el archivo correspondiente a la sesión en `brain/04_Logs/YYYY-MM-DD_Tema_Descriptivo.md` (añade un título corto con guiones bajos que resuma lo que se hizo en la sesión).
3. Si el archivo no existe, créalo. Si ya existe, edítalo añadiendo la nueva información (no borres lo que ya estaba de ese mismo día).
4. **Reglas de Calidad para el Diario**:
   - **Debe ser sumamente detallado y exhaustivo**. Escribe como un arquitecto senior.
   - Detalla comandos clave ejecutados, paquetes instalados, y la lógica detrás de las decisiones arquitectónicas tomadas (ej. por qué se usó un `record struct`, por qué CQRS, etc.).
   - Utiliza YAML Frontmatter de Obsidian (con properties como `fecha`, `tags: [log, ...]` y `tipo: log`).
   - Usa alertas de GitHub/Obsidian (`> [!success]`, `> [!info]`) para resaltar hitos.

## 3. Notificación Final
Una vez terminados los pasos 1 y 2, infórmale al usuario que su comando `/doc` se ha ejecutado exitosamente, listando rápidamente qué archivos se tocaron y confirmando que el contexto ("cerebro") está a salvo.
