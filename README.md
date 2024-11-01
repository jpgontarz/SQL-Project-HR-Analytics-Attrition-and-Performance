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

1. What is the average tenure for employees within each department?

2. How many employees in each department are still working at the company?

3. How does job satisfaction for employees compare with different tenure levels?

4. What percentage of employees who work overtime have left the company?

5. Rank departments by average manager ratings, separated by business travel.

6. Is there a positive correlation between the number of training opportunities an employee has taken and their job satisfaction?

7. Identify the top three employees by their manager rating in each department.

8. Categorize employees based on their distance from work and show average job satisfaction in each category.

9. Is there a relationship between the number of promotions and the years an employee has spent with their current manager?

10. For each department, identify the percentage of employees who have left the company and had a job satisfaction score below 3.

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

![image](https://github.com/user-attachments/assets/6c3a6a03-c744-42e4-9406-6a2e3d686281)

_Before measurement change: DistanceFromHome (KM)_

![image](https://github.com/user-attachments/assets/3f2ab5d2-d6d2-4253-aff2-83f6cfde3721)

_After measurement change: DistanceFromHome (MI)_

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
![image](https://github.com/user-attachments/assets/f656cbc0-6e4e-4488-a8e9-9da4a6b14e01)

_Before date format edit: mm-dd-YYYY_

![image](https://github.com/user-attachments/assets/8bfa8a5e-c4f8-4ddf-9cbd-2deed717ae60)

_After date format edit: YYYY-mm-dd_

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

### Question #1: What is the average tenure for employees within each department?

I found the average tenure for each department by using the ROUND, AVG, and GROUP BY functions. Since YearsAtCompany is already just a whole number, I decided to keep the average tenure as a whole number as well.

```sql
-- Average tenure by department --

SELECT Department,
	ROUND(AVG(YearsAtCompany), 2) AS AvgTenure_YRS
FROM EmployeePerformance
GROUP BY Department;
```

![image](https://github.com/user-attachments/assets/30cf6690-f908-4afc-9618-6690f34dea12)


_Average tenure (in years) by department_

The Sales department had the shortest overall tenure at nearly 5.4 years, HR tenure was next at 5.55 years, and Technology the longest at almost 5.7 years.

### Question #2: How many employees in each department are still working at the company?

To find the number of current employees, just simple COUNT, WHERE, and GROUP BY functions were needed.

```sql
-- Active Employees --

SELECT Department,
	COUNT(*) AS ActiveEmployees,
	ROUND(COUNT(*) * 100 / (SELECT COUNT(*)
				FROM EmployeePerformance
				WHERE Attrition = 'No'), 0
				) AS PercentageOfActive
FROM EmployeePerformance
WHERE Attrition = 'No'
GROUP BY Department
ORDER BY ActiveEmployees DESC;
```

![image](https://github.com/user-attachments/assets/9cfe6a89-cd36-4123-9de1-1b6630e3465c)

_Active employees by department and percentage of total_

The Technology department by far made up the most of the company's active employees with more than 3,100 employees, over two-thirds of the entire company. At not even half the tally was Sales at 1,332 employees (29%), with HR being the smallest department at 4% with just under 200 employees.

### Question #3: How does job satisfaction for employees compare with different tenure levels?

Next, I utilized the CASE function to find the average job satisfaction rating in three categories: employees who had worked less than three years, those in between three and five years, and those over five years.

```sql
-- Average job satisfaction by tenure category --

SELECT 
    CASE 
        WHEN YearsAtCompany < 3 THEN '< 3 years'
        WHEN YearsAtCompany BETWEEN 3 AND 5 THEN '3-5 years'
        ELSE '> 5 years' 
    END AS TenureCategory,
    ROUND(AVG(JobSatisfaction), 4) AS AvgJobSatisfaction
FROM EmployeePerformance
GROUP BY TenureCategory
ORDER BY AvgJobSatisfaction DESC;
```

Interestingly, the category with the highest average job satisfaction ratings were those working less than three years at nearly 3.45, following closely by the three to five year group and the more than five years category, both at almost 3.43.

### Question #4: What percentage of employees who work overtime have left the company?

Again using the CASE function, I found the percentage of overtime workers

```sql

```



