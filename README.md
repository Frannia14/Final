# Final
Trabajo Final Frannia -20210058

Enlace al video:https://miucateciedu.sharepoint.com/sites/Tareas550/Documentos%20compartidos/General/Recordings/Reuni%C3%B3n%20en%20_General_-20241206_223518-Grabaci%C3%B3n%20de%20la%20reuni%C3%B3n.mp4?web=1&referrer=Teams.TEAMS-ELECTRON&referrerScenario=MeetingChicletGetLink.view

# Airflow + dbt + Snowflake Integration

Este proyecto configura un entorno de **Airflow** para ejecutar proyectos dbt con Snowflake como base de datos.

## Estructura del Proyecto

dbt_airflow/ │ ├── dags/ │ └── my_cosmos_dag.py # DAG de Airflow │ ├── data/ │ ├── clientes.csv # Datos de clientes │ ├── reservas_1.csv # Primer archivo de reservas │ └── reservas_2.csv # Segundo archivo de reservas │ ├── models/ │ ├── transform/ │ │ ├── combinado_bookings.sql │ │ ├── customer.sql │ │ └── prepped_data.sql │ └── analysis/ │ ├── hotel_count_by_day.sql │ └── twenty_day_avg_cost.sql │ ├── requirements.txt # Dependencias ├── Dockerfile # Configuración de Docker └── README.md 


## Requisitos

- **Docker**: Para ejecutar los contenedores de Airflow.
- **Astro CLI**: Para configurar Airflow localmente.

## Preparación de Archivos CSV

1. **reservas_1.csv**
    ```csv
    id,booking_reference,hotel,booking_date,cost
    1,232323231,Pan Pacific,2021-03-19,100
    1,232323232,Fullerton,2021-03-20,200
    ```
   
2. **reservas_2.csv**
    ```csv
    id,booking_reference,hotel,booking_date,cost
    2,332323231,Fullerton,2021-03-19,100
    2,332323232,Jackson Square,2021-03-20,300
    ```

3. **clientes.csv**
    ```csv
    id,first_name,last_name,birthdate,membership_no
    1,jim,jone,1989-03-19,12334
    ```

## Creación de Modelos dbt

1. **combined_bookings.sql**
    ```sql
    {{ dbt_utils.union_relations(
        relations=[ref('bookings_1'), ref('bookings_2')]
    ) }}
    ```

2. **customer.sql**
    ```sql
    SELECT ID, FIRST_NAME, LAST_NAME, birthdate
    FROM {{ ref('customers') }}
    ```

3. **prepped_data.sql**
    ```sql
    SELECT A.ID, FIRST_NAME, LAST_NAME, BOOKING_REFERENCE, HOTEL, BOOKING_DATE, COST
    FROM {{ ref('customer') }} A
    JOIN {{ ref('combined_bookings') }} B ON A.ID = B.ID
    ```

## Dockerfile

Modifica el archivo `Dockerfile` para configurar un entorno virtual para dbt:

```Dockerfile
FROM quay.io/astronomer/astro-runtime:9.1.0-python-3.9-base

RUN python -m venv dbt_venv && source dbt_venv/bin/activate && pip install --no-cache-dir

## 
Aquí tienes una versión más compacta del README.mdpara que sea más adecuada para GitHub:

reducción

Copiar código
# Airflow + dbt + Snowflake Integration

Este proyecto configura un entorno de **Airflow** para ejecutar proyectos **dbt** con **Snowflake** como base de datos.

## Estructura del Proyecto

dbt_airflow/ │ ├── dags/ │ └── my_cosmos_dag.py # DAG de Airflow │ ├── data/ │ ├── clientes.csv # Datos de clientes │ ├── reservas_1.csv # Primer archivo de reservas │ └── reservas_2.csv # Segundo archivo de reservas │ ├── models/ │ ├── transform/ │ │ ├── combinado_bookings.sql │ │ ├── customer.sql │ │ └── prepped_data.sql │ └── analysis/ │ ├── hotel_count_by_day.sql │ └── twenty_day_avg_cost.sql │ ├── requirements.txt # Dependencias ├── Dockerfile # Configuración de Docker └── README.md # Este archivo

Perla

Copiar código

## Requisitos

- **Docker**: Para ejecutar los contenedores de Airflow.
- **Astro CLI**: Para configurar Airflow localmente.

## Preparación de Archivos CSV

1. **reservas_1.csv**
    ```csv
    id,booking_reference,hotel,booking_date,cost
    1,232323231,Pan Pacific,2021-03-19,100
    1,232323232,Fullerton,2021-03-20,200
    ```
   
2. **reservas_2.csv**
    ```csv
    id,booking_reference,hotel,booking_date,cost
    2,332323231,Fullerton,2021-03-19,100
    2,332323232,Jackson Square,2021-03-20,300
    ```

3. **clientes.csv**
    ```csv
    id,first_name,last_name,birthdate,membership_no
    1,jim,jone,1989-03-19,12334
    ```

## Creación de Modelos dbt

1. **combined_bookings.sql**
    ```sql
    {{ dbt_utils.union_relations(
        relations=[ref('bookings_1'), ref('bookings_2')]
    ) }}
    ```

2. **customer.sql**
    ```sql
    SELECT ID, FIRST_NAME, LAST_NAME, birthdate
    FROM {{ ref('customers') }}
    ```

3. *prepped_data.sql
    ```sql
    SELECT A.ID, FIRST_NAME, LAST_NAME, BOOKING_REFERENCE, HOTEL, BOOKING_DATE, COST
    FROM {{ ref('customer') }} A
    JOIN {{ ref('combined_bookings') }} B ON A.ID = B.ID
    ```

## Dockerfile

Modifica el archivo `Dockerfile` para configurar un entorno virtual para dbt:

```Dockerfile
FROM quay.io/astronomer/astro-runtime:9.1.0-python-3.9-base

RUN python -m venv dbt_venv && source dbt_venv/bin/activate && pip install --no-cache-dir


##Configuración del DAG en Airflow
Crea my_cosmos_dag.pyen dags/:
from datetime import datetime
import os
from cosmos import DbtDag, ProjectConfig, ProfileConfig, ExecutionConfig
from cosmos.profiles import SnowflakeUserPasswordProfileMapping
from pathlib import Path

dbt_project_path = Path("/usr/local/airflow/dags/dbt/cosmosproject")

profile_config = ProfileConfig(profile_name="default", target_name="dev", 
                               profile_mapping=SnowflakeUserPasswordProfileMapping(conn_id="snowflake_default", 
                               profile_args={"database": "demo_dbt", "schema": "public"}))

dbt_snowflake_dag = DbtDag(project_config=ProjectConfig(dbt_project_path),
                            profile_config=profile_config,
                            execution_config=ExecutionConfig(dbt_executable_path=f"{os.environ['AIRFLOW_HOME']}/dbt_venv/bin/dbt"),
                            schedule_interval="@daily", start_date=datetime(2023, 9, 10), catchup=False, dag_id="dbt_snowflake_dag")

