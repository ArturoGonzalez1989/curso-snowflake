# Caso Práctico: Uso de Tablas Externas y Vistas Materializadas con menús

Este caso práctico sirve como refuerzo de los conocimientos adquiridos en los apartados:  
📄 **[Stages en Snowflake](02_snowflake_objetos_seguros.md#stages-en-snowflake)**  
📄 **[Tablas Externas](02_snowflake_objetos_seguros.md#tablas-externas)**  
📄 **[Vistas Materializadas](02_snowflake_objetos_seguros.md#vistas-materializadas)**

En caso de dudas sobre alguno de los conceptos utilizados, se recomienda revisar nuevamente los apartados correspondientes en el material teórico.

## Objetivo del caso práctico

- Crear una tabla externa que haga referencia a archivos almacenados en un Data Lake (S3).  
- Copiar los datos en una tabla interna en Snowflake.  
- Crear una vista materializada para mejorar el rendimiento de consultas frecuentes.  
- Comparar el rendimiento de consultas sobre una tabla con y sin vista materializada.  
- Observar el comportamiento de actualización de las vistas materializadas.  

---

### Paso 1: Configuración del entorno y creación del Stage

En este paso se crea un nuevo esquema para organizar los objetos del caso práctico. Además, se configura un **Stage externo** que apunta a un bucket de S3 con datos del menú, y se verifica que los archivos estén accesibles.

```sql
USE DATABASE curso_snowflake_db;
CREATE OR REPLACE SCHEMA caso_external_menu;
USE SCHEMA caso_external_menu;

CREATE OR REPLACE STAGE blob_stage
URL = 's3://sfquickstarts/tastybytes/'
FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"');

-- Listar archivos disponibles
LIST @blob_stage/raw_pos/menu/;
```

---

### Paso 2: Creación de la tabla externa

En este paso se define una **tabla externa** que referencia directamente los archivos CSV almacenados en S3, sin necesidad de cargarlos físicamente en Snowflake.

```sql
CREATE OR REPLACE EXTERNAL TABLE menu_externo (
    menu_id NUMBER AS (VALUE:c1::NUMBER),
    menu_type_id NUMBER AS (VALUE:c2::NUMBER),
    menu_type STRING AS (VALUE:c3::STRING),
    truck_brand_name STRING AS (VALUE:c4::STRING),
    menu_item_id NUMBER AS (VALUE:c5::NUMBER),
    menu_item_name STRING AS (VALUE:c6::STRING),
    item_category STRING AS (VALUE:c7::STRING),
    item_subcategory STRING AS (VALUE:c8::STRING),
    cost_of_goods_usd NUMBER AS (VALUE:c9::NUMBER),
    sale_price_usd NUMBER AS (VALUE:c10::NUMBER),
    menu_item_health_metrics_obj VARIANT AS (VALUE:c11::VARIANT)
)
WITH LOCATION = @blob_stage/raw_pos/menu/
AUTO_REFRESH = TRUE
FILE_FORMAT = (TYPE = CSV SKIP_HEADER = 1 FIELD_OPTIONALLY_ENCLOSED_BY='"');
```

> ℹ️ **Nota**: `VALUE:c1` hace referencia a la primera columna del archivo. En archivos sin encabezado, se utilizan `c1`, `c2`, etc., por posición.

---

### Paso 3: Verificación de los datos

Aquí se realizan consultas de validación para asegurarse de que los datos de la tabla externa son accesibles y están bien estructurados.

```sql
SELECT * FROM menu_externo LIMIT 10;

SELECT COUNT(*) FROM menu_externo;
```

---

### Paso 4: Creación de una tabla interna

Ahora se crea una **tabla interna** copiando los datos desde la tabla externa. Esto permitirá comparar el rendimiento entre ambas estructuras.

```sql
CREATE OR REPLACE TABLE menu_interno AS 
SELECT * FROM menu_externo;

SELECT COUNT(*) FROM menu_interno;
```

---

### Paso 5: Creación de una vista materializada

Se crea una **vista materializada** para optimizar una consulta frecuente. Esta vista calculará el precio promedio por categoría de producto y marca del camión de comida.

```sql
CREATE OR REPLACE MATERIALIZED VIEW menu_resumen AS
SELECT 
    item_category, 
    truck_brand_name, 
    COUNT(*) AS total_items, 
    AVG(sale_price_usd) AS precio_promedio
FROM menu_interno
GROUP BY item_category, truck_brand_name;

-- Verificar estructura de la vista
DESC MATERIALIZED VIEW menu_resumen;
```

---


### Paso 6: Creación de una vista materializada sobre la tabla externa

En este paso se creará una **vista materializada directamente sobre la tabla externa** `menu_externo`. Esto permite acelerar el acceso a consultas agregadas sobre datos que aún residen fuera de Snowflake (por ejemplo, en S3), sin necesidad de cargarlos completamente en tablas internas.

```sql
CREATE OR REPLACE MATERIALIZED VIEW menu_externo_resumen AS
SELECT 
    item_category, 
    truck_brand_name, 
    COUNT(*) AS total_items, 
    AVG(sale_price_usd) AS precio_promedio
FROM menu_externo
GROUP BY item_category, truck_brand_name;
```

> ℹ️ **Nota**: Las vistas materializadas sobre tablas externas son útiles para evitar repetir cálculos pesados sobre archivos remotos. Almacenan los resultados de la consulta y se actualizan automáticamente cuando los datos subyacentes cambian.

Puedes consultar su estructura con:

```sql
DESC MATERIALIZED VIEW menu_externo_resumen;
```

---

### Paso 7: Comparación del rendimiento entre consultas

Se comparan dos formas de consultar el resumen de ventas: una directamente sobre la tabla interna y otra utilizando la vista materializada.

```sql
-- Consulta sobre la tabla interna (puede ser más lenta)
SELECT 
    item_category, 
    truck_brand_name, 
    COUNT(*) AS total_items, 
    AVG(sale_price_usd) AS precio_promedio
FROM menu_interno
GROUP BY item_category, truck_brand_name;

-- Consulta optimizada usando la vista materializada
SELECT * FROM menu_resumen;

-- Y también desde la vista basada en la tabla externa
SELECT * FROM menu_externo_resumen;
```

---

### Paso 8: Forzar la actualización de las vistas materializadas

Se inserta un nuevo registro en la tabla interna, y se compara si las vistas reflejan el cambio.

```sql
INSERT INTO menu_interno (menu_id, menu_type_id, menu_type, truck_brand_name, menu_item_id, menu_item_name, item_category, item_subcategory, cost_of_goods_usd, sale_price_usd, menu_item_health_metrics_obj)
VALUES (9999, 3, 'Fast Food', 'Burger Truck', 501, 'Mega Burger', 'Burgers', 'Special', 2.50, 8.99, NULL);

-- Consultar la vista materializada antes del refresh
SELECT * FROM menu_resumen WHERE item_category = 'Burgers';

-- Verificar también la vista sobre la tabla externa (si los datos del stage se modificaran)
SELECT * FROM menu_externo_resumen WHERE item_category = 'Burgers';
```

---

### Paso 9: Limpieza de objetos

Eliminar los objetos creados para evitar costes de almacenamiento innecesarios:

```sql
DROP MATERIALIZED VIEW IF EXISTS menu_resumen;
DROP MATERIALIZED VIEW IF EXISTS menu_externo_resumen;
DROP TABLE IF EXISTS menu_interno;
DROP EXTERNAL TABLE IF EXISTS menu_externo;
DROP STAGE IF EXISTS blob_stage;
```

