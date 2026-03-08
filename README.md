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

### 📌 Task 3: Turnover Concentration by Employee Segment
Analyzing turnover concentration by employee segments helps identify which groups of employees contribute most to overall attrition. By examining factors such as tenure and position, organization can better understand where turnover is concentrated and design more targeted retention strategies

#### 3.1 Tenure Trend Over Time

### Code:
<details>
<summary><b>View SQL Code</b></summary>
  
```sql
-- Generate a continuous list of years for analysis/ Tạo danh sách năm liên tục cho phân tích
WITH year AS (
SELECT EXTRACT(YEAR FROM d) AS Year
FROM UNNEST(GENERATE_DATE_ARRAY(DATE '2006-01-01', DATE '2018-12-31', INTERVAL 1 YEAR)) AS d),

-- Active headcount by year and tenure group/ Headcount đang làm việc theo năm và nhóm tenure
active_headcount AS (
SELECT
  y.Year,
  COUNT(DISTINCT hr.EmployeeID) AS active_headcount,
  CASE 
    WHEN DATE_DIFF(DATE(y.Year,12,31), hr.DateofHire, YEAR) < 1 THEN '0-1 year'
    WHEN DATE_DIFF(DATE(y.Year,12,31), hr.DateofHire, YEAR) BETWEEN 1 AND 2 THEN '1-2 years'
    WHEN DATE_DIFF(DATE(y.Year,12,31), hr.DateofHire, YEAR) BETWEEN 2 AND 3 THEN '2-3 years'
    WHEN DATE_DIFF(DATE(y.Year,12,31), hr.DateofHire, YEAR) BETWEEN 3 AND 5 THEN '3-5 years'
    ELSE '5+ years' END AS Tenure
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
JOIN year AS y
ON hr.DateofHire <= DATE(y.Year,12,31)
AND (hr.DateofTermination >= DATE(y.Year,1,1) OR hr.DateofTermination IS NULL)
GROUP BY y.Year, Tenure),

-- Terminated headcount by year and tenure group/ Số nhân viên nghỉ việc theo năm và tenure
terminated_headcount AS (
SELECT
  EXTRACT(YEAR FROM DateofTermination) AS Year,
  COUNT(*) AS terminated_headcount,
  CASE
    WHEN DATE_DIFF(DateofTermination,DateofHire,YEAR) < 1 THEN '0-1 year'
    WHEN DATE_DIFF(DateofTermination,DateofHire,YEAR) BETWEEN 1 AND 2 THEN '1-2 years'
    WHEN DATE_DIFF(DateofTermination,DateofHire,YEAR) BETWEEN 2 AND 3 THEN '2-3 years'
    WHEN DATE_DIFF(DateofTermination,DateofHire,YEAR) BETWEEN 3 AND 5 THEN '3-5 years'
    ELSE '5+ years' END AS Tenure
FROM `hr-operations-analysis.hr_raw.new_employee_data`
WHERE DateofTermination IS NOT NULL
GROUP BY Year, Tenure),

-- Calculate turnover rate by year and tenure/ Tính tỷ lệ nghỉ việc theo năm và tenure
turnover AS (
SELECT
  a.Year,
  a.Tenure,
  ROUND((IFNULL(t.terminated_headcount,0) / a.active_headcount) * 100,2) AS turnover_rate -- Turnover = terminated / active headcount
FROM active_headcount a
LEFT JOIN terminated_headcount t
ON a.Year = t.Year
AND a.Tenure = t.Tenure)

-- Pivot tenure groups into columns for easier comparison/ Pivot các nhóm tenure thành cột để dễ so sánh
SELECT *
FROM turnover
PIVOT(MAX(turnover_rate) FOR Tenure IN ('0-1 year','1-2 years','2-3 years','3-5 years','5+ years'))
ORDER BY Year
```
</details>

### Result

<img width="901" height="565" alt="image" src="https://github.com/user-attachments/assets/41de9392-e73a-450b-b013-bb9448695946" />

Turnover begins to rise after the first year of employment and **remains highest among employees with 3–5 years of tenure.** This suggests that **mid career employees may face limited internal advancement opportunities,** prompting them to seek external career growth

### Recommendation
Develop targeted mid-career retention initiatives, including leadership development programs, internal promotion pathways and structured career planning discussions

#### 3.2 Turnover by Position

### Code:
<details>
<summary><b>View SQL Code</b></summary>
  
