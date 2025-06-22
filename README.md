# NYC Green Taxi Data Pipeline 

An end-to-end data engineering project that builds a robust pipeline to ingest, store, and analyze NYC Green Taxi trip data. This project leverages containerized services, cloud infrastructure provisioning, and analytical querying to demonstrate key data engineering concepts.


## Project Overview

This project automates the ingestion of NYC Green Taxi trip data into a PostgreSQL database and enables structured querying through pgAdmin and SQL. It uses Docker for containerization, Python for ingestion, and Terraform for cloud provisioning on GCP.

Key Capabilities:
- Ingest raw CSV data from a public source
- Transform and store structured data in PostgreSQL
- Use pgAdmin for database management
- Query data to generate business insights
- Provision cloud resources (GCS bucket & BigQuery dataset) using Terraform


##  Architecture

```
        +----------------+
        | Public NYC TLC |
        +-------+--------+
                |
                v
    +------------------------+
    | Python Ingest Script   |  --> Ingests CSV to Postgres
    +------------------------+
                |
                v
        +---------------+            +----------------+
        | PostgreSQL    | <--------> | pgAdmin (GUI)  |
        +---------------+            +----------------+

        +-----------------------------+
        | Jupyter (zones ingestion)   |
        +-----------------------------+

        +---------------------------+
        | Terraform (GCP Setup)    |
        +---------------------------+
```


## Tech Stack

- **Docker** & **Docker Compose** – for container orchestration
- **PostgreSQL** & **pgAdmin** – for relational data storage and inspection
- **Python** & **Pandas** – for data ingestion and transformation
- **Jupyter Notebook** – for structured data enrichment
- **Terraform** – for provisioning GCP cloud infrastructure
- **SQL** – for analytics and business insight extraction


## Steps to Ingest Green Taxi Trips Data

### Step 1: Set up PostgreSQL and pgAdmin using Docker Compose
- Create a `docker-compose.yaml` file that defines two services:
  - `pgdatabase`: the PostgreSQL database
  - `pgadmin`: for web-based database management
- Start the services:

```bash
docker-compose up -d
```

### Step 2: Access pgAdmin
- Log in using the credentials defined in `docker-compose.yaml` on [http://localhost:8080](http://localhost:8080)
- Register a new server:
  - **Host name/address**: `pgdatabase`
  - **Port**: `5432`
  - **Username**: `root`
  - **Password**: `root`


### Step 3: Create the Python Ingestion Script
- Create a script `ingest_data.py` that:
  - Downloads the dataset from a URL
  - Reads and processes the CSV file
  - Connects to the Postgres database
  - Creates a table (if it doesn’t exist)
  - Loads the data into the table


### Step 4: Build the Docker Image for Ingestion
- Write a `Dockerfile` that installs Python and required libraries
   - pandas: for data processing
   - sqlalchem: for database connections
   - psycopg2: PostgreSQL driver to connect from Python
- Build the image using the following command:

```bash
docker build -t taxi_ingest:v001 .
```
Note: It must be located in the same directory where I am running docker build)

### Step 5: Run the Ingestion Container
- Use the command below to ingest the Green Taxi CSV file into the Postgres container:

```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"

docker run -it \
  --network=project1_default \
  taxi_ingest:v001 \
  --user=root \
  --password=root \
  --host=pgdatabase \
  --port=5432 \
  --db=ny_taxi \
  --table_name=green_taxi_trips \
  --url="$URL"
```
### Step 6: Load zone data using Jupter Notebook
I use jupter to ingest zones data, please see ***ingest_zone_lookup.ipynb***

And check on pgAdmin if datas are loaded properly. 

![image](https://github.com/user-attachments/assets/29fc5643-3a6d-4e11-8d33-4cf92c0cfa84)


## 📊 Analytics Queries

### 1. Trip Segmentation by Distance

```sql
SELECT 
  CASE 
    WHEN trip_distance <= 1 THEN '1. Up to 1 mile'
    WHEN trip_distance > 1 AND trip_distance <= 3 THEN '2. In between 1 and 3 miles'
    WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3. In between 3 and 7 miles'
    WHEN trip_distance > 7 AND trip_distance <= 10 THEN '4. In between 7 and 10 miles'
    WHEN trip_distance > 10 THEN '5. Over 10 miles'
  END AS distance_category,
  COUNT(*) 
FROM green_taxi_trips
WHERE DATE(lpep_pickup_datetime) >= '2019-10-01'
  AND DATE(lpep_pickup_datetime) < '2019-11-01'
  AND DATE(lpep_dropoff_datetime) >= '2019-10-01'
  AND DATE(lpep_dropoff_datetime) < '2019-11-01'
GROUP BY distance_category;
```

### 2. Longest Trip by Day

```sql
SELECT
    DATE(lpep_pickup_datetime) AS pickup_date,
    MAX(trip_distance) AS max_distance
FROM green_taxi_trips
GROUP BY DATE(lpep_pickup_datetime)
ORDER BY max_distance DESC
LIMIT 1;
```

### 3. Highest Revenue Pickup Zones (October 18)

```sql
WITH CTE AS (
  SELECT
      lpep_pickup_datetime,
      total_amount,
      zpu."Zone" AS pickup_loc
  FROM green_taxi_trips t 
  LEFT JOIN zones zpu ON t."PULocationID" = zpu."LocationID"
  WHERE DATE(lpep_pickup_datetime) = '2019-10-18'
)
SELECT SUM(total_amount) AS revenue, pickup_loc
FROM CTE
GROUP BY pickup_loc
HAVING SUM(total_amount) > 13000;
```

### 4. Largest Tip from East Harlem North (Oct 2019)

```sql
WITH CTE AS (
  SELECT
      tip_amount,
      zpu."Zone" AS pickup_zone,
      zdo."Zone" AS dropoff_zone
  FROM green_taxi_trips t 
  LEFT JOIN zones zpu ON t."PULocationID" = zpu."LocationID"
  LEFT JOIN zones zdo ON t."DOLocationID" = zdo."LocationID"
  WHERE EXTRACT(MONTH FROM lpep_pickup_datetime) = 10
    AND zpu."Zone" = 'East Harlem North'
)
SELECT MAX(tip_amount) AS max_tip, dropoff_zone
FROM CTE
GROUP BY dropoff_zone
ORDER BY max_tip DESC
LIMIT 1;
```

## Data Source

- NYC TLC Green Taxi Trip Records  
  [https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)



## 👤 Author

**YITIAN WANG**  
[LinkedIn](www.linkedin.com/in/yitian-w-de) • [GitHub]([https://github.com/scarlett-de])



## 🧳 Skills Demonstrated

- Docker and container orchestration
- Custom ingestion scripts with Python
- Relational data modeling and ingestion
- SQL analytics and business insight generation

