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
--Count and Percent of Total Students by Major
SELECT Major,
       Count(StudentID) AS Count_Students,
       Round(Cast(Count(*) * 100.0 / (SELECT Count(*) FROM Students) AS FLOAT), 2) AS Percent_of_Total_Students
FROM Students
GROUP BY Major
ORDER BY Percent_of_Total_Students DESC;
```
```sql
--General Averages per Major
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
--Students Who Have a Higher GPA Than the Overall Average
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
--Which Students Are Performing Well or Could Use Some Improvement (Is GPA greater than 3.0?)
SELECT s.StudentID,
       Major,
       Round(GPA, 2) AS GPA,
       StudyHoursPerWeek,
       iif(GPA > 3.0, 'Performing Well', 'Needs Improvement') AS PerformanceCategory
FROM   Students s
JOIN Academic_Performance ap ON s.StudentID = ap.StudentId;
```
```sql
--Attendance Rate and GPA by Gender and Financial Aid
SELECT 
	s.Major,
    s.Gender,
	s.FinAid,
    Count(s.StudentID) AS Total_Students,
    Sum(CASE WHEN s.FinAid = 'Yes' THEN 1 ELSE 0 END) AS Students_With_FinAid,
    Round(Cast(Sum(CASE WHEN s.FinAid = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*) AS FLOAT), 2) AS Percent_With_FinAid,
	Round(Avg(ap.GPA), 2) AS Avg_GPA,
    Round(Avg(ap.AttendanceRate), 2) / 100 AS Avg_Attendance_Rate
FROM Students s
JOIN Academic_Performance ap ON s.StudentID = ap.StudentId
GROUP BY s.Gender, s.Major, s.FinAid
ORDER BY s.Major;
```
```sql
--Part-Time Job vs Attendance Rate and GPA
SELECT s.PartTimeJob,
	   Count(s.StudentID) as Count_Students,
       s.Major,
       Round(Avg(ap.GPA), 2) AS Avg_GPA,
       Round(Avg(ap.AttendanceRate), 2) / 100 AS Avg_Attendance_Rate
FROM Academic_Performance ap
JOIN Students s ON ap.StudentID = s.StudentID
GROUP BY s.PartTimeJob, s.Major
ORDER BY Major;
```
```sql
--Extra Curricular Activities vs Attendance Rate and GPA
SELECT s.ExtraCurricularActivities,
	   Count(s.StudentID) as Count_Students,
       s.Major,
       Round(Avg(ap.GPA), 2) AS Avg_GPA,
       Round(Avg(ap.AttendanceRate), 2) / 100 AS Avg_Attendance_Rate
FROM Academic_Performance ap
JOIN Students s ON ap.StudentID = s.StudentID
GROUP BY s.ExtraCurricularActivities, s.Major
ORDER BY Major;
```
```sql
--Different Housing Situations vs Attendance Rate and GPA
SELECT s.Housing,
	   Count(s.StudentID) as Count_Students,
	   s.Major,
       Round(Avg(ap.GPA), 2) AS Avg_GPA,
       Round(Avg(ap.AttendanceRate), 2) / 100 AS Avg_Attendance_Rate 
FROM Academic_Performance ap
JOIN Students s ON ap.StudentID = s.StudentID
GROUP BY s.Housing, s.Major
ORDER BY Major;
```
```sql
--Average Attendance Rate and Number of Study Hours per Week for Students With Different GPA's
WITH GPA_Range AS (
    SELECT ap.StudentID,
           ap.StudyHoursPerWeek,
           ap.AttendanceRate,
           s.Major,
           CASE
               WHEN ap.GPA < 1 THEN '< 1.0'
               WHEN ap.GPA >= 1 AND ap.GPA < 1.5 THEN '1.0 - 1.4'
               WHEN ap.GPA >= 1.5 AND ap.GPA < 2 THEN '1.5 - 1.9'
               WHEN ap.GPA >= 2 AND ap.GPA < 2.5 THEN '2.0 - 2.4'
               WHEN ap.GPA >= 2.5 AND ap.GPA < 3 THEN '2.5 - 2.9'
               WHEN ap.GPA >= 3 AND ap.GPA < 3.5 THEN '3.0 - 3.4'
               WHEN ap.GPA >= 3.5 AND ap.GPA < 4 THEN '3.5 - 3.9'
               ELSE '4.0'
           END AS GPA_Rating
       FROM Academic_Performance ap
       JOIN Students s ON ap.StudentID = s.StudentID
)

SELECT Major,
	   GPA_Rating,
	   COUNT(StudentID) AS Count_Students,
	   ROUND(AVG(AttendanceRate), 2) / 100 AS Avg_AttendanceRate,
	   AVG(StudyHoursPerWeek) AS Avg_StudyHoursPerWeek
FROM GPA_Range
GROUP BY Major, GPA_Rating
ORDER BY Major, GPA_Rating;
```
```sql
--GPA's of the Top 10 Students per Major
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
--GPA's of the Bottom 10 Students per Major
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



