# Prepare Postgres

First I need to run the container with the database

```docker
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

Then I will make the connection to the database using pgcli

```bash
pgcli -h localhost -p 5432 -u root -d ny_taxi
```

Once I have the connection, I will load the data running the code in datapipeline.ipynb

# SQL questions.

## Question 3. Trip Segmentation Count

```sql
SELECT
    COUNT(CASE WHEN trip_distance <= 1 THEN 1 END) AS "Up to 1 mile",
    COUNT(CASE WHEN trip_distance > 1 AND trip_distance <= 3 THEN 1 END) AS "1 to 3 miles",
    COUNT(CASE WHEN trip_distance > 3 AND trip_distance <= 7 THEN 1 END) AS "3 to 7 miles",
    COUNT(CASE WHEN trip_distance > 7 AND trip_distance <= 10 THEN 1 END) AS "7 to 10 miles",
    COUNT(CASE WHEN trip_distance > 10 THEN 1 END) AS "Over 10 miles"
FROM
    green_taxi_trips
WHERE
    lpep_pickup_datetime >= '2019-10-01' AND
    lpep_pickup_datetime < '2019-11-01';
```



I got the following for this question:

```bash
+--------------+--------------+--------------+---------------+---------------+
| Up to 1 mile | 1 to 3 miles | 3 to 7 miles | 7 to 10 miles | Over 10 miles |
|--------------+--------------+--------------+---------------+---------------|
| 104830       | 198995       | 109642       | 27686         | 35201         |
+--------------+--------------+--------------+---------------+---------------+
```

## Question 4. Longest trip for each day

```sql
SELECT
    DATE(lpep_pickup_datetime) AS pickup_day,
    MAX(trip_distance) AS longest_trip_distance
FROM
    green_taxi_trips
WHERE
    lpep_pickup_datetime >= '2019-10-01' AND
    lpep_pickup_datetime < '2019-11-01'
GROUP BY
    DATE(lpep_pickup_datetime)
ORDER BY
    longest_trip_distance DESC
LIMIT 1;
```

I got the following for this question:

```bash
+------------+-----------------------+
| pickup_day | longest_trip_distance |
|------------+-----------------------|
| 2019-10-31 | 515.89                |
+------------+-----------------------+
```

## Question 5. Three biggest pickup zones

```sql
SELECT
    zpu."Zone" AS pickup_zone,
    SUM(t.total_amount) AS total_amount_sum
FROM
    green_taxi_trips t
JOIN
    zones zpu ON t."PULocationID" = zpu."LocationID"
WHERE
    DATE(t.lpep_pickup_datetime) = '2019-10-18'
GROUP BY
    zpu."Zone"
HAVING
    SUM(t.total_amount) > 13000
ORDER BY
    total_amount_sum DESC;
```



I got the following for this question:

```bash
+---------------------+--------------------+
| pickup_zone         | total_amount_sum   |
|---------------------+--------------------|
| East Harlem North   | 18686.680000000084 |
| East Harlem South   | 16797.26000000006  |
| Morningside Heights | 13029.79000000003  |
+---------------------+--------------------+
```

## Question 6. Largest tip

```sql
SELECT
    zdo."Zone" AS dropoff_zone,
    MAX(t.tip_amount) AS largest_tip
FROM
    green_taxi_trips t
JOIN
    zones zpu ON t."PULocationID" = zpu."LocationID"
JOIN
    zones zdo ON t."DOLocationID" = zdo."LocationID"
WHERE
    zpu."Zone" = 'East Harlem North' AND
    DATE(t.lpep_pickup_datetime) >= '2019-10-01' AND
    DATE(t.lpep_pickup_datetime) < '2019-11-01'
GROUP BY
    zdo."Zone"
ORDER BY
    largest_tip DESC
LIMIT 1;
```



I got the following for this question:

```bash
+--------------+-------------+
| dropoff_zone | largest_tip |
|--------------+-------------|
| JFK Airport  | 87.3        |
+--------------+-------------+
```
