# Project Deforestation Exploration
----
## Steps to Complete
### Create a View 
called “forestation” by joining all three tables - forest_area, land_area, and regions in the workspace.

```
CREATE VIEW forestation AS
SELECT f.country_code,
  f.country_name,
  f.year,
  f.forest_area_sqkm,
  r.region,
  r.income_group,
  l.total_area_sq_mi * 2.59 AS total_area_sqkm,
  (f.forest_area_sqkm/(l.total_area_sq_mi*2.59))*100 AS forest_percent
FROM forest_area AS f 
JOIN land_area AS l ON f.country_code = l.country_code AND f.year = l.year
JOIN regions AS r ON f.country_code = r.country_code
ORDER BY country_code;
```

```
Select * from forestation
```
### 1. Part 1 - Global Situation
#### a. What was the total forest area (in sq km) of the world in 1990? Please keep in mind that you can use the country record denoted as “World" in the region table.

```
SELECT forest_area_sqkm AS total_forest_area
FROM forestation
WHERE year = 1990 AND country_name = 'World';
```

#### b. What was the total forest area (in sq km) of the world in 2016? Please keep in mind that you can use the country record in the table is denoted as “World.”

```
SELECT forest_area_sqkm AS total_forest_area
FROM forestation
WHERE year = 2016 AND country_name = 'World';
```

#### c. What was the change (in sq km) in the forest area of the world from 1990 to 2016?

```
WITH a AS(
  SELECT country_code,forest_area_sqkm AS total_forest_area_1990
  FROM forestation
  WHERE year = 1990
    AND country_name = 'World'
),
b AS(
  SELECT country_code,forest_area_sqkm AS total_forest_area_2016
  FROM forestation
  WHERE year = 2016
    AND country_name = 'World'
)
SELECT (a.total_forest_area_1990 - b.total_forest_area_2016) AS total_forest_area_change
FROM a join b on a.country_code =b.country_code ;
```

#### d. What was the percent change in forest area of the world between 1990 and 2016?

```
WITH a AS(
  SELECT forest_area_sqkm
  FROM forestation
  WHERE year = 1990
    AND country_name = 'World'
),
b AS(
  SELECT forest_area_sqkm
  FROM forestation
  WHERE year = 2016
    AND country_name = 'World'
)
SELECT ((a.forest_area_sqkm-b.forest_area_sqkm)/a.forest_area_sqkm)*100  AS percent_change
FROM a,b;
```

***OR USING SELF JOIN:***
```
SELECT ((a.forest_area_sqkm-b.forest_area_sqkm)/a.forest_area_sqkm)*100  AS percent_change
FROM forestation a
INNER JOIN forestation b
ON a.country_name=b.country_name where a.country_name = 'World'
and b.country_name = 'World'
and a.year = 1990
and b.year = 2016;
```

#### e. If you compare the amount of forest area lost between 1990 and 2016, to which country's total area in 2016 is it closest to?

```
WITH a AS (
  SELECT country_name,forest_area_sqkm
  FROM forestation
  WHERE year = 1990
    AND country_name = 'World'
),
b AS (
  SELECT country_name,forest_area_sqkm
  FROM forestation
  WHERE year = 2016
    AND country_name = 'World'
)
Select country_name,
       total_area_sqkm,
       ABS((total_area_sqkm)-(SELECT a.forest_area_sqkm - b.forest_area_sqkm AS diff from a,b)) as change
FROM forestation
    WHERE year = 2016
    ORDER BY 3 LIMIT 1;
```

### 2. Part 2 - Regional Outlook

#### Create table

```
Create view b as
  SELECT a.*,
  (a.total_forest_area_sqkm / a.total_total_area_sqkm) * 100 AS percent_forest
FROM(
    SELECT region,year,
      SUM(forest_area_sqkm) AS total_forest_area_sqkm,
      SUM(total_area_sqkm) AS total_total_area_sqkm
    FROM forestation
    GROUP BY region,year
    HAVING (year = 2016 or year = 1990)
  ) AS a
ORDER BY region,year;
```


#### a. What was the percent forest of the entire world in 2016? Which region had the HIGHEST percent forest in 2016, and which had the LOWEST, to 2 decimal places?

 ##### a1. What was the percent forest of the entire world in 2016?

 
```
SELECT ROUND(CAST(percent_forest AS numeric),2) AS percent_fa_region
FROM b WHERE year = 2016 AND region = 'World';
```

##### a2. Which region had the HIGHEST percent forest in 2016

```
SELECT region,
       ROUND(CAST(total_total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
       ROUND(CAST(percent_forest AS NUMERIC),2) AS percent_forest
       FROM b
       WHERE ROUND(CAST(percent_forest AS NUMERIC),2) = (SELECT MAX(ROUND(CAST(percent_forest AS numeric),2)) AS max_percent
FROM b
WHERE year = 2016) AND year=2016;
```

##### a3. which had the LOWEST, to 2 decimal places?

```
SELECT region,
       ROUND(CAST(total_total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
       ROUND(CAST(percent_forest AS NUMERIC),2) AS percent_forest
       FROM b
       WHERE ROUND(CAST(percent_forest AS NUMERIC),2) = (SELECT MIN(ROUND(CAST(percent_forest AS numeric),2)) AS max_percent
FROM b
WHERE year = 2016) AND year=2016;
```

#### b. What was the percent forest of the entire world in 1990? Which region had the HIGHEST percent forest in 1990, and which had the LOWEST, to 2 decimal places?

##### b1. What was the percent forest of the entire world in 1990?

```
SELECT ROUND(CAST(percent_forest AS numeric),2) AS percent_fa_region
FROM b WHERE year = 1990 AND region = 'World';
```


