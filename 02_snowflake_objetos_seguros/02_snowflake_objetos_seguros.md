# Creación y Gestión de Objetos Seguros en Snowflake

## Tabla de Contenidos

- [Creación y Gestión de Objetos Seguros en Snowflake](#creación-y-gestión-de-objetos-seguros-en-snowflake)
  - [Tabla de Contenidos](#tabla-de-contenidos)
  - [Bases de Datos](#bases-de-datos)
  - [Esquemas](#esquemas)
    - [INFORMATION\_SCHEMA](#information_schema)
    - [ACCOUNT\_USAGE](#account_usage)
    - [Diferencias clave entre INFORMATION\_SCHEMA y ACCOUNT\_USAGE](#diferencias-clave-entre-information_schema-y-account_usage)
    - [Jerarquía de Objetos en Esquemas](#jerarquía-de-objetos-en-esquemas)
  - [Tablas](#tablas)
    - [Tabla Permanente](#tabla-permanente)
    - [Tabla Temporal](#tabla-temporal)
    - [Tabla Transitoria](#tabla-transitoria)
    - [Comparación rápida](#comparación-rápida)
    - [Caso práctico: Creación de diferentes tipos de tablas en Snowflake](#caso-práctico-creación-de-diferentes-tipos-de-tablas-en-snowflake)
      - [Objetivos del caso práctico](#objetivos-del-caso-práctico)
  - [Vistas](#vistas)
    - [Vistas Estándar](#vistas-estándar)
    - [Vistas Materializadas](#vistas-materializadas)
    - [¿Qué significa que una vista sea `SECURE`?](#qué-significa-que-una-vista-sea-secure)
    - [Comparativa general](#comparativa-general)
  - [FILE FORMAT en Snowflake](#file-format-en-snowflake)
    - [Ejemplo de creación de un `FILE FORMAT` CSV](#ejemplo-de-creación-de-un-file-format-csv)
    - [Tipos de formatos soportados](#tipos-de-formatos-soportados)
    - [Opciones comunes por tipo](#opciones-comunes-por-tipo)
    - [Comandos útiles](#comandos-útiles)
  - [Stages en Snowflake](#stages-en-snowflake)
    - [Internal Stages (Internos)](#internal-stages-internos)
    - [External Stages (Externos)](#external-stages-externos)
    - [Ejemplos de uso](#ejemplos-de-uso)
  - [Tablas Externas](#tablas-externas)
    - [Características clave](#características-clave)
    - [Limitaciones](#limitaciones)
  - [Caso práctico: Uso de tablas externas y vistas materializadas](#caso-práctico-uso-de-tablas-externas-y-vistas-materializadas)
      - [Objetivos del caso práctico](#objetivos-del-caso-práctico-1)
  - [COPY INTO](#copy-into)
      - [Sintaxis básica](#sintaxis-básica)
      - [Opciones comunes](#opciones-comunes)
    - [Caso práctico: Carga de datos con COPY INTO desde Internal Stage](#caso-práctico-carga-de-datos-con-copy-into-desde-internal-stage)
    - [Caso práctico: Carga de datos con COPY INTO desde External Stage](#caso-práctico-carga-de-datos-con-copy-into-desde-external-stage)
  - [Procedimientos Almacenados y UDFs](#procedimientos-almacenados-y-udfs)
    - [Funciones Definidas por el Usuario (UDF)](#funciones-definidas-por-el-usuario-udf)
    - [UDTF (User-Defined Table Function)](#udtf-user-defined-table-function)
      - [UDTF: Ejemplo de análisis de cesta de productos (Market Basket Analysis)](#udtf-ejemplo-de-análisis-de-cesta-de-productos-market-basket-analysis)
    - [Procedimientos Almacenados](#procedimientos-almacenados)
  - [Pipes, Streams, Sequences y Tasks](#pipes-streams-sequences-y-tasks)
    - [**Pipes** (Carga continua con Snowpipe)](#pipes-carga-continua-con-snowpipe)
    - [**Streams** (Change Data Capture - CDC)](#streams-change-data-capture---cdc)
    - [**Sequences** (Generación de identificadores únicos)](#sequences-generación-de-identificadores-únicos)
    - [**Tasks** (Automatización programada)](#tasks-automatización-programada)
  - [Test de Conocimientos](#test-de-conocimientos)



---

En Snowflake se dispone de una serie de objetos para gestionar y organizar datos de manera eficiente. Estos objetos incluyen bases de datos, esquemas, tablas, vistas, stages, procedimientos almacenados, funciones definidas por el usuario (UDFs), pipes, streams, sequences y tasks. Cada uno de estos objetos tiene un propósito específico y se utiliza en diferentes escenarios para satisfacer las necesidades de almacenamiento, procesamiento y análisis de datos.

A continuación, exploraremos cómo crear y gestionar estos objetos, destacando las mejores prácticas y ejemplos prácticos para su implementación.

## Bases de Datos

En Snowflake, una **base de datos** es un contenedor lógico que agrupa esquemas, tablas y otros objetos. Algunos de los comandos más importantes son:

```sql
-- Crear una base de datos
CREATE DATABASE mi_base_datos;

-- Crear una base de datos con un comentario
CREATE DATABASE mi_base_datos COMMENT = 'Base de datos para pruebas';

-- Se muestran las bases de datos creadas y disponibles para el usuario.
SHOW DATABASES;

-- Borrar base de datos
DROP DATABASE [IF EXISTS] mi_base_datos;

-- Renombrar una base de datos
ALTER DATABASE mi_base_datos RENAME TO mi_base_datos_nuevo;

-- Mostrar información detallada de una base de datos
DESCRIBE DATABASE mi_base_datos;

-- Cambiar a una base de datos específica
USE DATABASE mi_base_datos;
```

> ℹ️ Las bases de datos se usan para organizar datos y aplicar políticas de acceso.

---

## Esquemas

Los **esquemas** son subdivisiones dentro de una base de datos. Sirven para agrupar objetos como tablas, vistas, procedimientos, etc.

```sql
-- Crear esquema
CREATE SCHEMA demo_db.mi_esquema;

-- Cambiar a un esquema específico
USE SCHEMA demo_db.mi_esquema;
```

---

### INFORMATION_SCHEMA

`INFORMATION_SCHEMA` es un **esquema especial** que Snowflake incluye dentro de **cada base de datos**. Contiene **vistas de solo lectura** que exponen metadatos sobre los objetos existentes en esa base, como tablas, vistas, columnas, funciones, procedimientos, etc.

Es útil para:

- **Auditar** objetos de una base de datos específica  
- **Documentar** estructuras de datos  
- **Verificar propiedades** como tipos de columnas, claves, propietarios, etc.

```sql
-- Ver todas las tablas de una base de datos
SELECT * 
FROM demo_db.INFORMATION_SCHEMA.TABLES
WHERE TABLE_TYPE = 'BASE TABLE';
```

```sql
-- Ver todas las columnas de una tabla específica
SELECT * 
FROM demo_db.INFORMATION_SCHEMA.COLUMNS 
WHERE TABLE_NAME = 'EMPLEADOS';
```

```sql
-- Ver todos los procedimientos almacenados
SELECT * 
FROM demo_db.INFORMATION_SCHEMA.PROCEDURES;
```

> ℹ️ Puedes acceder al `INFORMATION_SCHEMA` de cada base como:  
> `[nombre_db].INFORMATION_SCHEMA.[vista]`

---

### ACCOUNT_USAGE

`SNOWFLAKE.ACCOUNT_USAGE` es una **vista global** a nivel de cuenta que forma parte del esquema `SNOWFLAKE`, accesible solo para roles con privilegios elevados (como `ACCOUNTADMIN`).

Está orientada a:

- **Auditoría de actividad** (quién hizo qué y cuándo)
- **Seguimiento de consumo y costos**
- **Gobernanza de datos**
- **Seguridad** (roles, permisos, acceso a objetos)
- **Monitoreo de usuarios y sesiones**

```sql
-- Ver todos los usuarios creados en la cuenta
SELECT * 
FROM SNOWFLAKE.ACCOUNT_USAGE.USERS;
```

```sql
-- Ver historial de consultas ejecutadas (últimos días)
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE START_TIME >= DATEADD('day', -7, CURRENT_TIMESTAMP());
```

```sql
-- Ver actividad de uso de warehouses
SELECT *
FROM SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY
WHERE START_TIME >= DATEADD('hour', -24, CURRENT_TIMESTAMP());
```

> ℹ️ Estos datos tienen un pequeño **retraso (lag)**, que puede ir de unos minutos a varias horas, según la tabla.

---

### Diferencias clave entre INFORMATION_SCHEMA y ACCOUNT_USAGE

| Característica              | `INFORMATION_SCHEMA`                          | `ACCOUNT_USAGE`                                       |
|-----------------------------|-----------------------------------------------|--------------------------------------------------------|
| Nivel de acceso             | Base de datos específica                      | Toda la cuenta (global)                                |
| Propósito                   | Inspección de objetos locales                 | Auditoría y monitoreo completo                         |
| Actualización               | En tiempo real                                | Con retardo (varía según la vista)                     |
| Requiere rol avanzado       | No necesariamente                             | Sí (por ejemplo `ACCOUNTADMIN`)                        |
| Datos históricos            | No                                             | Sí (consultas, acceso, uso de créditos, etc.)          |
| Visibilidad                 | Solo objetos de la base consultada            | Objetos de todas las bases/esquemas de la cuenta       |

---

### Jerarquía de Objetos en Esquemas

![Jerarquía de Objetos en Snowflake](.\resources\02_img1.png)

> ℹ️ Este diagrama ilustra cómo los objetos como tablas, vistas y procedimientos se organizan dentro de un esquema.


---

## Tablas

En Snowflake, las **tablas** son objetos fundamentales para almacenar datos estructurados. Existen **tres tipos principales** de tablas según su persistencia y comportamiento frente a mecanismos como Time Travel o Fail-safe:

| Tipo de tabla     | Persistencia | Visibilidad | Fail-safe | Uso común |
|-------------------|--------------|-------------|-----------|-----------|
| **Permanente**     | Persistente  | Global      | Sí        | Producción, almacenamiento duradero |
| **Temporal**       | Temporal     | Solo sesión | No        | Datos temporales de una sesión |
| **Transitoria**    | Persistente  | Global      | No        | Datos intermedios, staging, procesamiento |

---

### Tabla Permanente

Las **tablas permanentes** son el tipo por defecto. Están sujetas tanto a **Time Travel** como a **Fail-safe**, lo que permite la recuperación de datos accidentalmente eliminados o modificados.

```sql
CREATE TABLE empleados (
  id INT,
  evento STRING
);
```

- **Persisten** en el tiempo hasta que se eliminan explícitamente.
- **Soportan** mecanismos de recuperación (`Time Travel` y `Fail-safe`).
- Ideales para datos críticos y a largo plazo.
- Pueden estar asociadas a esquemas versionados o compartidos.

---

### Tabla Temporal

Las **tablas temporales** solo existen **durante la sesión actual** del usuario. Al finalizar la sesión, Snowflake **elimina automáticamente** la tabla y sus datos.

```sql
CREATE TEMPORARY TABLE temp_data (
  col1 INT
);
```

- No son visibles fuera de la sesión en la que fueron creadas.
- No se pueden compartir con otros usuarios ni roles.
- No admiten **Fail-safe** ni **Time Travel**.
- Útiles para operaciones intermedias, cálculos, pruebas, joins temporales.

> ℹ️  **Importante**: Aunque se puede usar para staging, si necesitas que los datos sobrevivan a la sesión, opta por tablas transitorias.

---

### Tabla Transitoria

Las **tablas transitorias** son persistentes como las permanentes, pero **sin protección de Fail-safe**, lo que significa que consumen menos almacenamiento y costos.

```sql
CREATE TRANSIENT TABLE log_data (
  id INT,
  evento STRING
);
```

- Persisten hasta que son eliminadas manualmente.
- Admiten **Time Travel** (por defecto, hasta 1 día).
- **No cuentan con Fail-safe**, por lo que no se pueden recuperar tras el período de retención.
- Recomendadas para cargas ETL, staging, almacenamiento temporal o auditorías internas con retención controlada.

> ℹ️ **Consejo**: Usa tablas transitorias si quieres reducir costos de almacenamiento y puedes prescindir de la recuperación ante desastres.

---

### Comparación rápida

| Característica               | Permanente | Transitoria | Temporal |
|-----------------------------|------------|-------------|----------|
| Persistencia post-sesión    | ✅         | ✅          | ❌       |
| Time Travel                 | ✅         | ✅ (limitado)| ❌       |
| Fail-safe                   | ✅         | ❌          | ❌       |
| Costo de almacenamiento     | Alto       | Bajo        | N/A      |
| Uso típico                  | Producción | ETL/Staging | Cálculos rápidos |


### Caso práctico: Creación de diferentes tipos de tablas en Snowflake  

En este caso práctico, se crearán los diferentes tipos de tablas disponibles en Snowflake: tablas permanentes, temporales y transitorias. Cada una tiene características únicas y casos de uso específicos. Aprenderás a crear cada tipo utilizando SQL y a comprender cuándo conviene usar una u otra.

#### Objetivos del caso práctico  
- Aprender a crear tablas permanentes, temporales y transitorias en Snowflake.  
- Comprender las diferencias clave entre los distintos tipos de tablas.  
- Ejecutar comandos SQL para manipular datos dentro de cada tipo de tabla.  
- Observar el comportamiento de persistencia de datos tras cerrar sesión o terminar la sesión de trabajo.  

Para comenzar, sigue las instrucciones detalladas en el siguiente documento:  

📄 **[Caso Práctico: Creación de diferentes tipos de tablas en Snowflake](02_creacion_tipos_tablas.md)**  

✅ **Al completar este caso práctico, estarás listo para crear y utilizar distintos tipos de tablas en tus proyectos de Snowflake, seleccionando la opción más adecuada según las necesidades de persistencia, visibilidad y rendimiento.**

---

## Vistas

En Snowflake, existen **dos tipos principales de vistas**:

1. **Vistas Estándar (`VIEW`)**  
2. **Vistas Materializadas (`MATERIALIZED VIEW`)**

Ambos tipos pueden, opcionalmente, ser creados como **vistas seguras** usando la palabra clave `SECURE`, lo que impide que se exponga su lógica SQL a usuarios no autorizados, incluso si tienen privilegios sobre la vista.

---

### Vistas Estándar

Una vista estándar es una consulta SQL almacenada que no guarda datos, sino que se ejecuta en tiempo real cada vez que se consulta. Es útil para encapsular lógica compleja, restringir columnas sensibles o crear capas de acceso a los datos.

```sql
-- Vista estándar
CREATE VIEW vista_empleados AS
SELECT * FROM empleados WHERE activo = TRUE;
```

> ℹ️ Esta vista no almacena datos y su lógica es visible para los roles con acceso a la misma.

---

### Vistas Materializadas

Una **vista materializada** precalcula y almacena físicamente el resultado de una consulta, mejorando significativamente el rendimiento cuando se trata de datos complejos o muy consultados.

```sql
-- Vista materializada
CREATE MATERIALIZED VIEW vista_ventas_agregadas AS
SELECT cliente_id, SUM(monto) AS total
FROM ventas
GROUP BY cliente_id;
```

> ℹ️ Snowflake actualiza automáticamente las vistas materializadas en segundo plano cuando los datos subyacentes cambian (near real-time), y cobra por su almacenamiento y mantenimiento.

---

### ¿Qué significa que una vista sea `SECURE`?

Cuando una vista se define como `SECURE`, su lógica interna queda oculta para los usuarios, incluso si tienen privilegios sobre la vista. Esto impide que puedan ver el SQL subyacente mediante introspección (`SHOW VIEWS`, herramientas de BI, exploradores de metadatos, etc.).

Esto aplica tanto a vistas estándar como a materializadas:

```sql
-- Vista estándar segura
CREATE SECURE VIEW vista_segura AS
SELECT id, nombre FROM empleados;

-- Vista materializada segura
CREATE SECURE MATERIALIZED VIEW vista_segura_ventas AS
SELECT cliente_id, SUM(monto) AS total
FROM ventas
GROUP BY cliente_id;
```

✅ **Ventajas de las vistas seguras:**

- Ideal para **entornos multiusuario** o **data sharing**.
- Protege la lógica del negocio y reglas de transformación.
- Se puede auditar fácilmente y es compatible con políticas de gobernanza.

---

### Comparativa general

| Tipo de Vista               | Almacena resultados | Oculta definición (`SECURE`) | Aumenta rendimiento | Usa almacenamiento |
|-----------------------------|---------------------|------------------------------|---------------------|--------------------|
| Estándar                    | ❌                  | Opcional                     | ❌                  | ❌                 |
| Estándar segura             | ❌                  | ✅                            | ❌                  | ❌                 |
| Materializada               | ✅                  | Opcional                     | ✅                  | ✅                 |
| Materializada segura        | ✅                  | ✅                            | ✅                  | ✅                 |


---

## FILE FORMAT en Snowflake

En Snowflake, un **`FILE FORMAT`** define cómo deben interpretarse los archivos de datos durante operaciones de carga (`COPY INTO`), exportación o consultas externas. Es un objeto reutilizable que facilita el manejo de formatos como **CSV, JSON, Avro, ORC, Parquet** y XML.

> ℹ️ Es recomendable definir un `FILE FORMAT` cuando se trabaja con Stages o cargas automatizadas, para mantener consistencia en la interpretación del contenido.


### Ejemplo de creación de un `FILE FORMAT` CSV

```sql
CREATE OR REPLACE FILE FORMAT my_csv_format
TYPE = 'CSV'
FIELD_OPTIONALLY_ENCLOSED_BY = '"'
SKIP_HEADER = 1
EMPTY_FIELD_AS_NULL = TRUE;
```

Este ejemplo define un formato compatible con archivos CSV que:
- Interpreta campos encerrados opcionalmente por comillas (`"`).
- Omite la primera fila (usualmente cabecera).
- Considera los campos vacíos como `NULL` en lugar de cadenas vacías.


### Tipos de formatos soportados

| Tipo        | Descripción                                |
|-------------|--------------------------------------------|
| `CSV`       | Formato separado por comas u otro delimitador |
| `JSON`      | Archivos con estructura en formato JSON    |
| `AVRO`      | Formato binario utilizado en Big Data      |
| `ORC`       | Optimized Row Columnar, usado en Hadoop    |
| `PARQUET`   | Muy eficiente para análisis columnar       |
| `XML`       | Documentos estructurados en XML            |



### Opciones comunes por tipo

| Opción                      | CSV     | JSON    | PARQUET | XML     |
|----------------------------|---------|---------|---------|---------|
| `FIELD_DELIMITER`          | ✅      | ❌      | ❌      | ❌      |
| `SKIP_HEADER`              | ✅      | ❌      | ❌      | ❌      |
| `FILE_EXTENSION`           | ✅      | ✅      | ✅      | ✅      |
| `TRIM_SPACE`               | ✅      | ❌      | ❌      | ❌      |
| `NULL_IF`                  | ✅      | ❌      | ❌      | ❌      |
| `STRIP_NULL_VALUES`        | ❌      | ✅      | ❌      | ✅      |
| `RECORD_DELIMITER`         | ✅      | ❌      | ❌      | ❌      |

Consulta todos los parámetros disponibles según el tipo en la [documentación oficial de Snowflake](https://docs.snowflake.com/en/sql-reference/sql/create-file-format.html).


### Comandos útiles

```sql
-- Ver formatos de archivo disponibles
SHOW FILE FORMATS;

-- Describir un formato existente
DESC FILE FORMAT my_csv_format;

-- Eliminar un formato
DROP FILE FORMAT my_csv_format;
```

---

✅ **Buenas prácticas**
- Crea `FILE FORMAT` reutilizables por proyecto o tipo de dato.
- Usa nombres descriptivos para cada formato.
- Evita formatos con `SKIP_HEADER = 0` si los archivos tienen encabezado.
- Usa `TRY_CAST` o `TRY_TO_DATE` al aplicar `COPY INTO` para prevenir errores en la carga si los datos no coinciden perfectamente.

---

## Stages en Snowflake

Un **Stage** es un área de almacenamiento utilizada para **almacenar archivos antes o después de cargarlos en Snowflake**. Los Stages permiten mover datos desde tu entorno local o desde la nube, de forma segura y controlada.

Snowflake clasifica los stages en **dos tipos principales**:

### Internal Stages (Internos)

Son gestionados **completamente por Snowflake** y no requieren configuración de almacenamiento externa. Dentro de los internos existen tres subtipos:

| Subtipo              | Descripción |
|----------------------|-------------|
| **User Stage**       | Asociado a cada usuario Snowflake. Accesible vía `@~` |
| **Table Stage**      | Asociado automáticamente a cada tabla. Ideal para cargas rápidas o temporales. Se accede con `@%nombre_tabla` |
| **Named Stage**      | Stage interno creado explícitamente por el usuario. Reutilizable en múltiples cargas. Se accede con `@nombre_stage` |

---

### External Stages (Externos)

Permiten conectar Snowflake con servicios de almacenamiento externo como:

- Amazon S3  
- Azure Blob Storage  
- Google Cloud Storage  

Requieren configuración adicional (credenciales, integración segura o almacenamiento externo).

```sql
CREATE STAGE s3_stage
  URL = 's3://mi-bucket/data/'
  STORAGE_INTEGRATION = mi_integracion;
```

---

### Ejemplos de uso

```sql
-- User Stage (Stage personal del usuario actual)
PUT file://local/archivo.csv @~;

-- Table Stage (para tabla llamada clientes)
PUT file://clientes.csv @%clientes;

-- Named Stage
CREATE OR REPLACE STAGE stage_ventas;
PUT file://ventas.csv @stage_ventas;

-- Ver archivos en un stage
LIST @stage_ventas;

-- Cargar/copiar datos desde stage a una tabla
COPY INTO empleados
FROM @stage_ventas/empleados.csv
FILE_FORMAT = (FORMAT_NAME = 'formato_csv');

-- Descargar archivo desde tabla a un stage
COPY INTO @stage_ventas
FROM (SELECT * FROM ventas WHERE region = 'Sur')
FILE_FORMAT = (TYPE = 'CSV');
```

> ℹ️ **Consejo**: Usa **Named Stages** para mayor control y reutilización, **Table Stages** para cargas rápidas y **External Stages** si trabajas con almacenamiento en la nube.

---

## Tablas Externas

Las **tablas externas** permiten consultar datos almacenados fuera de Snowflake (como en Amazon S3, Azure Blob Storage o Google Cloud Storage), **sin necesidad de cargarlos físicamente**. Son ideales cuando se quiere analizar archivos directamente desde un data lake sin duplicar datos.

Para crear una tabla externa, se requiere:

1. Un **Stage externo** que apunte al almacenamiento (ya definido anteriormente).
2. Un **formato de archivo** (`FILE FORMAT`) que describa cómo interpretar los datos.
3. La definición de la tabla externa, incluyendo expresiones para extraer valores de los archivos.

```sql
-- Crear formato CSV
CREATE OR REPLACE FILE FORMAT formato_csv
  TYPE = 'CSV'
  FIELD_OPTIONALLY_ENCLOSED_BY = '"'
  SKIP_HEADER = 1;

-- Crear tabla externa apuntando al Stage previamente creado
CREATE OR REPLACE EXTERNAL TABLE clientes_ext (
  id INT AS (VALUE:c1::INT),
  nombre STRING AS (VALUE:c2::STRING),
  pais STRING AS (VALUE:c3::STRING)
)
WITH LOCATION = @ext_stage_clientes
FILE_FORMAT = formato_csv
AUTO_REFRESH = true;
```
> ℹ️ **Nota**: `VALUE:c1` hace referencia a la **primera columna** del archivo externo (sin nombre), y se convierte al tipo deseado (`::INT` en este caso).  
> En archivos sin encabezado, se utiliza `c1`, `c2`, `c3`, etc., para acceder a cada campo por posición.  
> Si el archivo tuviera encabezados y se utilizara `HEADER = TRUE`, se podrían usar los nombres de las columnas directamente.

> ℹ️ **Nota**: El Stage **@ext_stage_clientes** debe haber sido creado con anterioridad y debe apuntar a una ubicación válida en S3, Azure o GCP.

### Características clave
- Los datos no se almacenan dentro de Snowflake.
- Snowflake actúa como motor de consulta sobre los archivos externos.
- Se pueden usar funciones de particionado, y AUTO_REFRESH para mantener metadatos actualizados.
- Ideal para entornos tipo data lake o para exploración rápida de datos sin necesidad de cargas previas.

### Limitaciones
- Solo admite operaciones de lectura (SELECT).
- No es posible hacer INSERT, UPDATE o DELETE.
- El rendimiento depende del formato y localización de los archivos.

> ℹ️ Recomendación: Utilizar tablas externas para consultar grandes volúmenes de datos en bruto directamente desde almacenamiento en la nube, sin incurrir en costes de duplicación de datos ni almacenamiento interno en Snowflake.

## Caso práctico: Uso de tablas externas y vistas materializadas

En este caso práctico se trabajará con archivos almacenados en un Data Lake (S3) para crear una tabla externa en Snowflake. A partir de esta estructura, se realizarán operaciones de carga, análisis y optimización mediante la creación de vistas materializadas sobre tablas tanto internas como externas.

#### Objetivos del caso práctico
- Crear una tabla externa a partir de un stage externo con archivos CSV.
- Cargar los datos en una tabla interna.
- Crear una vista materializada para optimizar consultas sobre la tabla interna.
- Crear una vista materializada directamente sobre la tabla externa.
- Comparar el rendimiento de las distintas estructuras.

Para comenzar, sigue las instrucciones detalladas en el siguiente documento:

📄 **[Caso Práctico: Uso de tablas externas y vistas materializadas](02_external_tables_y_materialized_views.md)**


✅ **Al completar este caso práctico, estarás listo para integrar datos externos y aplicar vistas materializadas para acelerar consultas en Snowflake de forma eficiente.**

---


## COPY INTO

El comando `COPY INTO` permite cargar datos de forma eficiente desde Stages (internos o externos) hacia tablas en Snowflake. Es una de las herramientas clave para implementar pipelines de datos o procesos ETL.

#### Sintaxis básica

```sql
COPY INTO <nombre_tabla>
FROM <ubicacion_stage>
FILE_FORMAT = (FORMAT_NAME = '<formato>');
```

#### Opciones comunes

- `ON_ERROR`: cómo manejar errores de carga (`ABORT_STATEMENT`, `CONTINUE`, `SKIP_FILE`).
- `PATTERN`: permite aplicar una expresión regular para filtrar archivos específicos dentro del Stage.
- `VALIDATION_MODE`: para validar los archivos sin cargarlos (últil para auditoría).

```sql
-- Validar archivos antes de cargarlos
COPY INTO mi_tabla
FROM @mi_stage
FILE_FORMAT = (FORMAT_NAME = 'mi_formato')
VALIDATION_MODE = RETURN_ALL_ERRORS;
```

### Caso práctico: Carga de datos con COPY INTO desde Internal Stage

En este caso práctico se aplican los conocimientos aprendidos sobre el uso del comando `COPY INTO` y los Stages internos de Snowflake. Se trabajarán dos ejercicios reales de carga y transformación de datos a partir de archivos CSV incluidos en los recursos del curso.

Para comenzar, accede al documento detallado:

📄 **[Caso Práctico: Carga de datos con COPY INTO desde Internal Stage](02_copy_into_internal_stage.md)**

✅ **Al completar este caso práctico, estarás preparado para realizar cargas controladas de datos desde archivos locales en Snowflake, aplicando buenas prácticas de staging y transformación.**

### Caso práctico: Carga de datos con COPY INTO desde External Stage

En este caso práctico se aplican los conocimientos aprendidos sobre el uso del comando `COPY INTO` y los Stages externos de Snowflake. Se trabajará un ejercicio real de carga y transformación de datos a partir de archivos CSV incluidos en un bucket S3 de AWS.

Para comenzar, accede al documento detallado:

📄 **[Caso Práctico: Carga de datos con COPY INTO desde External Stage](02_copy_into_external_stage.md)**

✅ **Al completar este caso práctico, estarás preparado para realizar cargas controladas de datos desde archivos remotos en Snowflake, aplicando buenas prácticas de staging y transformación.**


---

## Procedimientos Almacenados y UDFs

Snowflake permite extender sus capacidades SQL mediante objetos programables como **procedimientos almacenados** y **funciones definidas por el usuario (UDFs)**. Estos objetos encapsulan lógica reutilizable que se puede invocar en consultas, flujos ETL o tareas automáticas. Son ideales para ejecutar operaciones complejas, personalizar transformaciones o implementar lógica condicional, iterativa o dinámica directamente dentro del entorno de Snowflake, sin necesidad de herramientas externas.

### Funciones Definidas por el Usuario (UDF)

Permiten encapsular lógica SQL o código en JavaScript.

```sql
-- Crear o reemplazar una función llamada 'suma' que recibe dos enteros
CREATE OR REPLACE FUNCTION suma(a INT, b INT)
  RETURNS INT
  LANGUAGE SQL
  AS 'a + b';

-- Llamada a la función con dos valores
SELECT suma(3, 7) AS resultado;
```

---

### UDTF (User-Defined Table Function)

Una **UDTF** es una función definida por el usuario que, en lugar de devolver un único valor escalar como una UDF tradicional, **retorna una tabla completa**. Es útil cuando necesitas generar **varias filas como salida** a partir de una sola fila de entrada, especialmente en operaciones como descomposición de arrays, generación de combinaciones, u operaciones analíticas avanzadas.

> ℹ️ Las UDTFs pueden escribirse en **SQL** o **JavaScript**, y permiten que el resultado sea tratado como una tabla dentro de una consulta.


#### UDTF: Ejemplo de análisis de cesta de productos (Market Basket Analysis)

```sql
-- Crear una UDTF que descompone un array de productos en filas individuales
CREATE OR REPLACE FUNCTION basket_analysis(items ARRAY)
RETURNS TABLE(item STRING)
AS
$$
  SELECT VALUE AS item 
  FROM TABLE(FLATTEN(INPUT => items))
$$;

-- Ejemplo de uso: devuelve cada producto como una fila
SELECT * 
FROM TABLE(basket_analysis(ARRAY_CONSTRUCT('Pan', 'Leche', 'Queso')));
```
El resultado sería el siguiente:

| item   |
|--------|
| Pan    |
| Leche  |
| Queso  |


> ℹ️ Este tipo de función es ideal para desglosar datos semiestructurados como arrays, listas de elementos, o estructuras anidadas, facilitando su análisis en Snowflake.

---

### Procedimientos Almacenados

Un **procedimiento almacenado** (stored procedure) en Snowflake encapsula un bloque de código que puede incluir **lógica condicional, bucles, control de flujo, asignaciones, operaciones DML**, etc. Es ideal para tareas repetitivas, cargas automatizadas, transformación de datos o acciones que no pueden resolverse con simples consultas SQL.

Los procedimientos pueden escribirse en **lenguaje SQL** o en **JavaScript** (más potente y flexible).

```sql
-- Crear o reemplazar un procedimiento llamado 'sp_saludo'
CREATE OR REPLACE PROCEDURE sp_saludo(nombre STRING)
RETURNS STRING
LANGUAGE SQL
AS
$$
  DECLARE mensaje STRING;
BEGIN
  LET mensaje = 'Hola ' || nombre || '!';
  RETURN mensaje;
END;
$$;

-- Llamar al procedimiento pasando el nombre como argumento
CALL sp_saludo('Arturo');
```
El resultado esperado es:
| SALUDO        |
|---------------|
| Hola Arturo!  |


---

## Pipes, Streams, Sequences y Tasks

Snowflake ofrece objetos programables que permiten automatizar flujos de datos, capturar cambios y ejecutar tareas de forma controlada. Estos elementos son clave para implementar pipelines de datos robustos y eficientes.


### **Pipes** (Carga continua con Snowpipe)

Un **Pipe** permite automatizar la carga de archivos desde un Stage hacia una tabla, utilizando internamente el comando `COPY INTO`. Es parte del servicio **Snowpipe**, diseñado para cargas **casi en tiempo real**.

```sql
-- Crear un pipe que cargue archivos CSV desde un Stage
CREATE OR REPLACE PIPE pipe_clientes
  AUTO_INGEST = TRUE
  AS
  COPY INTO clientes
  FROM @stage_clientes
  FILE_FORMAT = (TYPE = 'CSV');
```

> ℹ️ Si `AUTO_INGEST = TRUE`, necesitarás configurar notificaciones externas (ej. S3 + SQS en AWS).

Puedes consultar el estado de un pipe con:

```sql
SHOW PIPES;
```


### **Streams** (Change Data Capture - CDC)

Un **Stream** permite rastrear los cambios ocurridos en una tabla: **inserciones, actualizaciones y eliminaciones**. Es el mecanismo de **CDC (Change Data Capture)** de Snowflake, ideal para auditoría o cargas incrementales.

```sql
-- Crear un stream sobre la tabla 'empleados'
CREATE OR REPLACE STREAM mi_stream ON TABLE empleados;
```

```sql
-- Consultar los cambios desde la última vez que se leyó el stream
SELECT * FROM mi_stream;
```

El stream expone columnas especiales como:

- `_METADATA$ACTION`: `INSERT`, `DELETE`, `UPDATE`
- `_METADATA$ISUPDATE`: indica si es una actualización

> ℹ️ Los datos del stream representan **solo los cambios** desde la última lectura. Una vez leídos, el stream se "reinicia" y comenzará a registrar cambios nuevos.

Consulta detalles de los streams:

```sql
SHOW STREAMS;
```



### **Sequences** (Generación de identificadores únicos)

Una **Sequence** genera números secuenciales, típicamente usados para columnas ID, control de versiones o claves técnicas.

```sql
-- Crear una secuencia
CREATE SEQUENCE seq_id START = 1 INCREMENT = 1;
```

```sql
-- Usar la secuencia en una inserción
INSERT INTO empleados (id, nombre)
VALUES (seq_id.nextval, 'Arturo');
```

Puedes ver el próximo valor sin insertarlo con:

```sql
SELECT seq_id.nextval;
```

> ℹ️ Las secuencias son persistentes y no se reinician al reiniciar sesiones.



### **Tasks** (Automatización programada)

Los **Tasks** permiten programar la ejecución de SQL o procedimientos almacenados, en intervalos regulares o encadenados a otras tareas.

```sql
-- Crear una tarea diaria que ejecuta un procedimiento
CREATE OR REPLACE TASK tarea_diaria
  WAREHOUSE = mi_warehouse
  SCHEDULE = '1 DAY'
AS
  CALL sp_actualizar_estadisticas();
```

```sql
-- Activar la tarea
ALTER TASK tarea_diaria RESUME;
```

> ℹ️ Puedes usar `SCHEDULE = '5 MINUTE'`, `1 HOUR`, `1 WEEK`, etc., o usar `AFTER otra_tarea` para crear flujos secuenciales.

Consultar historial de ejecución:

```sql
SELECT * 
FROM INFORMATION_SCHEMA.TASK_HISTORY
WHERE NAME = 'TAREA_DIARIA';
```

Diagrama tipo "pipeline de carga" combinando Stage → Pipe → Stream → Procedure → Task:
![Pipeline de datos](.\resources\02_img2.png)

---


## Test de Conocimientos

1. ¿Qué tipo de tabla utiliza Fail-safe? *(Selección única)*  
- A) Permanente  
- B) Temporal  
- C) Transitoria  
- D) Ninguna de las anteriores  
<details><summary>Respuesta</summary>A) Permanente</details>

2. ¿Cuál es el propósito de `ACCOUNT_USAGE`? *(Selección única)*  
- A) Visualizar estructuras de tablas  
- B) Consultar usuarios, roles y eventos  
- C) Crear bases de datos  
- D) Restaurar datos eliminados  
<details><summary>Respuesta</summary>B) Consultar usuarios, roles y eventos</details>

3. ¿Qué objeto permite ejecutar código automáticamente con una programación? *(Selección única)*  
- A) Stream  
- B) Pipe  
- C) Task  
- D) Stage  
<details><summary>Respuesta</summary>C) Task</details>

4. ¿Cuál es el propósito de un `Stream` en Snowflake? *(Selección única)*  
- A) Detectar cambios en una tabla  
- B) Cargar datos desde archivo  
- C) Crear secuencias automáticas  
- D) Definir lógica almacenada  
<details><summary>Respuesta</summary>A) Detectar cambios en una tabla</details>

5. ¿Qué tipo de vista permite mejorar el rendimiento y almacena físicamente el resultado? *(Selección única)*  
- A) Vista estándar  
- B) Vista materializada  
- C) Vista temporal  
- D) Vista según tabla  
<details><summary>Respuesta</summary>B) Vista materializada</details>

6. ¿Cuál de los siguientes comandos sirve para cargar datos desde un stage a una tabla? *(Selección única)*  
- A) PUT  
- B) GET  
- C) SELECT INTO  
- D) COPY INTO  
<details><summary>Respuesta</summary>D) COPY INTO</details>

7. ¿Qué componente permite generar valores numéricos incrementales? *(Selección única)*  
- A) Stream  
- B) Pipe  
- C) Sequence  
- D) Task  
<details><summary>Respuesta</summary>C) Sequence</details>

8. ¿Qué ventaja ofrece una vista `SECURE`? *(Selección única)*  
- A) Oculta la lógica SQL a usuarios no autorizados  
- B) Mejora el rendimiento  
- C) Permite Time Travel  
- D) Evita bloqueos por concurrencia  
<details><summary>Respuesta</summary>A) Oculta la lógica SQL a usuarios no autorizados</details>

9. ¿Qué stage se accede con `@~`? *(Selección única)*  
- A) Table Stage  
- B) User Stage  
- C) External Stage  
- D) Named Stage  
<details><summary>Respuesta</summary>B) User Stage</details>

10. ¿Qué comandos permiten cambiar el nombre de una base de datos? *(Selección múltiple)*  
- A) RENAME TO  
- B) UPDATE DATABASE  
- C) ALTER DATABASE  
- D) SET DATABASE  
<details><summary>Respuesta</summary>C) ALTER DATABASE y A) RENAME TO</details>

