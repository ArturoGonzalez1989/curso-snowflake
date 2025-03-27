---
title: "01. Introducción a Snowflake"
---

## Introducción y Arquitectura de Snowflake



- [Introducción y Arquitectura de Snowflake](#introducción-y-arquitectura-de-snowflake)
- [Gestión de roles y permisos](#gestión-de-roles-y-permisos)
  - [**RBAC (Role-Based Access Control)**](#rbac-role-based-access-control)
  - [**Roles predeterminados en Snowflake**](#roles-predeterminados-en-snowflake)
  - [**Herencia de privilegios**](#herencia-de-privilegios)
  - [**Gestión de permisos con comandos en Snowflake**](#gestión-de-permisos-con-comandos-en-snowflake)
  - [**Caso práctico: Gestión de roles y permisos en Snowflake**](#caso-práctico-gestión-de-roles-y-permisos-en-snowflake)
- [Conclusión](#conclusión)

---



## Gestión de roles y permisos

En Snowflake, los permisos y accesos a objetos se gestionan mediante un sistema de **roles y permisos**. Esto permite administrar de manera centralizada el acceso a los datos y controlar los privilegios de los usuarios de forma escalable.

### **RBAC (Role-Based Access Control)**
- Utiliza roles en lugar de asignaciones directas a usuarios.
- La estructura de asignación de permisos es **Permisos -> Roles -> Usuarios**.
- Simplifica la gestión de accesos y permite una mejor administración de los permisos.

### **Roles predeterminados en Snowflake**
- `ACCOUNTADMIN`: Nivel más alto de privilegio, administra toda la cuenta.
- `SYSADMIN`: Gestiona objetos de la base de datos como tablas y esquemas.
- `SECURITYADMIN`: Administra usuarios, roles y permisos de seguridad.
- `USERADMIN`: Diseñado para la gestión de usuarios y asignación de roles.
- `PUBLIC`: Rol predeterminado con permisos básicos para todos los usuarios.

### **Herencia de privilegios**
- Los roles pueden heredar permisos de otros roles asignados como "roles padres".
- Los permisos otorgados a un rol superior también están disponibles para sus roles hijos.
- Esto facilita la administración de permisos y evita configuraciones redundantes.

### **Gestión de permisos con comandos en Snowflake**

Algunos comandos clave para gestionar roles y permisos en Snowflake incluyen:

```sql
-- Crear un nuevo rol\CREATE ROLE data_analyst;

-- Asignar permisos a un rol
GRANT SELECT ON ALL TABLES IN SCHEMA my_db.my_schema TO ROLE data_analyst;

-- Asignar un rol a un usuario
GRANT ROLE data_analyst TO USER analyst_user;

-- Cambiar el rol activo
USE ROLE securityadmin;
```

### **Caso práctico: Gestión de roles y permisos en Snowflake**

En este caso práctico, aprenderás a:
- Crear y asignar roles a usuarios.
- Configurar permisos sobre bases de datos y esquemas.
- Administrar accesos de forma centralizada en Snowflake.

📄 **[Caso Práctico: Gestión de Roles y Permisos en Snowflake](01-caso-practico4.md)**



## Conclusión

Snowflake proporciona una solución escalable, segura y eficiente para el almacenamiento y análisis de datos en la nube. Su arquitectura única permite a las organizaciones optimizar sus operaciones sin preocuparse por la administración de infraestructura.  