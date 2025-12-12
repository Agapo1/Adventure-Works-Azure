# Descripción General

Este proyecto implementa un pipeline de datos completo basado en la base AdventureWorks2019 utilizando Azure Databricks, Apache Spark y el patrón Medallion Architecture (Bronze → Silver → Gold).

El propósito es simular un flujo real de ingeniería de datos en la nube, incorporando:
- Lago de datos en Azure Data Lake Storage Gen2 (ADLS)
- Control de acceso mediante Unity Catalog
- Notebooks modulares para cada etapa del pipeline
- Seguridad mediante Storage Credentials, External Locations y GRANTs
- Tablas Delta bien estructuradas listas para análisis y visualización en Power BI

## Arquitectura General
### Azure Data Lake Storage Gen2

Se utiliza un único contenedor con las capas:

`/raw/` → Archivos CSV originales

`/bronze/` → Tablas Delta sin transformar

`/silver/` → Datos limpios y tipificados

`/golden/` → Modelo dimensional (hechos y dimensiones)


## Azure Databricks + Unity Catalog
- Metastore asignado al workspace
- Storage Credential creado con Service Principal
- External Locations para cada capa
- Catalog y Schemas:

`adventure_works.bronze`

`adventure_works.silver`

`adventure_works.golden`

## 1. Notebook – Environment Preparation (Scripts folder)

Este notebook prepara todo el entorno para operar con Unity Catalog:
Tareas realizadas:
- Lectura de secrets desde Azure Key Vault mediante Secret Scope
- Creación del Storage Credential
- Creación de las External Locations para bronze, silver y gold
- Creación del Catalog adventure_works
- Creación de los schemas:

`adventure_works.bronze`

`adventure_works.silver`

`adventure_works.golden`

Este notebook se ejecuta promero y solo una vez

## 2. Notebook – ingest_adventureworks (RAW → BRONZE)

Este notebook representa la capa Bronze.
Funciones principales:
Lectura de los 5 archivos CSV desde raw/ usando schemas explícitos:
- Customer
- Product
- SalesOrderDetail
- SalesOrderHeader
- SalesTerritory
Inclusión de columnas de auditoría:
- fecha_carga
- archivo_origen
Creación de tablas Delta en: `adventure_works.bronze.*`

## 3. Notebook – load_adventureworks (BRONZE → SILVER)

La capa Silver se enfoca en:
- Limpieza
- Duplicados
- Normalización de tipos
- Renombrado de columnas
- Conversión de fechas
- Selección de atributos relevantes

Agrupación lógica

Se crean datasets Silver estandarizados:
- silver.customer
- silver.product
- silver.so_header
- silver.so_detail
- silver.territory

## 4. Notebook – transform_adventureworks (SILVER → GOLD)

La capa Gold construye el modelo dimensional final.

Dimensiones creadas:

- dim_customer
- dim_product
- dim_territory

Fact table:

- fact_sales

Incluye:

- SalesOrderID
- CustomerID
- ProductID
- TerritoryID
- Fecha de orden
- Cantidad pedida
- LineTotal
- Integración con dimensión de producto y cliente

Salida final:

Tablas Delta optimizadas para BI:

`adventure_works.golden.dim_*`

`adventure_works.golden.fact_sales`

##5. Notebook – Permissions (Security folder)

Implementa control de acceso mediante Unity Catalog:

Creación de grupos lógicos:

- DataEngineers

- DataAnalysts

- Admins

Permisos:

- USAGE sobre el catálogo

- ALL PRIVILEGES para ingeniería sobre bronze / silver

- SELECT para analistas sobre golden

- Control total para administradores


##6. Notebook – Revoke (Reversion folder)

- Revierte todos los GRANTs asignados:

- Revoca permisos por esquema

- Revoca permisos por tabla

- Revoca uso del catálogo

##Dashboards (Power BI)

Se incluye un dashboard que consume directamente:

`golden.fact_sales`

`golden.dim_product`

`golden.dim_customer`

`golden.dim_territory`

##Orden de Ejecución del Proyecto

1. Scripts/Environment Preparation.ipynb

2. Process/ingest_adventureworks.ipynb

3. Process/load_adventureworks.ipynb

4. Process/transform_adventureworks.ipynb

5. Security/Permissions.ipynb

6. (Opcional) Reversion/Revoke.ipynb

7. Power BI Dashboard

##Tecnologías utilizadas

Azure Databricks

Azure Data Lake Storage Gen2

Unity Catalog

Secret Scopes + Azure Key Vault

Delta Lake

Power BI

#Autor

Giovani Curo

Proyecto final – Ingeniería de Datos e IA con Azure Databricks
