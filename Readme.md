# Introducción a Self Hosted Integration Runtime con Synapse Analytics 

- [Introducción a Self Hosted Integration Runtime con Synapse Analytics](#introducción-a-self-hosted-integration-runtime-con-synapse-analytics)
  - [Introducción](#introducción)
  - [Paso 1: Crear una cuenta de Azure](#paso-1-crear-una-cuenta-de-azure)
  - [Paso 2: Crear el servicio de Synapse Analytics](#paso-2-crear-el-servicio-de-synapse-analytics)
  - [Paso 3: Crear e instalar el entorno de ejecución (SHIR)](#paso-3-crear-e-instalar-el-entorno-de-ejecución-shir)
  - [Paso 4: Crear los Pipelines de prueba](#paso-4-crear-los-pipelines-de-prueba)
    - [Pipeline 1. Pasar archivo Parquet local a Azure Blob Storage](#pipeline-1-pasar-archivo-parquet-local-a-azure-blob-storage)
      - [Creación del pipeline](#creación-del-pipeline)
      - [Source (Fuente)](#source-fuente)
      - [Sink (Destino)](#sink-destino)
      - [Ejecutar el _pipeline_](#ejecutar-el-pipeline)
    - [Pipeline 2. Pasar una tabla de **SQLite** a **Azure Table Storage**](#pipeline-2-pasar-una-tabla-de-sqlite-a-azure-table-storage)
      - [Configurar ODBC](#configurar-odbc)
      - [Creación del pipeline](#creación-del-pipeline-1)
      - [Source](#source)
      - [Crear una tabla Azure Table Storage](#crear-una-tabla-azure-table-storage)
      - [Sink](#sink)
      - [Ejecutar el _pipeline_](#ejecutar-el-pipeline-1)
    - [Pipeline 3. Pasar tabla **Access** local a **Parquet** en **Azure Blob Storage**](#pipeline-3-pasar-tabla-access-local-a-parquet-en-azure-blob-storage)
      - [Configurar ODBC](#configurar-odbc-1)
      - [Creación del pipeline](#creación-del-pipeline-2)
  - [Paso 4: Consultando archivos parquet con serverless SQL Pools](#paso-4-consultando-archivos-parquet-con-serverless-sql-pools)
  - [Paso 5: Conectar Power BI](#paso-5-conectar-power-bi)
  - [Paso 6: Configurar github](#paso-6-configurar-github)



## Introducción
- Este repositorio contiene una descripción -a grandes rasgos- de los pasos y material para replicar el video-tutrorial [Introducción a Self Hosted Integration Runtime (SHIR) con Synapse Analytics](). 

- El contenido está inspirado en el video [How to Create Self Hosted IR and Load Data From On-Prem to Azure Cloud - Azure Data Factory Tutorial](https://youtu.be/YgfNzNdK7Y8), sin embargo allí muestran una conexión con SQL Server.
   
- El [Self-Hosted Integration Runtime](https://docs.microsoft.com/es-es/azure/data-factory/create-self-hosted-integration-runtime) (SHIR) es similar al [On-premise data gateway de Power BI](https://docs.microsoft.com/es-es/power-bi/connect-data/service-gateway-onprem) y su función es transferir datos locales (_on-premise_) a los servicio de Azure en la nube.
  
- Si no estás familiarizado con git, en lugar de clonar este repositorio, puedes simplemente bajar los archivos la carpeta [data](./data) que necesites localmente.
  
- La documentación de microsoft está disponible en inglés y español. Para pasar de un idioma al otro debes colocar en el url (www.docs.microsoft.com/{idioma}/) **en-us** o **es-es** según sea el caso.

## Paso 1: Crear una cuenta de Azure
Si no tienes acceso a una cuenta en Azure, tienes varias opciones gratuitas para empezar:
- ¿Tienes tarjeta de crédito? -> [Cree soluciones en la nube con una cuenta gratuita de Azure](https://azure.microsoft.com/es-es/free/)
- ¿Eres estudiante? -> [Compilación gratuita en la nube con Azure for Students](https://azure.microsoft.com/es-es/free/students/)

## Paso 2: Crear el servicio de Synapse Analytics
1. Crear el _Resource Group_: `shir-test-rg`
2. Crear el recurso _Azure Synapse Analytics_: `synapse-test-ws`
   1. _Managed Resourse Group_: `synapse-mngd-rg`
   2. _Datalake Gen2 Storage Account_: `datalake-test-st`
   3. _File system name_: `filesys-test-fs`

## Paso 3: Crear e instalar el entorno de ejecución (SHIR)
Una vez creado el recurso, puedes acceder al ambiente de trabajo haciendo click en el _Workspace web URL_. A partir de lo cual se procede a crear el SHIR:
1. En el espacio de trabajo o _Synapse Analytics workspace_, en el menú izquierdo seguir la opción **Manage > Integration runtimes**. Hacer click en **+ New** y seguir los pasos seleccionando la opción **Azure, Self Hosted**.
   1. _Integration runtime_: `shir-test`
2. Descargar el instalador. Luego instalar y configurar el SHIR localmente.

## Paso 4: Crear los Pipelines de prueba
A continuación vamos a crear tres _pipelines_, cada uno con sus particularidades. La actividad más básica de un _pipeline_ es copiar datos al entorno de Synapse. La actividad de copia está definida por una fuente y un destino. La estructura es básicamente la siguiente:

- _Source data_
  - _Integration dataset_
  - _Linked Service_
- _Sink dataset_
  - _Integration dataset_
  - _Linked Service_
    
### Pipeline 1. Pasar archivo Parquet local a Azure Blob Storage
- Apache Parquet es un tipo de archivo columnar especialmente diseñado para almacenar y consultar datos de manera eficiente. Ver [Apache Parquet](https://parquet.apache.org/).
  
- Para subir archivos locales se requiere compartir en Windows la carpeta local que los contiene. De esta manera el SHIR podrá accederlos.  

#### Creación del pipeline
1. En el portal de Azure se crea un _pipeline_ (**Integrate > + Pipeline**) y se añade la actividad _Copy data_ (**Activities > Move and Transform > Copy data**) con la siguiente configuración:

#### Source (Fuente)
1. Crear un nuevo _Source dataset_.
2. Seleccionar _File System_ en _New Integration dataset_ y seleccionar el formato _Parquet_.
3. Crear un _linked service_ haciendo referencia el _integration runtime_ `shir-test` y a la carpeta compartida.

#### Sink (Destino)
1. Crear _Sink dataset_.
2. Seleccionar _Azure Blob Storage_ en _Integration dataset_ y luego formato _Parquet_.

#### Ejecutar el _pipeline_
Ejecutar el _pipeline_ en **Add Trigger > Trigger Now**.

### Pipeline 2. Pasar una tabla de **SQLite** a **Azure Table Storage**
- En este caso hacemos uso del conector [_Open Database Connectivity_ (ODBC)](https://docs.microsoft.com/es-es/sql/odbc/reference/odbc-overview?view=sql-server-ver16) para copiar una tabla [SQLite](https://www.sqlite.org/index.html).
  
- SQLite es una base de datos transaccional de código abierto, auto-contenida y _serverless_. Es la más usada del mundo, instalada en billones de dispositivos de todo tipo. Ver [About SQLite](https://www.sqlite.org/about.html)
  
- El destino de este _pipeline_ será una tabla [Azure Table Storage](https://azure.microsoft.com/es-es/services/storage/tables/#overview).

#### Configurar ODBC
1. Instalar SQLite ODBC driver descargando y ejecutando el archivo `sqliteodbc_w64.exe` [link](http://www.ch-werner.de/sqliteodbc/).
2. Configurar la base de datos en Windows, con **ODBC Data Sources Administrator (64-bit)**
3. En _System DSN_, de nombre `sqlite_chinook`, que conecte el _Data Source Name_ (DSN) al archivo (base de datos) [./data/chinook.db](./chinook.db). El archivo fue descargado de [SQLite Sample Database](https://www.sqlitetutorial.net/sqlite-sample-database/)

#### Creación del pipeline
Se siguen los mismos pasos anteriores, con la siguiente configuración:  
  
#### Source
1. Crear _Source dataset_.
2. Seleccionar _ODBC_ en _Integration dataset_.
3. Crear un _linked service_ haciendo referencia al _integration runtime_ `shir-test`.
4. En _Connection string_: `dsn=sqlite_chinook` (Mismo nombre que el paso anterior)
5. _Authentication type_: `Anonymous`

#### Crear una tabla Azure Table Storage
1. Ir al _Storage Account_.
2. Seleccionar **Table Service > +Table**
3. Seleccionar un nombre para la tabla.

#### Sink
1. Crear un nuevo _Sink dataset_.
2. Seleccionar _Azure Table Storage_ en _Integration dataset_.
3. Crear un nuevo _Linked Service_ and y seleccionar la tabla creada anteriormente.

#### Ejecutar el _pipeline_
Ejecutar el _pipeline_ en **Add Trigger > Trigger Now**.

### Pipeline 3. Pasar tabla **Access** local a **Parquet** en **Azure Blob Storage**

- Usamos nuevamente el conector ODBC, esta vez para copiar una tabla de una base de datos Access. 
  
- Esta base de datos se usaba hace unos siete años para la capacitación de Power BI [Analyzing-Visualizing-Data-PowerBI](https://github.com/MicrosoftLearning/Analyzing-Visualizing-Data-PowerBI). Está disponible aquí: [PowerBI AccessDB.zip](https://github.com/MicrosoftLearning/Analyzing-Visualizing-Data-PowerBI/blob/master/Lab1/PowerBI%20AccessDB.zip)
  
- Este caso es muy similar al anterior.

#### Configurar ODBC
- El driver de Access está disponible en Windows. 
- En **ODBC Data Sources Administrato (64-bit)**, _System DSN_, crear `power_bi_access`. Conectando con la base de datos `.accdb`.

#### Creación del pipeline
Los pasos para la creación del pipeline son muy similares a lo realizado anteriormente.

## Paso 4: Consultando archivos parquet con serverless SQL Pools
Hacer una consulta (_query_) a alguno de los archivos Parquet que fueron pasados a Synapse.

## Paso 5: Conectar Power BI
Conectar Power BI con las tablas y archivos parquet creados en Azure.

## Paso 6: Configurar github
Sincronizar los desarrollos realizados con github para control de versiones y CI/CD.