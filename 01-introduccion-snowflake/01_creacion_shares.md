# Caso Práctico: Creación y uso de Shares en Snowflake

Este caso práctico servirá de refuerzo de los conocimientos aprendidos en el documento:  
📄 **[Introducción y Arquitectura de Snowflake](01_introduccion_snowflake.md)**  
Ante cualquier duda que se pueda tener acerca de los conceptos adquiridos, se recomienda revisar nuevamente el documento y su sección correspondiente.



## Objetivo del caso práctico

- Compartir datos entre diferentes cuentas de Snowflake usando `Shares`.
- Administrar permisos sobre vistas seguras.
- Simular la configuración desde la cuenta del proveedor y del consumidor.
- Crear cuentas Reader para empresas sin Snowflake.
- Consolidar conocimientos aprendidos.

---

### Paso 1: Creación de objetos iniciales en Snowflake

Desde el rol `SYSADMIN`, crea tres bases de datos: `sales`, `marketing` y `sharedb`.  
En cada una de ellas, crea un esquema llamado `my_schema` (o `shares` en el caso de `sharedb`) y las siguientes tablas:

- En `sales.my_schema`, crea una tabla `customer` con los campos:  
  - `customer_id` (INT)  
  - `customer_name` (STRING)

- En `marketing.my_schema`, crea una tabla `conversions` con los campos:  
  - `customer_id` (INT)  
  - `campaign` (STRING)  
  - `converted` (BOOLEAN)

- En `sharedb`, crea un esquema llamado `shares` (no se requiere tabla por ahora).

<details><summary>Mostrar solución</summary>

```sql
USE ROLE SYSADMIN;

CREATE DATABASE sales;
CREATE SCHEMA sales.my_schema;
CREATE TABLE sales.my_schema.customer (
    customer_id INT,
    customer_name STRING
);

CREATE DATABASE marketing;
CREATE SCHEMA marketing.my_schema;
CREATE TABLE marketing.my_schema.conversions (
    customer_id INT,
    campaign STRING,
    converted BOOLEAN
);

CREATE DATABASE sharedb;
CREATE SCHEMA sharedb.shares;
```

</details>

---

### Paso 2: Crear una vista segura para compartir

Crea una vista segura (`SECURE VIEW`) en `sharedb.shares` llamada `sharedview`.  
La vista debe combinar datos de clientes y conversiones, mostrando únicamente aquellos clientes que hayan convertido (`converted = TRUE`).

<details><summary>Mostrar solución</summary>

```sql
CREATE SECURE VIEW sharedb.shares.sharedview AS
SELECT 
    customer.customer_id, 
    customer.customer_name, 
    conversions.campaign, 
    conversions.converted
FROM sales.my_schema.customer customer
JOIN marketing.my_schema.conversions conversions 
    ON customer.customer_id = conversions.customer_id
WHERE converted;
```

</details>

---

### Paso 3: Crear el Data Share

Desde el rol `ACCOUNTADMIN`, crea un objeto `SHARE` llamado `myshare`.

<details><summary>Mostrar solución</summary>

```sql
USE ROLE ACCOUNTADMIN;

CREATE SHARE myshare;
```

</details>

---

### Paso 4: Otorgar permisos al Share

Desde el rol `SECURITYADMIN`, otorga los siguientes permisos al objeto `SHARE` creado:

- `USAGE` sobre la base de datos `sharedb` y su esquema `shares`.
- `REFERENCE_USAGE` sobre las bases de datos `sales` y `marketing`.
- `SELECT` sobre la vista `sharedb.shares.sharedview`.

<details><summary>Mostrar solución</summary>

```sql
USE ROLE SECURITYADMIN;

GRANT USAGE ON DATABASE sharedb TO SHARE myshare;
GRANT USAGE ON SCHEMA sharedb.shares TO SHARE myshare;

GRANT REFERENCE_USAGE ON DATABASE sales TO SHARE myshare;
GRANT REFERENCE_USAGE ON DATABASE marketing TO SHARE myshare;

GRANT SELECT ON VIEW sharedb.shares.sharedview TO SHARE myshare;
```

</details>

---

