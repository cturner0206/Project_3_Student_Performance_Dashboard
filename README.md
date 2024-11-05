# Project 3: Student Performance Dashboard 
   
<img src="https://github.com/user-attachments/assets/5a705fcf-1411-49a3-a444-2afb2e629e95" alt="dash" width="900" />


# Table of Contents 
- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Loading Data into SSMS ](#loading-data-into-ssms)
- [Creating the Queries](#creating-the-queries)
- [Process of Building the Report](#process-of-building-the-report)
- [Findings](#findings)
- [Recommendations](#recommendations)


# Project Overview

The primary goal of this project was to identify and analyze the various factors that affect student GPA. To do this, I used SQL queries to calculate averages, distributions, and other relevant data and imported the queries into PowerBI to create an interactive dashboard. The dashboard aims to provide insights that can help educators and administrators make informed decisions to enhance student performance.

# Data Source
The data used for this project comes from the `student_performance_data.csv` and `student_performance_data1.csv` files. The original dataset file had both files combined, but I split it up to segment the data based on student data and academic performance metrics. 


# Loading Data into SSMS  

Loading the data into Microsoft SQL Server Management Studio involved the following steps:

- Created new Student_Data database.
- Imported `student_performance_data.csv` and `student_performance_data1.csv` files in SQL Server Management Studio as two seperate tables.
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

1. Identifying Relevant Data Points that can affect GPA:
- Attendance rate: How does housing location, extracurricular activity participation, and having a part-time job outside of school affect attendance rates and GPA?
- Major: Does the difficulty of a major affect overall GPA or attendance rates? Ex: Engineering vs Arts.
- Effort: Total study hours per week and attendance rate affect on GPA. 
- Assistance: Does receiving financial aid affect GPA or attendance?
- Demographics: Age and Gender affects on GPA.
- Distribution: How many students are in each GPA range, who is performing well, top 10 and bottom 10 students by major.
  
2. Deciding on visuals
- I believe that simple matrices, cards, bar charts, and tables would be the most effective visuals for this dashboard as they are easy to read while conveying all the data appropriately. 
- Slicers by major and by student performance will allow for further filtering and analysis.
- Wont be any time series analysis or visuals as there wasn't any time data provied in the dataset. 

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

### 1. **Importing the Data:**  
Imported all of the different SQL queries and both of the CSV files into Power BI. 

### 2. **Connect Tables in Model View:**  
The fact tables in the report consisted of the two different CSV files (student data and academic performance metrics).
 > **Note:** One fact table in PowerBI is the Students table from the SSMS database while the other is a SQL query that takes the averages of all of the columns that were in the Academic Performance table in SSMS.

Used a STAR schema and established relationships between the SQL query tables and the fact tables.
<p align="center">   
<img src="https://github.com/user-attachments/assets/21b9c917-ae76-45b7-a740-93323e65f981" alt="model" width="600" />
</p> 

### 3. **Building the Different Visuals and Adding Pages** 

   - Decided to have the main page of the dashboard be more of an overview while having an additional attendance rate and GPA page that would have additional data. 
   - Created cards, bar charts, tables, and matrices from the SQL queries.
   - Altered the color palette and segmented the visuals using boxes.
   - Applied conditional formatting to improve visual clarity on some visuals. 
   - Created slicers for selecting between majors and for selecting between student performance on the additional GPA page.
   - Created buttons to navigate between the different pages. When on the overview page, clicking on the arrows on the average attendance rate and average GPA by major bar charts swap to the additional metrics pages respectivly. When on the additional pages, you can go back to the overview page by clicking on the back button on the top bar.
   - Added buttons to clear all slicers.
  


# Findings

## GPA 

- **Business** has the highest average GPA at **3.25**.
- **Engineering** has the lowest average GPA at **2.88**, while **Science** follows closely with an average GPA of **2.96**. This difference is likely due to the challenging nature of these majors, leading to lower average GPAs for students in these fields.
- **Across all majors**, GPA distribution follows a bell curve, with students in higher GPA ranges reporting higher study hours per week and better attendance rates.
- **Gender** and **financial aid status** do not significantly affect average GPA across all majors.
- **Top-performing Engineering students** (those with a GPA of 3.5+) report the highest average study hours per week, ranging from **27-30 hours**.


## Attendance Rate 

- **Engineering** students have the highest average attendance rate at **74.1%**.
- **Science** students have the lowest average attendance rate at **70.9%**.
- **Arts**, **Education**, and **Science** majors tend to have a higher average attendance rate for students who receive **financial aid** (**1%-3%** higher).
- **Education** and **Engineering** majors show higher attendance rates among students who **do not receive financial aid** (**1%** higher).

- Students who live **on-campus** have the highest average attendance rate at **76.5%**, compared to those who **commute** (**71.5%**) or live **off-campus** (**69.9%**).
- **Students without a part-time job** tend to have a higher average attendance rate (**75.1%**) and a higher average GPA (**3.23**) compared to those with a **part-time job** whose attendance rate is **70.5%** and average GPA is **2.97**.
- Students who **do not participate in extracurricular activities** have a slightly higher average attendance rate (**73.3%**) compared to those who do (**71.9%**).
- There is a **slight gender difference** in attendance rates. **Males** generally have a **1%-3%** lower attendance rate across most majors compared to females, except in **Education**, where males have a **2.4% higher** attendance rate.



## Other

**Science** majors make up **16%** of the student population, which is notably lower compared to other majors, which range from **19.4% to 22.8%**.





# Recommendations
- look into science major (lowest avg attendance rate, second lowest avg gpa, lowest amount of study hours per week, lowest number of students)


