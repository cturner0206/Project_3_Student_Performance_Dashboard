# Project 3: Student Performance Dashboard 
   
<img src="" alt="dash" width="900" />

# Table of Contents 
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Loading Data into SSMS ](#loading-data-into-ssms)
- [Creating the Queries](#creating-the-queries)
- [Process of Building the Report](#process-of-building-the-report)
- [Findings](#findings)
- [Recommendations](#recommendations)


# Project Overview

The primary goal of this project is to identify and analyze the various factors that affect student GPA. To do this, I used SQL queries to calculate averages, distributions, and other relevant data and imported the queries into PowerBI to create an interactive dashboard. The dashboard aims to provide insights that can help educators and administrators make informed decisions to enhance student performance.

# Data Source
The data used for this project comes from the `student_performance_data.csv` and `student_performance_data1.csv` files. The original dataset file had both files combined, but I split it up to segment the data based on student data and academic performance metrics. 


# Loading Data into SSMS  

Loading the data into Microsoft SQL Server Management Studio involved the following steps:

- Created new Student_Data database.
- Imported `student_performance_data.csv` and `student_performance_data1.csv` files in SQL Server Management Studio.
- Chose relevent data types for the different columns in both tables
   - StudentID was used as the PK in Students table and PK/FK in Academic_Performance table to maintain data integrity
   - Set everything to not include nulls
        > **Note:** I saw there were no null values to begin with in the dataset as there are only 500 rows
   - Ran SELECT statements to review the data in the two tables
      

![image](https://github.com/user-attachments/assets/6c0a99fd-6c70-4ea7-9e6c-0beec0672e1d)
![image](https://github.com/user-attachments/assets/44339b0c-6d34-4dbe-a9ae-55d52c858827)



# Creating the Queries

## **Query Brainstorm:**  

To be able to create relevant queries that would allow for an end user to gain meaningful insights, I first had a brainstorming session to: 

1. Identifying Relevant Data Points:
  -  FINISH THIS
  -   FINISH THIS
  -    FINISH THIS
  -     FINISH THIS
  -  FINISH THIS
  -   FINISH THIS
  -    FINISH THIS
  -     FINISH THIS
  -  FINISH THIS

  - 
## **Data Cleaning:**  
  Any data cleaning that took place was in the queries themselves as the data itself was completely clean. The only altering I did was rounding and casting values.

## **SQL Used:**
   - Group By and Where clauses
   - Aggregate functions: Avg, Sum, Count
   - Joins
   - Scalar Functions: Round, Cast
   - Conditional: Case, IIF
   - Subqueries
   - CTE's
   - Window functions: row_number()


## **All Queries**
  
```sql
--Count and percent of total students by major
SELECT Major,
       Count(StudentID) AS Cnt_Students,
       Round(Cast(Count(*) * 100.0 / (SELECT Count(*) FROM Students) AS FLOAT), 2) AS Percent_of_Total_Students
FROM Students
GROUP BY Major
ORDER BY Percent_of_Total_Students DESC;
```
```sql
--General averages per major
SELECT Major,
       Avg(Age) AS Avg_Age,
       Round(Avg(Gpa), 2) AS Avg_Gpa,
       Round(Avg(AttendanceRate), 2) / 100 AS Avg_Attendance,
       Avg(StudyHoursPerWeek) AS Avg_Study_Hours
FROM Students s
JOIN Academic_Performance ap ON s.Studentid = ap.Studentid
GROUP BY Major
ORDER BY Avg_Gpa DESC;
```
```sql
--Students who have a higher GPA than the overall average
SELECT s.StudentID,
       Gender,
       Age,
       Major,
       Round(GPA, 2) AS GPA
FROM Students AS s
JOIN Academic_Performance ap ON s.StudentID = ap.StudentID
WHERE GPA > (SELECT Avg(GPA) FROM Academic_Performance)
ORDER BY GPA DESC;
```
```sql
--Which students are performing well or could use some improvement (Is GPA greater than 3.0?)
SELECT s.StudentID,
       Major,
       Round(GPA, 2) AS GPA,
       StudyHoursPerWeek,
       iif(GPA > 3.0, 'Performing Well', 'Needs Improvement') AS PerformanceCategory
FROM   Students s
JOIN Academic_Performance ap ON s.StudentID = ap.StudentId;
```
```sql
--Financial aid percentages by gender 
SELECT Gender,
       Count(Studentid) AS Total_Students,
       Sum(CASE WHEN FinAid = 'Yes' THEN 1 ELSE 0 END) AS Students_With_FinAid,
       Round(Cast(Sum(CASE WHEN FinAid = 'Yes' THEN 1 ELSE 0 END) * 100.0 / Count(*) AS FLOAT), 2) AS Percent_With_FinAid
FROM Students
GROUP BY Gender
```
```sql
--Part-time job vs attendance rate and gpa
SELECT PartTimeJob,
       Round(Avg(GPA), 2) AS Avg_GPA,
       Round(Avg(AttendanceRate), 2) / 100 AS Avg_Attendance_Rate
FROM Academic_Performance ap
JOIN Students s ON ap.StudentID = s.StudentID
GROUP BY PartTimeJob
ORDER BY Avg_GPA DESC
```
```sql
--Extra curricular activities vs attendance rate and gpa
SELECT ExtraCurricularActivities,
       Round(Avg(GPA), 2) AS Avg_GPA,
       Round(Avg(AttendanceRate), 2) / 100 AS Avg_Attendance_Rate
FROM  Academic_Performance ap
JOIN Students s ON ap.StudentID = s.StudentID
GROUP BY ExtraCurricularActivities
ORDER BY Avg_GPA DESC
```
```sql
--Different housing situations vs attendance rate and gpa
SELECT Housing,
       Round(Avg(GPA), 2) AS Avg_GPA,
       Round(Avg(AttendanceRate), 2) / 100 AS Avg_Attendance_Rate
FROM  Academic_Performance ap
JOIN Students s ON ap.StudentID = s.StudentID
GROUP BY Housing
ORDER BY Avg_GPA DESC
```
```sql
--Average attendance rate and number of study hours per week for students with different GPAs
WITH GPA_Range AS (
	SELECT StudentID,
           StudyHoursPerWeek,
		   AttendanceRate,
           CASE
              WHEN GPA < 1 THEN '< 1.0'
              WHEN GPA >= 1 AND GPA < 1.5 THEN '1.0 - 1.4'
              WHEN GPA >= 1.5 AND GPA < 2 THEN '1.5 - 1.9'
              WHEN gpa >= 2 AND GPA < 2.5 THEN '2.0 - 2.4'
              WHEN GPA >= 2.5 AND GPA < 3 THEN '2.5 - 2.9'
              WHEN GPA >= 3 AND GPA < 3.5 THEN '3.0 - 3.4'
              WHEN GPA >= 3.5 AND GPA < 4 THEN '3.5 - 3.9'
              ELSE '4.0'
          END AS GPA_Rating
	FROM Academic_Performance)

SELECT GPA_Rating,
       Count(StudentID) AS cnt_Students,
       Round(Avg(AttendanceRate), 2) / 100 Avg_AttandanceRate,
       Avg(StudyHoursPerWeek)AS Avg_StudyHoursPerWeek
FROM GPA_Range
GROUP BY GPA_Rating;
```
```sql
--GPA's of the top 10 students per major
WITH RankedStudentsTop AS (
	SELECT s.StudentID,
           Major,
           GPA,
           Row_number() OVER (partition BY Major ORDER BY GPA DESC) AS TopRank
    FROM Students s
	JOIN Academic_Performance ap ON s.StudentID = ap.StudentID)

SELECT StudentID,
       Major,
       Round(GPA, 2) AS GPA,
       TopRank
FROM RankedStudentstop
WHERE TopRank <= 10;
```
```sql
--GPA's of the bottom 10 students per major
WITH RankedStudentsBottom AS (
	SELECT s.StudentID,
           Major,
           GPA,
           Row_number() OVER (partition BY Major ORDER BY GPA ASC) AS BottomRank
    FROM Students s
	JOIN Academic_Performance ap ON s.StudentID = ap.StudentID)

SELECT StudentID,
       Major,
       Round(GPA, 2) AS GPA,
       BottomRank
FROM RankedStudentsBottom
WHERE BottomRank <= 10;
```


# Process of Building the Report

### Steps in Creating the Report:

### 1. **Import SQL Queries:**  
   Imported the SQL queries into Power BI to create the fact and dimension tables.

### 2. **Connect Tables in Model View:**  
   Established connections between different tables in the model view, using a STAR schema design.
<p align="center">   
<img src="" alt="model" width="600" />
</p> 

### 3. **Built the different visuals** 

### 4. **Added more pages** 



# Findings



# Recommendations



