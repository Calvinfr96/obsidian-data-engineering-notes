---
company:
role:
salary_range:
status: 💰 Pipeline (Applied | Screening | Technical | System Design | HM | Offer)
applied_date: 2026-07-22
url:
---

# 🏢 [[#]] Interview Prep

## 📌 Logistics & Context
- **Recruiter Contact:** 
- **Tech Stack Found:** 
  - **Storage/Compute:** (e.g., Snowflake, BigQuery, Databricks, Redshift)
  - **Ingestion/Orchestration:** (e.g., Airflow, dbt, Flink, Kafka)
  - **Cloud Infrastructure:** (e.g., AWS, GCP, Azure)
- **Engineering Blog Notes:** *(Paste 1-2 interesting takeaways from their tech blog here)*
- **Company Context:** *(Helps answer the question 'Why XYZ Company?'*
	- **Product Facts:**
	- **Culture Facts:**

---

## 🛠️ Data Engineering Checklist
*Before this interview, I must review:*
- [ ] [[Data Modeling Cheat Sheet]] (Star vs. Snowflake, SCD Types, Fact vs. Dim)
- [ ] [[SQL Optimization Principles]] (Window functions, CTEs, indexing, partitioning)
- [ ] [[Distributed Systems Fundamentals]] (Sharding, replication, CAP theorem, Lambda vs. Kappa)
- [ ] Python streaming solutions (Generators, `yield`, memory-efficient batch processing 

---

## 🎭 Behavioral Alignment
*Select 2-3 stories from your `03_Behavioral/` folder that match this company's scale:*
- 🟢 **Scale/Throughput story:** [[STAR - Optimizing High Throughput Data Ingestion]]
- 🟡 **Conflict/Stakeholder story:** [[STAR - Disagreeing with Product on Data Schema Change]]
- 🔴 **Failure/Outage story:** [[STAR - Troubleshooting a Broken Production Pipeline]]

---

## 📓 Interview Round Logs

### 📞 Round 1: Recruiter / Screening
- **Date:** 
- **Interviewer:** 
- **Key Notes:** 
- **Questions I Asked:** 

### 💻 Round 2: Technical Screening (SQL & Python Coding)
- **Date:** 
- **Interviewer:** 
- **Coding Prompt:** 
```sql
-- Paste the exact SQL question/schema or your solved query here
SELECT 
    user_id,
    DENSE_RANK() OVER (PARTITION BY Department ORDER BY Salary DESC) as rank
FROM employees;
```
- **Python Prompt:**
```python
# Paste your Python solution or optimization problem here
def stream_large_log_file(file_path):
    with open(file_path, "r") as file:
        for line in file:
            if "ERROR" in line:
                yield line.strip()
```
- **Post-interview reflection:** *(What went well? Where did I stumble?)*

### 📐 Round 3: Data Architecture & System Design
- **Date:** 
- **Interviewer:** 
- **The Prompt:** (e.g., "Design a real-time leaderboard" or "Design a billing pipeline")
- **My Architecture Choices:** (e.g., Kafka ➔ Spark Streaming ➔ ClickHouse)
- **Trade-offs Discussed:** (Latency vs. Accuracy, Eventual vs. Strong consistency)

### 👥 Round 4: Hiring Manager / Culture Fit
- **Date:** 
- **Interviewer:** 
- **Core Focus:** (Team fit, cross-functional collaboration, long-term technical vision)
