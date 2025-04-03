# Caso Práctico: Creación de Warehouses según necesidades de cada rol

Este caso práctico servirá de refuerzo de los conocimientos aprendidos en el documento:
 📄 **[Introducción y Arquitectura de Snowflake](01_introduccion_snowflake.md)**
Ante cualquier duda que se pueda tener acerca de los conceptos adquiridos, se recomienda revisar nuevamente el documento y su sección correspondiente.

## Objetivo del caso práctico

- Crear diferentes warehouses para distintos roles en la empresa.
- Configurar parámetros adecuados para cada tipo de workload.
- Optimizar el rendimiento y el consumo de créditos en Snowflake.
- Consolidar conocimientos aprendidos.
  

---

### Ejercicio 1. Creación de Warehouses según requerimientos

#### Warehouse para Analistas de Datos

Se necesita un warehouse pequeño para los analistas de datos. Este warehouse debe permitir consultas ligeras y análisis exploratorio. Debe suspenderse tras 1 minuto de inactividad para reducir costos. No debe permitir escalado automático y debe reactivarse automáticamente cuando sea necesario.

<details><summary>Mostrar solución</summary>

```sql
CREATE WAREHOUSE IF NOT EXISTS WH_ANALYTICS
WITH WAREHOUSE_TYPE = 'STANDARD'
     WAREHOUSE_SIZE = 'SMALL'
     AUTO_SUSPEND = 60
     AUTO_RESUME = TRUE
     MIN_CLUSTER_COUNT = 1
     MAX_CLUSTER_COUNT = 1
     SCALING_POLICY = 'STANDARD'
     INITIALLY_SUSPENDED = TRUE;
```

</details>

---

#### Warehouse para Data Scientists

Se necesita un warehouse mediano para los Data Scientists. Debe ser capaz de manejar grandes volúmenes de datos y cargas variables. Se debe suspender tras 5 minutos de inactividad para evitar costos innecesarios. Debe permitir el escalado automático hasta 3 clusters y utilizar Query Acceleration.

<details><summary>Mostrar solución</summary>

```sql
CREATE WAREHOUSE IF NOT EXISTS WH_DATASCIENCE
WITH WAREHOUSE_TYPE = 'STANDARD'
     WAREHOUSE_SIZE = 'MEDIUM'
     AUTO_SUSPEND = 300
     AUTO_RESUME = TRUE
     MIN_CLUSTER_COUNT = 1
     MAX_CLUSTER_COUNT = 3
     SCALING_POLICY = 'ECONOMY'
     ENABLE_QUERY_ACCELERATION = TRUE
     INITIALLY_SUSPENDED = TRUE;
```

</details>

---

#### Warehouse para Data Engineers

Se necesita un warehouse grande para los Data Engineers. Debe estar optimizado para la ejecución de procesos ETL y transformaciones de datos. No debe suspenderse automáticamente para evitar interrupciones en los pipelines de datos. Debe permitir el escalado automático hasta 5 clusters para manejar cargas intensivas.

<details><summary>Mostrar solución</summary>

```sql
CREATE WAREHOUSE IF NOT EXISTS WH_ENGINEERING
WITH WAREHOUSE_TYPE = 'STANDARD'
     WAREHOUSE_SIZE = 'LARGE'
     AUTO_SUSPEND = 0
     AUTO_RESUME = TRUE
     MIN_CLUSTER_COUNT = 1
     MAX_CLUSTER_COUNT = 5
     SCALING_POLICY = 'STANDARD'
     INITIALLY_SUSPENDED = FALSE;
```

</details>

---

#### Warehouse para Procesos de Machine Learning

Se necesita un warehouse optimizado para Snowpark para ejecutar entrenamientos de modelos. Debe ser grande y permitir el escalado dinámico hasta 8 clusters. Se debe suspender tras 10 minutos de inactividad para evitar costos innecesarios. Debe permitir Query Acceleration para mejorar el rendimiento de consultas complejas.

<details><summary>Mostrar solución</summary>

```sql
CREATE WAREHOUSE IF NOT EXISTS WH_ML
WITH WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
     WAREHOUSE_SIZE = 'LARGE'
     AUTO_SUSPEND = 600
     AUTO_RESUME = TRUE
     MIN_CLUSTER_COUNT = 1
     MAX_CLUSTER_COUNT = 8
     SCALING_POLICY = 'STANDARD'
     ENABLE_QUERY_ACCELERATION = TRUE
     INITIALLY_SUSPENDED = FALSE;
```

</details>

---

### Ejercicio 2. Prueba de funcionamiento de los Warehouses

Activar manualmente el warehouse de analistas y verificar su estado

<details><summary>Mostrar solución</summary>

```sql
ALTER WAREHOUSE WH_ANALYTICS RESUME;
SELECT CURRENT_WAREHOUSE();
```

</details>

<p>

Suspender manualmente el warehouse de analistas

<details><summary>Mostrar solución</summary>

```sql
ALTER WAREHOUSE WH_ANALYTICS SUSPEND;
```

</details>

---

### Ejercicio 3. Optimización y ajustes adicionales

Reducir tamaño del warehouse de Data Engineers de ``LARGE`` a ``MEDIUM``

<details><summary>Mostrar solución</summary>

```sql
ALTER WAREHOUSE WH_ENGINEERING SET WAREHOUSE_SIZE = 'MEDIUM';
```

</details>

<p>

Aumentar tamaño del warehouse de analistas de SMALL a MEDIUM

<details><summary>Mostrar solución</summary>

```sql
ALTER WAREHOUSE WH_ANALYTICS SET WAREHOUSE_SIZE = 'MEDIUM';
```

</details>

<p>

Reducir suspensión automática del warehouse de Data Scientists a 2 minutos

<details><summary>Mostrar solución</summary>

```sql
ALTER WAREHOUSE WH_DATASCIENCE SET AUTO_SUSPEND = 120;
```

</details>

---

### Ejercicio 4. Finalización del caso práctico

Suspender todos los warehouses al finalizar

<details><summary>Mostrar solución</summary>

```sql
ALTER WAREHOUSE WH_ANALYTICS SUSPEND;
ALTER WAREHOUSE WH_DATASCIENCE SUSPEND;
ALTER WAREHOUSE WH_ENGINEERING SUSPEND;
ALTER WAREHOUSE WH_ML SUSPEND;
```

</details>
