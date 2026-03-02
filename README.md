# 📊 Turnover Rate Across Periods Analysis - mid sized technology enabled organization | SQL
  
### Author: Vo Tran Mai Khanh
### Date: 02/2026
### Tools Used: SQL

## 📌 Background & Overview  

### Objective - This project uses SQL to analyze turnover rate from technology organization to:
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

### 📊 Data Source  
- Source: HR workforce dataset representing a mid-sized technology-enabled organization   
- Size: 311 employee records across multiple HR related tables
- Format: .csv

### 📊 Data Structure & Relationships  

### 1️⃣ Tables Used:  
- **Employees data** – workforce master data covering employment status, termination details, hire/termination dates, organizational identifiers (Department ID, Position ID)   
- **Positions** – job titles and department mapping  

### 2️⃣ Table Schema & Data Snapshot  

**Table 1:** Employee Table


<img width="892" height="200" alt="image" src="https://github.com/user-attachments/assets/7beb1553-92b6-487e-a3b0-f4948e5c14b1" />

**Table 2:** Position


<img width="306" height="107" alt="image" src="https://github.com/user-attachments/assets/5733e6ab-11a5-45ac-898d-80f28d748b5d" />


## ⚒️ Main Process

1️⃣ Data Cleaning & Preprocessing  
2️⃣ Exploratory Data Analysis (EDA)  
3️⃣ SQL Analysis 

### Task 1: Analyze turnover rate by year to identify trends over time
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

### Task 2: Analyze turnover rate contribution by Employment Status - Voluntarily terminated & Terminate by Cause
Following the annual turnover trend analysis, examining turnover by Employment Status (Voluntarily Terminated vs. Terminated for Cause) helps clarify the underlying drivers of attrition. While the overall rate shows whether turnover is rising or falling, it does not explain why employees leave. Breaking it down by status distinguishes between employee driven exits and company initiated terminations, providing clearer insight into whether turnover reflects retention challenges or internal management decisions

### Code:
```sql
WITH years AS(
  SELECT DISTINCT EXTRACT (YEAR FROM hr.DateofTermination)
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr),

avg_headcount AS(
SELECT
  b.Year,
  (b.beginning_headcount + e.end_headcount)/2 AS avg_headcount
FROM `hr-operations-analysis.hr_raw.beginning_headcount` AS b
LEFT JOIN `hr-operations-analysis.hr_raw.end_headcount` AS e
ON b.Year = e.Year),

terminated_by_status AS(
SELECT
  EXTRACT (YEAR FROM DateofTermination) AS Year,
  COUNT(EmployeeID) AS total_terminated,
  EmploymentStatus
FROM `hr-operations-analysis.hr_raw.new_employee_data`
WHERE EmploymentStatus IN ('Voluntarily Terminated','Terminated for Cause')
GROUP BY Year, EmploymentStatus)

SELECT
  t.Year,
  t.EmploymentStatus,
  t.total_terminated,
  ROUND(SAFE_DIVIDE(t.total_terminated,a.avg_headcount) * 100,2) AS turnover_by_status
FROM terminated_by_status AS t
LEFT JOIN avg_headcount AS a
ON t.Year = a.Year
ORDER BY t.Year, t.EmploymentStatus;
```

### Result

<img width="418" height="272" alt="image" src="https://github.com/user-attachments/assets/d8906e85-6d5f-4359-a099-8969442109aa" />

The breakdown of turnover by Employment Status shows that **voluntary resignations consistently account** for the majority of annual turnover. From 2010 to 2018, **voluntary turnover** remains significantly higher than terminations for cause, **peaking in 2015–2016**, while **involuntary** turnover stays low and stable, **generally below 2.5%**. This indicates that fluctuations in overall turnover are **primarily driven by employee initiated exits** rather than company decisions. The spike in 2015–2016 suggests a retention challenge during that period, whereas the decline in 2017 reflects improved stability. Overall, attrition appears more related to engagement, career progression, or external opportunities, implying that retention strategies rather than stricter performance management should be the primary focus

### Task 3: Analyze overall turnover rate by position
Building on the turnover analysis by status and year, turnover was further examined by position to identify roles contributing most to employee exits. While the time based analysis reveals how turnover evolves across years, position level analysis highlights where turnover is concentrated within the organization. This provides clearer insight into functional areas that may require deeper investigation or targeted retention strategies

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


Full period position analysis shows that attrition is primarily **concentrated in operational roles rather than strategic or executive functions.** Production related positions exhibit the highest cumulative turnover, with **Production Technician II (45.61%), Production Technician I (37.96%)** and **Production Manager II (38.46%)** recording the most significant exits, supported by relatively large headcounts. In contrast, technical and IT roles such as **Software Engineer (33.33%), Data Analyst (25%) and IT Manager (25%)** remain at moderate levels, while leadership roles show minimal or no turnover. Although a few positions display 100% turnover, these are based on very small headcounts and do not represent systemic risk. he pattern suggests that attrition is more operationally driven rather than indicative of a strategic talent retention issue

### Task 4: Analyze overall turnover rate by tenure
In addition to position level analysis, turnover was examined by tenure to identify the career stages where attrition is most concentrated. While position analysis highlights “where” employees leave, tenure analysis clarifies “when” they leave. This helps determine whether turnover is linked to onboarding gaps, mid-career stagnation or long-term retention challenges, supporting more stage specific retention strategies

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
### Result

<img width="304" height="89" alt="image" src="https://github.com/user-attachments/assets/868d97bd-24ad-4235-a28d-8e9280451c0d" />

