# Exercise notebooks from Kaggle micro courses
for quick review and interview prep: 
https://www.kaggle.com/learn/overview
# Contents

## SQL
### Intro to SQL ✅ 

### Advanced SQL
*(in-progress)*

---
*(upcoming)*
## Python
## Pandas
## Data Visualization
## Feature Engineering
## Machine Learning
### Intro, Intermediate, Explanability


----

## Notes

## SQL
### Intro to SQL
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

***Use COUNT(1) to count the rows in each group*** -- More readable, also scans less data than if supplied column names.

GROUP BY takes the name of one or more columns, and treats all rows with the same value in that column as a single group when you apply aggregate functions like COUNT().

*HAVING is used in combination with GROUP BY* to ignore groups that don't meet certain criteria. In this example, only include groups that have more than one ID in them:
![having example](https://i.imgur.com/2ImXfHQ.png) (image source: https://www.kaggle.com/dansbecker/group-by-having-count)

        Another example (*doesn't work if put COUNT(1) >= 175 under the WHERE clause*):
        ```
         SELECT indicator_code, indicator_name, COUNT(1) AS num_rows
                   FROM `bigquery-public-data.world_bank_intl_education.international_education`
                   WHERE year = 2016
                   GROUP BY indicator_name, indicator_code
                   HAVING COUNT(1) >= 175
                   ORDER BY COUNT(1) DESC
        ```

Note: It doesn't make sense to use GROUP BY without an aggregate function, because GROUP BY tells SQL how to apply aggregate functions (like COUNT()). Similarly, if you have any GROUP BY clause, then all variables must be passed to either a GROUP BY command, or an aggregation function. In other words, use GROUP BY whenever you have functions like avg(), count(), max(), min(), etc. Basically it groups all identical records as single entity based on your selection of column

On its own, AS is a convenient way to clean up the data returned by your query. It's even more powerful when combined with WITH in what's called a "common table expression".

**GROUP BY 1** *means group by first column from SELECT. The same pattern could be used for ORDER BY.* For example:
```
select (salary * months)as earnings, count(*) 
from employee 
group by 1 
order by earnings desc limit 1;

- outputs 2 columns: 
- max earnings and number of employees with maximum earnings
```

*Use CAST AS DECIMAL, ROUND, or TRUNCATE to limit number of decimals:*
```
select CAST(MAX(LAT_N) AS DECIMAL (10,4)) from station
where LAT_N < 137.2345;
```
https://stackoverflow.com/questions/2030986/sql-server-cast-and-rounding 

ROUND changes the value, not the type. https://www.w3schools.com/sql/func_sqlserver_round.asp 
The CAST() function converts a value (of any type) into a specified datatype.

* AS and WITH

A **common table expression (or CTE)** is a temporary table that you return within your query. CTEs are helpful for splitting your queries into readable chunks, and you can write queries against them.
![CTE example](https://i.imgur.com/3xQZM4p.png)
The WITH...AS incomplete query creates a CTE that we can then refer to (as Seniors) while writing the rest of the query.
We can finish the query by pulling the information that we want from the CTE (return all the IDs in this example). 

It's important to note that *CTEs only exist inside the query where you create them, and you can't reference them in later queries.* So, any query that uses a CTE is always broken into two parts: 

        - first, we create the CTE, and then 
        
        - we write a query that uses the CTE.

Common table expressions (CTEs) let you *shift a lot of your data cleaning into SQL*. That's an especially good thing in the case of **BigQuery, because it is vastly faster than doing the work in Pandas.** 

Exercise (AS & WITH notebook has a lot more data exploration than previous ones, see notebook for example of more complex query with CTE): 

        - The SQL code to **SELECT** the year from `trip_start_timestamp` is <code>SELECT EXTRACT(YEAR FROM trip_start_timestamp)</code>

        - The **FROM** field can be a little tricky. The format is:
            1. A backick (the symbol \`).
            2. The project name. In this case it is `bigquery-public-data`.
            3. A period.
            4. The dataset name. In this case, it is `chicago_taxi_trips`.
            5. A period.
            6. The table name.
            7. A backtick (the symbol \`).
                
 

* JOIN Data
Example joining on PetID column, also using aliases for table names
![JOIN example](https://i.imgur.com/fLlng42.png)

> In general, when joining tables, it's a good habit to specify which table each of your columns comes from. That way, you don't have to pull up the schema every time you go back to read the query.

![JOIN example](https://i.imgur.com/QeufD01.png)
We begin with the JOIN (highlighted in blue above). This specifies the sources of data and how to join them. We use ON to specify that we combine the tables by matching the values in the repo_name columns in the tables.

Next, we'll talk about SELECT and GROUP BY (highlighted in yellow). The GROUP BY breaks the data into a different group for each license, before we COUNT the number of rows in the sample_files table that corresponds to each license. (Remember that you can count the number of rows with COUNT(1).)

Finally, the ORDER BY (highlighted in purple) sorts the results so that licenses with more files appear first.

Can use % as wildcard when selecting rows using text (WHERE column_name LIKE 'text_pattern'):
```
query = """
        SELECT * 
        FROM `bigquery-public-data.pet_records.pets` 
        WHERE Name LIKE '%ipl%'
        """
```


Another example of aliasing and JOIN:
```
SELECT c.*, f.name AS country_name FROM facts as f
INNER JOIN cities AS c
ON f.id = c.facts_id
LIMIT 5;
```
-first line, selected f.name column is followed by its own alias, before FROM facts as f (alias of table)
    
    
The following two queries, one using a left join and one using a right join, produce identical results.
```
SELECT f.name country, c.name city
FROM facts f
LEFT JOIN cities c ON c.facts_id = f.id
LIMIT 5;

SELECT f.name country, c.name city
FROM cities c
RIGHT JOIN facts f ON f.id = c.facts_id
LIMIT 5;
```
The main reason a right join would be used is when you are joining more than two tables. In these cases, using a right join is preferable because it can avoid restructuring your whole query to join one table. 
