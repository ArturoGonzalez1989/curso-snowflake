---
title: "Caso Práctico: Carga de Datos con COPY INTO desde Stage Externo"
---

# Caso Práctico: Carga de Datos con COPY INTO desde Stage Externo

Este caso práctico tiene como objetivo reforzar el uso del comando `COPY INTO` en Snowflake, permitiendo cargar archivos CSV almacenados en un bucket de Amazon S3 mediante un Stage externo. Se trabajará con datos reales del menú de la empresa ficticia **Tasty Bytes**, incluyendo análisis de datos estructurados y semiestructurados.

📄 **[Stages en Snowflake](02_snowflake_objetos_seguros.md#stages-en-snowflake)**  
📄 **[COPY INTO y carga de datos](02_snowflake_objetos_seguros.md#copy-into)**  
📄 **[Tablas Externas](02_snowflake_objetos_seguros.md#tablas-externas)**

---

## Objetivos del caso práctico

- Establecer el contexto adecuado (rol y warehouse).
- Crear una base de datos y esquema organizados por funcionalidad.
- Crear una tabla para almacenar datos de menú desde archivo CSV.
- Crear un stage externo que conecte con un Blob Storage en S3.
- Cargar los datos del archivo CSV en la tabla usando `COPY INTO`.
- Realizar consultas sobre los datos cargados.
- Trabajar con datos semiestructurados usando `FLATTEN` y JSON.

---

## Paso 1: Establecer el contexto de rol y warehouse

Establecemos el entorno de trabajo para tener permisos suficientes y recursos computacionales disponibles.

```sql
USE ROLE accountadmin;
USE WAREHOUSE compute_wh;
```

---

## Paso 2: Crear la base de datos, esquema y tabla `menu`

Creamos la base de datos, el esquema y la tabla que almacenará los datos de los menús, incluyendo campos numéricos, de texto y un campo semiestructurado (`VARIANT`).

```sql
CREATE OR REPLACE DATABASE tasty_bytes_sample_data;

CREATE OR REPLACE SCHEMA tasty_bytes_sample_data.raw_pos;

CREATE OR REPLACE TABLE tasty_bytes_sample_data.raw_pos.menu (
    menu_id NUMBER(19,0),
    menu_type_id NUMBER(38,0),
    menu_type VARCHAR(16777216),
    truck_brand_name VARCHAR(16777216),
    menu_item_id NUMBER(38,0),
    menu_item_name VARCHAR(16777216),
    item_category VARCHAR(16777216),
    item_subcategory VARCHAR(16777216),
    cost_of_goods_usd NUMBER(38,4),
    sale_price_usd NUMBER(38,4),
    menu_item_health_metrics_obj VARIANT
);
```

---

## Paso 3: Crear el `Stage` externo que apunta a S3

Definimos un Stage externo que nos conecta con un bucket de S3 que contiene archivos del menú en formato CSV. También se define el formato del archivo a utilizar.

```sql
CREATE OR REPLACE STAGE tasty_bytes_sample_data.public.blob_stage
URL = 's3://sfquickstarts/tastybytes/'
FILE_FORMAT = (TYPE = csv);
```

Verificamos el contenido del stage para asegurarnos de que los datos están accesibles:

```sql
LIST @tasty_bytes_sample_data.public.blob_stage/raw_pos/menu/;
```

---

## Paso 4: Cargar el archivo CSV del menú en la tabla

Utilizamos `COPY INTO` para importar los datos desde los archivos CSV del Stage hacia la tabla previamente creada.

```sql
COPY INTO tasty_bytes_sample_data.raw_pos.menu
FROM @tasty_bytes_sample_data.public.blob_stage/raw_pos/menu/;
```

---

## Paso 5: Consultas de validación y exploración de los datos

Validamos que los datos se hayan cargado correctamente y revisamos algunas filas de ejemplo.

```sql
-- Cantidad total de registros
SELECT COUNT(*) AS cantidad_filas 
FROM tasty_bytes_sample_data.raw_pos.menu;

-- Primeras 10 filas de la tabla
SELECT TOP 10 * 
FROM tasty_bytes_sample_data.raw_pos.menu;
```

---

## Paso 6: Consultas específicas de negocio

Ejecutamos consultas para responder preguntas comerciales concretas como: ¿qué vende una marca específica?, ¿cuál es la ganancia por ítem?

```sql
-- Menú ofrecido por la marca 'Freezing Point'
SELECT menu_item_name
FROM tasty_bytes_sample_data.raw_pos.menu
WHERE truck_brand_name = 'Freezing Point';

-- Ganancia sobre un ítem específico
SELECT 
   menu_item_name,
   (sale_price_usd - cost_of_goods_usd) AS ganancia_usd
FROM tasty_bytes_sample_data.raw_pos.menu
WHERE truck_brand_name = 'Freezing Point'
  AND menu_item_name = 'Mango Sticky Rice';
```

---

## Paso 7: Análisis de datos semiestructurados

Se utiliza la función `FLATTEN` para acceder a la estructura interna del campo tipo `VARIANT` y extraer información como la lista de ingredientes.

```sql
-- Extraer ingredientes desde campo tipo JSON (VARIANT)
SELECT 
    m.menu_item_name,
    obj.value:"ingredients"::ARRAY AS ingredientes
FROM tasty_bytes_sample_data.raw_pos.menu m,
     LATERAL FLATTEN (input => m.menu_item_health_metrics_obj:menu_item_health_metrics) obj
WHERE truck_brand_name = 'Freezing Point'
  AND menu_item_name = 'Mango Sticky Rice';
```
