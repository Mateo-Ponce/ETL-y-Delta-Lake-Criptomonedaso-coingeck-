# Coingecko ETL Delta Lake Pipeline

¡Bienvenido Este repositorio contiene un pipeline ETL en Python que extrae datos de la API de CoinGecko, realiza cargas incrementales y estáticas, y los almacena en formato parquet en un **Delta Lake** organizado en las capas **Bronze**, **Silver** y **Gold**.
---

## 📋 Tabla de Contenidos

- [Descripción del Proyecto](#descripción-del-proyecto)  
- [Arquitectura](#arquitectura)  
- [Estructura del Repositorio](#estructura-del-repositorio)  
- [Requisitos](#requisitos)  
- [Instalación](#instalación)  
- [Flujo de Trabajo](#flujo-de-trabajo)  
  - [Capa Bronze](#capa-bronze)  
  - [Capa Silver](#capa-silver)  
  - [Capa Gold](#capa-gold)  
- [Uso](#uso)
- [conclusión](#conclusión)
- [Contacto](#contacto)  

---

## Descripción del Proyecto

Este proyecto implementa un pipeline ETL de principio a fin para datos de criptomonedas:

1. **Extracción** desde la API de CoinGecko, con tres endpoints:
   - `coins/list` (datos estáticos)
   - `coins/markets` (datos diarios, extracción incremental)
   - `{coin_id}/market_chart` (datos históricos de los últimos 30 días)
2. **Transformación** y limpieza de datos en distintas capas del Delta Lake.
3. **Carga** en Delta Lake usando particionamiento y lógica de _merge_ para evitar duplicados y mantener la consistencia.

---

## Arquitectura
```bash
API CoinGecko
│
▼
Capa Bronze ──▶ Capa Silver ──▶ Capa Gold
│ │ │
Raw Data Datos Limpios Datos Enriquecidos & Agregados
```


1. **Bronze**  
   - Almacena los datos crudos tal cual vienen de la API, en formato Parquet y Delta Lake, particionados por fecha de extracción.  
   - Endpoints:  
     - `coins_list` → `bronze/coingecko_api/coins_list` (overwrite)  
     - `coins_markets` → `bronze/coingecko_api/coins_markets` (merge por `extract_date`)  
     - `market_chart` → `bronze/coingecko_api/market_chart` (merge por `coin_id` + `extract_date`) :contentReference[oaicite:1]{index=1}  

2. **Silver**  
   - Carga desde Bronze las particiones más recientes.  
   - _Transformaciones_ principales:  
     - Selección de 18 columnas relevantes de `coins_markets` y relleno de valores nulos (`max_supply` con mediana, `roi_percentage` con 0).  
     - Normalización del DataFrame de `market_chart`: extracción de fecha, conversión de tuplas a valores flotantes, reordenación de columnas, y merge por `coin_id` + `date` para evitar duplicados históricos :contentReference[oaicite:2]{index=2}.  

3. **Gold**  
   - Integra datos actuales y 30 días de históricos para las 5 criptomonedas más importantes.  
   - Cálculos avanzados: porcentaje de crecimiento mensual, visualizaciones, y agregaciones.  
   - Lógica de merge incremental con claves adecuadas y conversión de tipos (fechas como strings) para almacenamiento seguro en Delta Lake :contentReference[oaicite:3]{index=3}.  

---

## Estructura del Repositorio

```bash
.
├── README.md
├── requirements.txt
├── notebooks/
│   ├── TP_Mateo_Ponce_parte1_bronze.ipynb
│   ├── TP_Mateo_Ponce_parte2_silver.ipynb
│   └── TP_Mateo_Ponce_parte3_gold.ipynb
├── scripts/
│   ├── extract.py
│   ├── transform.py
│   └── load.py
├── delta_lake/
│   ├── bronze/
│   ├── silver/
│   └── gold/
└── docs/
    └── arquitectura.png
```

## Requisitos

- Python 3.8+  
- Delta Lake (`delta-spark`)  
- PySpark  
- Requests  
- Pandas  

## Instalación:

```bash
pip install -r requirements.txt
```

## Uso
### Flujo de Trabajo  
#### Capa Bronze  
Ejecuta todas las celdas de TP_Mateo_Ponce_parte1_bronze.ipynb para:

Crear el directorio delta_lake/bronze/.

Extraer datos de los tres endpoints.

Guardar los Parquet/Delta de forma overwrite o merge según el caso 
.

#### Capa Silver  
Ejecuta TP_Mateo_Ponce_parte2_silver.ipynb para:

Cargar datos recientes desde Bronze.

Aplicar transformaciones (selección de columnas, relleno de nulos, limpieza de duplicados en market_chart).

Guardar en delta_lake/silver/.

#### Capa Gold  
Ejecuta TP_Mateo_Ponce_parte3_gold.ipynb para:

Cargar datos limpios desde Silver.

Integrar datos actuales e históricos.

Calcular métricas (crecimiento porcentual de las top 5 criptomonedas).

generar visualización y obtener insights 

Guardar resultados finales en delta_lake/gold/ y generar visualizaciones


## conclusión 
Al ejecutar el codigo tendras datos de diferentes endpoints de coingecko procesados y almacenados en formato delta lake en distintas capas;
Bronce: contiene los datos crudos en formato parquet extraidos de distintos edpoint de la API coingecko.  
Silver: en esta capa los datos crudos se normalizan y se procesan dandoles una estuctura mas adecuada para el análisis, la visualizacion o para modelos de machine learning  
Gold: se realizan tranformaciones avanzadas combinando dataframes lo cual permite realizar calculos y agregaciones, analizar tendencias históricas y visualizar estas tendencias en distintos graficos.

## Contacto
Desarrollado por Mateo Ponce.
Correo: mate.ponce.prog@gmail.com
