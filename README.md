# Laboratorio 11: Infraestructura Big Data como Código (Terraform + Floci)

Proyecto de Infraestructura como Código (IaC) para el aprovisionamiento automatizado de un **Data Lake** organizado por dominios y capas (arquitectura Medallón), junto con su catálogo de metadatos en **AWS Glue Data Catalog**, ejecutado de forma local mediante **Terraform** y el emulador **Floci**.

## Arquitectura Aprovisionada

El despliegue gestiona **13 recursos** declarativos en el entorno local (`http://localhost:4566`):

* **Almacenamiento (Amazon S3):**
  * Bucket principal: `ucv-data-platform`
  * **9 prefijos** estructurados bajo el patrón `<dominio>/<capa>/`:
    * **Dominios:** `sales`, `customers`, `products`
    * **Capas:** `bronze`, `silver`, `gold`
* **Catálogo de Metadatos (AWS Glue Data Catalog):**
  * `sales_domain`
  * `customers_domain`
  * `products_domain`

## Estructura del Proyecto

```text
lab11-iac-bigdata/
├── .gitignore
├── docker-compose.yml       # Definición del servicio local Floci (puerto 4566)
├── README.md                # Documentación del proyecto
└── terraform/
    ├── versions.tf          # Restricciones de versión de Terraform y provider AWS
    ├── provider.tf          # Configuración de endpoints locales apuntando a Floci
    ├── variables.tf         # Parámetros reutilizables (bucket, dominios, capas, entorno)
    ├── main.tf              # Declaración del bucket S3, prefijos y bases de datos Glue
    └── outputs.tf           # Salidas clave tras el despliegue