### Paso 5: Añadir una cuenta consumidora al Share

Desde `ACCOUNTADMIN`, añade una cuenta externa al Share.  
(Sustituye `DROWAQJ.YJ40678` por el ID real si fuera necesario).

<details><summary>Mostrar solución</summary>

```sql
USE ROLE ACCOUNTADMIN;

ALTER SHARE myshare ADD ACCOUNT = DROWAQJ.YJ40678;
SHOW SHARES;
SHOW GRANTS OF SHARE myshare;
```

</details>

---

### Paso 6: Configuración del consumidor del Data Share

Simula la configuración en la cuenta consumidora.  
Utiliza `DESC SHARE` para ver los detalles del Share, crea una base de datos a partir de él y otorga los permisos necesarios para acceder a los datos.

<details><summary>Mostrar solución</summary>

```sql
USE ROLE ACCOUNTADMIN;

DESC SHARE WMXCUWM.UR83766.myshare;
CREATE DATABASE UR83766_share FROM SHARE WMXCUWM.UR83766.myshare;
GRANT IMPORTED PRIVILEGES ON DATABASE UR83766_share TO SYSADMIN;

USE ROLE SYSADMIN;
SHOW VIEWS;
SELECT * FROM UR83766_SHARE.SHARES.SHAREDVIEW;
```

</details>

---

### Paso 7: Crear un Reader Account para empresas externas

Desde `ACCOUNTADMIN`, crea una cuenta Reader para una empresa que no dispone de cuenta en Snowflake.  
A continuación, añade dicha cuenta al Share.

<details><summary>Mostrar solución</summary>

```sql
USE ROLE ACCOUNTADMIN;

CREATE MANAGED ACCOUNT reader_acct1
    ADMIN_NAME = 'MySecretUsername',
    ADMIN_PASSWORD = 'MySuperSecretPassword',
    TYPE = READER;

/* Resultado esperado:
| {"accountName":"RE47190","loginUrl":"https://re47190.snowflakecomputing.com"} |
*/

ALTER SHARE myshare ADD ACCOUNTS=RE47190;
```

</details>

---

### Paso 8: Validación del Data Share

Valida que los datos sean accesibles tanto desde el proveedor como desde un consumidor simulado.  
(En este paso se utiliza la opción `SIMULATED_DATA_SHARING_CONSUMER`).

<details><summary>Mostrar solución</summary>

```sql
SELECT COUNT(*) FROM sharedb.shares.sharedview;
SELECT TOP 1 * FROM sharedb.shares.sharedview;

ALTER SESSION SET SIMULATED_DATA_SHARING_CONSUMER='ABC123';

SELECT COUNT(*) FROM sharedb.shares.sharedview;
SELECT TOP 10 * FROM sharedb.shares.sharedview;
```

</details>

---

### Paso 9: Monitoreo y auditoría del Share

Consulta los permisos y el estado actual del objeto `SHARE`.

<details><summary>Mostrar solución</summary>

```sql
SHOW SHARES;
SHOW GRANTS TO SHARE myshare;
SHOW GRANTS OF SHARE myshare;
```

</details>

---

### Paso 10: Añadir y remover objetos del Share

Crea una nueva vista a partir de una tabla clonada y agrégala al Share.  
Luego, elimina del Share una vista que ya no sea necesaria (`campaigns`, por ejemplo).

<details><summary>Mostrar solución</summary>

```sql
CREATE TABLE sharedb.shares.BRONZE_CUSTOMER_RAW_CLONE 
AS SELECT * FROM CURSO_SNOWFLAKE_DB.CASO_ETL.BRONZE_CUSTOMER_RAW;

CREATE OR REPLACE SECURE VIEW sharedb.shares.BRONZE_CUSTOMER_RAW AS
SELECT * FROM sharedb.shares.BRONZE_CUSTOMER_RAW_CLONE;

USE ROLE SECURITYADMIN;
GRANT SELECT ON VIEW sharedb.shares.BRONZE_CUSTOMER_RAW TO SHARE myshare;

REVOKE SELECT ON VIEW sharedb.shares.campaigns FROM SHARE myshare;
```

</details>
