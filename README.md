# My-SQL-query
this is some of my SQL queries that show my ability to clean, prepare and manipulate data


# SQL Queries Documentation

## Aggregation, Filtering, and Analysis Queries

```sql
-- 1. Aggregation with HAVING and CASE Statement
SELECT
    Warehouse.warehouse_id,
    CONCAT(Warehouse.state, ': ', Warehouse.warehouse_alias) AS warehouse_name,
    COUNT(Orders.order_id) AS number_of_orders,
    (SELECT COUNT(*) FROM porfolio-project-432006.warehouse_orders.orders AS Orders) AS total_orders,
    CASE
        WHEN COUNT(Orders.order_id)/(SELECT COUNT(*) FROM porfolio-project-432006.warehouse_orders.orders AS Orders) <= 0.20
        THEN 'Fulfilled 0-20% of Orders'
        WHEN COUNT(Orders.order_id)/(SELECT COUNT(*) FROM porfolio-project-432006.warehouse_orders.orders AS Orders) > 0.20
            AND COUNT(Orders.order_id)/(SELECT COUNT(*) FROM porfolio-project-432006.warehouse_orders.orders AS Orders) <= 0.60
        THEN 'Fulfilled 21-60% of Orders'
        ELSE 'Fulfilled more than 60% of Orders'
    END AS fulfillment_summary
FROM porfolio-project-432006.warehouse_orders.warehouse AS Warehouse
LEFT JOIN porfolio-project-432006.warehouse_orders.orders AS Orders
    ON Orders.warehouse_id = Warehouse.warehouse_id
GROUP BY
    Warehouse.warehouse_id,
    warehouse_name
HAVING
    COUNT(Orders.order_id) > 0;

-- 2. Counting Percentage of NULL Values
WITH ColumnStats AS (
    SELECT
        'CustomerID' AS column_name, COUNTIF(CustomerID IS NULL) AS null_count, COUNT(*) AS total_count
    FROM `porfolio-project-432006.Adventure_Work_Sales.JoinedData`
    UNION ALL
    SELECT
        'CustomerFirstName' AS column_name, COUNTIF(CustomerFirstName IS NULL) AS null_count, COUNT(*) AS total_count
    FROM `porfolio-project-432006.Adventure_Work_Sales.JoinedData`
)
SELECT
    column_name,
    null_count,
    total_count,
    (null_count / total_count) * 100 AS null_percentage
FROM ColumnStats;

-- 3. Removing NULL Values
CREATE OR REPLACE TABLE `porfolio-project-432006.Adventure_Work_Sales.CleanedDataFinal` AS
SELECT *
FROM `porfolio-project-432006.Adventure_Work_Sales.CleanedData`
WHERE
    OrderDate IS NOT NULL
    AND ProductName IS NOT NULL
    AND TotalDue IS NOT NULL;

-- 4. Subquery for Comparison
SELECT
    starttime,
    start_station_id,
    tripduration,
    (
        SELECT ROUND(AVG(tripduration), 2)
        FROM bigquery-public-data.new_york_citibike.citibike_trips
        WHERE start_station_id = outer_trips.start_station_id
    ) AS avg_duration_for_station,
    ROUND(tripduration - (
        SELECT AVG(tripduration)
        FROM bigquery-public-data.new_york_citibike.citibike_trips
        WHERE start_station_id = outer_trips.start_station_id), 2) AS difference_from_avg
FROM bigquery-public-data.new_york_citibike.citibike_trips AS outer_trips
ORDER BY difference_from_avg DESC
LIMIT 25;

-- 5. Subquery within Subquery to Find Top 5
SELECT
    tripduration,
    start_station_id
FROM bigquery-public-data.new_york_citibike.citibike_trips
WHERE start_station_id IN
(
    SELECT
        start_station_id
    FROM
    (
        SELECT
            start_station_id,
            AVG(tripduration) AS avg_duration
        FROM bigquery-public-data.new_york_citibike.citibike_trips
        GROUP BY start_station_id
    ) AS top_five
    ORDER BY avg_duration DESC
    LIMIT 5
);

-- 6. Simple Join Statement
SELECT
    seasons.market AS university,
    seasons.name AS team_name,
    mascots.mascot AS team_mascot,
    AVG(seasons.wins) AS avg_wins,
    AVG(seasons.losses) AS avg_losses,
    AVG(seasons.ties) AS avg_ties
FROM `bigquery-public-data.ncaa_basketball.mbb_historical_teams_seasons` AS seasons
LEFT JOIN `bigquery-public-data.ncaa_basketball.mascots` AS mascots
    ON seasons.team_id = mascots.id
WHERE seasons.season BETWEEN 1990 AND 1999
    AND seasons.division = 1
GROUP BY 1, 2, 3
ORDER BY avg_wins DESC;

-- 7. Using Temporary Table for Analysis
WITH longest_used_bike AS (
    SELECT
        bike_id,
        SUM(duration_minutes) AS trip_duration
    FROM bigquery-public-data.austin_bikeshare.bikeshare_trips
    GROUP BY bike_id
    ORDER BY trip_duration DESC
    LIMIT 1
)
SELECT
    trips.start_station_id,
    COUNT(*) AS trip_ct
FROM longest_used_bike AS longest
INNER JOIN bigquery-public-data.austin_bikeshare.bikeshare_trips AS trips
    ON longest.bike_id = trips.bike_id
GROUP BY trips.start_station_id
ORDER BY trip_ct DESC
LIMIT 1;

-- 8. Creating or Replacing a Permanent Table
CREATE OR REPLACE TABLE `porfolio-project-432006.Adventure_Work_Sales.JoinedData` AS
SELECT
    c.CustomerID,
    c.FullName AS CustomerFullName,
    o.SalesOrderID,
    o.OrderDate,
    o.TotalDue
FROM `porfolio-project-432006.Adventure_Work_Sales.customers` AS c
JOIN `porfolio-project-432006.Adventure_Work_Sales.orders` AS o
    ON c.CustomerID = o.CustomerID;

-- 9. Manipulating Strings
SELECT
    CONCAT(first_name, ' ', last_name) AS full_name
FROM customers;
