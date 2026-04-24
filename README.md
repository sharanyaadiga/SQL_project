# 📊 SQL Job Market Analysis

A deep-dive SQL analysis of the data job market, exploring top-paying roles, in-demand skills, and where high salary meets high demand — helping data professionals make smarter career decisions.

> 🎓 **Project inspired by:** [Luke Barousse's SQL for Data Analytics Course](https://www.youtube.com/watch?v=7mz73uXD9DA)

---

## 📁 Table of Contents

- [Introduction](#introduction)
- [Background](#background)
- [Tools Used](#tools-used)
- [The Analysis](#the-analysis)
  - [1. Top Paying Jobs](#1-top-paying-data-analyst-jobs)
  - [2. Skills for Top Paying Jobs](#2-skills-required-for-top-paying-jobs)
  - [3. Most In-Demand Skills](#3-most-in-demand-skills)
  - [4. Skills Based on Salary](#4-top-skills-based-on-salary)
  - [5. Most Optimal Skills to Learn](#5-most-optimal-skills-to-learn)
- [What I Learned](#what-i-learned)
- [Conclusions](#conclusions)

---

## Introduction

This project analyzes the **data job market** — specifically data analyst roles — by querying a real-world job postings dataset using SQL. The goal is to answer key questions like:

- What are the highest-paying data jobs?
- What skills are required for those roles?
- What skills are most in demand overall?
- Which skills offer the best return on investment?

All queries were written in **PostgreSQL** and results were visualized to surface actionable career insights.

---

## Background

The data job market is evolving fast. With so many tools, certifications, and roles available, it can be hard to know **what to focus on**. This project was born from a desire to answer that question with data — not guesswork.

The dataset used in this project contains thousands of real job postings with details on:
- Job titles and locations
- Salary ranges (annual average)
- Required skills per posting
- Company names and posting dates

By analyzing this data with SQL, I aimed to find patterns that help aspiring and existing data analysts make informed decisions about their career path.

> 📺 **Tutorial Reference:** This project follows the structure taught in [Luke Barousse's SQL Course on YouTube](https://www.youtube.com/watch?v=7mz73uXD9DA). Huge thanks to Luke for making this content freely available!

---

## Tools Used

| Tool | Purpose |
|------|---------|
| **SQL** | Core language for querying and analyzing the dataset |
| **PostgreSQL** | Database management system used to run all queries |
| **Visual Studio Code** | Code editor with SQL extensions for writing and running queries |
| **Git & GitHub** | Version control and project hosting |

---

## The Analysis

Each query in this project tackles a specific analytical question about the job market. Here's a breakdown of each one:

---

### 1. Top Paying Data Analyst Jobs

**Question:** What are the highest-paying data analyst roles available?

**Goal:** Identify the top 10 highest-paying data analyst jobs, filtering for postings that include salary data.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name AS company_name
FROM
    job_postings_fact
LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst'
    AND job_location = 'Anywhere'
    AND salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

**Key Insight:** Top-paying remote data analyst roles range from **$184,000 to $650,000**, showing a huge ceiling for the role depending on company and specialization.

![Top Paying Jobs](assets/1_top_paying_jobs.png)
*Bar chart of the top 10 highest-paying data analyst roles (remote)*

---

### 2. Skills Required for Top Paying Jobs

**Question:** What skills are required for the top-paying data analyst jobs?

**Goal:** Join the job postings with skills data to identify what top-paying employers are looking for.

```sql
WITH top_paying_jobs AS (
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        name AS company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim ON job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst'
        AND job_location = 'Anywhere'
        AND salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC;
```

**Key Insight:** **SQL**, **Python**, and **Tableau** are the most frequently required skills among the top 10 highest-paying roles.

---

### 3. Most In-Demand Skills

**Question:** What skills are most frequently requested in data analyst job postings?

**Goal:** Find the top 5 most demanded skills across all data analyst postings.

```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) AS demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND job_work_from_home = TRUE
ORDER BY
    demand_count DESC
LIMIT 5;
```

**Results:**

| Skill | Demand Count |
|-------|-------------|
| SQL | 7,291 |
| Excel | 4,611 |
| Python | 4,330 |
| Tableau | 3,745 |
| Power BI | 2,609 |

**Key Insight:** SQL dominates demand — it's the foundational skill for any data analyst.

![Most In-Demand Skills](assets/3_in_demand_skills.png)
*Top 5 most in-demand skills for remote data analyst roles*

---

### 4. Top Skills Based on Salary

**Question:** What is the average salary associated with each skill?

**Goal:** Find which skills correlate with higher salaries — revealing what to learn for maximum pay.

```sql
SELECT
    skills,
    ROUND(AVG(salary_year_avg), 0) AS avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
    AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    avg_salary DESC
LIMIT 25;
```

**Key Insight:** Niche and big data skills like **PySpark**, **Bitbucket**, **Couchbase**, and **Watson** command salaries above **$160,000**, while common tools like Excel average around $85,000.

![Top Skills by Salary](assets/4_top_paying_skills.png)
*Top 10 skills ranked by average salary for remote data analyst roles*

---

### 5. Most Optimal Skills to Learn

**Question:** Which skills are both high in demand AND high in salary — making them the smartest skills to learn?

**Goal:** Identify optimal skills by combining demand count and average salary.

```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_dim.skill_id
),
average_salary AS (
    SELECT
        skills_job_dim.skill_id,
        ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_job_dim.skill_id
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    demand_count,
    avg_salary
FROM
    skills_demand
INNER JOIN average_salary ON skills_demand.skill_id = average_salary.skill_id
WHERE
    demand_count > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25;
```

**Key Insight:** **Go**, **Confluence**, and **Hadoop** offer high average salaries with solid demand. **Python**, **Tableau**, and **SQL** remain the optimal foundational trio — high demand, competitive salaries.

![Optimal Skills](assets/5_optimal_skills.png)
*Bubble chart of skills plotted by demand (x-axis) vs average salary (y-axis)*

---

## What I Learned

Through this project I significantly leveled up my SQL skills:

- **🔗 Complex Joins** — Combining multiple tables (job postings, skills, companies) to build comprehensive views of the data.
- **🧮 Aggregations & GROUP BY** — Using `COUNT()`, `AVG()`, and `ROUND()` to summarize data at scale.
- **📦 CTEs (Common Table Expressions)** — Writing clean, readable multi-step queries using `WITH` clauses instead of nested subqueries.
- **🔍 Filtering with WHERE** — Narrowing results by job type, remote status, and salary availability.
- **📊 Turning Data into Insights** — Not just writing queries that work, but framing them around real business questions.
- **💡 Analytical Thinking** — Learning to connect query results to actionable career advice.

---

## Conclusions

### Key Findings

1. **Top-paying data analyst jobs** can reach up to **$650,000/year** for remote roles — salary potential is much higher than many expect.
2. **SQL is non-negotiable** — It appears in both the highest-paying AND most in-demand skill lists.
3. **Python and Tableau** round out the essential toolkit for well-paid analysts.
4. **Niche/cloud skills** like PySpark, Databricks, and GCP command premium salaries but appear in fewer postings.
5. **The optimal learning path** combines foundational skills (SQL, Python, Excel) with one visualization tool (Tableau or Power BI) and exposure to cloud platforms.

### Closing Thoughts

This project reinforced that data-driven decision-making applies just as much to career planning as it does to business strategy. For anyone entering or growing in the data field, focusing on **SQL + Python + a visualization tool** offers the best combination of job availability and earning potential.

---

## 📂 Project Structure

```
sql-job-analysis/
│
├── queries/
│   ├── 1_top_paying_jobs.sql
│   ├── 2_top_paying_job_skills.sql
│   ├── 3_in_demand_skills.sql
│   ├── 4_top_paying_skills.sql
│   └── 5_optimal_skills.sql
│
├── assets/
│   ├── 1_top_paying_jobs.png
│   ├── 3_in_demand_skills.png
│   ├── 4_top_paying_skills.png
│   └── 5_optimal_skills.png
│
├── README.md
└── .gitignore
```

---

## 🔗 Resources

- 📺 [Luke Barousse - SQL for Data Analytics (YouTube)](https://www.youtube.com/watch?v=7mz73uXD9DA)
- 🐘 [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- 💻 [Visual Studio Code](https://code.visualstudio.com/)
- 🐙 [GitHub](https://github.com/)

---

*Feel free to fork this project, explore the queries, or reach out if you have suggestions!*
