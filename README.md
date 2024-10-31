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

Original data can be found [here](https://www.kaggle.com/datasets/mahmoudemadabdallah/hr-analytics-employee-attrition-and-performance/data?select=Employee.csv).

The dataset includes five tables, capturing information regarding performance reviews, employee demographics, satisfaction levels, and ratings.

## Task

In this analysis, I help the HR department with the following:

### Performance Enhancement

1. Who are each department's top performers?

2. Analyze the affect of training on performance.

3. Is there a positive relationship between a good work-life balance and performance?

4. Are there correlations/discrepancies with employee self-assessments and manager evaluations in a given department?

5. Does a higher job satisfaction rating align with improved performance metrics?

### Retention and Turnover Insights

6. Are there any trends between low job satisfaction and attrition rates, specifically across a department or demographic?

7. Is there a relationship between overtime and turnover?

8. Does the distance from office seem to affect retention?

9. How are low relationship satisfaction scores with managers associated with attrition, especially in any specific departments?

10. Do employees who go longer without promotions have a higher tendency to leave?

### Boosting Job Satisfaction

11. Do employees who participate in more training opportunities report higher satisfaction in areas like work-life balance or job contentment?

12. Examine if frequent business travel has any negative impact on job satisfaction, work-life balance, or relationships at work.

13. How satisfied are employees who live further from the workplace than others?

14. Are there notable trends in satisfaction scores by gender, age, marital status, and other demographics?

15. How likely are employees who have been at the company longer to be satisfied with their job?

16. Does stock options seem to affect employee satisfaction?

### Managerial Effectiveness

17. Determine if a longer tenure with the same manager correlates with better job performance and satisfaction scores.

18. Do lower manager ratings contribute to higher attrition rates?

19. Which managers facilitate more training opportunities than others, and does this affect their employees' satisfaction and performance?

20. Examine if employees with longer relationships with their managers experience more frequent promotions.

## Exploratory Data Analysis

Before helping with the task, I first need to make sure the data is clean and ready to use in the analysis. As the EducationLevel, RatingLevel, and SatisfiedLevel tables are for reference, the main work uses the Employee and PerformanceRating tables.

#### Null/Missing Values

First, I checked for any missing values in the two most important fields: EmployeeID and PerformanceID.

```sql
-- Check for missing values in key fields --

SELECT COUNT(*) AS MissingValues
FROM Employee
WHERE EmployeeID IS NULL;

SELECT COUNT(*) AS MissingValues
FROM PerformanceRating
WHERE PerformanceID IS NULL
	OR EmployeeID IS NULL;
```

Next, it is vital to make sure duplicate rows are removed, if any are found, again in the key fields.

```sql
-- Check for duplicate values in key fields --

SELECT EmployeeID, COUNT(*)
FROM Employee
GROUP BY EmployeeID
HAVING COUNT(*) > 1;

SELECT PerformanceID, COUNT(*)
FROM PerformanceRating
GROUP BY PerformanceID
HAVING COUNT(*) > 1;
```

In order to standardize the data, I decided to first change the measurement of the DistanceFromHome column from kilometers (KM) to miles (MI), and then change the name of the column to DistanceFromHome (MI).

```sql
-- Change DistanceFromHome (KM) measurement from kilometers (KM) to miles (MI); change column name to DistanceFromHome (MI) --

UPDATE Employee
SET `DistanceFromHome (KM)` = ROUND(`DistanceFromHome (KM)` * 0.621371, 0);

ALTER TABLE Employee CHANGE COLUMN `DistanceFromHome (KM)` `DistanceFromHome (MI)` INT;
```

I noticed in the ReviewDate column in the PerformanceRating table is not in standard MySQL date format YYYY-MM-DD.

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



## Insights

### Question #1: Who are each department's top performers?



**Question #1 Conclusion:** asdfjkl;

## Recommendations
