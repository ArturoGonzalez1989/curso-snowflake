# Caso Práctico: Carga de datos con COPY INTO desde Internal Stage

Este caso práctico tiene como objetivo reforzar el uso del comando `COPY INTO` en Snowflake, permitiendo cargar archivos CSV desde el equipo del usuario a través de un Stage interno. Se trabajarán dos ejercicios independientes: uno con datos de títulos de Netflix y otro con rankings de vídeos de YouTube.

📄 **[Stages en Snowflake](02_snowflake_objetos_seguros.md#stages-en-snowflake)**  
📄 **[COPY INTO y carga de datos](02_snowflake_objetos_seguros.md#copy_into)**

---

## Objetivos del caso práctico

- Crear estructuras de tablas compatibles con archivos CSV.
- Configurar un formato de archivo (`FILE FORMAT`) en Snowflake.
- Crear y utilizar un Stage interno para subir datos.
- Usar el comando `COPY INTO` para cargar datos desde el Stage a Snowflake.
- Aplicar transformaciones sobre los datos brutos y crear tablas limpias.
- Verificar y validar la carga de datos.

---

## Ejercicio 1: Carga de datos de títulos de Netflix

### Paso 1: Preparación del entorno

Se utiliza la base de datos `CURSO_SNOWFLAKE_DB` y se crea un nuevo esquema llamado `ejercicio_1` para organizar los objetos de este ejercicio.

```sql
USE DATABASE CURSO_SNOWFLAKE_DB;
CREATE OR REPLACE SCHEMA ejercicio_1;
USE SCHEMA ejercicio_1;
```

---

### Paso 2: Creación de la tabla bruta `netflix_titles_raw`

Se define una tabla con todos los campos tal como se encuentran en el archivo [`netflix_titles.csv`](./resources/netflix_titles.csv). Esta tabla almacenará los datos en bruto, sin transformación.

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

---

### Paso 3: Creación del formato de archivo CSV

Se define el formato que Snowflake utilizará para interpretar los archivos CSV que se subirán. Este formato asume encabezado en la primera fila, permite campos encerrados entre comillas y trata los vacíos como `NULL`.

```sql
CREATE OR REPLACE FILE FORMAT my_csv_format
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
SKIP_HEADER = 1
EMPTY_FIELD_AS_NULL = TRUE;
```

---

### Paso 4: Creación del Stage interno

El Stage es el área temporal de Snowflake donde se cargarán los archivos antes de insertarlos en una tabla. Aquí se asocia el formato de archivo previamente creado.

```sql
CREATE OR REPLACE STAGE my_stage
FILE_FORMAT = my_csv_format;
```

---

### Paso 5: Subida del archivo `netflix_titles.csv` desde Snowsight

En este paso el usuario debe subir el archivo local a Snowflake:

1. Iniciar sesión en Snowsight.  
2. Ir a la sección **"Data" → "Stages"**.  
3. Seleccionar el Stage `my_stage`.  
4. Subir el archivo [`netflix_titles.csv`](./resources/netflix_titles.csv)   desde tu equipo.

---

### Paso 6: Verificación del archivo en el Stage

Se verifica que el archivo fue subido correctamente al Stage.

```sql
LIST @my_stage;
```

---

### Paso 7: Carga de datos con COPY INTO

Se carga el contenido del archivo CSV desde el Stage hacia la tabla bruta.

```sql
COPY INTO netflix_titles_raw
FROM @my_stage/netflix_titles.csv
FILE_FORMAT = my_csv_format;
```

---

### Paso 8: Creación de tabla transformada `netflix_titles`

Se transforma la tabla bruta: se convierte el campo de fecha a tipo `DATE` y se tipifican correctamente algunos campos.

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

---

### Paso 9: Verificación de los datos cargados

Se realiza una consulta para validar que los datos han sido cargados y transformados correctamente.

```sql
SELECT * FROM netflix_titles;
```

---

## Ejercicio 2: Carga de rankings de vídeos de YouTube

### Paso 1: Preparación del entorno

Se crea un nuevo esquema llamado `ejercicio_2` para este segundo ejercicio, manteniendo organizada la base de datos.

```sql
CREATE OR REPLACE SCHEMA ejercicio_2;
USE SCHEMA ejercicio_2;
```

---

### Paso 2: Creación de la tabla bruta `video_rankings_raw`

Se define la estructura de la tabla que recibirá los datos del archivo [`resources/Most_popular_1000_Youtube_videos.csv`](./resources/Most_popular_1000_Youtube_videos.csv).

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

> ℹ️ El formato de archivo y el Stage ya fueron creados en el Ejercicio 1 (`my_csv_format` y `my_stage`), por lo tanto se reutilizan.

---

### Paso 3: Subida del archivo CSV al Stage

Repite el procedimiento en Snowsight:

1. Ir a **Snowsight > Data > Stages**  
2. Seleccionar `my_stage`  
3. Subir el archivo: [`resources/Most_popular_1000_Youtube_videos.csv`](./resources/Most_popular_1000_Youtube_videos.csv)

---

### Paso 4: Verificación del archivo

Confirmamos que el archivo ha sido correctamente subido:

```sql
LIST @my_stage;
```

---

### Paso 5: Carga con COPY INTO

Se carga el contenido del archivo CSV desde el Stage hacia la tabla bruta.

```sql
COPY INTO video_rankings_raw
FROM '@my_stage/Most popular 1000 Youtube videos.csv'
FILE_FORMAT = my_csv_format;
```

---

### Paso 6: Creación de la tabla transformada `video_rankings`

Se transforman los datos: se eliminan las comas de los valores numéricos y se realiza el casteo adecuado.

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

---

### Paso 7: Verificación de los datos cargados

Se realiza una consulta de validación de los datos procesados.

```sql
SELECT * FROM video_rankings;
```

---

## Limpieza opcional

Una vez finalizada la carga de ambos archivos, es posible liberar espacio en el Stage:

```sql
REMOVE @my_stage;
```