Tenure level analysis shows that **attrition is most concentrated among mid tenure employees.** **The 3–5 years group accounts for 49.04% of total terminations,** representing nearly half of all exits, followed by the 1–2 years group (31.73%). In contrast, early stage employees (0–1 year) contribute only 3.85%, while long-tenured employees (5+ years) represent 15.38% of exits. This pattern suggests that **turnover is not primarily driven by onboarding or early hiring issues** but rather by mid career dynamics. The high proportion of exits within the 3–5 year band may indicate potential career progression bottlenecks, limited advancement opportunities or compensation related factors affecting employees at a critical growth stage

### Task 5: Time series analysis of high turnover positions
Based on Task 3 findings, three high risk positions — **Production Technician II (45.61%), Production Manager II (38.46%) and Production Technician I (37.96%)** — meeting both scale (headcount ≥10) and turnover threshold (≥20%) criteria were selected for longitudinal analysis

### Code
```sql
WITH top_positions AS (
SELECT Position
FROM `hr-operations-analysis.hr_raw.overall_turnover_position`
WHERE headcount_per_position >= 10
AND turnover_rate_by_position >= 20),

yearly_base AS (
SELECT
  EXTRACT(YEAR FROM hr.DateofTermination) AS Year,
  po.Position,
  COUNT(*) AS total_terminated
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
JOIN `hr-operations-analysis.hr_raw.Position` AS po
ON hr.PositionID = po.PositionID
WHERE hr.DateofTermination IS NOT NULL
AND po.Position IN (SELECT Position FROM top_positions)
GROUP BY Year, po.Position),

position_headcount_year AS (
  SELECT
    y.Year,
    po.Position,
    COUNT(DISTINCT hr.EmployeeID) AS headcount_year
  FROM (
    SELECT DISTINCT EXTRACT(YEAR FROM DateofTermination) AS Year
    FROM `hr-operations-analysis.hr_raw.new_employee_data`
    WHERE DateofTermination IS NOT NULL) AS y
  JOIN `hr-operations-analysis.hr_raw.new_employee_data` AS hr
    ON hr.DateofHire <= DATE(y.Year,12,31)
   AND (hr.DateofTermination IS NULL 
        OR hr.DateofTermination >= DATE(y.Year,1,1))
  JOIN `hr-operations-analysis.hr_raw.Position` AS po
    ON hr.PositionID = po.PositionID
  WHERE po.Position IN (SELECT Position FROM top_positions)
  GROUP BY y.Year, po.Position)

SELECT
  t.Position,
  t.Year,
  t.total_terminated,
  h.headcount_year,
  ROUND(SAFE_DIVIDE(t.total_terminated, h.headcount_year) * 100, 2) AS turnover_rate
FROM yearly_base t
LEFT JOIN position_headcount_year h
ON t.Year = h.Year AND t.Position = h.Position
ORDER BY t.Position, t.Year;
```
### Result

<img width="508" height="357" alt="image" src="https://github.com/user-attachments/assets/fe91cabf-ff64-42a5-83eb-27b373ac9ba6" />

Production Technician I shows a structural upward trend, **rising from 2.63% (2012) to 13.04% (2016)**. In contrast, Production Technician II experienced a **one-time spike at 24.24% (2013)** before stabilizing, while Production Manager II **peaked at 25% (2012) but remained around ~11% afterward**. The findings indicate sustained attrition pressure primarily in Production Technician I.

## Final Conclusion & Recommendations  

Based on the insights above, the following strategic actions are recommended for HR Team:  

### 1️⃣ Growth-Driven Attrition, Not Structural Instability
Turnover peaked in 2015 (10.34%) during rapid workforce expansion, then stabilized as growth slowed. This suggests attrition was largely associated with scaling pressure rather than systemic retention failure.

### Key Actions
- Implement structured workforce planning before large scale hiring phases
- Forecast expected attrition buffers in expansion models
- Strengthen onboarding capacity during rapid growth periods

### 2️⃣ Attrition is Primarily Voluntary
Most separations were employee driven rather than company initiated, indicating retention risk rather than performance cleansing

### Key Actions
- Standardize exit reason categorization for voluntary exits
Conduct proactive stay interviews for mid-tenure employees
- Introduce early retention risk flags based on tenure and role

### 3️⃣ Operational Roles Drive Attrition Risk
Turnover is concentrated in production related roles (Technician I, Technician II, Manager II), while leadership and strategic roles remain stable. This signals operational continuity risk rather than strategic talent loss

### Key Actions
- Redesign career progression pathways for production roles
- Establish structured skill ladders with defined promotion timelines
- Build operational staffing buffers during peak production periods

### 4️⃣ Mid Tenure Employees (3–5 Years) Represent the Highest Risk Segment
Nearly half of all terminations occur within the 3–5 year tenure band, indicating potential stagnation, compensation gaps or limited advancement opportunities at a critical career stage

### Key Actions
- Launch mid-career progression programs (3–5 year focus)
- Conduct targeted compensation benchmarking for this segment
- Offer internal mobility opportunities before the 4-year mark

### 5️⃣ Structural Risk in Production Technician I
Production Technician I shows a sustained upward turnover trend, indicating a structural pressure point rather than a one-time spike.

### Key Actions
- Implement milestone-based pay and title progression within the first 24 months
- Strengthen frontline manager training and engagement capability
- Introduce retention check-ins at 18–24 months for entry-level production staff
