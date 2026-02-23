# Fantasy-Football-Proyect

# Fantasy Premier League (FPL) - Data Pipeline & BI

Este repositorio contiene una prueba de concepto de un pipeline de datos *batch* que extrae, procesa y modela estadísticas de jugadores de la API pública de la Fantasy Premier League.

El objetivo del proyecto es aplicar los principios de la Arquitectura Medallion utilizando Databricks, procesando estructuras JSON complejas y preparando los datos para su explotación analítica en Power BI.

## Arquitectura y Flujo de Datos

El pipeline se divide en tres capas lógicas, utilizando Delta Lake como formato de almacenamiento:

* **Capa Bronze (Ingesta):** Almacenamiento del JSON raw extraído de la API de FPL (`https://fantasy.premierleague.com/api/bootstrap-static/`). 
* **Capa Silver (Limpieza):** Uso de PySpark para aplanar jerarquías complejas (`explode` de arrays). Se extraen las tablas de dimensiones (Equipos y Posiciones) y se aplica lógica MERGE para realizar *Upserts* diarios sobre la tabla de hechos de los jugadores, actualizando sus métricas y precios.
* **Capa Gold (Analítica):** Uso de Spark SQL para consolidar el modelo. Se calculan métricas de negocio en el servidor (ej. `points_per_million`).

## Stack Tecnológico

* **Procesamiento:** Databricks, PySpark, Spark SQL, Pandas (para exportación final a CSV).
* **Almacenamiento:** Delta Lake, Unity Catalog Volumes.
* **Visualización:** Power BI Desktop.

## Decisiones de Diseño y Limitaciones Conocidas

Para mantener el realismo técnico, este proyecto documenta explícitamente las siguientes concesiones de arquitectura:

* **OBT (One Big Table) vs. Star Schema:** En la capa Gold, se ha optado por generar una tabla desnormalizada en lugar de un modelo en estrella estricto. Esto delega la carga computacional de los *JOINs* al clúster de Databricks, optimizando el rendimiento de lectura del dashboard en Power BI a costa de redundancia de datos.
* **Ausencia de Orquestación:** Al ser un proyecto de portfolio ejecutado en un entorno de pruebas, los *notebooks* se ejecutan de forma secuencial manual. En un entorno de producción, este flujo requeriría integración con Databricks Workflows o Apache Airflow.
* **SCD Tipo 1:** La tabla de jugadores en la capa Silver mantiene el estado actual. No se ha implementado versionado histórico (Type 2) para trackear la fluctuación de precios de los jugadores a lo largo de las semanas, manteniendo el modelo simplificado para la visualización de la jornada actual.

## Cómo ejecutar este proyecto

1. Clonar el repositorio en un *Workspace* de Databricks.
2. Ejecutar `01_bronze_ingest` para conectar con la API y guardar el JSON raw.
3. Ejecutar `02_silver_clean` para aplanar los datos y ejecutar el *Upsert*.
4. Ejecutar `03_gold_metrics` para aplicar transformaciones SQL y generar el archivo de exportación.
5. Conectar el archivo `.csv` resultante a Power BI Desktop.
