### Create a new dataset to store our tables

the name of the  dataset is **fruit_store**

### Loading semi-structured JSON into BigQuery

1. Create a new table in the fruit_store dataset

2. Add the following details for the table:

* Source: Choose **Google Cloud Storage** in the Create table from dropdown.

* Select file from **GCS bucket** (type or paste the following): **data-insights-course/labs/optimizing-for-performance/shopping_cart.json**

* File format: **JSONL (Newline delimited JSON) {This will be auto-populated}**

* Schema: **Check Auto detect (Schema and input parameters)**.

Call the new table **"fruit_details"**.

### Creating your own arrays with ARRAY_AGG()

```bash
SELECT
  fullVisitorId,
  date,
  v2ProductName,
  pageTitle
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
ORDER BY date
```
* Now we can use **ARRAY_AGG()** function to aggregate our string values into an array.

```bash
SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(v2ProductName) AS products_viewed,
  ARRAY_AGG(pageTitle) AS pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
```
* then we use **ARRAY_LENGTH()** function to count the number of pages and products that were viewed

```bash
SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(v2ProductName)) AS num_products_viewed,
  ARRAY_AGG(pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(pageTitle)) AS num_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
```
* let's deduplicate the pages and products so we can see how many unique products were viewed. We'll simply add DISTINCT to our **ARRAY_AGG()**:

```bash
SELECT
  fullVisitorId,
  date,
  ARRAY_AGG(DISTINCT v2ProductName) AS products_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT v2ProductName)) AS distinct_products_viewed,
  ARRAY_AGG(DISTINCT pageTitle) AS pages_viewed,
  ARRAY_LENGTH(ARRAY_AGG(DISTINCT pageTitle)) AS distinct_pages_viewed
  FROM `data-to-insights.ecommerce.all_sessions`
WHERE visitId = 1501570398
GROUP BY fullVisitorId, date
ORDER BY date
```
###  Querying datasets that already have ARRAYs

* The BigQuery Public Dataset for Google Analytics **bigquery-public-data.google_analytics_sample** has many more fields and rows than our course dataset **data-to-insights.ecommerce.all_sessions**.
Run the below query to explore the available data and see if you can find fields with repeated values (arrays)

```bash
SELECT
  *
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398
```

* The amount of fields available in the **Google Analytics** schema can be overwhelming for our analysis. Let's try to query just the visit and page name fields like we did before:
```bash
SELECT
  visitId,
  hits.page.pageTitle
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398
```

### Introduction to STRUCTs
On the BigQuery go to the **bigquery-public-data** then select the dataset **google_analytics_sample**
then select the table **ga_sessions**

```bash
SELECT
  visitId,
  totals.*,
  device.*
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801`
WHERE visitId = 1501570398
LIMIT 10
```


#### Practice with STRUCTs and ARRAYs

* Try out the STRUCT syntax and note the different field types within the struct container
```bash
#standardSQL
SELECT STRUCT("Rudisha" as name, 23.4 as split) as runner
```

```bash
#standardSQL
SELECT STRUCT("Rudisha" as name, [23.4, 26.3, 26.4, 26.1] as splits) AS runner

```
* **Practice ingesting JSON data**
1. Create a new dataset titled **racing**.

2. Create a new table titled **race_results**.

3. Ingest this **Google Cloud Storage JSON file**: **data-insights-course/labs/optimizing-for-performance/race_results.json**


4. ource: **Google Cloud Storage** under Create table from dropdown.

5. Select file from GCS bucket: **data-insights-course/labs/optimizing-for-performance/race_results.json**

6. File format: **JSONL (Newline delimited JSON)**

7. In Schema, move the Edit as text slider and add the following:

```bash
[
    {
        "name": "race",
        "type": "STRING",
        "mode": "NULLABLE"
    },
    {
        "name": "participants",
        "type": "RECORD",
        "mode": "REPEATED",
        "fields": [
            {
                "name": "name",
                "type": "STRING",
                "mode": "NULLABLE"
            },
            {
                "name": "splits",
                "type": "FLOAT",
                "mode": "REPEATED"
            }
        ]
    }
]
```
* **Practice querying nested and repeated fields
```bash
#standardSQL
SELECT * FROM racing.race_results
```

```bash
#standardSQL
SELECT race, participants.name
FROM racing.race_results
CROSS JOIN
race_results.participants # full STRUCT name
```
```bash
#standardSQL
SELECT race, participants.name
FROM racing.race_results AS r, r.participants

```

```bash
#standardSQL
SELECT COUNT(participants.name) AS racer_count
FROM racing.race_results
```

```bash 
#standardSQL
SELECT COUNT(p.name) AS racer_count
FROM racing.race_results AS r, UNNEST(r.participants) AS p
```

```bash
#standardSQL
SELECT
  p.name,
  SUM(split_times) as total_race_time
FROM racing.race_results AS r
, UNNEST(r.participants) AS p
, UNNEST(p.splits) AS split_times
WHERE p.name LIKE 'R%'
GROUP BY p.name
ORDER BY total_race_time ASC;
```

```bash
#standardSQL
SELECT
  p.name,
  split_time
FROM racing.race_results AS r
, UNNEST(r.participants) AS p
, UNNEST(p.splits) AS split_time
WHERE split_time = 23.2;
```
