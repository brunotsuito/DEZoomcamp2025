# Week 1 Homework

## Question 1:
Run docker with the python:3.12.8 image in an interactive mode, use the entrypoint bash.

What's the version of pip in the image?

- 24.3.1
- 24.2.1
- 23.3.1
- 23.2.1

### Solution

#### Step 1: Run Docker with the Python Image
I ran the following command to start a Docker container with the `python:3.12.8` image in interactive mode and dropped into a `bash` shell:

```bash
docker run -it python:3.12.8 bash 
```
#### Step 2: Checking pip version
Inside the container, I ran the following command to check the pip version:
```bash
pip --version
```
#### The Output was:
```bash
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
```
So the correct pip version in the python:3.12.8 image is 24.3.1.
<br><br>


## Question 2:

Given the following docker-compose.yaml, what is the hostname and port that pgadmin should use to connect to the postgres database?
```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

    postgres:5433
    localhost:5432
    db:5433
    postgres:5432
    db:5432


### Solution

In Docker Compose, services can communicate with each other using their service names as hostnames and sinse the Postgres was named db, the hostname to pgadmin to conect is db.

With a few google search, PostgreSQL container exposes port 5432 internally defined by the posgres image. The 5433:5432 is the maps the container's port to the host's port. But for docker network, is should be used the container port.

so the answer is:

Hostname: db

Port: 5432
<br><br>


## Preparing Postgres

First starting the docker-compose where we start postgres and pgadmin (docker-compose.yaml)

```bash
docker-compose up -d
```

Then running the python script to ingest the greentrip url data

```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz"
docker run -it \
  --network=2_docker_sql_default \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pgdatabase \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_trip_hmwk \
    --url=${URL}
```

Digesting the taxi zone data

```python
import pandas as pd
from sqlalchemy import create_engine

engine = create_engine('postgresql://root:root@localhost:5432/ny_taxi')

!wget https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
df_zones = pd.read_csv('taxi_zone_lookup.csv')
df_zones.head()
df_zones.to_sql(name='zones', con=engine, if_exists='replace')
```
<br><br>


## Question 3:

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, respectively, happened:

1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles

Answers:

- 104,802; 197,670; 110,612; 27,831; 35,281
- 104,802; 198,924; 109,603; 27,678; 35,189
- 104,793; 201,407; 110,612; 27,831; 35,281
- 104,793; 202,661; 109,603; 27,678; 35,189
- 104,838; 199,013; 109,645; 27,688; 35,202

### Solution
1.
```sql
select 
	count(1)
from 
	green_trip_hmwk g
where
	g.lpep_pickup_datetime >= '2019-10-01' and g.lpep_pickup_datetime < '2019-11-01'
AND
	g.lpep_dropoff_datetime >= '2019-10-01' and g.lpep_dropoff_datetime < '2019-11-01'
AND
	trip_distance <= 1;
```
Output: 104802

2.
```sql
select 
	count(1)
from 
	green_trip_hmwk g
where
	g.lpep_pickup_datetime >= '2019-10-01' and g.lpep_pickup_datetime < '2019-11-01'
AND
	g.lpep_dropoff_datetime >= '2019-10-01' and g.lpep_dropoff_datetime < '2019-11-01'
AND
	trip_distance > 1 and trip_distance <= 3;
```
Output: 198924

3.
```sql
select 
	count(1)
from 
	green_trip_hmwk g
where
	g.lpep_pickup_datetime >= '2019-10-01' and g.lpep_pickup_datetime < '2019-11-01'
AND
	g.lpep_dropoff_datetime >= '2019-10-01' and g.lpep_dropoff_datetime < '2019-11-01'
AND
	trip_distance > 3 and trip_distance <= 7;
````
Output: 109603

4.
```sql
select 
	count(1)
from 
	green_trip_hmwk g
where
	g.lpep_pickup_datetime >= '2019-10-01' and g.lpep_pickup_datetime < '2019-11-01'
AND
	g.lpep_dropoff_datetime >= '2019-10-01' and g.lpep_dropoff_datetime < '2019-11-01'
AND
	trip_distance > 7 and trip_distance <= 10;
```
Output: 27678

5.
```sql

select 
	count(1)
from 
	green_trip_hmwk g
where
	g.lpep_pickup_datetime >= '2019-10-01' and g.lpep_pickup_datetime < '2019-11-01'
AND
	g.lpep_dropoff_datetime >= '2019-10-01' and g.lpep_dropoff_datetime < '2019-11-01'
AND
	trip_distance > 10;
```
Output: 35189


## Question 4:

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance.

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31

### Solution

```sql
select 
	cast(lpep_pickup_datetime as DATE) as "Day",
	max(trip_distance) as longest_trip
from 
	green_trip_hmwk g
group by
	1
order by 
	longest_trip desc
Limit 1;
```

Output:
Day: 2019-10-31
Distance: 515.89
