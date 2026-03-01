# 📊 Turnover Rate Across Periods Analysis - mid sized technology enabled organization | SQL
  
### Author: Vo Tran Mai Khanh
### Date: 02/2026
### Tools Used: SQL

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
4. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)

## 📌 Background & Overview  

### Objective:
- This project uses SQL to analyze turnover rate from technology organization to:
- Examine turnover trends over time
- Differentiate turnover types (voluntary vs. involuntary)
- Identify root causes of employee exits
- Segment workforce to detect retention risks
- Support strategic retention planning  

### 👤 Who is this project for?  
- HR Leadership & People Analytics Team – to monitor turnover trends and design targeted retention strategies
- Department Managers – to identify high risk teams and address engagement or performance gaps
- Executive Leadership – to understand workforce stability and its impact on operational continuity
- Talent Acquisition Team – to adjust hiring strategy based on turnover patterns and tenure risks
- Workforce Planning / Operations Team – to support succession planning and capacity forecasting

## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: HR workforce dataset representing a mid-sized technology-enabled organization   
- Size: 311 employee records across multiple HR related tables
- Format: .csv

### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used:  
- **Employees** – employee demographic and information  
- **Positions** – job titles and department mapping  
- **Departments** – department details  
- **Terminations** – employee exit records and reasons

#### 2️⃣ Table Schema & Data Snapshot  

Table 1: Employee Table

<img width="892" height="200" alt="image" src="https://github.com/user-attachments/assets/7beb1553-92b6-487e-a3b0-f4948e5c14b1" />

Table 2: Department

<img width="307" height="107" alt="image" src="https://github.com/user-attachments/assets/1121950f-2a86-4ab7-8b54-eb36baf3815a" />

Table 3: Position

<img width="306" height="107" alt="image" src="https://github.com/user-attachments/assets/5733e6ab-11a5-45ac-898d-80f28d748b5d" />

Table 4: Manager

<img width="203" height="106" alt="image" src="https://github.com/user-attachments/assets/fba234dc-2cbd-4014-8954-aa879227117b" />

Taable 5: Performance

<img width="226" height="88" alt="image" src="https://github.com/user-attachments/assets/4401a26e-c556-431c-b3b3-89aeedd0c396" />

## ⚒️ Main Process

1️⃣ Data Cleaning & Preprocessing  
2️⃣ Exploratory Data Analysis (EDA)  
3️⃣ SQL Analysis 

## Task 1: Analyze turnover rate by year to identify trends over time
Annual turnover analysis is essential to detect long term workforce trends, identify potential retention risks and assess whether turnover changes are structural or event driven. This metric provides an early signal of workforce instability and supports strategic workforce planning

### Code:
```sql
 WITH years AS(
  SELECT DISTINCT EXTRACT (YEAR FROM DateofHire) AS Year
FROM `hr-operations-analysis.hr_raw.new_employee_data`),

beginning_headcount AS(
  SELECT 
  y.Year,
  COUNT(EmployeeID) AS beginning_headcount
FROM years AS y
  LEFT JOIN `hr-operations-analysis.hr_raw.new_employee_data` AS hr
  ON hr.DateofHire <= DATE(CONCAT(y.Year,'-01-01'))
  AND (hr.DateofTermination IS NULL OR (hr.DateofTermination >= DATE(CONCAT(y.Year,'-01-01'))))
GROUP BY y.Year),

end_headcount AS(
SELECT 
  y.Year,
  COUNT(EmployeeID) AS end_headcount
FROM years AS y
  LEFT JOIN `hr-operations-analysis.hr_raw.new_employee_data` AS hr
  ON hr.DateofHire <= DATE(CONCAT(y.Year,'-12-31'))
  AND (hr.DateofTermination IS NULL OR (hr.DateofTermination >= DATE(CONCAT(y.Year,'-12-31'))))
GROUP BY y.Year),

termination_by_year AS(
SELECT
  EXTRACT (YEAR FROM DateofTermination) AS Year,
  COUNT(EmployeeID) AS total_terminated
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
WHERE DateofTermination IS NOT NULL
GROUP BY Year)

SELECT
  y.Year,
  IFNULL(b.beginning_headcount,0) AS Beginning_headcount,
  IFNULL(e.end_headcount,0) AS End_headcount,
  IFNULL(t.total_terminated,0) AS Total_terminated,
  IFNULL(
    ROUND(
    t.total_terminated / ((b.beginning_headcount + e.end_headcount) / 2) * 100,2),0) AS Turnover_rate_by_year
FROM years AS y
  LEFT JOIN beginning_headcount AS b ON y.Year = b.Year
  LEFT JOIN end_headcount AS e ON y.Year = e.Year
  LEFT JOIN termination_by_year AS t ON y.Year = t.Year
ORDER BY y.Year
```
### Result

