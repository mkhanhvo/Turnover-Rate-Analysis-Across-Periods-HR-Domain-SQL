<img width="1440" height="817" alt="image" src="https://github.com/user-attachments/assets/de9e242a-b212-4a2d-a837-68b2930ba846" />

# 📊 Turnover Rate Across Periods Analysis - mid sized technology enabled organization | SQL
  
### Author: Vo Tran Mai Khanh
### Date: 02/2026
### Tools Used: SQL

## 📌 Background & Overview  

### 📖 Objective - This project uses SQL to analyze turnover rate from technology organization to:
- Examine turnover trends over time
- Differentiate turnover types (voluntary vs. involuntary)
- Analyze turnover rates by position and tenure
- Segment workforce to detect retention risks

### 👤 Who is this project for?  
- HR Leadership & People Analytics Team – to monitor turnover trends and design targeted retention strategies
- Department Managers – to identify high risk teams
- Executive Leadership – to understand workforce stability and its impact on operational continuity
- Talent Acquisition Team – to adjust hiring strategy based on turnover patterns and tenure risks
- Workforce Planning / Operations Team – to support succession planning and capacity forecasting 

### 📊 Data Source  
- Source: HR workforce dataset representing a mid-sized technology-enabled organization   
- Size: 311 employee records across multiple HR related tables
- Format: .csv

### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used:  
- **Employees data** – workforce master data covering employment status, termination details, hire/termination dates, organizational identifiers (Department ID, Position ID)   
- **Positions** – job titles and department mapping  

#### 2️⃣ Table Schema & Data Snapshot  

**Table 1:** Employee Table


<img width="892" height="200" alt="image" src="https://github.com/user-attachments/assets/7beb1553-92b6-487e-a3b0-f4948e5c14b1" />

**Table 2:** Position


<img width="306" height="107" alt="image" src="https://github.com/user-attachments/assets/5733e6ab-11a5-45ac-898d-80f28d748b5d" />


## ⚒️ Main Process

1️⃣ Data Cleaning & Preprocessing  
2️⃣ Exploratory Data Analysis (EDA)  
3️⃣ SQL Analysis 

### 📌 Task 1: Analyze turnover rate by year to identify trends over time
Annual turnover analysis is essential to detect long term workforce trends, identify potential retention risks and assess whether turnover changes are structural or event driven. This metric provides an early signal of workforce instability and supports strategic workforce planning

### Code:
<details>
<summary><b>View SQL Code</b></summary>
  
```sql
 -- Get the list of years appearing in hire or termination dates/ Lấy danh sách các năm xuất hiện trong ngày vào làm & nghỉ việc
WITH year AS (
SELECT DISTINCT Year
FROM (
  SELECT EXTRACT(YEAR FROM DateofHire) AS Year
  FROM `hr-operations-analysis.hr_raw.new_employee_data`
  UNION ALL
  SELECT EXTRACT(YEAR FROM DateofTermination) AS Year
  FROM `hr-operations-analysis.hr_raw.new_employee_data`
  WHERE DateofTermination IS NOT NULL)),

-- Calculate headcount at the beginning of each year/ Tính số nhân viên đang làm việc tại thời điểm đầu năm
beginning_headcount AS(
  SELECT 
  y.Year,
  COUNT(EmployeeID) AS Beginning_headcount
FROM year AS y
  LEFT JOIN `hr-operations-analysis.hr_raw.new_employee_data` AS hr
  ON hr.DateofHire <= DATE(y.Year,1,1)
  AND (hr.DateofTermination IS NULL OR (hr.DateofTermination >= DATE(y.Year,1,1)))
GROUP BY y.Year),

-- Calculate headcount at the end of each year/ Tính số nhân viên đang làm việc tại thời điểm cuối năm
end_headcount AS(
SELECT 
  y.Year,
  COUNT(EmployeeID) AS End_headcount
FROM year AS y
  LEFT JOIN `hr-operations-analysis.hr_raw.new_employee_data` AS hr
  ON hr.DateofHire <= DATE(y.Year,12,31)
  AND (hr.DateofTermination IS NULL OR (hr.DateofTermination > DATE(y.Year,12,31)))
GROUP BY y.Year),

-- Count total employee exits in each year/ Đếm số nhân viên nghỉ việc trong từng năm
termination_by_year AS(
SELECT
  EXTRACT (YEAR FROM DateofTermination) AS Year,
  COUNT(EmployeeID) AS total_terminated
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
WHERE DateofTermination IS NOT NULL
GROUP BY Year)

-- Calculate annual turnover rate/ Tính tỷ lệ nghỉ việc
SELECT
  y.Year,
  IFNULL(b.beginning_headcount,0) AS Beginning_headcount,
  IFNULL(e.end_headcount,0) AS End_headcount,
  IFNULL(t.total_terminated,0) AS Total_terminated,
  IFNULL(
    ROUND(
    t.total_terminated / ((b.beginning_headcount + e.end_headcount) / 2) * 100,2),0) AS Turnover_rate_by_year -- số người nghỉ / headcount trung bình trong năm
FROM year AS y
  LEFT JOIN beginning_headcount AS b ON y.Year = b.Year
  LEFT JOIN end_headcount AS e ON y.Year = e.Year
  LEFT JOIN termination_by_year AS t ON y.Year = t.Year
ORDER BY y.Year
```
</details>

