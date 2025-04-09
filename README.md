<p align="center">
  <img src=""C:\Users\kobit\Downloads\life exp.jpg"alt="Map with Pins" width="500">
</p>
# World Life Expectancy Data Cleaning & Exploratory Data Analysis Project

## Project Overview
This project focuses on cleaning and analyzing a dataset related to global life expectancy using SQL. The goal was to gain insights into health indicators, socioeconomic factors, and country-level differences that may influence life expectancy across the world

##  Objectives
- Clean and standardize raw life expectancy data for better usability.
- Identify trends in life expectancy over time and across countries.
- Explore the impact of factors like GDP, BMI, and development status on life expectancy.
- Develop SQL queries to extract meaningful insights from the dataset.

## Dataset
The data for this project is sourced from the Kaggle dataset:
- **Dataset Name:** Life Expectancy (WHO)
- **Dataset Link:** [Kaggle - Life Expectancy Dataset](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who)
---

## 1 Data Cleaning Process

###  1.1 Remove Duplicate Records
```sql
SELECT Country, Year, CONCAT(Country, Year),
COUNT(CONCAT(Country, Year))
FROM world_life_expectancy
GROUP BY Country, Year, CONCAT(Country, Year)
HAVING COUNT(CONCAT(Country, Year)) > 1;
```

```sql
DELETE FROM world_life_expectancy
WHERE Row_ID IN (
  SELECT Row_ID FROM (
    SELECT Row_ID,
    ROW_NUMBER() OVER (PARTITION BY CONCAT(Country, Year) ORDER BY CONCAT(Country, Year)) AS row_num
    FROM world_life_expectancy
  ) AS row_num
  WHERE row_num > 1
);
```

---

###  1.2 Handle Missing Values
```sql
SELECT * FROM world_life_expectancy WHERE Status IS NULL;
SELECT DISTINCT(Status) FROM world_life_expectancy WHERE Status <> '';
```

```sql
UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2 ON t1.Country = t2.Country
SET t1.Status = 'Developing'
WHERE t1.Status = '' AND t2.Status = 'Developing';

UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2 ON t1.Country = t2.Country
SET t1.Status = 'Developed'
WHERE t1.Status = '' AND t2.Status = 'Developed';
```

---

### 1.3 Insert Missing `Life expectancy` Values
```sql
SELECT * FROM world_life_expectancy WHERE `Life expectancy` = '';
```

```sql
UPDATE world_life_expectancy t1
JOIN world_life_expectancy t2 ON t1.Country = t2.Country AND t1.Year = t2.Year - 1
JOIN world_life_expectancy t3 ON t1.Country = t3.Country AND t1.Year = t3.Year + 1
SET t1.`Life expectancy` = ROUND((t2.`Life expectancy` + t3.`Life expectancy`) / 2, 1)
WHERE t1.`Life expectancy` = '';
```

---

## 2️ Exploratory Data Analysis (EDA)

### 📈 2.1 Life Expectancy Change Over Time
```sql
SELECT Country, 
MIN(`Life expectancy`), 
MAX(`Life expectancy`),
ROUND(MAX(`Life expectancy`)-MIN(`Life expectancy`),1) AS life_increse_15_years
FROM world_life_expectancy
GROUP BY Country
HAVING MIN(`Life expectancy`) <> 0
AND MAX(`Life expectancy`) <> 0
ORDER BY life_increse_15_years DESC;
```

---

###  2.2 Global Life Expectancy by Year
```sql
SELECT Year, ROUND(AVG(`Life expectancy`),2)
FROM world_life_expectancy
WHERE `Life expectancy` <> 0
GROUP BY Year
ORDER BY Year;
```

---

###  2.3 Life Expectancy vs GDP
```sql
SELECT Country, ROUND(AVG(`Life expectancy`),1) AS Life_exp, ROUND(AVG(GDP),1) AS GDP
FROM world_life_expectancy
GROUP BY Country
HAVING Life_exp > 0 AND GDP > 0
ORDER BY GDP DESC;
```

```sql
SELECT 
SUM(CASE WHEN GDP >= 1500 THEN 1 ELSE 0 END) AS High_GDP,
AVG(CASE WHEN GDP >= 1500 THEN `Life expectancy` ELSE NULL END) AS High_GDP_Life_Exp,
SUM(CASE WHEN GDP <= 1500 THEN 1 ELSE 0 END) AS Low_GDP,
AVG(CASE WHEN GDP <= 1500 THEN `Life expectancy` ELSE NULL END) AS Low_GDP_Life_Exp
FROM world_life_expectancy;
```

---

###  2.4 Life Expectancy by Development Status
```sql
SELECT Status, ROUND(AVG(`Life expectancy`),1)
FROM world_life_expectancy
GROUP BY Status;
```

```sql
SELECT Status, COUNT(DISTINCT Country), ROUND(AVG(`Life expectancy`),1)
FROM world_life_expectancy
GROUP BY Status;
```

---

###  2.5 Life Expectancy vs BMI
```sql
SELECT Country, ROUND(AVG(`Life expectancy`),1) AS Life_exp, ROUND(AVG(BMI),1) AS BMI
FROM world_life_expectancy
GROUP BY Country
HAVING Life_exp > 0
AND BMI > 0
ORDER BY BMI ASC;
```

---

###  2.6 Adult Mortality Over Time (Rolling Total)
```sql
SELECT Country, Year, `Life expectancy`, `Adult Mortality`,
SUM(`Adult Mortality`) OVER(PARTITION BY Country ORDER BY Year) AS Rolling_total
FROM world_life_expectancy
WHERE Country LIKE '%United%';
```

---

 With the data now clean and explored, this analysis provides insight into global health trends and the relationship between life expectancy and socioeconomic factors. These SQL queries lay the foundation for deeper statistical analysis and potential data visualizations.