<img width="449" height="253" alt="image" src="https://github.com/user-attachments/assets/b31519d5-f0a0-4682-b048-546a1da32805" />

Overall, turnover ranged from 3.6% to 10.3% between 2006 and 2018, **peaking in 2015 (10.34%)** during the company’s rapid expansion phase, when headcount grew significantly from 13 to 229 employees. Turnover **gradually increased** alongside workforce growth from **2010 to 2015,** suggesting that attrition was linked to scaling pressure rather than structural retention issues. Following 2016, turnover **declined notably to 3.64% in 2017,** indicating improved stability as organizational growth slowed. **No abnormal spikes or retention crisis were observed **and hiring consistently offset attrition during peak years

## Task 2: Analyze overall turnover rate by position
After reviewing the overall turnover trend over time, analyzing turnover by position helps identify which roles contribute most to employee exits. While the time trend shows how turnover changes across years, position-level analysis explains where the turnover is happening within the organization. This step provides a clearer view of which functional areas may require deeper investigation or targeted retention strategies

### Code:
```sql
WITH terminate_by_position AS(
SELECT
  Position,
  COUNT(EmployeeID) AS total_terminated
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
  LEFT JOIN `hr-operations-analysis.hr_raw.Position` AS po
  ON hr.PositionID = po.PositionID
WHERE DateofTermination IS NOT NULL
GROUP BY Position),

headcount_per_position AS(
SELECT
  Position,
  COUNT(DISTINCT EmployeeID) AS headcount_per_position
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
  LEFT JOIN `hr-operations-analysis.hr_raw.Position` AS po
  ON hr.PositionID = po.PositionID
GROUP BY Position)

SELECT
  h.Position,
  h.headcount_per_position,
  IFNULL(t.total_terminated,0) AS total_terminated,
  IFNULL(ROUND(SAFE_DIVIDE(t.total_terminated,headcount_per_position)*100,2),0) AS turnover_rate_by_position
FROM headcount_per_position AS h
  LEFT JOIN terminate_by_position AS t
  ON h.Position = t.Position
ORDER BY turnover_rate_by_position DESC;
```
### Result

<img width="458" height="359" alt="image" src="https://github.com/user-attachments/assets/203335d1-2c71-459f-802c-933b03d34121" />

<img width="452" height="197" alt="image" src="https://github.com/user-attachments/assets/2a3b9ee7-305e-451d-b84d-91fd0ed675b1" />

Overall position level analysis shows that attrition is primarily **concentrated in operational roles rather than strategic or executive functions.** Production related positions exhibit the highest cumulative turnover, with **Production Technician II (45.61%), Production Technician I (37.96%)** and **Production Manager II (38.46%)** recording the most significant exits, supported by relatively large headcounts. In contrast, technical and IT roles such as **Software Engineer (33.33%), Data Analyst (25%) and IT Manager (25%)** remain at moderate levels, while leadership roles show minimal or no turnover. Although a few positions display 100% turnover, these are based on very small headcounts and do not represent systemic risk. Overall, the pattern suggests that attrition is more operationally driven rather than indicative of a strategic talent retention issue

## Task 3: Analyze overall turnover rate by tenure
After identifying where turnover occurs by position, analyzing turnover by tenure helps determine at which stage of the employee lifecycle attrition is most concentrated. While position analysis explains “where” employees leave, tenure analysis explains “when” they leave in their career journey within the organization. This provides insight into whether turnover is driven by early-stage hiring and onboarding challenges, mid-career stagnation, or long-term retention issues. Understanding tenure-based patterns allows the organization to design more targeted retention strategies aligned with different career stages

### Code:
```sql
SELECT
CASE
  WHEN DATE_DIFF(DateofTermination, DateofHire, YEAR) <1 THEN '0-1 year'
  WHEN DATE_DIFF(DateofTermination, DateofHire, YEAR) BETWEEN 1 AND 2 THEN '1-2 years'
  WHEN DATE_DIFF(DateofTermination, DateofHire, YEAR) BETWEEN 3 AND 5 THEN '3-5 years'
  ELSE '+5 years' END AS Tenure,

COUNT(*) AS terminated_by_tenure,

ROUND(
  SAFE_DIVIDE(
    COUNT(*),
    SUM(COUNT(*)) OVER ()) * 100,2) AS pct_terminated_by_tenure

FROM `hr-operations-analysis.hr_raw.new_employee_data`
WHERE Termination = 1
GROUP BY Tenure;
```
## 🔎 Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we would recommend the [stakeholder team] to consider the following:  

📍 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3

**_📌Remember to summarize the most core insights/ observations you extract from the entire projects. 
 Recap ONLY key actions/ recommendations. DO NOT copy paste everything above_**