### Result

<img width="449" height="253" alt="image" src="https://github.com/user-attachments/assets/b31519d5-f0a0-4682-b048-546a1da32805" />

Employee turnover remained negligible before 2010 due to the small workforce size. As the organization expanded rapidly, turnover gradually increased and peaked at **10.34% in 2015**. After a temporary decline in 2016–2017, the rate **increased again in 2018 (~6.1%)**

This pattern suggests that **organizational growth and workforce expansion are closely associated with higher attrition risk**, likely due to onboarding challenges, role adjustments and cultural adaptation during scaling phases

### Recommendation
- Monitor turnover closely during periods of rapid workforce growth
- Strengthen onboarding and early engagement programs to support new employees
- Implement early retention monitoring for employees within their **first 12–24 months**

### 📌 Task 2: Analyze turnover rate contribution by Employment Status - Voluntarily Terminated & Terminate by Cause
While yearly turnover trend highlights long-term changes in employee attrition, analyzing monthly turnover patterns helps identify potential seasonal fluctuations within each year. This allows organization to detect recurring periods of higher turnover risk and implement targeted retention strategies in advance

### Code:
<details>
<summary><b>View SQL Code</b></summary>
  
```sql
-- Create a calendar table with month start and month end for the analysis period/ Tạo bảng calendar gồm ngày đầu & cuối tháng cho toàn bộ giai đoạn phân tích/ 
WITH calendar AS(
SELECT
  month AS month_start,
  LAST_DAY(month) AS month_end
FROM UNNEST(
GENERATE_DATE_ARRAY(DATE '2006-01-01',DATE'2018-12-31',INTERVAL 1 MONTH)) AS month),

-- Tính các chỉ số workforce theo từng tháng/ Calculate monthly workforce metrics
monthly_metric AS(
SELECT
  c.month_start,
  COUNTIF(hr.DateofHire >= c.month_start
    AND (hr.DateofTermination IS NULL OR (hr.DateofTermination >= c.month_start))) AS Beginning_headcount, -- Headcount tại đầu tháng
  COUNTIF(hr.DateofHire <= c.month_end
    AND (hr.DateofTermination IS NULL OR (hr.DateofTermination >= c.month_end))) AS End_headcount, -- Headcount tại cuối tháng
  COUNTIF(DATE_TRUNC(hr.DateofTermination,MONTH) = c.month_start) AS Terminated_count -- Số nhân viên nghỉ việc trong tháng

-- Join HR data with each month to evaluate employment status/ Join dữ liệu nhân sự với từng tháng để kiểm tra trạng thái làm việc
FROM calendar AS c
LEFT JOIN `hr-operations-analysis.hr_raw.new_employee_data` AS hr
ON TRUE
GROUP BY c.month_start, c.month_end),

-- Calculate monthly turnover rate/ Tính turnover rate theo từng tháng
turnover AS(
SELECT
  EXTRACT(YEAR FROM month_start) AS Year,
  EXTRACT(MONTH FROM month_start) AS Month,
  ROUND((Terminated_count/((Beginning_headcount + End_headcount)/2)) * 100,2) AS Turnover_rate -- Số nhân viên nghỉ việc / Headcount trung bình
FROM monthly_metric)

-- Pivot the table so each year becomes a column/ Chuyển bảng thành dạng pivot với mỗi năm là một cột
SELECT*
FROM turnover
PIVOT(MAX(Turnover_rate)
FOR YEAR IN (2006,2007,2008,2009,2010,2011,
2012,2013,2014,2015,2016,2017,2018))
```
</details>


### Result

<img width="1885" height="520" alt="image" src="https://github.com/user-attachments/assets/003ea132-b555-4cf6-91b2-7e4f158e1278" />


Turnover shows noticeable seasonal patterns with increases occurring primarily around **April – May and August – September.** These periods often align with common labor market cycles such as post bonus transitions, fiscal year planning or peak recruitment seasons when external job opportunities increase

### Recommendation
- Conduct retention reviews before high risk periods, particularly during Q2 and Q3
- Implement proactive measures such as: compensation reviews, career discussions and internal mobility opportunities. These actions can reduce the likelihood of employees leaving during peak turnover periods

### 📌 Task 3: Analyze overall turnover rate by position
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

### 📌 Task 4: Analyze overall turnover rate by tenure
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

## 🔎 Final Conclusion & Recommendations  

Based on the insights above, the following strategic actions are recommended for HR Team:  

### 1️⃣ Growth Driven Attrition, Not Structural Instability
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
