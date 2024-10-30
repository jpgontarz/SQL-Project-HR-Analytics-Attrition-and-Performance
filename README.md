# SQL Project: HR-Analytics Employee Attrittion & Performance

## Overview

This project demonstrates my SQL problem-solving skills, analyzing employee performance, job satisfaction, and attrition trends to solve key HR-related business problems. The goal is to clean and process the data to uncover patterns related to job performance, employee retention, and satisfaction. The insights gained from this analysis will help addresss HR challenges such as improving performance, boosting retention, and improving overall job satisfaction.

## Project Structure

- [About the Data](#about-the-data)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Business Problems](#business-problems)
- [Problems Solved](#problems-solved)
- [Insights and Recommendations](#insights-and-recommendations)

## About the Data

Original dataset can be found [here](https://www.kaggle.com/datasets/mahmoudemadabdallah/hr-analytics-employee-attrition-and-performance/data?select=Employee.csv).

The dataset includes five tables, capturing information regarding performance reviews, employee demographics, satisfaction levels, and ratings.

### 1. PerformanceRating

Employees performance ratings.
- PerformanceID: Unique identifier for each performance review.
- EmployeeID: Unique identifier for the employee being reviewed.
- ReviewDate: The date of the performance review.
- EnvironmentSatisfaction: Rating of the employee's satisfaction with their work environment.
- JobSatisfaction: Rating of the employee's satisfaction with their job.
- RelationshipSatisfaction: Rating of the employee's satisfaction with workplace relationships.
- TrainingOpportunitiesWithinYear: Number of training opportunities available to the employee within the year.
- TrainingOpportunitiesTaken: Number of training opportunities the employee has taken.
- WorkLifeBalance: Rating of the employee's work-life balance.
- SelfRating: The employee's self-assessment rating.
- ManagerRating: The manager's rating of the employee's performance.

### 2. Employee

General employee information.
- EmployeeID: Unique identifier for each employee.
- FirstName: The first name of the employee.
- LastName: The last name of the employee.
- Gender: The gender of the employee.
- Age: The age of the employee.
- BusinessTravel: The frequency of business travel for the employee.
- Department: The department in which the employee works.
- DistanceFromHome (KM): The distance between the employee's home and workplace in kilometers.
- State: The state in which the employee resides.
- Ethnicity: The ethnicity of the employee.
- MaritalStatus: The marital status of the employee.
- Salary: The annual salary of the employee.
- StockOptionLevel: The level of stock options granted to the employee.
- OverTime: Whether the employee works overtime (Yes/No).
- HireDate: The date the employee was hired.
- Attrition: Whether the employee has left the company (Yes/No).
- YearsAtCompany: The number of years the employee has been with the company.
- YearsInMostRecentRole: The number of years the employee has been in their most recent role.
- YearsSinceLastPromotion: The number of years since the employee's last promotion.
- YearsWithCurrManager: The number of years the employee has worked with their current manager.

### 3. SatisfiedLevel

Satisfaction level scale.
- SatisfactionID: Unique identifier for the satisfaction level.
- SatisfactionLevel: The level of satisfaction, ranging from "Very Dissatisfied" to "Very Satisfied."

### 4. RatingLevel

Performance level scale.
- RatingID: Unique identifier for the rating level.
- RatingLevel: The performance rating, ranging from "Unacceptable" to "Above and Beyond."

### 5. EducationLevel

Education level scale.
- EducationLevelID: Unique identifier for the education level.
- EducationLevel: The level of education achieved, ranging from "No Formal Qualifications" to "Doctorate."

## Exploratory Data Analysis

BeforeData cleaning is essential to ensure the dataset is accurate and useful for further analysis. I did not find any data that needed to be standardized, so I only needed to perform simple checks for null values and duplicates.

### 1. Null Values


### 2. Duplicates



### 3. Standardization




### 4. Errors/Incorrect Entries



### 5. 