##### b2. Which region had the HIGHEST percent forest in 1990
```
SELECT region,
       ROUND(CAST(total_total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
       ROUND(CAST(percent_forest AS NUMERIC),2) AS percent_forest
       FROM b
       WHERE ROUND(CAST(percent_forest AS NUMERIC),2) = (SELECT MAX(ROUND(CAST(percent_forest AS numeric),2)) AS max_percent
FROM b
WHERE year = 1990) AND year=1990;
```


##### b3. which had the LOWEST, to 2 decimal places?

```
SELECT region,
       ROUND(CAST(total_total_area_sqkm AS NUMERIC),2) AS total_area_sqkm,
       ROUND(CAST(percent_forest AS NUMERIC),2) AS percent_forest
       FROM b
       WHERE ROUND(CAST(percent_forest AS NUMERIC),2) = (SELECT MIN(ROUND(CAST(percent_forest AS numeric),2)) AS max_percent
FROM b
WHERE year = 1990) AND year=1990;
```

##### c. Based on the table you created, which regions of the world DECREASED in forest area from 1990 to 2016?

```
WITH c AS (SELECT * FROM b WHERE year =1990),
	d AS (SELECT * FROM b WHERE year = 2016)
SELECT c.region,
       ROUND(CAST(c.percent_forest AS NUMERIC),2) AS percent_forest_1990,
       ROUND(CAST(d.percent_forest AS NUMERIC),2) AS percent_forest_2016
    FROM c JOIN d ON c.region = d.region
    WHERE c.percent_forest > d.percent_forest;
```

### Part 3 - Country-Level Detail

#### a. Which 5 countries saw the largest amount decrease in forest area from 1990 to 2016? What was the difference in forest area for each?

```
WITH y90 AS 
(SELECT * FROM forest_area 
   WHERE year = 1990 AND forest_area_sqkm IS NOT NULL AND country_name != 'World'
 ),

y2016 AS (SELECT * FROM forest_area f
WHERE year = 2016 AND forest_area_sqkm IS NOT NULL AND country_name != 'World'
	)
 SELECT y90.country_code,
        y90.country_name,
        r.region,
        y90.forest_area_sqkm AS sqkm1990,
        y2016.forest_area_sqkm AS sqkm2016,
        y90.forest_area_sqkm - y2016.forest_area_sqkm AS change_sqkm
      FROM y90
      JOIN y2016
      ON y90.country_code = y2016.country_code
      AND (y90.forest_area_sqkm IS NOT NULL AND y2016.forest_area_sqkm IS NOT NULL)
      JOIN regions r ON y2016.country_code = r.country_code
      ORDER BY change_sqkm DESC
      LIMIT 5;
```
#### b. Which 5 countries saw the largest percent decrease in forest area from 1990 to 2016? What was the percent change to 2 decimal places for each?


```
WITH y90 AS 
(SELECT * FROM forest_area 
   WHERE year = 1990 AND forest_area_sqkm IS NOT NULL AND country_name != 'World'
 ),

y2016 AS (SELECT * FROM forest_area f
WHERE year = 2016 AND forest_area_sqkm IS NOT NULL AND country_name != 'World'
	)
 SELECT y90.country_code,
        y90.country_name,
        r.region,
        y90.forest_area_sqkm AS sqkm1990,
        y2016.forest_area_sqkm AS sqkm2016,
        y90.forest_area_sqkm - y2016.forest_area_sqkm AS change_sqkm,
        ABS(ROUND(CAST(((y2016.forest_area_sqkm-y90.forest_area_sqkm)/y90.forest_area_sqkm*100) AS NUMERIC),2)) AS change_per_1,
        ROUND(CAST(((y2016.forest_area_sqkm-y90.forest_area_sqkm)/y90.forest_area_sqkm*100) AS NUMERIC),2) as change_per
      FROM y90
      JOIN y2016
      ON y90.country_code = y2016.country_code
      AND (y90.forest_area_sqkm IS NOT NULL AND y2016.forest_area_sqkm IS NOT NULL)
      JOIN regions r ON y2016.country_code = r.country_code
      ORDER BY change_per 
      LIMIT 5;
```

#### c. If countries were grouped by percent forestation in quartiles, which group had the most countries in it in 2016?

```
WITH a AS (
  SELECT country_name,
    CASE WHEN forest_percent < 25 THEN 'Q1'
	WHEN forest_percent >= 25 AND forest_percent < 50 THEN 'Q2'
	WHEN forest_percent >= 50 AND forest_percent < 75 THEN 'Q3'
	ELSE 'Q4' END AS quartiles
  FROM forestation
  WHERE year = 2016 AND forest_percent IS NOT NULL
)
SELECT DISTINCT quartiles, (COUNT(country_name) OVER (PARTITION BY quartiles)) AS count
FROM a ORDER BY quartiles;
```

#### d. List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.

```
WITH a AS (
  SELECT country_name, region,
    CASE WHEN forest_percent < 25 THEN 'Q1'
	WHEN forest_percent >= 25 AND forest_percent < 50 THEN 'Q2'
	WHEN forest_percent >= 50 AND forest_percent < 75 THEN 'Q3'
	ELSE 'Q4' END AS quartiles
  FROM forestation
  WHERE year = 2016 AND forest_percent IS NOT NULL
)
SELECT country_name,region, quartiles
FROM a WHERE quartiles = 'Q4';
```

#### e. How many countries had a percent forestation higher than the United States in 2016?

```
With a as(
    SELECT DISTINCT country_name FROM forestation
    WHERE forest_percent > (SELECT forest_percent FROM forestation WHERE country_name = 'United States' AND year = 2016)
    ORDER BY country_name
  ) Select count(*) from a
```