11. ¿Cuál es la diferencia clave entre `INFORMATION_SCHEMA` y `ACCOUNT_USAGE`? *(Selección única)*  
- A) Ambos son iguales  
- B) `ACCOUNT_USAGE` tiene más permisos por defecto  
- C) `INFORMATION_SCHEMA` tiene información histórica  
- D) `INFORMATION_SCHEMA` es en tiempo real, `ACCOUNT_USAGE` tiene retardo  
<details><summary>Respuesta</summary>D) INFORMATION_SCHEMA es en tiempo real, ACCOUNT_USAGE tiene retardo</details>

12. ¿Qué tipo de tabla se elimina automáticamente al cerrar sesión? *(Selección única)*  
- A) Permanente  
- B) Temporal  
- C) Transitoria  
- D) Materializada  
<details><summary>Respuesta</summary>B) Temporal</details>

13. ¿Qué objeto encapsula lógica con control de flujo y puede retornar valores? *(Selección única)*  
- A) View  
- B) Sequence  
- C) Procedimiento almacenado  
- D) Task  
<details><summary>Respuesta</summary>C) Procedimiento almacenado</details>

14. ¿Qué sintaxis se usa para crear una vista materializada segura? *(Selección única)*  
- A) CREATE VIEW ... SECURE  
- B) CREATE SECURE VIEW ... MATERIALIZED  
- C) CREATE MATERIALIZED VIEW ... SECURE  
- D) CREATE SECURE MATERIALIZED VIEW  
<details><summary>Respuesta</summary>D) CREATE SECURE MATERIALIZED VIEW</details>

