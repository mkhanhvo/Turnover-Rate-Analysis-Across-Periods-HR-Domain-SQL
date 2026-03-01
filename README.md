
👉🏻Change Icon emoji 🔥🔍📘🚩 to your likings by clicking "Start" + "."

# 📊 Turnover Rate Across Periods Analysis - mid sized technology enabled organization | SQL
  
Author: Vo Tran Mai Khanh
Date: 02/2026
Tools Used: SQL
---
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

 **_If your result is a very long table with many records, only show top 5/10 and bottom 5/10 rows, or records that relevant to the insights/ observation below_**

*_Example_*

### Project Results:

| Period   | Name                | Count Items | Count Orders | Sales        |
|:---------|:--------------------|------------:|-------------:|-------------:|
| Apr 2014 | Bib-Shorts          |           4 |            1 |       233.97 |
| Feb 2014 | Bib-Shorts          |           4 |            2 |       233.97 |
| Jul 2013 | Bib-Shorts          |           2 |            1 |       116.99 |
| Jun 2013 | Bib-Shorts          |           2 |            1 |       116.99 |
| Apr 2014 | Bike Racks          |          45 |           45 |     5,400.00 |
| Aug 2013 | Bike Racks          |         222 |           63 |    17,387.18 |
| Dec 2013 | Bike Racks          |         162 |           48 |    12,582.29 |
| Feb 2014 | Bike Racks          |          27 |           27 |     3,240.00 |
| Jan 2014 | Bike Racks          |         161 |           53 |    12,840.00 |
| Jul 2013 | Bike Racks          |         422 |           75 |    29,802.30 |
| ...      | ...                 |         ... |          ... |          ... |
| May 2014 | Vests               |         610 |          103 |    23,640.71 |
| Nov 2013 | Vests               |         315 |           75 |    12,937.24 |
| Oct 2013 | Vests               |         611 |           93 |    23,255.74 |
| Sep 2013 | Vests               |         623 |          102 |    24,100.47 |
| Jul 2013 | Wheels              |           4 |            1 |       698.63 |
| Jun 2013 | Wheels              |           3 |            1 |       450.91 |
| Sep 2013 | Wheels              |           1 |            1 |        83.30 |

*A summary of the full results. The complete dataset is available in the repository.*

👉🏻 Finally, explain your observations/ findings from the results 
  
 _Describe trends, key metrics, and patterns._  

---

## 🔎 Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we would recommend the [stakeholder team] to consider the following:  

📍 Key Takeaways:  
✔️ Recommendation 1  
✔️ Recommendation 2  
✔️ Recommendation 3

**_📌Remember to summarize the most core insights/ observations you extract from the entire projects. 
 Recap ONLY key actions/ recommendations. DO NOT copy paste everything above_**
