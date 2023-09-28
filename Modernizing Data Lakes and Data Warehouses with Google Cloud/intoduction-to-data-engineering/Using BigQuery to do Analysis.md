### Explore bicycle rental data 
On the BigQuery console, select bigquery-public-data > new_york_citibike > citibike_trips table.

** On the query Editor we run our firt query query1 which return typical duration for the 10 most common one-way rentals

** Then we run a new query with return the  total distance traveled by each bicycle in the dataset. Note that the query limits the results to only top 5 

###  Explore the weather dataset 

On BigQuery Console, select the newly added bigquery-public-data project and select ghcn_d > ghcnd_2015

** Run a new query that will return the rainfall (in mm) for all days in 2015 from a weather station in New York whose id is provided in the query

### Find correlation between rain and bicycle rentals

We apply a joining between the bicycle rentals data against weather data to learn whether there are fewer bicycle rentals on rainy days

** For that let's run the query3 for getting the correlation.
