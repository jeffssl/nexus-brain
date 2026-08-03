---
name: sync-db
description: Sincroniza la arquitectura de base de datos (DBML) con la documentación local y la nube (dbdiagram)
---

# Workflow: Sincronización de Base de Datos

Este workflow se ejecuta cuando el usuario ha realizado cambios manuales en el archivo de diseño de base de datos (`Nexus.dbml`) o en las migraciones de Entity Framework, y necesita empujarlos a la documentación y a la nube.

## Paso 1: Leer y validar DBML
1. Lee todo el contenido del archivo `brain/01_Architecture/Nexus.dbml`.
2. Asegúrate de que el formato sea correcto.

## Paso 2: Actualizar Documentación Local
1. Sobrescribe y sincroniza el archivo `brain/01_Architecture/Esquema_Nexus.md` con la versión más reciente del esquema.
2. Identifica qué tablas o columnas se añadieron/modificaron y actualiza la nota `brain/Agendamiento-Clinicas-Laboratorios/01_Departamentos/Base_de_Datos/Diccionario_de_Datos.md` (o el equivalente en el nuevo cerebro si se ha movido).

## Paso 3: Sincronizar con el Servidor MCP (dbdiagram.io)
1. Ejecuta el servidor MCP `dbdiagramflow` usando la tool `apply_dbml`.
2. Pasa como argumento el id del diagrama: `e4a55818-1c08-4d0b-8adb-faec8022d60b`.
3. Pasa como el argumento `dbml` todo el contenido de texto que extrajiste en el Paso 1.

## Paso 4: Registrar en el Diario
1. Escribe en el Changelog o diario del proyecto una breve nota explicando qué entidades se detectaron como modificadas en esta sincronización.
2. Invoca el comando `/doc-updater` internamente si es necesario para guardar el contexto en la bitácora del día.

## Paso 5: Confirmación
1. Envíale un resumen al usuario confirmando que los Markdowns y la plataforma en la nube fueron actualizados exitosamente.
