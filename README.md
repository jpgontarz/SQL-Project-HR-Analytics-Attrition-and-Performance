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

ALTER TABLE Employee
CHANGE COLUMN `DistanceFromHome (KM)` `DistanceFromHome (MI)` INT;
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

There were no further issues I found.

## Insights

### Question #1: What is the average tenure for employees within each department?

I found the average tenure for each department by using the ROUND, AVG, and GROUP BY functions. Since YearsAtCompany is already just a whole number, I decided to keep the average tenure as a whole number as well.

```sql
-- Average tenure by department --

SELECT Department,
    ROUND(AVG(YearsAtCompany), 2) AS AvgTenure_Years
FROM Employee
GROUP BY Department
ORDER BY AvgTenure_Years;
```

![image](https://github.com/user-attachments/assets/58629302-062f-4eae-97e0-7bdd64709dab)

_Average tenure (in years) by department_

The Technology department had the longest overall tenure at 4.61 years, with HR and Sales departments following slightly behind at 4.47 and 4.46 years, respectively.

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
FROM Employee
WHERE Attrition = 'No'
GROUP BY Department
ORDER BY ActiveEmployees DESC;
```

![image](https://github.com/user-attachments/assets/8d081323-7275-412e-8688-bb0069320b2f)

_Active employees by department and percentage of total_

The Technology department by far made up the most of the company's active employees with 828 employees, over two-thirds of the entire company. At not even half the tally was Sales at 354 employees (29%), with HR being the smallest department at 4% with just 51 employees.

### Question #3: How does job satisfaction for employees compare with different tenure levels?

Next, I utilized the CASE and JOIN functions to find the average job satisfaction rating in three categories: employees who had worked less than three years, those in between three and five years, and those over five years.

```sql
-- Average job satisfaction by tenure category --

SELECT 
    CASE 
	WHEN e.YearsAtCompany < 3 THEN '< 3 years'
        WHEN e.YearsAtCompany BETWEEN 3 AND 5 THEN '3-5 years'
        ELSE '> 5 years' 
    END AS TenureCategory,
    ROUND(AVG(p.JobSatisfaction), 2) AS AvgJobSatisfaction
FROM Employee e
JOIN PerformanceRating p ON e.EmployeeID = p.EmployeeID
GROUP BY TenureCategory
ORDER BY AvgJobSatisfaction DESC;

```

![image](https://github.com/user-attachments/assets/23bfb5f3-11a8-4518-b518-b6cfa105db4e)

_Average job satisfaction by tenure category_

Interestingly, the category with the highest average job satisfaction ratings were those working less than three years at nearly 3.45, following closely by the three to five year group and the more than five years category, both at almost 3.43.

### Question #4: Examine how many employees who worked overtime have left the company versus those who did not work overtime.

Again using the CASE function, I found the percentage of overtime workers would did not work anymore.

```sql
SELECT OverTime,
    ROUND(COUNT(CASE
		    WHEN Attrition = 'Yes'
		    THEN EmployeeID
		END) * 100.0 / COUNT(EmployeeID), 0) AS OverTimeAttritionPercentage
FROM Employee
GROUP BY OverTime
ORDER BY OverTime DESC;
```

![image](https://github.com/user-attachments/assets/bcad7c1f-635c-452a-b65c-4f4b22c78ec2)

_Overtime attrition percentage_

31% of overtime workers did not work for the company anymore, compared to only 10% of regular-houred employees.

### Question #5: Rank departments by average manager ratings, separated by business travel.

For this problem, I used the RANK, OVER(PARTITION BY), and JOIN functions to find and rank the average manager rating of each business travel type for each department then grouped the results by business travel type.

```sql
-- Average manager ratings by department and travel --

SELECT e.Department,
    e.BusinessTravel,
    ROUND(AVG(p.ManagerRating), 2) AS AvgManagerRating,
    RANK() OVER (PARTITION BY e.BusinessTravel
		ORDER BY AVG(p.ManagerRating) DESC) AS DepartmentRank
FROM Employee e
JOIN PerformanceRating p ON e.EmployeeID = p.EmployeeID
GROUP BY Department,
	BusinessTravel;
```

![image](https://github.com/user-attachments/assets/3a4571f3-447e-47b6-8c0d-e54b5e2ce6c1)

_Average manager rating, ranked by department, separated by travel type_

I also put together the total average ManagerRating by just Department, and then just by BusinessTravel.

```sql
SELECT e.Department,
	ROUND(AVG(p.ManagerRating), 2) AS AvgManagerRating
FROM Employee e
JOIN PerformanceRating p ON e.EmployeeID = p.EmployeeID
GROUP BY Department
ORDER BY AvgManagerRating DESC;
```

![image](https://github.com/user-attachments/assets/dead48f4-430e-4ad7-8217-3eb8b3cf4730)

_Average manager rating by department_

```sql
SELECT e.BusinessTravel,
	ROUND(AVG(p.ManagerRating), 2) AS AvgManagerRating
FROM Employee e
JOIN PerformanceRating p ON e.EmployeeID = p.EmployeeID
GROUP BY BusinessTravel
ORDER BY AvgManagerRating DESC;
```

![image](https://github.com/user-attachments/assets/ebe7522f-7f31-41ec-ad48-a2f36e773b5f)

_Average manager rating by travel type_

Technology boasts the highest average manager rating at 3.49, followed by Sales at 3.45, and HR at 3.44.

Meanwhile, the highest average manager rating by travel type was No Travel at 3.50, followed by Some Travel and Frequent Traveller both at 3.47.

### Question #6: Is there a positive correlation between the number of training opportunities an employee has taken and their job satisfaction?

Here, I found the average job satisfaction rating and grouped them with the number of training opportunities employees had taken.

```sql
-- Average job satisfaction by training opportunities taken --

SELECT TrainingOpportunitiesTaken,
    ROUND(AVG(JobSatisfaction), 2) AS AvgJobSatisfaction
FROM PerformanceRating
GROUP BY TrainingOpportunitiesTaken
ORDER BY TrainingOpportunitiesTaken DESC;
```

![image](https://github.com/user-attachments/assets/a46bc708-264c-44ac-abe7-37aebbee98ad)

_Average job satisfaction rating by training opportunities taken_

There seems to be a slight positive correlation between the amount of training opportunities employees take and their job satisfaction. Employees who took up three training opportunities boasted an average satisfaction rating of 3.48, those who had taken two and one both at 3.42, while employees who did not take any had an average of 3.44.

### Question #7: Identify the top three employees by their manager rating in each department.

While trying to write this query, I found that there were more than three employees in each department who had a five for manager rating. I decided to expand the criteria, where the employees not only had to have a manager rating of five but also had to have taken three training opportunities. Then, since there were slightly less results, I included a row number column that would randomly select only three employees who matched the criteria. The row numbers then would randomize every time the query was run.

```sql
-- Random top three employees by department, manager rating, and training opportunities taken --

WITH RandomTop3Performers AS (
    SELECT 
        CONCAT(e.FirstName, ' ', e.LastName) AS FullName,
        e.Department,
        p.ManagerRating,
        p.TrainingOpportunitiesTaken,
        ROW_NUMBER() OVER (PARTITION BY e.Department
	    ORDER BY p.ManagerRating,
		p.TrainingOpportunitiesTaken,
                RAND()) AS RowNum
    FROM Employee e
    JOIN PerformanceRating p ON e.EmployeeID = p.EmployeeID
    WHERE p.ManagerRating = 5
        AND p.TrainingOpportunitiesTaken = 3
        AND e.Attrition = 'No'
)
SELECT FullName,
    Department
FROM RandomTop3Performers
WHERE RowNum <= 3;
```

![image](https://github.com/user-attachments/assets/20ad3efc-d912-41be-b3b1-0137c610c1fb)

_Random top three performers by department_
