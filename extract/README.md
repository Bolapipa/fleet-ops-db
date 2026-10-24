# extract/

Extração do Postgres para a camada Bronze de um lakehouse (Fase 8).

1. Incremental por watermark (`updated_at`) para Parquet.
2. CDC com replicação lógica.

Use apenas um Databricks **pessoal** (Free Edition).