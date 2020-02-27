# Exercise notebooks from Kaggle micro courses https://www.kaggle.com/learn/overview
# Contents

## SQL
### Intro to SQL
* SQL and BigQuery

### Advanced SQL

---

## Python
## Pandas
## Data Visualization
## Feature Engineering
## Machine Learning
### Intro, Intermediate, Explanability


----

## Notes
* SQL and Google BigQuery

When working on big datasets, can estimate the size of any query before running it. Here is an example using the (very large!) Hacker News dataset. To see how much data a query will scan, we create a QueryJobConfig object and set the dry_run parameter to True.

```
# Query to get the score column from every row where the type column has value "job"
query = """
        SELECT score, title
        FROM `bigquery-public-data.hacker_news.full`
        WHERE type = "job" 
        """

# Create a QueryJobConfig object to estimate size of query without running it
dry_run_config = bigquery.QueryJobConfig(dry_run=True)

# API request - dry run query to estimate costs
dry_run_query_job = client.query(query, job_config=dry_run_config)

print("This query will process {} bytes.".format(dry_run_query_job.total_bytes_processed))


# Only run the query if it's less than 1 MB  
ONE_MB = 1000*1000   #ONE_GB = 1000*1000*1000
safe_config = bigquery.QueryJobConfig(maximum_bytes_billed=ONE_MB)

# Set up the query (will only run if it's less than 1 MB)
safe_query_job = client.query(query, job_config=safe_config)

# API request - try to run the query, and return a pandas DataFrame
safe_query_job.to_dataframe()
```

* GROUP BY, HAVING, and COUNT
GROUP BY in SQL is similar to [`groupby()`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.groupby.html) in pandas. **But BigQuery works quickly with far larger datasets.**

COUNT() is an example of an aggregate function, which takes many values and returns one. (Other examples of aggregate functions include SUM(), AVG(), MIN(), and MAX().) If you pass COUNT() the name of a column, it will return the number of entries in that column.

GROUP BY takes the name of one or more columns, and treats all rows with the same value in that column as a single group when you apply aggregate functions like COUNT().

HAVING is used in combination with GROUP BY to ignore groups that don't meet certain criteria:
![having example](https://i.imgur.com/2ImXfHQ.png) 
