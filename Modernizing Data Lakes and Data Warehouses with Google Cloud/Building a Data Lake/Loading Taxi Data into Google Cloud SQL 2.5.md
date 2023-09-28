### Activate Google Cloud Shell

* Google Cloud Shell provides command-line access to your Google Cloud resources.

*  We can list the active account name with the following command:

```bash
gcloud auth list
```

**  To list the project ID we just type the following command:
```bash
gcloud config list project
 ```

###  Preparing the environment

The environment variables that will be used later in the lab for the project ID and the storage bucket that will contain our data
```bash
export PROJECT_ID=$(gcloud info --format='value(config.project)')
export BUCKET=${PROJECT_ID}-ml
```
###  Create a Cloud SQL instance

1. I use the following commands to create a Cloud SQL instance

```bash
gcloud sql instances create taxi \
    --tier=db-n1-standard-1 --activation-policy=ALWAYS
```
2. For setting a root password for the Cloud SQL instance:

```bash
gcloud sql users set-password root --host % --instance taxi \
 --password Passw0rd

```
3. Let's create an environment variable with the IP address of the Cloud Shell
```bash
export ADDRESS=$(wget -qO - http://ipecho.net/plain)/32
```
4. Whitelist the Cloud Shell instance for management access to your SQL instance

```bash
gcloud sql instances patch taxi --authorized-networks $ADDRESS
```
Then press Y

5. Get the IP address of your Cloud SQL instance by running

```bash
MYSQLIP=$(gcloud sql instances describe \
taxi --format="value(ipAddresses.ipAddress)")
```

6. Check the variable MYSQLIP

```bash
echo $MYSQLIP
```

7. Create the taxi trips table by logging into the mysql command line interface

```bash 
mysql --host=$MYSQLIP --user=root \
      --password --verbose
```

8. When prompted for a password enter Passw0rd

9. Type the following command to create our database and table

```bash 
create database if not exists bts;
use bts;
drop table if exists trips;
create table trips (
  vendor_id VARCHAR(16),		
  pickup_datetime DATETIME,
  dropoff_datetime DATETIME,
  passenger_count INT,
  trip_distance FLOAT,
  rate_code VARCHAR(16),
  store_and_fwd_flag VARCHAR(16),
  payment_type VARCHAR(16),
  fare_amount FLOAT,
  extra FLOAT,
  mta_tax FLOAT,
  tip_amount FLOAT,
  tolls_amount FLOAT,
  imp_surcharge FLOAT,
  total_amount FLOAT,
  pickup_location_id VARCHAR(16),
  dropoff_location_id VARCHAR(16)
);

```

### Add data to Cloud SQL instance

1. Notice that we store  the New York City taxi trips CSV files in our cloud storage .
Let's copy it :)

```bash
gcloud storage cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_1.csv trips.csv-1
gcloud storage cp gs://cloud-training/OCBL013/nyc_tlc_yellow_trips_2018_subset_2.csv trips.csv-2
```
2. Then we connect to the mysql interactive console to load local infile data

```bash
mysql --host=$MYSQLIP --user=root  --password  --local-infile
```
When prompted for a password enter Passw0rd

3. In the mysql interactive console select the database then load the local CSV file data using local-infile

```bash
use bts;

LOAD DATA LOCAL INFILE 'trips.csv-1' INTO TABLE trips
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);

LOAD DATA LOCAL INFILE 'trips.csv-2' INTO TABLE trips
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 LINES
(vendor_id,pickup_datetime,dropoff_datetime,passenger_count,trip_distance,rate_code,store_and_fwd_flag,payment_type,fare_amount,extra,mta_tax,tip_amount,tolls_amount,imp_surcharge,total_amount,pickup_location_id,dropoff_location_id);


```

### Checking for data integrity

1. Query the trips table for unique pickup location region
```bash
select distinct(pickup_location_id) from trips;
```
2. let's find the min max trip_distance

```bash
select
  max(trip_distance),
  min(trip_distance)
from
  trips;
```
3. The number of trips that have distance 0

```bash
select count(*) from trips where trip_distance = 0;
```
4. let's investigate the payment_type

```bash
select
  payment_type,
  count(*)
from
  trips
group by
  payment_type;
```

