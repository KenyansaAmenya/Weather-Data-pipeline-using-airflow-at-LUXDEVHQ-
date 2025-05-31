# Weather-Data-pipeline-using-airflow-at-LUXDEVHQ-
# Airflow Weather Data Pipeline Explanation
This code defines an Apache Airflow DAG (Directed Acyclic Graph) that fetches weather data from the OpenWeatherMap API and stores it in a PostgreSQL database. Here's a breakdown of what it does and what problems it solves:

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