15. ¿Cómo se consulta el historial de ejecución de tareas? *(Selección única)*  
- A) SHOW TASKS  
- B) INFORMATION_SCHEMA.TASK_HISTORY  
- C) ACCOUNT_USAGE.RUN_HISTORY  
- D) DESCRIBE TASK  
<details><summary>Respuesta</summary>B) INFORMATION_SCHEMA.TASK_HISTORY</details>

16. ¿Qué comando permite listar las tablas existentes en un esquema específico? *(Selección única)*  
- A) SHOW TABLES  
- B) DESCRIBE SCHEMA  
- C) SELECT * FROM TABLES  
- D) LIST OBJECTS  
<details><summary>Respuesta</summary>A) SHOW TABLES</details>

17. ¿Cuál es la palabra clave para suspender automáticamente un warehouse? *(Selección única)*  
- A) SUSPEND_WHEN_IDLE  
- B) IDLE_TIMEOUT  
- C) AUTO_RESUME  
- D) AUTO_SUSPEND  
<details><summary>Respuesta</summary>D) AUTO_SUSPEND</details>

18. ¿Qué tipo de función devuelve una tabla como resultado? *(Selección única)*  
- A) UDF escalar  
- B) UDTF  
- C) SECURE FUNCTION  
- D) MATERIALIZED FUNCTION  
<details><summary>Respuesta</summary>B) UDTF</details>

19. ¿Qué comando crea una secuencia que empieza en 1 y aumenta de 1 en 1? *(Selección única)*  
- A) CREATE SEQUENCE ... START = 1 INCREMENT = 1  
- B) CREATE SEQUENCE STARTING 1 BY 1  
- C) NEW SEQUENCE 1 STEP 1  
- D) GENERATE SEQUENCE  
<details><summary>Respuesta</summary>A) CREATE SEQUENCE ... START = 1 INCREMENT = 1</details>

20. ¿Qué objeto permite cargar datos en tiempo real desde un stage con notificación automática? *(Selección única)*  
- A) Task  
- B) Stream  
- C) Pipe con AUTO_INGEST  
- D) External Stage  
<details><summary>Respuesta</summary>C) Pipe con AUTO_INGEST</details>