```sql
-- Generate a list of years with the start and end date of each year/ Tạo danh sách các năm kèm ngày bắt đầu & kết thúc của từng năm
WITH year AS (
SELECT
  EXTRACT(YEAR FROM y) AS Year,
  DATE_TRUNC(y, YEAR) AS Year_start,
  LAST_DAY(y, YEAR) AS Year_end
FROM UNNEST(
GENERATE_DATE_ARRAY(DATE '2006-01-01',DATE '2018-12-31',INTERVAL 1 YEAR)) AS y),

-- Combine employee information with their position title/ Kết hợp thông tin nhân viên với tên vị trí công việc
employee_position AS (
SELECT
  hr.EmployeeID,
  hr.DateofHire,
  hr.DateofTermination,
  po.Position
FROM `hr-operations-analysis.hr_raw.new_employee_data` AS hr
LEFT JOIN `hr-operations-analysis.hr_raw.Position` AS po
ON hr.PositionID = po.PositionID),

-- Calculate headcount at the start of each year/ Tính số nhân viên đang làm việc tại đầu năm (active vào ngày 1/1)
headcount_start AS (
SELECT
  y.Year,
  hr2.Position,
  COUNT(hr2.EmployeeID) AS headcount_start
FROM employee_position AS hr2
JOIN year AS y
ON hr2.DateofHire < y.Year_start
AND (hr2.DateofTermination IS NULL OR hr2.DateofTermination >= y.Year_start)
GROUP BY y.Year, hr2.Position),

-- Calculate headcount at the end of each year/ Tính số nhân viên đang làm việc tại cuối năm (active vào ngày 31/12)
headcount_end AS (
SELECT
  y.Year,
  hr2.Position,
  COUNT(hr2.EmployeeID) AS headcount_end
FROM employee_position AS hr2
JOIN year AS y
ON hr2.DateofHire <= y.year_end
AND (hr2.DateofTermination IS NULL OR hr2.DateofTermination > y.year_end)
GROUP BY y.Year, hr2.Position),

-- Count how many employees terminated in each year by position/ Đếm số nhân viên nghỉ việc trong từng năm theo từng vị trí
terminate_count AS (
SELECT
  EXTRACT(YEAR FROM DateofTermination) AS Year,
  Position,
  COUNT(*) AS total_terminated
FROM employee_position
WHERE DateofTermination IS NOT NULL
GROUP BY Year, Position),

-- Calculate turnover rate using average headcount/ Tính tỷ lệ nghỉ việc dựa trên headcount trung bình của năm
turnover AS (
SELECT
  b.Year,
  b.Position,
  ROUND(
    COALESCE(t.total_terminated,0) /
    ((COALESCE(s.headcount_start,0) + COALESCE(e.headcount_end,0)) / 2) * 100,2) AS turnover_rate 
FROM ( -- Create a base table of all Year + Position combinations / Tạo bảng nền gồm tất cả các cặp Year + Position xuất hiện trong các bảng metric
  SELECT Year, Position FROM headcount_start -- Attach headcount at the start of year/ Ghép thêm headcount đầu năm
  UNION DISTINCT
  SELECT Year, Position FROM headcount_end -- Attach headcount at the end of year/ Ghép thêm headcount cuối năm
  UNION DISTINCT
  SELECT Year, Position FROM terminate_count -- Attach number of employees who terminated in that year/ Ghép thêm số nhân viên nghỉ việc trong năm đó
) AS b
LEFT JOIN headcount_start AS s ON b.Year = s.Year AND b.Position = s.Position
LEFT JOIN headcount_end AS e ON b.Year = e.Year AND b.Position = e.Position
LEFT JOIN terminate_count AS t ON b.Year = t.Year AND b.Position = t.Position)

-- Pivot years into columns to create a year-by-year turnover table/ Chuyển các năm thành cột để tạo bảng turnover theo từng năm
SELECT *
FROM turnover
PIVOT(MAX(Turnover_rate)FOR Year IN (2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017,2018))
ORDER BY Position
```
</details>

#### *Turnover rate by all positions*

<img width="1810" height="760" alt="image" src="https://github.com/user-attachments/assets/d01d8f34-8917-4cb7-a7f9-f861a0d0f435" />

<img width="1792" height="487" alt="image" src="https://github.com/user-attachments/assets/1849cac4-4c90-48d0-9632-b2d327863acd" />

#### *Positions with Increasing Turnover Trends*

<img width="664" height="721" alt="image" src="https://github.com/user-attachments/assets/91332a2b-9557-4b2d-a4bd-9ca9d1e44da0" />


