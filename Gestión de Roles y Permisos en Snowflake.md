# Gestión de Roles y Permisos en Snowflake

## Introducción

En Snowflake, la gestión de roles y permisos es crucial para garantizar un acceso seguro y organizado a los datos y recursos de la plataforma. Snowflake utiliza un modelo basado en roles (RBAC - Role-Based Access Control), lo que permite asignar permisos a roles en lugar de directamente a usuarios.

### RBAC (Role-Based Access Control)
RBAC se utiliza para administrar permisos mediante la asignación de roles, lo que facilita la administración de accesos en entornos con múltiples usuarios.

- **Jerarquía:** Los permisos se asignan a roles, y los roles se asignan a usuarios.
- **Herencia de privilegios:** Un rol puede heredar permisos de otros roles, lo que permite organizar y simplificar la administración de accesos.

### Roles predefinidos en Snowflake
Snowflake proporciona una serie de roles predefinidos que cubren diferentes necesidades de administración:

- **ACCOUNTADMIN:** Nivel más alto de privilegio, con control total sobre la cuenta.
- **SYSADMIN:** Administrador de bases de datos, esquemas y objetos dentro de Snowflake.
- **SECURITYADMIN:** Responsable de la administración de usuarios, roles y permisos de seguridad.
- **USERADMIN:** Encargado de gestionar usuarios y sus roles, pero sin permisos sobre datos.
- **PUBLIC:** Rol predeterminado asignado a todos los usuarios con permisos mínimos.

### Jerarquía de Roles y Herencia de Permisos

En Snowflake, los roles pueden heredar permisos de otros roles asignados como "roles padres". Esto simplifica la gestión de permisos y evita configuraciones redundantes. Los permisos otorgados a un rol superior también están disponibles para sus roles hijos.

### Comandos importantes en la gestión de roles y permisos
Algunos comandos esenciales para administrar roles y permisos en Snowflake incluyen:

```sql
CREATE USER usuario1 PASSWORD = 'Password123'; -- Creación de usuario
GRANT ROLE SYSADMIN TO USER usuario1; -- Asignación de un rol a un usuario
SHOW ROLES; -- Visualización de roles disponibles en la cuenta
SHOW USERS; -- Listado de usuarios creados en la cuenta
CREATE ROLE data_analyst; -- Creación de un rol personalizado
GRANT USAGE ON DATABASE my_db TO ROLE data_analyst; -- Asignación de permisos a un rol
```

---

## Caso práctico: Gestión de roles y permisos en Snowflake

### Objetivo
Explorar la jerarquía de roles en Snowflake, asignar permisos y evaluar el acceso de usuarios.

### Pasos

#### 1. Creación de la base de datos y el esquema (Ejecutar como ACCOUNTADMIN)
> **Nota:** Para ejecutar los comandos SQL, abre una SQL Worksheet desde "Projects -> Worksheets" en el menú lateral izquierdo y haz clic en el botón añadir (+) en la esquina superior derecha.

Introduce y ejecuta los siguientes comandos:

```sql
CREATE OR REPLACE DATABASE my_db;
CREATE OR REPLACE SCHEMA my_db.my_schema;
USE DATABASE my_db;
USE SCHEMA my_db.my_schema;
```

