# Censo Data Pipeline & Geo-Analytics (Salta 2022)

Este proyecto implementa un pipeline de ingeniería de datos de extremo a extremo (End-to-End Data Pipeline) para la extracción, limpieza, transformación, modelado relacional y análisis geoespacial de los microdatos correspondientes al Censo Nacional de Población, Hogares y Viviendas 2022 para la Provincia de Salta, Argentina.

A partir de registros crudos y desestructurados provenientes de la plataforma REDATAM, se diseñó una arquitectura de base de datos relacional y espacial sobre PostgreSQL y PostGIS. El flujo de trabajo abarca desde la normalización de las variables hasta la validación automatizada de integridad estructural mediante aserciones y la generación de componentes de visualización analítica de alta fidelidad.

## Características Técnicas de la Arquitectura

*   **Procesamiento y ETL Dinámico:** Limpieza, tipado y transformación de archivos de texto plano masivos a formatos estructurados CSV de alta compatibilidad con el motor de base de datos.
*   **Modelado Multidimensional (Star Schema):** Diseño de un modelo en estrella optimizado para analítica (OLAP), compuesto por tablas de hechos (`fact_persona`, `fact_hogar`, `fact_vivienda`) y tablas de dimensiones (`dim_geografia`, `dim_variables`, `dim_categorias`) destinadas a reducir el costo de cómputo en consultas complejas.
*   **Ingeniería de Datos Espaciales:** Ingesta masiva y eficiente de geometrías vectoriales complejas (`.gpkg`) mediante bindings de Python (GeoPandas y el motor analítico PyOgrio) hacia tablas nativas de PostGIS, optimizadas mediante la indexación espacial GiST.
*   **Corrección y Modelado Demográfico:** Mitigación del sesgo de disociación dimensional en la base de datos de origen (Edad vs. Sexo) mediante la implementación de algoritmos de fraccionamiento demográfico a nivel quinquenal, logrando representaciones estadísticas de alta precisión.
*   **Suite de Aseguramiento de Calidad (QA Assertions):** Implementación de controles y aserciones matemáticas cruzadas que auditan la consistencia interna del total poblacional e instruccional frente a las métricas oficiales del censo antes de liberar la capa analítica.

---

## Stack Tecnológico

*   **Entorno de Ejecución:** Python 3.11
*   **Motor de Base de Datos:** PostgreSQL 18+ y Extensión Espacial PostGIS 3.6+
*   **Librerías del Ecosistema Core:** Pandas, GeoPandas, PyOgrio, Psycopg2, SQLAlchemy, Matplotlib, NumPy, Python-Dotenv.

---

## Protocolo de Replicación del Entorno

Siga las instrucciones secuenciales detalladas a continuación para inicializar el entorno de desarrollo local y ejecutar el pipeline:

### 1. Aprovisionamiento de Dependencias del Sistema
Clone el repositorio localmente e instale las librerías nativas binarias requeridas por el sistema operativo para el manejo de abstracciones espaciales y soporte de GDAL/PostGIS:

```bash
# Clonar el repositorio institucional
git clone https://github.com
cd censo-pipeline-geoanalytics

# Instalar dependencias espaciales core en sistemas basados en Ubuntu/Debian
sudo apt update && sudo apt install gdal-bin libgdal-dev postgis postgresql-18-postgis-3.6 -y
```

### 2. Inicialización del Servidor de Base de Datos
Acceda a su terminal interactiva de `psql` (o cliente DBMS equivalente) y cree de forma exclusiva la instancia de destino:

```sql
CREATE DATABASE censo_geolocalizado;
```

### 3. Configuración de Variables de Entorno Seguras
Genere un archivo denominado **`.env`** en el directorio raíz del proyecto para parametrizar las credenciales de red local, evitando la exposición de datos sensibles en el código fuente:

```ini
POSTGRES_DB=censo_geolocalizado
POSTGRES_USER=postgres
POSTGRES_PASSWORD=TuContraseñaServidorLocal
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
```

### 4. Despliegue del Entorno Virtual de Aislamiento
Inicialice el entorno virtual de Python, proceda con su activación e instale de forma determinista el catálogo de dependencias del pipeline:

```bash
# Generar el entorno virtual aislado bajo Python 3.11
python3.11 -m venv .venv

# Activar el contexto del entorno
source .venv/bin/activate

# Actualizar el gestor de paquetes e instalar dependencias indexadas
pip install --upgrade pip
pip install -r requirements.txt
```

### 5. Estructuración y Disposición de Data Assets
Descomprima los microdatos tabulares extraídos del sistema REDATAM y sitúe los componentes (carpeta de origen y mapas vectoriales) exactamente bajo la siguiente jerarquía física en la raíz del espacio de trabajo:

```text
tu-repositorio/
├── 66-salta-2022/              <-- Directorio con extractos CSV desestructurados
│   ├── geografia.csv
│   ├── persona.csv
│   ├── hogar.csv
│   └── vivienda.csv
├── radios2022c_vivVacias.gpkg   <-- Capa de datos geoespaciales vectoriales
├── .env
├── main.ipynb                  <-- Jupyter Notebook / Orquestador ejecutable
└── requirements.txt
```

### 6. Ejecución y Orquestación del Pipeline
Abra el archivo interactivo `main.ipynb` dentro de su entorno de desarrollo integrado (IDE). El pipeline ejecutará de forma secuencial y automatizada las siguientes fases:
1.  Inyección y registro de la extensión `postgis` en el esquema físico.
2.  Compilación del esquema de datos relacionales (DDL y constraints de clave foránea).
3.  Operación de carga masiva de datos vectoriales y tabulares mediante el protocolo de alta velocidad `COPY FROM STDIN`.
4.  Normalización, reproyección y cálculo de proyecciones geográficas a EPSG:4326 (WGS 84).
5.  Evaluación cuantitativa en la suite de auditoría interna de datos (QA).
6.  Generación de artefactos analíticos visuales en el directorio local.

---