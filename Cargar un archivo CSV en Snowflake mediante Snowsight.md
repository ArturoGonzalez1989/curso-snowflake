# 📌 Caso guiado: Cargar un archivo CSV en Snowflake mediante Snowsight

Este tutorial describe paso a paso cómo cargar un archivo CSV en Snowflake utilizando la interfaz web Snowsight.

---

## 1. Crear una base de datos en Snowflake

1. Ir a **"Databases"** en el menú lateral.
2. Hacer clic en **"+ Database"** en la esquina superior derecha.
3. Escribir el nombre de la base de datos en el campo **"Name"** (opcionalmente, añadir un comentario).
4. Hacer clic en **"Create"** para finalizar.

![Crear base de datos](./resources/Imagen4.png)


---

## 2. Crear un esquema dentro de la base de datos

1. En el menú lateral, expandir la base de datos creada.
2. Hacer clic en **"+ Schema"** en la esquina superior derecha.
3. Escribir el nombre del esquema en el campo **"Name"** (opcionalmente, añadir un comentario).
4. Hacer clic en **"Create"** para finalizar.

![Crear esquema](./resources/Imagen5.png)

---

## 3. Crear un Stage en Snowflake

1. En el menú lateral, expandir la base de datos y seleccionar el esquema creado.
2. Hacer clic en **"Create"** en la esquina superior derecha.
3. En el menú desplegable, seleccionar **"Stage" → "Snowflake Managed"**.
4. Completar los detalles y confirmar con **"Create"**.

![Crear Stage](./resources/Imagen6.png)

---

## 4. Configurar el Stage

1. Escribir un nombre en el campo **"Stage Name"**.
2. (Opcional) Activar el interruptor para ver los archivos almacenados en el Stage.
3. Elegir **"Client-side encryption"** o la opción preferida.
4. Hacer clic en **"Create"** para finalizar.

![Configurar Stage](./resources/Imagen7.png)

---

## 5. Subir un archivo CSV al Stage

1. En el menú lateral, expandir el esquema creado y seleccionar el Stage recién creado.
2. Hacer clic en **"+ Files"** en la esquina superior derecha.
3. Seleccionar el archivo a subir ([netflix_titles.csv](resources/netflix_titles.csv)).
4. (Opcional) Especificar un directorio dentro del Stage.
5. Hacer clic en **"Upload"** para finalizar.

![Subir CSV](./resources/Imagen8.png)

---

## 6. Verificar archivos en el Stage y cargar en tabla

1. Verificar que el archivo subido aparece en el Stage.
2. Repetir el proceso de carga con otro archivo si es necesario.
3. Hacer clic en el menú de **tres puntos** junto al archivo.
4. Seleccionar **"Load into table"** para iniciar la carga en Snowflake.

![Verificar archivos](./resources/Imagen9.png)

---

## 7. Configurar la carga del archivo a la tabla

1. Asegurarse de que **COMPUTE_WH** (warehouse) está activo.
2. Confirmar que el archivo CSV seleccionado es el correcto.
3. Verificar que el esquema de destino es correcto.
4. Definir el nombre de la tabla, por ejemplo: `NETFLIX_TITLES`.
5. Hacer clic en **"Next"** para continuar.

![Configurar carga](./resources/Imagen10.png)

---

## 8. Ajustar configuración y solucionar errores

1. Asegurar que **"Delimited Files (CSV or TSV)"** está seleccionado.
2. Configurar **"First line contains header"** para detectar los nombres de las columnas.
3. Usar **"Comma (default)"** como delimitador de campos.
4. Resolver errores por caracteres inválidos (ejemplo: separadores de miles en números).
5. Hacer clic en **"Load"** para cargar los datos.

![Ajustar configuración](./resources/Imagen11.png)

---

## 9. Verificar la carga y consultar datos

1. Aparecerá un mensaje de carga exitosa.

![Verificar carga](./resources/Imagen12.png)

2. Opción de visualizar datos cargados con **"Query data"**.
3. Snowflake generará automáticamente una consulta SQL de ejemplo.
4. Ejecutar la consulta para verificar los datos.

![.](./resources/Imagen13.png)

---

¡Listo! Ahora has aprendido a cargar un archivo CSV en Snowflake usando Snowsight.
