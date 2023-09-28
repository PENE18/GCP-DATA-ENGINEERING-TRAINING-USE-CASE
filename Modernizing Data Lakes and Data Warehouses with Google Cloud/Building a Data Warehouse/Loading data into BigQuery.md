### Create a new dataset to store tables

To create a dataset, we click on the View actions icon (the three vertical dots) next to our project ID and select Create dataset.

Next, name our Dataset ID **nyctaxi** and leave all other options at their default values, and then click Create dataset.

###  Ingest a new dataset from a CSV

1. In my local machine i have the **NYC taxi 2018 trips** data .

2. In the BigQuery Console, Select the **nyctaxi** dataset then click Create Table

3. Specify the below table options:

**Source**:

Create table from: Upload
Choose File: select the file you have on your local machine
File format: CSV
Destination:

Table name: **2018trips** Leave all other settings at default.
Schema:

Check Auto Detect (tip: Not seeing the checkbox? Ensure the file format is CSV and not Avro)
Advanced Options

#### Running SQL Queries

run the query to list the top 5 most expensive trips of the year

```bash
#standardSQL
SELECT
  *
FROM
  nyctaxi.2018trips
ORDER BY
  fare_amount DESC
LIMIT  5
```

### Ingest a new dataset from Google Cloud Storage

In our cloud storage we have **nyc_tlc_yellow_trips_2018_subset_2.csv**

In your Cloud Shell type the following command to ingest csv file to nyctaxi.2018trips

```bash
bq load \
--source_format=CSV \
--autodetect \
--noreplace  \
nyctaxi.2018trips \
gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv
```

### Create tables from other tables with DDL
The 2018trips table now has trips from throughout the year. What if you were only interested in January trips?

In the Query Editor, run the following CREATE TABLE command

```bash
#standardSQL
CREATE TABLE
  nyctaxi.january_trips AS
SELECT
  *
FROM
  nyctaxi.2018trips
WHERE
  EXTRACT(Month
  FROM
    pickup_datetime)=1;

```
