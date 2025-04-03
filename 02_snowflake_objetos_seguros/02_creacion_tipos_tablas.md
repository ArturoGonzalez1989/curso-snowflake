# Caso Práctico: Procesamiento de clientes con tablas temporales en Snowflake

Este caso práctico sirve como refuerzo de los conocimientos presentados en el documento:  
📄 **[Creación y Gestión de Objetos Seguros en Snowflake](02_objetos_seguros_snowflake.md)**  
En caso de dudas sobre los conceptos utilizados, se recomienda revisar nuevamente el documento y su sección correspondiente sobre tipos de tablas.

## Objetivo del caso práctico

- Crear una base de datos y un esquema en Snowflake.  
- Utilizar tablas temporales para filtrar y transformar datos.  
- Insertar los datos transformados en una tabla permanente.  
- Crear una tabla de tipo `TRANSIENT` y observar su comportamiento.  
- Comprender cómo las tablas temporales optimizan procesos ETL.  
- Consolidar conocimientos mediante preguntas de reflexión y validación final.  

---

#### Paso 1: Configuración del entorno en Snowflake

En este paso se configurará el entorno necesario para trabajar con los datos. Se seleccionará el rol de administrador, el warehouse adecuado, y se crearán la base de datos y el esquema donde se almacenarán las tablas utilizadas a lo largo del caso práctico.

```sql
USE ROLE ACCOUNTADMIN;
USE WAREHOUSE COMPUTE_WH;

CREATE DATABASE IF NOT EXISTS RETAIL_DB;
USE DATABASE RETAIL_DB;

CREATE SCHEMA IF NOT EXISTS CUSTOMER_ANALYSIS;
USE SCHEMA CUSTOMER_ANALYSIS;
```

---

#### Paso 2: Creación de la tabla temporal

En este paso se creará una tabla temporal con datos de clientes nacidos en Estados Unidos, extraídos de una tabla de muestra de Snowflake. Las tablas temporales permiten trabajar con datos durante una sesión sin ocupar espacio de almacenamiento permanente.

```sql
CREATE TEMPORARY TABLE temp_customers AS 
SELECT 
    c_customer_sk, 
    c_first_name, 
    c_last_name, 
    c_birth_country, 
    c_login, 
    c_preferred_cust_flag, 
    c_customer_id, 
    c_current_cdemo_sk 
FROM SNOWFLAKE_SAMPLE_DATA.TPCDS_SF100TCL.CUSTOMER 
WHERE c_birth_country = 'UNITED STATES';
```

> ℹ️ **Importante**: Se debe recordar que este tipo de tablas desaparecerán al cerrar la sesión en la que se han creado.

---

#### Paso 3: Transformación de datos

En este paso se realizará una transformación sobre los datos de la tabla temporal. El objetivo es clasificar a los clientes según su nivel de ingresos utilizando una nueva columna. Esta transformación puede hacerse añadiendo y actualizando una columna, o recreando la tabla con la columna calculada incluida.

**Opción 1: Utilizar `ALTER TABLE` + `UPDATE`**

```sql
ALTER TABLE temp_customers ADD COLUMN c_income_level STRING;

UPDATE temp_customers 
SET c_income_level = 
    CASE 
        WHEN c_current_cdemo_sk BETWEEN 1 AND 10000 THEN 'Low Income'
        WHEN c_current_cdemo_sk BETWEEN 10001 AND 50000 THEN 'Middle Income'
        WHEN c_current_cdemo_sk > 50000 THEN 'High Income'
        ELSE 'Unknown'
    END;
```

**Opción 2: Reemplazar la tabla temporal con la columna calculada**

```sql
CREATE OR REPLACE TEMPORARY TABLE temp_customers AS 
SELECT 
    c_customer_sk, 
    c_first_name, 
    c_last_name, 
    c_birth_country, 
    c_login, 
    c_preferred_cust_flag, 
    c_customer_id, 
    c_current_cdemo_sk,
    CASE 
        WHEN c_current_cdemo_sk BETWEEN 1 AND 10000 THEN 'Low Income'
        WHEN c_current_cdemo_sk BETWEEN 10001 AND 50000 THEN 'Middle Income'
        WHEN c_current_cdemo_sk > 50000 THEN 'High Income'
        ELSE 'Unknown'
    END AS c_income_level
FROM temp_customers;
```

---

#### Paso 4: Carga en tabla permanente

En este paso se guardarán los datos procesados en una tabla permanente. Dado que las tablas temporales desaparecen al finalizar la sesión, esta operación asegura la persistencia de los datos transformados para su uso posterior.

```sql
CREATE OR REPLACE TABLE processed_customers AS 
SELECT * FROM temp_customers;
```

---

#### Paso 5: Creación de una tabla transitoria

Además de las tablas permanentes y temporales, Snowflake permite el uso de **tablas transitorias (TRANSIENT)**. Estas tablas conservan datos entre sesiones, pero no mantienen historial para funciones como Time Travel o Fail-safe, lo que permite optimizar almacenamiento y costes.

En este paso se creará una tabla `TRANSIENT` a partir de los mismos datos ya procesados:

```sql
CREATE OR REPLACE TRANSIENT TABLE transient_customers AS 
SELECT * FROM processed_customers;
```

---

#### Paso 6: Validación de persistencia tras cerrar sesión

En este paso se comprobará el comportamiento de persistencia de los diferentes tipos de tabla.

1. Cerrar la sesión actual de Snowflake.  
2. Volver a iniciar sesión en Snowflake con el mismo usuario.  
3. Ejecutar las siguientes consultas para verificar qué tablas están disponibles:

```sql
SHOW TABLES IN SCHEMA RETAIL_DB.CUSTOMER_ANALYSIS;
```

4. Confirmar que:
   - La tabla `temp_customers` ya **no está disponible**.
   - Las tablas `processed_customers` y `transient_customers` **se conservan**.

---

#### Preguntas de reflexión

1. **¿Es posible modificar una tabla temporal después de su creación?**  
<details><summary>Mostrar solución</summary>
   Sí, es posible utilizando `ALTER TABLE` o `UPDATE`, aunque los cambios solo persistirán durante la sesión activa.
</details>
<p></p>

2. **¿Qué ventajas ofrece el uso de tablas temporales en procesos ETL?**
<details><summary>Mostrar solución</summary>  
   Mejoran el rendimiento, reducen costos, facilitan la transformación de datos y evitan almacenamiento innecesario.
</details>
<p></p>

3. **¿Cuál es la principal diferencia entre `temp_customers`, `transient_customers` y `processed_customers`?**  
<details><summary>Mostrar solución</summary>
   `temp_customers` es temporal y desaparece al cerrar sesión.  
   `transient_customers` persiste entre sesiones, pero no guarda historial.  
   `processed_customers` es una tabla permanente y conserva historial y opciones de recuperación.
</details>
<p></p>

4. **¿Qué ocurre si se intenta consultar `temp_customers` en una nueva sesión?**
<details><summary>Mostrar solución</summary>
   La consulta fallará, ya que la tabla temporal habrá sido eliminada automáticamente.
</details>
<p></p>

5. **¿Cómo puede utilizarse este enfoque en un proceso ETL en producción?**
<details><summary>Mostrar solución</summary>
   Puede aplicarse como paso intermedio para limpieza y transformación antes de insertar los datos en tablas permanentes.
</details>

