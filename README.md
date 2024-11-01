# SQL Project: HR-Analytics Employee Attrittion & Performance

![](https://github.com/user-attachments/assets/717e02c3-ee79-4a05-812a-845bec6252d6)

## Overview

_GreatPlaceToWork human resource personnel would like to improve performance, boost retention, and improve overall job satisfaction. However, they do not have good insights into the pertinent employee data. My goal is to utilize SQL within MySQL Workbench, analyzing their data to provide recommendations to the HR department that will facilitate successful improvements._

## Project Structure

- [About the Data](#about-the-data)
- [Task](#task)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Insights](#insights)
- [Recommendations](#recommendations)

## About the Data

Original data, along with an explanation of each column, can be found [here](https://www.kaggle.com/datasets/mahmoudemadabdallah/hr-analytics-employee-attrition-and-performance/data?select=Employee.csv).

The dataset includes five tables, capturing performance reviews, employee demographics, satisfaction levels, and ratings within over 8,100 records and 40 columns.

## Task

In this analysis, I help the HR department with the following:

1. How many employees in each department are still with the company and have been there for 3 or more years?

2. What is the average JobSatisfaction score for employees who have left versus those who remain?

3. How does the attrition rate vary for employees who work overtime in each department?

4. Is there a connection between frequent business travel and lower scores in job satisfaction or work-life balance?

5. Which departments have the most consistent manager ratings, as indicated by the lowest variation in ManagerRating?

6. Which employees show consistent improvement or decline in ManagerRating over their review dates?

7. What combination of demographics (department, age, years at the company) best aligns with high ManagerRating and JobSatisfaction scores?

8. Does a longer tenure with the current manager correlate with higher job satisfaction or lower turnover rates?

9. Are employees who complete more training opportunities given higher ManagerRating scores?

10. Does the time since an employee's last promotion correlate with attrition, particularly within specific departments?

## Exploratory Data Analysis

Before helping with the task, I first need to make sure the data is clean and ready to use in the analysis. As the EducationLevel, RatingLevel, and SatisfiedLevel tables are for reference, the main work uses the Employee and PerformanceRating tables.

#### Null/Missing Values

First, I checked for any missing values in the two most important fields: EmployeeID and PerformanceID. None were found.

```sql
-- Check for missing values in Employee table --

SELECT COUNT(*) AS MissingValues
FROM Employee
WHERE EmployeeID IS NULL;

-- Check for missing values in PerformanceRating table --

SELECT COUNT(*) AS MissingValues
FROM PerformanceRating
WHERE PerformanceID IS NULL
	OR EmployeeID IS NULL;
```

Next, it is vital to make sure duplicate rows are removed, if any are found, again in the key fields. None were found.

```sql
-- Check for duplicate values in Employee table --

SELECT EmployeeID, COUNT(*)
FROM Employee
GROUP BY EmployeeID
HAVING COUNT(*) > 1;

-- Check for duplicate values in PerformanceRating table --

SELECT PerformanceID, COUNT(*)
FROM PerformanceRating
GROUP BY PerformanceID
HAVING COUNT(*) > 1;
```

In order to standardize the data, I decided to first change the measurement of the DistanceFromHome column from kilometers (KM) to miles (MI), and then change the name of the column to DistanceFromHome (MI), since the data and myself are from the United States.

```sql
-- Change DistanceFromHome (KM) measurement from kilometers (KM) to miles (MI) --

UPDATE Employee
SET `DistanceFromHome (KM)` = ROUND(`DistanceFromHome (KM)` * 0.621371, 0);

-- Change column name to DistanceFromHome (MI) --

ALTER TABLE Employee CHANGE COLUMN `DistanceFromHome (KM)` `DistanceFromHome (MI)` INT;
```

![image](https://github.com/user-attachments/assets/6c3a6a03-c744-42e4-9406-6a2e3d686281)   ![image](https://github.com/user-attachments/assets/3f2ab5d2-d6d2-4253-aff2-83f6cfde3721)

I noticed in the ReviewDate column in the PerformanceRating table is not in standard MySQL date format YYYY-MM-DD, so I quickly changed this as well.

```sql
-- Change ReviewDate to standard date format --
-- (due to Safe Updates being on in MySQL, I had to momentarily turn them off to perform this query) --

SET SQL_SAFE_UPDATES = 0;

UPDATE PerformanceRating
SET ReviewDate = CASE
    WHEN LENGTH(ReviewDate) = 10 THEN DATE_FORMAT(STR_TO_DATE(ReviewDate, '%m/%d/%Y'), '%Y-%m-%d')
    ELSE DATE_FORMAT(STR_TO_DATE(ReviewDate, '%m/%d/%Y'), '%Y-%m-%d')
END
WHERE ReviewDate IS NOT NULL;

SET SQL_SAFE_UPDATES = 1;
```
![image](https://github.com/user-attachments/assets/f656cbc0-6e4e-4488-a8e9-9da4a6b14e01)   ![image](https://github.com/user-attachments/assets/8bfa8a5e-c4f8-4ddf-9cbd-2deed717ae60)

There were no further issues I found. With the data cleaning complete, I decided to create a View that combined the Employee and PerformanceRating tables in order for simpler analysis.

```sql
-- Create View for Employee and PerformanceRating tables --

CREATE VIEW EmployeePerformance AS
SELECT 
    e.EmployeeID, 
    CONCAT(e.FirstName, ' ', e.LastName) AS EmployeeName, 
    e.Gender, 
    e.Age, 
    e.Department, 
    e.`DistanceFromHome (MI)`, 
    e.State, 
    e.Ethnicity, 
    e.MaritalStatus, 
    e.Salary, 
    e.StockOptionLevel, 
    e.OverTime, 
    e.HireDate, 
    e.Attrition, 
    e.YearsAtCompany, 
    e.YearsInMostRecentRole, 
    e.YearsSinceLastPromotion, 
    e.YearsWithCurrManager,
    p.PerformanceID, 
    p.ReviewDate, 
    p.EnvironmentSatisfaction, 
    p.JobSatisfaction, 
    p.RelationshipSatisfaction, 
    p.TrainingOpportunitiesWithinYear, 
    p.TrainingOpportunitiesTaken, 
    p.WorkLifeBalance, 
    p.SelfRating, 
    p.ManagerRating
FROM Employee e
LEFT JOIN PerformanceRating p ON e.EmployeeID = p.EmployeeID;
```

## Insights

### Question #1: 


