# Caso Práctico: Carga de Datos en Snowflake con COPY INTO

## Objetivo
Cargar un archivo CSV en Snowflake usando un stage y el comando `COPY INTO`.

---

## Paso 1: Creación de la base de datos y el esquema

Se debe crear una base de datos y un esquema nuevos o usar algunos existentes.

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE DATABASE CURSO_SNOWFLAKE_DB;
USE DATABASE CURSO_SNOWFLAKE_DB;

CREATE OR REPLACE SCHEMA CASO1;
USE SCHEMA CASO1;
```
</details>

---

## Paso 2: Creación de la tabla destino en Snowflake

Se debe crear una tabla en Snowflake que almacene los datos del archivo CSV con los siguientes atributos:
- `rank` VARCHAR(10)
- `video` VARCHAR(500)
- `video_views` VARCHAR(50)
- `likes` VARCHAR(50)
- `dislikes` VARCHAR(50)
- `category` VARCHAR(100)
- `published` VARCHAR(10)

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE TABLE video_rankings_raw (
    rank VARCHAR(10),
    video VARCHAR(500),
    video_views VARCHAR(50),
    likes VARCHAR(50),
    dislikes VARCHAR(50),
    category VARCHAR(100),
    published VARCHAR(10)
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

Se debe crear un stage interno en Snowflake para almacenar archivos antes de cargarlos en la tabla destino. En la definición del Stage, se recomienda usar el File Format creado previsamente.

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
4. Subir el archivo CSV **[Most popular 1000 Youtube videos.csv](resources/Most_popular_1000_Youtube_videos.csv)** desde su equipo.

---

## Paso 6: Verificación de los archivos en el stage

Se debe verificar que el archivo ha sido correctamente subido al stage mediante el uso del comando LIST y el nombre del Stage.

<details><summary>Mostrar solución</summary>

```sql
LIST @my_stage;
```
</details>

---

## Paso 7: Uso del comando COPY INTO para cargar los datos en la tabla destino

Se debe cargar el archivo CSV desde el stage a la tabla `video_rankings_raw` utilizando `COPY INTO`.

<details><summary>Mostrar solución</summary>

```sql
COPY INTO video_rankings_raw
FROM '@my_stage/Most_popular_1000_Youtube_videos.csv'
FILE_FORMAT = my_csv_format;
```
</details>

---

## Paso 8: Creación de la tabla final con datos transformados

Se debe crear una nueva tabla transformando los datos cargados en la tabla raw:
- Convertir `rank` a `INT`.
- Eliminar comas de `video_views`, `likes` y `dislikes`, y convertirlos a `INT`.
- Convertir `published` a `INT`.

La conversión de datos se puede realizar utilizando la notación :: (por ejemplo `::INT`) o con la función CAST (`CAST(... AS INT)`).

<details><summary>Mostrar solución</summary>

```sql
CREATE OR REPLACE TABLE video_rankings AS
SELECT
    rank::INT AS rank,
    video,
    REPLACE(video_views, ',', '')::INT AS video_views,
    REPLACE(likes, ',', '')::INT AS likes,
    REPLACE(dislikes, ',', '')::INT AS dislikes,
    category,
    published::INT AS published
FROM video_rankings_raw;
```
</details>

---

## Paso 9: Verificación de la carga de datos

Se debe consultar la tabla final para verificar que los datos han sido correctamente transformados y cargados.

<details><summary>Mostrar solución</summary>

```sql
SELECT * FROM video_rankings;
```
</details>

---

## Notas

1. Reemplazar `'ruta/al/archivo.csv'` con la ubicación y nombre real del archivo CSV.
2. Asegurarse de que la estructura del archivo CSV coincide con la de la tabla creada.
3. Revisar los registros cargados y corregir posibles errores en los datos.
4. Si se requiere, limpiar el stage con `REMOVE @my_stage` después de la carga.

