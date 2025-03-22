# Caso Práctico: Conexión de Snowflake con Visual Studio Code

Este caso práctico te guiará en la instalación y configuración de la extensión de **Snowflake** en **Visual Studio Code**, permitiéndote conectarte a tu cuenta de Snowflake y ejecutar consultas SQL directamente desde el editor.

---

## Parte 1: Configuración de la Extensión de Snowflake en VS Code

### Paso 1: Descargar e Instalar Visual Studio Code

1. Ve al sitio oficial de Visual Studio Code y descarga la versión más reciente:
   - [Descargar VS Code](https://code.visualstudio.com/download)
2. Instala el software en tu sistema operativo siguiendo las instrucciones del instalador.

---

### Paso 2: Instalar la Extensión de Snowflake en VS Code

1. Abre **Visual Studio Code**.
2. Dirígete a la sección de **Extensiones** (`Ctrl+Shift+X` o `Cmd+Shift+X` en Mac).
3. En la barra de búsqueda, escribe **Snowflake**.
4. Selecciona la extensión **"Snowflake"** (publicada por Snowflake) y haz clic en **Instalar**.

---

### Paso 3: Configurar la Conexión con Snowflake

#### Obtener el identificador de la cuenta en Snowflake

1. Inicia sesión en **Snowflake** a través del navegador.
2. En la inferior izquierda, haz clic sobre tu perfil y sitúa el cursor sobre "Account".
3. Selecciona "View account details".
3. Encuentra el nombre de tu cuenta de Snowflake donde se indique "Account Name" (ejemplo: `NAB28306`) y copialo para tenerlo siempre a mano.
4. Realiza lo mismo con el valor **Account/Server URL**.

#### Añadir la cuenta en VS Code

1. En VS Code, abre la sección de **Snowflake** en la barra lateral.
2. Haz clic en **Agregar cuenta**.
3. Pega el **identificador de la cuenta** (Account/Server URL que hemos copiado anteriormente) en el campo correspondiente.
4. Configura los datos de acceso:
   - **Método de autenticación:** `Username/Password`
   - **Usuario:** *(Tu usuario de Snowflake)*
   - **Contraseña:** *(Tu contraseña de Snowflake)*
5. Haz clic en **Sign in** para autenticarte.

---

### Paso 4: Ejecutar una Consulta de Ejemplo

1. Abre una nueva pestaña en VS Code.
1. En la sección **Object Explorer** del menu lateral, selecciona el icono para añadir una **New Snowflake SQL File**.
2. Escribe la siguiente consulta SQL:

   ```sql
   SELECT * FROM SNOWFLAKE_SAMPLE_DATA.TPCDS_SF100TCL.CALL_CENTER;
   ```

3. Asegúrate de que estás conectado a Snowflake.
4. Haz clic en el botón de **Ejecutar consulta** (`▶`).
   
   Si no aparece este botón, podría ser que se necesite instalar la extensión **Python** en **Visual Studio Code**.
5. Verifica los resultados en la sección de **Query Results**.

---

## Parte 2: Instalación de Python y Dependencias para Snowflake

Además de la configuración de la extensión en VS Code, es necesario tener **Python** instalado junto con algunas librerías para interactuar con **Snowflake** mediante código.

### Paso 1: Validar la instalación de Python

Ejecuta el siguiente comando en la terminal de VS Code para verificar si tienes Python instalado:

```sh
python --version
```

Si no tienes Python instalado, instálalo con el siguiente comando:

```sh
winget install Python.Python.3.11
```

---

### Paso 2: Actualizar `pip`

Para asegurarte de tener la última versión de `pip`, ejecuta:

```sh
python -m pip install --upgrade pip
```

---

### Paso 3: Instalar las Librerías Necesarias

Instala las librerías requeridas con el siguiente comando:

```sh
pip install snowflake-snowpark-python pandas numpy altair matplotlib scipy
```

Si después de la instalación obtienes errores con **pandas**, ejecuta:

```sh
pip install --upgrade pandas pyarrow numpy
```

---

## Parte 3: Ejecutar Código desde VS Code

### Requisitos Previos

1. Copia el contenido del archivo **"1.3. Ejecución de código desde VS.txt"**.
2. Pega el contenido en un nuevo archivo con extensión `.py`.
3. Guarda el archivo y ejecútalo desde la terminal de **VS Code**.

---

### Código de Ejemplo en Python para Snowflake

```python
import snowflake.snowpark as snowpark
import pandas as pd
import numpy as np
import altair as alt
import matplotlib.pyplot as plt

# Parámetros de conexión a Snowflake (usa credenciales propias en un entorno seguro)
connection_parameters = {
    "account": "account",  # Nombre de la cuenta en Snowflake
    "user": "user",  # Usuario de Snowflake
    "password": "account",  # Contraseña (¡Nunca expongas credenciales en código real!)
    "warehouse": "COMPUTE_WH"  # Almacén de cómputo para ejecutar consultas
}

# Crear sesión en Snowflake para interactuar con la base de datos
session = snowpark.Session.builder.configs(connection_parameters).create()

# Crear base de datos y esquema si no existen
session.sql("CREATE DATABASE IF NOT EXISTS CURSO_SNOWFLAKE_DB").collect()
session.sql("USE DATABASE CURSO_SNOWFLAKE_DB").collect()
session.sql("CREATE SCHEMA IF NOT EXISTS CURSO_SNOWFLAKE_DB.CURSO_SNOWFLAKE_SC").collect()
session.sql("USE SCHEMA CURSO_SNOWFLAKE_DB.CURSO_SNOWFLAKE_SC").collect()

# Crear una tabla temporal con datos sintéticos simulando productos tecnológicos
query = """
CREATE OR REPLACE TEMPORARY TABLE PRODUCTS_TECH AS
SELECT 
    CONCAT('TECH-', UNIFORM(1000,9999, RANDOM())) AS PRODUCT_ID,  -- ID único para cada producto
    CASE 
        WHEN UNIFORM(0,5,RANDOM()) < 1 THEN 'Laptop'
        WHEN UNIFORM(0,5,RANDOM()) < 2 THEN 'Smartphone'
        WHEN UNIFORM(0,5,RANDOM()) < 3 THEN 'Tablet'
        WHEN UNIFORM(0,5,RANDOM()) < 4 THEN 'Monitor'
        ELSE 'Accesorio' 
    END AS CATEGORY,  -- Categoría del producto generada aleatoriamente
    ABS(NORMAL(750, 200::FLOAT, RANDOM())) AS PRICE,  -- Precio con distribución normal centrada en 750
    ABS(NORMAL(4, 1, RANDOM())) AS RATING  -- Puntuación con distribución normal centrada en 4
FROM TABLE(GENERATOR(ROWCOUNT => 500))  -- Generar 500 filas de datos
"""
session.sql(query).collect()

# Obtener los datos generados en un DataFrame de Pandas
# Esto permite analizar los datos sin necesidad de ejecutar consultas adicionales en Snowflake
df = session.sql("SELECT * FROM PRODUCTS_TECH").to_pandas()
print(df.head())  # Muestra las primeras filas de la tabla

# Análisis exploratorio de los datos generados
print("Precio promedio por categoría:")
print(df.groupby("CATEGORY")["PRICE"].mean())  # Promedio de precio por categoría

print("\nProducto más caro:")
print(df.loc[df["PRICE"].idxmax()])  # Producto con el precio más alto

print("\nNúmero de productos con rating mayor a 4:")
print(len(df[df["RATING"] > 4]))  # Contar cuántos productos tienen un rating mayor a 4

# Visualización de la distribución de ratings con Altair
chart = alt.Chart(df, title="Distribución de Ratings").mark_bar().encode(
    alt.X("RATING", bin=alt.Bin(step=0.5)),  # Agrupa los valores de rating en intervalos de 0.5
    y='count()'  # Cuenta la cantidad de productos en cada intervalo de rating
)
chart

# Visualización de la distribución de precios con Matplotlib
fig, ax = plt.subplots(figsize=(6,3))
df["PRICE"].plot(kind="hist", density=True, bins=15, alpha=0.6)  # Histograma de precios
df["PRICE"].plot(kind="kde", color='#c44e52')  # Curva de densidad de precios
plt.title("Distribución de Precios")
plt.xlabel("Precio")
plt.show()

# Cerrar la sesión de Snowflake al finalizar
session.close()

```

---

## Conclusión

Con estos pasos, ahora puedes:

✅ **Conectar Snowflake con VS Code** mediante la extensión oficial.  
✅ **Ejecutar consultas SQL** directamente desde el editor.  
✅ **Configurar Python** e instalar librerías necesarias.  
✅ **Ejecutar código en Snowflake** usando `snowflake-snowpark-python`.  
✅ **Realizar análisis y visualización de datos** en **pandas**, **Altair** y **Matplotlib**.

Si encuentras errores, revisa:
- La conexión a Snowflake (credenciales y permisos).
- La instalación de dependencias (`pip list` para ver librerías instaladas).
- Que tienes Python actualizado (`python --version`).

**¡Listo para trabajar con Snowflake desde VS Code y Python!**