Full period position analysis shows that attrition is primarily **concentrated in operational roles rather than strategic or executive functions.** Production related positions exhibit the highest cumulative turnover, with **Production Technician II (45.61%), Production Technician I (37.96%)** and **Production Manager II (38.46%)** recording the most significant exits, supported by relatively large headcounts. In contrast, technical and IT roles such as **Software Engineer (33.33%), Data Analyst (25%) and IT Manager (25%)** remain at moderate levels, while leadership roles show minimal or no turnover. Although a few positions display 100% turnover, these are based on very small headcounts and do not represent systemic risk. he pattern suggests that attrition is more operationally driven rather than indicative of a strategic talent retention issue

### Recommendation
- Improve working conditions and shift flexibility
- Review compensation competitiveness relative to the market
- Create skill progression pathways to support career development within operational roles
#### With positions with increasing turnover trends as Production Technician
- Conduct targeted exit interviews for operational staff
- Identify root causes such as workload, management practices or career stagnation
- Introduce internal mobility or upskilling opportunities

### 📌 Task 4: Voluntary vs Involuntary Turnover
Examining turnover by Employment Status (Voluntarily Terminated vs. Terminated for Cause) helps clarify the underlying drivers of attrition. While the overall turnover rate shows whether attrition is increasing or decreasing, it does not explain why employees leave. Breaking turnover down by status distinguishes between employee driven exits and company initiated terminations, providing clearer insight into whether attrition reflects retention challenges or internal management decisions

### Code:
<details>
<summary><b>View SQL Code</b></summary>
  
```sql
WITH avg_headcount AS(
  SELECT
  b.Year,
  (b.beginning_headcount + e.end_headcount)/2 AS avg_headcount
FROM `hr-operations-analysis.hr_raw.beginning_headcount` AS b
LEFT JOIN `hr-operations-analysis.hr_raw.end_headcount` AS e
ON b.Year = e.Year),

terminated_by_status AS(
SELECT
  EXTRACT (YEAR FROM DateofTermination) AS Year,
  COUNT(EmployeeID) AS Total_terminated,
  EmploymentStatus
FROM `hr-operations-analysis.hr_raw.new_employee_data`
WHERE EmploymentStatus IN ('Voluntarily Terminated','Terminated for Cause')
GROUP BY Year, EmploymentStatus)

SELECT
  t.Year,
  t.EmploymentStatus AS Employment_Status,
  t.Total_terminated,
  ROUND(SAFE_DIVIDE(t.total_terminated,a.avg_headcount) * 100,2) AS Turnover_by_status
FROM terminated_by_status AS t
LEFT JOIN avg_headcount AS a
ON t.Year = a.Year
ORDER BY t.Year, Employment_Status;
```
</details>

### Result

<img width="853" height="601" alt="image" src="https://github.com/user-attachments/assets/4a34e821-64d4-4bd7-87ee-e61b7656ec91" />

Tenure level analysis shows that **attrition is most concentrated among mid tenure employees.** **The 3–5 years group accounts for 49.04% of total terminations,** representing nearly half of all exits, followed by the 1–2 years group (31.73%). In contrast, early stage employees (0–1 year) contribute only 3.85%, while long-tenured employees (5+ years) represent 15.38% of exits. This pattern suggests that **turnover is not primarily driven by onboarding or early hiring issues** but rather by mid career dynamics. The high proportion of exits within the 3–5 year band may indicate potential career progression bottlenecks, limited advancement opportunities or compensation related factors affecting employees at a critical growth stage

### Recommendation
- Strengthen mid career retention programs targeting employees in the 2–5 year tenure range where the majority of exits occur
- Introduce clearer career progression pathways and promotion criteria to reduce potential advancement bottlenecks for mid tenure employees
- Expand professional development and upskilling opportunities to support employees at a critical career growth stage
- Review compensation competitiveness for mid tenure roles, ensuring pay progression aligns with market benchmarks and internal equity
- Conduct targeted stay interviews with employees in the 2–5 year tenure band to identify early signals of disengagement and address potential retention risks

### Task 5: Reasons for Employee Exit
Analyzing the reasons for employee exit helps provide deeper insight into the underlying drivers of turnover. While previous analyses identify when and where attrition occurs, examining exit reasons reveals why employees leave organization. Understanding these drivers allows business to address root causes more effectively and design targeted retention strategies to reduce future turnover

### Code:
<details>
<summary><b>View SQL Code</b></summary>

