# Plataforma de Datos Lakehouse Local (Terraform + Floci + AWS Glue 5 + Apache Iceberg)

Proyecto integral de Ingeniería de Datos e Infraestructura como Código (IaC) que evoluciona desde el aprovisionamiento declarativo del Data Lake (**Laboratorio 11**) hasta la implementación de un pipeline analítico Medallón sobre **Apache Iceberg** con **AWS Glue 5** y **Data Contracts** (**Laboratorio 12**), ejecutado íntegramente de forma local.

---

## Arquitectura del Sistema

El entorno local opera sobre contenedores Docker comunicados en red interna:

1. **Infraestructura Base (Lab 11 — Terraform + Floci):**
   - **Almacenamiento (Amazon S3 emulado en `http://localhost:4566`):** Bucket `ucv-data-platform` organizado en **9 prefijos** (`<dominio>/<capa>/`) para los dominios `sales`, `customers` y `products` a través de las capas `bronze`, `silver` y `gold`.
   - **Catálogo de Metadatos (AWS Glue Data Catalog):** Bases de datos `sales_domain`, `customers_domain` y `products_domain`.

2. **Procesamiento y Lakehouse (Lab 12 — AWS Glue 5 + Apache Iceberg):**
   - **Ingesta (`Bronze`):** Extracción en Python desde la API REST de DummyJSON (`https://dummyjson.com/products?limit=0`) y almacenamiento particionado por fecha (`products/bronze/ingestion_date=YYYY-MM-DD/`).
   - **Limpieza y Deduplicación (`Silver`):** Job de PySpark en AWS Glue 5 que normaliza tipos, filtra registros inválidos, deduplica por `product_id` y escribe en la tabla Iceberg `lakehouse.silver.products`.
   - **Agregación Analítica (`Gold`):** Job de PySpark que calcula métricas de negocio por categoría y las persiste en la tabla Iceberg `lakehouse.gold.product_category_summary`.
   - **Gobernanza:** Contrato de producto de datos definido en YAML (`contracts/product_category_summary.yaml`).

---

## Estructura del Monorepo

```text
lab11-iac-bigdata/
├── .gitignore
├── docker-compose.yml                                      # Servicios Floci (S3/Glue) y AWS Glue 5 (Spark 3.5)
├── requirements.txt                                        # Dependencias Python para ingesta local (requests, boto3)
├── README.md                                               # Documentación general del proyecto
├── contracts/
│   └── product_category_summary.yaml                       # Data Contract de la capa Gold
├── pipelines/
│   └── products/
│       ├── ingest/
│       │   └── ingest_products.py                          # Ingesta API REST -> S3 Bronze (JSON)
│       ├── glue_jobs/
│       │   ├── bronze_to_silver.py                         # Glue Job: Bronze -> Iceberg Silver
│       │   └── silver_to_gold.py                           # Glue Job: Iceberg Silver -> Iceberg Gold
│       └── jobs/
│           └── query_iceberg_tables.py                     # Validación independiente de tablas Iceberg
└── terraform/
    ├── versions.tf                                         # Restricciones de versión de Terraform y provider AWS
    ├── provider.tf                                         # Configuración de endpoints locales apuntando a Floci
    ├── variables.tf                                        # Parámetros reutilizables (bucket, dominios, capas)
    ├── main.tf                                             # Declaración del bucket S3, prefijos y Glue Catalog
    └── outputs.tf                                          # Salidas clave tras el despliegue