---
title: "01. Introducción a Snowflake"
---

## Presentación docente

**Autor:** Arturo González Martínez  
**Certificaciones:**  
- SnowPro Core Certified  
- Microsoft Certified: Fabric Data Engineer Associate, Fabric Analytics Engineer Associate, Azure Fundamentals, Azure AI Fundamentals, Azure Data Fundamentals  
- AWS Certified Cloud Practitioner  

**Formación:**  
- Ingeniero Informático (UNIR)  
- MSc. en Inteligencia de Negocio y Big Data Analytics (UOC)  
- Postgrado en Inteligencia de Negocio y Data Analytics (UOC)  

**Rol actual:**  
- Senior Data Engineer & Data Architect  
- Profesor Colaborador, autor de contenidos, coordinador y tutor de TFGs en UOC  
- Formador IT  

## Agenda de la sesión

1. [Conceptos básicos e introducción a Snowflake](#conceptos-básicos-e-introducción)
2. [Arquitectura de tres capas: Service, Compute y Storage](#arquitectura-de-tres-capas)  
   - [Service Layer: Cloud Services](#arquitectura-de-tres-capas)
   - [Compute Layer: Query Processing](#arquitectura-de-tres-capas)
   - [Storage Layer: Database Storage](#arquitectura-de-tres-capas)
3. [Arquitectura compartida y multi-cluster de Snowflake](#arquitectura-multi-cluster-shared-disk)  
4. [Conexión a Snowflake utilizando conectores y drivers](#conexión-a-snowflake)  
   - [Ecosistema de conectividad: Python, Spark, Node.js, JDBC, ODBC](#ecosistema-de-snowflake)  
   - [Integración con herramientas ETL, BI y Machine Learning](#ecosistema-de-snowflake)  
   - [Uso de Snowflake Partner Connect](#ecosistema-de-snowflake)  
5. [Uso de Snowsight y SnowSQL para la gestión de datos](#snowsight-interfaz-web-de-snowflake)  
   - [Snowsight: interfaz web para consultas SQL y análisis](#snowsight-interfaz-web-de-snowflake)  
   - [SnowSQL: herramienta CLI para consultas y administración](#snowsql-cli-client)  
   - [Extensión de Snowflake para Visual Studio Code](#extensión-de-snowflake-para-visual-studio-code)  
6. [Gestión de roles y permisos](#Gestión-de-roles-y-permisos)  

## Conceptos básicos e introducción

### ¿Qué es Snowflake?

Snowflake es una plataforma en la nube diseñada para el almacenamiento y análisis de datos. Es una solución:  
- **Escalable**  
- **Segura**  
- **Eficiente**  

A diferencia de bases de datos convencionales, Snowflake es un servicio totalmente administrado, permitiendo a las organizaciones enfocarse en el análisis en lugar de la administración de infraestructura.

### ¿Para qué se utiliza Snowflake?
Snowflake es ampliamente utilizado en diversos sectores para satisfacer necesidades de almacenamiento y análisis de datos.

- **Data Warehousing**  
- **Business Intelligence (BI) y Analítica**  
- **Integración de Datos (ETL/ELT)**  
- **Machine Learning e Inteligencia Artificial**  
- **Big Data y procesamiento en tiempo real**  
- **Gobernanza y seguridad de datos**  

### Características principales
Snowflake ofrece una serie de características clave que lo diferencian de otras soluciones de almacenamiento y análisis de datos

- **Arquitectura de almacenamiento y computación separada**  
- **Almacenamiento en la nube**  
- **Escalabilidad automática**  
- **Soporte para datos estructurados y semiestructurados**  
- **Concurrencia y aislamiento de cargas de trabajo**  
- **Modelo de pago por uso**  

## Caso práctico: Creación de una cuenta en Snowflake  

Como primer caso práctico del curso, crearemos una cuenta en Snowflake desde cero. Este ejercicio permitirá familiarizarse con la plataforma y explorar sus capacidades desde el primer acceso.  

### Objetivos del caso práctico  
- Aprender a registrarse en **Snowflake** de forma gratuita.  
- Configurar los primeros ajustes de la cuenta.  
- Acceder a la interfaz web **Snowsight** y conocer sus funcionalidades básicas.  

Para comenzar, sigue las instrucciones detalladas en el siguiente documento:  

📄 **[Caso Práctico: Creación de Cuenta en Snowflake](01-caso-practico1.md)**  

✅ **Al completar este caso práctico, estarás listo para explorar Snowflake y empezar a trabajar con datos en la nube.**  


## Arquitectura de Snowflake

### Arquitectura de tres capas

![Arquitectura de tres capas en Snowflake](./resources/Imagen1.png)

*Figura: Diagrama de la arquitectura de tres capas en Snowflake, que incluye Cloud Services, Query Processing y Database Storage.*

#### **1. Service Layer (Cloud Services)**  
Esta capa es la encargada de gestionar la seguridad, autenticación y control de acceso en Snowflake. Incluye funciones como la optimización de consultas, el manejo de metadatos y la integración con herramientas externas de Business Intelligence (BI) y procesos de extracción, transformación y carga (ETL). Además, administra la infraestructura subyacente de la plataforma en la nube, permitiendo que Snowflake se ofrezca como un servicio completamente gestionado.

#### **2. Compute Layer (Query Processing)**  
El procesamiento de consultas en Snowflake se realiza a través de los **Virtual Warehouses**. Estas instancias de cómputo operan de manera independiente, lo que permite ejecutar múltiples cargas de trabajo simultáneamente sin interferencias. Snowflake permite escalar los warehouses de forma automática tanto horizontal como verticalmente, asegurando un rendimiento óptimo según la demanda. Esta independencia evita problemas de concurrencia y permite una ejecución eficiente de consultas sobre grandes volúmenes de datos.

#### **3. Storage Layer (Database Storage)**  
La capa de almacenamiento en Snowflake está diseñada para ser altamente escalable y segura. Utiliza un modelo de almacenamiento en formato columnar optimizado, lo que mejora la compresión y la eficiencia en la consulta de datos. Los datos almacenados en Snowflake están distribuidos automáticamente en múltiples regiones de la nube y pueden residir en proveedores como AWS, Azure o Google Cloud Platform. Además, Snowflake implementa **técnicas avanzadas de compresión** y **cifrado automático**, garantizando la seguridad y el rendimiento de los datos almacenados.  

Un aspecto clave de esta capa es la capacidad de **pruning inteligente**, que permite minimizar la cantidad de datos escaneados en consultas, optimizando el rendimiento sin necesidad de intervención manual.

![Ejemplo de Pruning Inteligente en Snowflake](./resources/Imagen2.png)

*Figura: Ejemplo de pruning inteligente en Snowflake, donde se minimiza la cantidad de datos escaneados al optimizar el acceso a micro-particiones.*

## Comparación entre bases de datos por filas y por columnas  

Las bases de datos pueden organizar su información de diferentes maneras según su propósito. En esta tabla se comparan las bases de datos orientadas a filas y a columnas en función de su estructura, recuperación de datos, tipos de operación y ejemplos de cada una.  

- **Bases de datos por filas**: Optimizadas para transacciones rápidas, donde los registros completos suelen recuperarse con frecuencia. Son ideales para sistemas **OLTP** (Procesamiento de Transacciones en Línea).  
  - **Ejemplos**: *Postgres*, MySQL, Oracle, SQL Server.  

- **Bases de datos por columnas**: Diseñadas para análisis de grandes volúmenes de datos, ya que permiten escanear solo las columnas necesarias en lugar de registros completos. Son ideales para entornos **OLAP** (Procesamiento Analítico en Línea).  
  - **Ejemplos**: Snowflake, Amazon *Redshift*, *BigQuery*.  

Este modelo de almacenamiento impacta directamente en el rendimiento y eficiencia de las consultas dependiendo del tipo de carga de trabajo que se maneje.


| Categoría              | Filas                          | Columnas                          |
|------------------------|------------------------------|-----------------------------------|
| Organización de datos  | Por filas                     | Por columnas                     |
| Recuperación de datos  | Registros completos          | Columnas relevantes              |
| Tipos de operación     | Transaccionales              | Analíticas                       |
| Ejemplos               | *Postgres*, MySQL, Oracle, SQL Server | Snowflake, Amazon *Redshift*, *BigQuery* |



### Arquitectura Multi-Cluster, Shared-Disk

Snowflake implementa un modelo de arquitectura híbrido que combina elementos de **shared-disk** y **shared-nothing**. Esta combinación permite aprovechar lo mejor de ambos enfoques: la facilidad de administración del almacenamiento compartido y la eficiencia del procesamiento distribuido.

#### **Modelo de almacenamiento centralizado**  
A diferencia de las bases de datos tradicionales de procesamiento distribuido, Snowflake utiliza un **repositorio de almacenamiento centralizado** donde todos los nodos pueden acceder a los datos. Este enfoque elimina la necesidad de replicar datos entre nodos, lo que simplifica la gestión del almacenamiento y permite un acceso más eficiente a la información sin afectar el rendimiento.

#### **Procesamiento distribuido mediante Virtual Warehouses**  
El cómputo en Snowflake se lleva a cabo mediante **clústeres de warehouses virtuales**, donde cada nodo dentro del clúster procesa diferentes porciones de los datos de forma independiente. Gracias a este modelo, es posible ejecutar múltiples consultas en paralelo sin que unas interfieran con otras. Además, Snowflake permite escalar dinámicamente los warehouses, agregando más recursos cuando la demanda aumenta y reduciéndolos cuando no se necesitan.

#### **Escalabilidad y concurrencia optimizadas**  
Uno de los mayores beneficios de esta arquitectura es su capacidad de **escalar horizontalmente** de forma automática y sin interrupciones. Si un alto volumen de usuarios o consultas genera carga en el sistema, Snowflake puede desplegar nodos adicionales para equilibrar la carga y mejorar el tiempo de respuesta. Esta escalabilidad evita los bloqueos y problemas de concurrencia típicos de las bases de datos tradicionales.

Gracias a este enfoque híbrido, Snowflake logra combinar una administración sencilla con alto rendimiento y escalabilidad, adaptándose tanto a cargas de trabajo analíticas como a grandes volúmenes de procesamiento de datos en tiempo real.

![Comparación entre arquitectura Shared-Disk y Shared-Nothing](./resources/Imagen3.png)

*Figura: Comparación entre la arquitectura Shared-Disk y Shared-Nothing. En la arquitectura Shared-Disk, múltiples nodos comparten el mismo almacenamiento, mientras que en la arquitectura Shared-Nothing, cada nodo tiene su propio almacenamiento independiente, permitiendo un procesamiento distribuido más eficiente.*


## Conexión a Snowflake

### Ecosistema de Snowflake

A través de una red extensiva de conectores, drivers , lenguajes de programación, y utilidades, incluyendo:

- Partners certificados que proveen soluciones on-premise o en la nube para conectarse a Snowflake

- Otras herramientas y tecnologías tercerizadas que son conocidas para trabajar con Snowflake

- Clientes de Snowflake, incluyendo SnowSQL (_command line interface_), conectores para Python y Spark, y drivers para Node.js, JDBC, ODBC, y más.

>ℹ️ **Info**: Snowflake trabaja con una amplia variedad de herramientas y tecnologías lideres en la industria.

![](./resources/01-image_4.png)

- Data Integration. ETL

- Business Intelligence

- Machine Learning and Data Science

- Security and Governance

- SQL Development and Management

- Native Programmatic Interfaces

- Snowflake Partner Connect

### Snowsight: The Snowflake web interface

Snowsight provee una experiencia unificada para trabajar con sus datos de Snowflake utilizando SQL o Python:

- Queries de SQL

- ML workflows utilizando notebooks de Snowflake

- Análisis de datos con Snowflake Copilot

- Snowpark Python

- Monitoreo de desempeño de queries e historia.

- Visualización

- Desarrollar y compartir Snowflake Native Apps

- Snowflake Marketplace

>ℹ️ **Info**: Snowflake resume el potente soporte SQL de Snowflake en una experiencia unificada y fácil de usar, importante para operaciones críticas de Snowflake.

![](./resources/01-image_5.png)

Administrar el despliegue de Snowflake y desarrollar operaciones críticas de Snowflake.

- Administración de costos.

- Riesgos de seguridad, monitoreo y evaluación

- Monitoreo de la carga de datos

- Creación y administración de usuarios

- Creación y administración de roles.

- Etc, etc, etc.

### SnowSQL (CLI Client)

Snowsight provee una experiencia unificada para trabajar con sus datos de Snowflake utilizando SQL o Python:

- SnowSQL es una herramienta poderosa y flexible para interactuar con Snowflake, proporcionando una interfaz de línea de comandos robusta para la ejecución de consultas, administración de bases de datos y manipulación de datos, todo ello con un enfoque en la seguridad y la eficiencia.

- SnowSQL permite a los usuarios ejecutar consultas SQL, scripts y comandos de administración directamente desde su terminal o línea de comandos.

- Está diseñado para interactuar con el entorno de Snowflake, facilitando tareas como la ejecución de consultas, la carga y descarga de datos, y la administración de bases de datos y objetos de almacenamiento.

>ℹ️ **Info**: SnowSQL es el cliente de línea de comandos para conectarse a Snowflake y ejecutar consultas SQL y realizar todas las operaciones DDL y DML, incluida la carga y descarga de datos de las tablas de la base de datos.

## Caso práctico: Uso de SnowSQL para la gestión de datos

En este caso práctico, exploraremos SnowSQL, la interfaz de línea de comandos (CLI) de Snowflake, y aprenderemos a utilizarlo para ejecutar consultas y administrar datos en Snowflake.

### Objetivos del caso práctico  
- Instalar y configurar **SnowSQL** en tu equipo.  
- Conectarse a una cuenta de **Snowflake** mediante SnowSQL.  
- Ejecutar consultas SQL para la gestión de datos.  
- Cargar y descargar datos en Snowflake desde la línea de comandos.  

Para comenzar, sigue las instrucciones detalladas en el siguiente documento:  

📄 **[Caso Práctico: Uso de SnowSQL](01-caso-practico2.md)**  

✅ **Al completar este caso práctico, serás capaz de manejar Snowflake desde la línea de comandos de manera eficiente.**


### Snowflake Extension for Visual Studio Code

Snowflake proporciona una extensión para Visual Studio Code (VS Code) que permite a los usuarios de Snowflake escribir y ejecutar sentencias SQL de Snowflake directamente en VS Code. La extensión también se integra con Snowpark Python para proporcionar funciones de depuración, resaltado de sintaxis y autocompletado para SQL en código Python.

Las extensiones son funcionalidades preempaquetadas, a menudo proporcionadas por terceros, que añaden nuevas características y funcionalidades a VS Code.

>ℹ️ **Info**: Utilice la extensión Snowflake para Visual Studio Code para conectarse a Snowflake dentro de Visual Studio Code y realizar operaciones SQL.

### **Caso práctico: Conexión de Snowflake con VSC**

Este caso práctico te guiará en la instalación y configuración de la extensión de Snowflake en Visual Studio Code, permitiéndote conectarte a tu cuenta de Snowflake y ejecutar consultas SQL directamente desde el editor

📄 **[Caso práctico: Conexión de Snowflake con VSC](01-caso-practico3.md)**

---

## Gestión de roles y permisos

En Snowflake, los permisos y accesos a objetos se gestionan mediante un sistema de **roles y permisos**. Esto permite administrar de manera centralizada el acceso a los datos y controlar los privilegios de los usuarios de forma escalable.

### **RBAC (Role-Based Access Control)**
- Utiliza roles en lugar de asignaciones directas a usuarios.
- La estructura de asignación de permisos es **Permisos -> Roles -> Usuarios**.
- Simplifica la gestión de accesos y permite una mejor administración de los permisos.

### **Roles predeterminados en Snowflake**
- `ACCOUNTADMIN`: Nivel más alto de privilegio, administra toda la cuenta.
- `SYSADMIN`: Gestiona objetos de la base de datos como tablas y esquemas.
- `SECURITYADMIN`: Administra usuarios, roles y permisos de seguridad.
- `USERADMIN`: Diseñado para la gestión de usuarios y asignación de roles.
- `PUBLIC`: Rol predeterminado con permisos básicos para todos los usuarios.

### **Herencia de privilegios**
- Los roles pueden heredar permisos de otros roles asignados como "roles padres".
- Los permisos otorgados a un rol superior también están disponibles para sus roles hijos.
- Esto facilita la administración de permisos y evita configuraciones redundantes.

### **Gestión de permisos con comandos en Snowflake**

Algunos comandos clave para gestionar roles y permisos en Snowflake incluyen:

```sql
-- Crear un nuevo rol\CREATE ROLE data_analyst;

-- Asignar permisos a un rol
GRANT SELECT ON ALL TABLES IN SCHEMA my_db.my_schema TO ROLE data_analyst;

-- Asignar un rol a un usuario
GRANT ROLE data_analyst TO USER analyst_user;

-- Cambiar el rol activo
USE ROLE securityadmin;
```

### **Caso práctico: Gestión de roles y permisos en Snowflake**

En este caso práctico, aprenderás a:
- Crear y asignar roles a usuarios.
- Configurar permisos sobre bases de datos y esquemas.
- Administrar accesos de forma centralizada en Snowflake.

📄 **[Caso Práctico: Gestión de Roles y Permisos en Snowflake](01-caso-practico4.md)**



## Conclusión

Snowflake proporciona una solución escalable, segura y eficiente para el almacenamiento y análisis de datos en la nube. Su arquitectura única permite a las organizaciones optimizar sus operaciones sin preocuparse por la administración de infraestructura.  