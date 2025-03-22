# Caso Práctico: Carga de Datos en Snowflake con COPY INTO

## Objetivo
Cargar un archivo CSV en Snowflake usando un stage y el comando `COPY INTO`.

---

## Paso 1: Creación del esquema en Snowflake

Usaremos la base de datos ya creada (`CURSO_SNOWFLAKE_DB`), pero crearemos un nuevo esquema para este caso práctico.

<details><summary>Mostrar solución</summary>

```sql
USE DATABASE CURSO_SNOWFLAKE_DB;

CREATE OR REPLACE SCHEMA CASO2;
USE SCHEMA CASO2;
```
</details>

---

## Paso 2: Creación de la tabla destino en Snowflake

Se debe crear una tabla en Snowflake (puedes llamarla `netflix_titles_raw`) que almacene los datos del archivo CSV con los siguientes atributos:
- `show_id` VARCHAR(10)
- `type` VARCHAR(50)
- `title` VARCHAR(500)
- `director` VARCHAR(500)
- `cast_d` VARCHAR(1000)
- `country` VARCHAR(500)
- `date_added` VARCHAR(50)
- `release_year` VARCHAR(10)
- `rating` VARCHAR(10)
- `duration` VARCHAR(50)
- `listed_in` VARCHAR(500)
- `description` VARCHAR(2000)

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE TABLE netflix_titles_raw (
    show_id VARCHAR(10),
    type VARCHAR(50),
    title VARCHAR(500),
    director VARCHAR(500),
    cast_d VARCHAR(1000),
    country VARCHAR(500),
    date_added VARCHAR(50),
    release_year VARCHAR(10),
    rating VARCHAR(10),
    duration VARCHAR(50),
    listed_in VARCHAR(500),
    description VARCHAR(2000)
);
```
</details>

---

## Paso 3: Creación de un FILE FORMAT para CSV

Se debe definir un formato de archivo CSV en Snowflake para especificar la estructura de los datos que se cargarán.

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE FILE FORMAT my_csv_format
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
SKIP_HEADER = 1
EMPTY_FIELD_AS_NULL = TRUE;
```
</details>

---

## Paso 4: Creación de un stage interno en Snowflake

Se debe crear un stage interno en Snowflake para almacenar archivos antes de cargarlos en la tabla destino.

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE STAGE my_stage
FILE_FORMAT = my_csv_format;
```
</details>

---

## Paso 5: Instrucciones para cargar el archivo en el stage desde Snowsight

Se debe subir el archivo CSV al stage siguiendo estos pasos:

1. Iniciar sesión en Snowsight.
2. Ir a la sección **"Data"** y luego **"Stages"**.
3. Seleccionar el stage **"my_stage"**.
4. Subir el archivo CSV **"netflix_titles.csv"** desde su equipo.
**[netflix_titles.csv](resources/netflix_titles.csv)**
---

## Paso 6: Verificación de los archivos en el stage

Se debe verificar que el archivo ha sido correctamente subido al stage.

<details><summary>Mostrar solución</summary>

```sql
LIST @my_stage;
```
</details>

---

## Paso 7: Uso del comando COPY INTO para cargar los datos en la tabla destino

Se debe cargar el archivo CSV desde el stage a la tabla `netflix_titles_raw` utilizando `COPY INTO`.

<details><summary>Mostrar solución</summary>

```sql
COPY INTO netflix_titles_raw
FROM @my_stage/netflix_titles.csv
FILE_FORMAT = my_csv_format;
```
</details>

---

## Paso 8: Creación de la tabla final con datos transformados

Se debe crear una nueva tabla transformando los datos cargados en la tabla bruta:
- Convertir `release_year` a `INT`.
- Transformar `date_added` a formato de fecha utilizando `TRY_TO_DATE`.

La conversión de datos se puede realizar utilizando la notación `::` o con la función `CAST`.

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE TABLE netflix_titles AS
SELECT
    show_id,
    type,
    title,
    director,
    cast_d,
    country,
    TRY_TO_DATE(date_added, 'MMMM DD, YYYY') AS date_added,
    release_year::INT AS release_year,
    rating,
    duration,
    listed_in,
    description
FROM netflix_titles_raw;
```
</details>

---

## Paso 9: Verificación de la carga de datos

Se debe consultar la tabla final para verificar que los datos han sido correctamente transformados y cargados.

<details><summary>Mostrar solución</summary>

```sql
SELECT * FROM netflix_titles;
```
</details>

---

## Notas

1. Reemplazar `'ruta/al/archivo.csv'` con la ubicación y nombre real del archivo CSV.
2. Asegurarse de que la estructura del archivo CSV coincide con la de la tabla creada.
3. Revisar los registros cargados y corregir posibles errores en los datos.
4. Si se requiere, limpiar el stage con `REMOVE @my_stage` después de la carga.