```sql
WITH terminated_headcount AS (
SELECT
  EXTRACT(YEAR FROM DateofTermination) AS Year,
  COUNT(DISTINCT EmployeeID) AS Terminated_headcount
FROM `hr-operations-analysis.hr_raw.new_employee_data`
WHERE DateofTermination IS NOT NULL
GROUP BY Year),

terminated_reason_count AS (
SELECT
  EXTRACT(YEAR FROM DateofTermination) AS Year,
	COUNT(*) AS Reason_count,
	CASE
    WHEN TerminationReason IN ('Another position','career change','more money') THEN 'Career opportunity'
    WHEN TerminationReason IN ('unhappy','hours') THEN 'Job dissatisfaction'
    WHEN TerminationReason IN ('attendance','no-call, no-show','gross misconduct','performance') THEN 'Performance / conduct'
    WHEN TerminationReason IN ('maternity leave - did not return','medical issues','military','return to school','relocation out of area') THEN 'Personal reasons'
    WHEN TerminationReason = 'retiring' THEN 'Retirement'
    ELSE 'Other' END AS reason_group
FROM `hr-operations-analysis.hr_raw.new_employee_data`
WHERE DateofTermination IS NOT NULL
GROUP BY Year, reason_group),

reason_distribution AS (
SELECT
  t1.Year,
  t2.reason_group,
  ROUND((Reason_count/Terminated_headcount) * 100,2) AS Terminate_reason_distribution
FROM terminated_headcount AS t1
LEFT JOIN terminated_reason_count AS t2
ON t1.Year = t2.Year)

SELECT *
FROM reason_distribution
PIVOT(
  MAX(Terminate_reason_distribution)
  FOR Year IN (2006,2007,2008,2009,2010,2011,2012,2013,2014,2015,2016,2017,2018))
ORDER BY reason_group
```
</details>

### Result

<img width="1782" height="282" alt="image" src="https://github.com/user-attachments/assets/b683bd8b-022f-459a-a3af-270ba5113170" />

**Career opportunity** consistently represents the largest share of exits, **accounting for 66.67% in 2011,** then gradually declining but still remaining high at **38.46% in 2018.** Meanwhile, **job dissatisfaction peaked at 46.15% in 2013 before steadily decreasing to 7.69% by 2018,** suggesting some improvement in employee experience over time. In contrast, **performance or conduct related** exits show a gradual increase in later years, **rising from 7.69% in 2013 to around 23–25% between 2016 and 2018.** Overall, these patterns indicate that turnover is primarily driven by career advancement opportunities and job satisfaction factors rather than personal circumstances

### Recommendation
- Strengthen internal career mobility by providing clearer promotion pathways and internal job opportunities
- Expand professional development programs to support employee growth and reduce the need to seek opportunities outside the organization
- Improve employee engagement and job satisfaction through regular feedback, engagement surveys  and manager training
- Review compensation competitiveness, particularly for roles where career opportunity appears to drive turnover

## 🔎 Final Conclusion and Recommendation

The turnover analysis reveals that employee attrition is influenced by several structural patterns within the organization. Overall turnover increased during periods of workforce expansion, suggesting that rapid organizational growth may create retention challenges. Seasonality analysis indicates that turnover tends to rise during certain months of the year, implying potential external labor market effects

**Employee segmentation** further shows that attrition is highly concentrated among mid tenure employees, particularly those **with 3–5 years of tenure** who account for nearly half of all exits. In addition, turnover is primarily observed in **operational roles, especially production related positions.** Finally, exit reason analysis indicates that **career opportunities and job dissatisfaction are the main drivers of turnover,** while performance-related terminations represent a smaller share. Together, these findings suggest that most turnover is **voluntary and opportunity driven** rather than caused by onboarding issues or unavoidable personal factors

To address the primary drivers of turnover, the organization should focus on strengthening employee retention strategies, particularly for mid-tenure employees and operational roles:
- Prioritize retention initiatives for employees with 2–5 years of tenure, where attrition risk is highest. Clear promotion pathways, internal mobility programs, and leadership development opportunities can help retain employees during this critical career stage.
- Improve career growth visibility within the organization by providing structured career progression frameworks and skill development opportunities that allow employees to advance without leaving the company
- Enhance engagement and job satisfaction, particularly in operational roles through regular employee feedback, improved job design and stronger managerial support
- Monitor turnover seasonality and workforce expansion periods to proactively implement retention actions before high risk periods