#### 2. Creación de una tabla de prueba (Ejecutar como ACCOUNTADMIN)
```sql
CREATE OR REPLACE TABLE my_db.my_schema.dummy_table (
    id INT AUTOINCREMENT PRIMARY KEY,
    nombre STRING,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 3. Creación de usuarios de prueba (Ejecutar como ACCOUNTADMIN)
```sql
CREATE USER usuario1 PASSWORD = 'Password123';
CREATE USER usuario2 PASSWORD = 'Password123';
```

#### 4. Asignación de roles predefinidos (Ejecutar como ACCOUNTADMIN)
```sql
GRANT ROLE ACCOUNTADMIN TO USER usuario1;
GRANT ROLE SYSADMIN TO USER usuario2;
```

#### 5. Consulta para ver los roles asignados a los usuarios creados (Ejecutar como ACCOUNTADMIN)
```sql
SHOW GRANTS TO USER usuario1;
SHOW GRANTS TO USER usuario2;
```

#### 6. Verificación de permisos (Ejecutar como usuario1)
> **Nota:** Para ejecutar los comandos SQL, abre una SQL Worksheet desde "Projects -> Worksheets" en el menú lateral izquierdo y haz clic en el botón añadir (+) en la esquina superior derecha.
```sql
USE ROLE ACCOUNTADMIN;
SHOW ROLES;
SHOW USERS;
SHOW DATABASES;
```

#### 7. Verificación de permisos (Ejecutar como usuario2)
> **Nota:** Para ejecutar los comandos SQL, abre una SQL Worksheet desde "Projects -> Worksheets" en el menú lateral izquierdo y haz clic en el botón añadir (+) en la esquina superior derecha.
```sql
USE ROLE SYSADMIN;
SHOW ROLES;
SHOW USERS;
SHOW DATABASES;
```

#### 8. Intento de creación de un usuario desde SYSADMIN (debería fallar porque no tiene permiso, Ejecutar como usuario2)
```sql
CREATE USER usuario3 PASSWORD = 'Password123'; -- ERROR esperado
```

#### 9. Creación de un rol personalizado para análisis de datos (Ejecutar como ACCOUNTADMIN)
```sql
CREATE ROLE data_analyst;
```

#### 10. Asignación de permisos al rol personalizado (Ejecutar como ACCOUNTADMIN un usuario que tenga los permisos)
```sql
GRANT USAGE ON DATABASE my_db TO ROLE data_analyst;
GRANT USAGE ON SCHEMA my_db.my_schema TO ROLE data_analyst;
GRANT SELECT ON ALL TABLES IN SCHEMA my_db.my_schema TO ROLE data_analyst;
```

#### 11. Asignación del rol al usuario2 (Ejecutar como ACCOUNTADMIN un usuario que tenga los permisos)
```sql
GRANT ROLE data_analyst TO USER usuario2;
```

#### 12. Cambio de rol y verificación de permisos (Ejecutar como usuario2)
```sql
USE ROLE data_analyst;
USE DATABASE my_db;
USE SCHEMA my_db.my_schema;
SHOW GRANTS TO USER usuario2;
SELECT * FROM my_db.my_schema.dummy_table; /* Debería ser permitido. Puesto que la 
tabla no contiene datos no veremos resultados pero tampoco obtendremos errores.*/
```

#### 13. Intento de insertar datos (debería fallar porque solo tiene SELECT, Ejecutar como usuario2)
```sql
INSERT INTO my_db.my_schema.dummy_table (nombre) VALUES ('prueba'); -- ERROR esperado
```

#### 14. Ampliación de permisos para permitir inserción de datos (Ejecutar como ACCOUNTADMIN)
```sql
GRANT INSERT ON ALL TABLES IN SCHEMA my_db.my_schema TO ROLE data_analyst;
```

#### 15. Validación de permisos nuevamente (Ejecutar como usuario2)
```sql
USE ROLE data_analyst;
USE DATABASE my_db;
USE SCHEMA my_db.my_schema;
USE WAREHOUSE COMPUTE_WH;
INSERT INTO my_db.my_schema.dummy_table (nombre) VALUES ('prueba'); -- Ahora debería funcionar
```

#### 16. Conclusiones y eliminación de usuarios y roles (opcional, para limpiar el entorno, Ejecutar como ACCOUNTADMIN)
```sql
DROP USER usuario1;
DROP USER usuario2;
DROP ROLE data_analyst;
DROP DATABASE my_db;
```

---

## Conclusión
Este ejercicio práctico permite entender la gestión de roles en Snowflake, incluyendo la asignación de permisos, la verificación de accesos y la creación de roles personalizados. Además, destaca la importancia de asignar privilegios de manera estructurada para garantizar un acceso seguro y eficiente a los recursos de la base de datos.

