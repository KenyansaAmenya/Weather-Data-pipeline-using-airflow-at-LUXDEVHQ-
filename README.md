# Weather-Data-pipeline-using-airflow-at-LUXDEVHQ-
# Airflow Weather Data Pipeline Explanation
This code defines an Apache Airflow DAG (Directed Acyclic Graph) that fetches weather data from the OpenWeatherMap API and stores it in a PostgreSQL database. Here's a breakdown of what it does and what problems it solves:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta
import requests
import psycopg2
import json
import pandas as pd
import http.client

# PostgreSQL configuration
DB_CONFIG = {
    "host": "139.59.57.88",
    "port": 5400,
    "dbname": "bcp",
    "user": "bcp",
    "password": "developer@123",
    "schema": "weather_ke"  # Using your specified schema name
}

# List of cities and countries to fetch weather for
cities = [("Nairobi", "KE"), ("London", "GB")]

def fetch_and_store_weather():
    """
    Fetches weather data from OpenWeatherMap API and stores it in PostgreSQL
    """
    conn = None
    cursor = None
    try:
        # Connect to the database
        conn = psycopg2.connect(
            dbname=DB_CONFIG["dbname"],
            user=DB_CONFIG["user"],
            password=DB_CONFIG["password"],
            host=DB_CONFIG["host"],
            port=DB_CONFIG["port"]
        )
        cursor = conn.cursor()

        # Create schema if it doesn't exist
        cursor.execute(f"CREATE SCHEMA IF NOT EXISTS {DB_CONFIG['schema']}")
        
        # Set the schema search path
        cursor.execute(f"SET search_path TO {DB_CONFIG['schema']}")

        # Create table if it doesn't exist
        cursor.execute(f"""
            CREATE TABLE IF NOT EXISTS {DB_CONFIG['schema']}.weather_data (
                id SERIAL PRIMARY KEY,
                city VARCHAR(100),
                country VARCHAR(100),
                description VARCHAR(100),
                temperature FLOAT,
                timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        """)
        conn.commit()

        # Fetch and store weather for each city
        for city, country in cities:
            params = {
                "q": f"{city},{country}",
                "appid": "API *******",
                "units": "metric"
            }

            response = requests.get(
                "https://api.openweathermap.org/data/2.5/weather",
                params=params,
                timeout=10
            )
            data = response.json()

            if response.status_code == 200:
                description = data['weather'][0]['description']
                temperature = data['main']['temp']

                cursor.execute(
                    f"""INSERT INTO {DB_CONFIG['schema']}.weather_data 
                        (city, country, description, temperature)
                        VALUES (%s, %s, %s, %s)
                    """,
                    (city, country, description, temperature)
                )
                print(f"Successfully stored weather for {city}, {country}")
            else:
                print(f"Failed to fetch weather for {city}: {data.get('message')}")

        conn.commit()

    except Exception as e:
        print(f"Error occurred: {str(e)}")
        if conn:
            conn.rollback()
        raise  # Re-raise the exception to mark task as failed in Airflow
    finally:
        if cursor:
            cursor.close()
        if conn:
            conn.close()

# DAG default arguments
default_args = {
    'owner': 'airflow',
    'start_date': datetime(2024, 1, 1),
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
    'catchup': False
}

# Define the DAG
with DAG(
    dag_id='weather_data_pipeline',
    default_args=default_args,
    schedule_interval='@daily',
    description='A pipeline that fetches weather data and stores it in PostgreSQL',
    tags=['weather', 'data_engineering']
) as dag:
    
    fetch_and_store_task = PythonOperator(
        task_id='fetch_and_store_weather_data',
        python_callable=fetch_and_store_weather
    )
```
# What the Code Does
1. Imports: The code imports necessary libraries including Airflow components, database connectors, and HTTP request tools.

2. Database Configuration: Sets up connection details for PostgreSQL including host, port, credentials, and schema name.

3. City List: Defines which cities to fetch weather for (Nairobi, Kenya and London, UK).

4. Main Function (fetch_and_store_weather):

- Connects to PostgreSQL

- Creates a schema and table if they don't exist

- For each city, fetches current weather data from OpenWeatherMap API

- Stores the city name, country, weather description, and temperature in the database

- Handles errors and ensures proper connection cleanup

5. Airflow DAG Configuration:

- Sets default arguments (owner, start date, retry settings)

- Defines the DAG with a daily schedule

- Creates a single task that calls the main function

# Problems This Solves
- Automated Data Collection: Automates the daily collection of weather data, eliminating manual work.

- Data Centralization: Stores weather data from multiple locations in a single database for easy analysis.

- Historical Tracking: By running daily, it builds a historical record of weather patterns over time.

- Reliability: Includes error handling and retry logic to deal with temporary API or database issues.

- Scalability: The structure makes it easy to add more cities by simply expanding the cities list.

# Potential Use Cases
- Weather Analysis: Track temperature trends over time in different locations

- Application Backend: Provide historical weather data for a weather application

- Research: Support climate studies or urban planning decisions

- Business Intelligence: Correlate weather data with business metrics (e.g., retail sales)

# Key Components
- PostgreSQL: Used for reliable, structured data storage

- OpenWeatherMap API: Provides the weather data

- Airflow: Orchestrates the daily execution and handles scheduling

- Python: Implements the data fetching and storage logic

The pipeline runs daily (as per the schedule_interval='@daily' parameter) to keep the weather data up to date.

# The output
![2025-05-31 16 46 47](https://github.com/user-attachments/assets/8794cb0f-c83b-4f8a-85cd-8bdf4922d658)
