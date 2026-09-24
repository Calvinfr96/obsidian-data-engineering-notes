---
tags: [interview-prep, audit]
created: 2026-09-24
---

# Interview Preparation Audit

Your preparation has strong breadth and meaningful practice history. The highest-value next step is to demonstrate reliable execution and concise explanations using your existing materials and projects. The evidence does not support starting the curriculum again.

Action plan: [[Six Week Interview Study Plan]]. Budget: **30 hours per week for six weeks**, assuming no fixed interview deadline.

## Scope and interpretation

This audit inventories the 85 Markdown notes present before this report, the relevant local learning/project repositories, and accessible Google Drive learning, coaching, tracker, and project-story materials. It also uses your Notion Job Search Workspace for targeting. The vault contains approximately 334,000 whitespace-delimited words, including code and metadata; this measures volume, not learning or proficiency. Imported notes and their source repositories overlap and are not independent accomplishments.

The review combined corpus inventory and topic scans with detailed inspection of relevant exercises, project code, and mock feedback. It is not an exhaustive line-by-line correctness review of every file. Books were inspected through available text, contents, and selected sections; course videos and interview recordings were not watched. Project observations below come from local code, not a fresh deployment or execution. Older story copies and checkboxes are not proof of current performance. No study files or project code were repaired as part of this audit.

Relevant local sources include `dea-general-learning`, `dea-structured-query-language-learning`, 12 guided-project directories, six end-to-end project directories, and the Calendly, Wistia, and CRM pipeline repositories under `/Users/Apple/dea-workspace`. The 17 AWS question files excluded from the earlier migration remain source-only; this audit does not import them into the vault.

Your target is junior/entry data engineering and adjacent ETL, analytics, data-platform, and software/data hybrid roles. Your Amazon Ads experience supplies professional engineering examples; your DE projects are independent portfolio work. You confirmed implementing Kinesis streaming and not implementing LLM/RAG. Preserve that distinction in every project explanation.

## Comparison with the interview topics

The five original topics—SQL, Python, behavioral, data modeling, and system design—are a useful starting point. For preparation, make pipeline reliability/data quality explicit, and reserve a small amount for the employer's tools and business metrics. Spark, Airflow, cloud services, testing, and warehouse concepts fit inside those categories rather than becoming separate large study tracks.

| Topic | Material already covered | Weakness or uncertainty | Priority |
|---|---|---|---|
| SQL | 16 dedicated notes; joins, aggregation, windows, dates, subqueries, worked interview exercises, optimization notes; positive technical-mock feedback | Current timed accuracy is unmeasured; verify tie handling, nulls, grain, date boundaries, fan-out, and query-plan reasoning | High: retain regular unseen timed work |
| Python | Seven notes covering fundamentals, data structures, algorithms, Pandas, and DE utilities; project transformation tests exist | Some saved solutions contain defects; add evidence for parsing, validation, generators, exception handling, and tested transformations under time limits | High: executable correctness before more algorithms |
| Data modeling | Four dedicated notes plus warehouse content; seven worked business cases, facts/dimensions, normalization, SCD concepts | Business metric definitions, precise grain, cardinality, history, and queries that prove the model are uneven | High: model and query the same dataset |
| Pipelines, ETL, quality | Airflow, dbt, Kafka, AWS, Spark/Delta labs and multiple portfolio pipelines; retries/checkpoints/CDC discussed | Reproducible replay, late/out-of-order handling, historical output retention, backfills, and quality checks need stronger demonstrated evidence | High: focused project experiments |
| Behavioral and project explanation | Ten story notes, checklist, retrospective, project narratives, coaching feedback | Recent mock identifies clarity, impact, and composure; inconsistent metrics and overly detailed explanations weaken otherwise relevant stories | High: recorded practice and factual reconciliation |
| Data system design | Twelve folder notes, worked architectures, distributed systems, concurrency, warehouses, Spark, plus external playbooks | Breadth exceeds current need; practice concise requirements → model → pipeline → failure handling explanations | Maintain, not expand |
| Role-specific tools and business context | Snowflake, Databricks, AWS, Airflow, dbt, Streamlit; significant project exposure | Tools must be matched to actual postings; metrics and business outcomes need clearer definitions | Small, targeted allocation |

Folder counts understate cross-topic coverage: the system-design folder contains substantial Spark and warehousing material. They should not be interpreted as time spent or interview frequency.

## What the mock evidence actually says

- The [August 15 technical mock](https://docs.google.com/document/d/1HgLouRiNJ16anXI-MfTV6c7Jh9Hc0oRmUDVK4r3WCZw/edit) rated technical knowledge, communication, and problem-solving 5/5 and recommended applying. It highlighted Airflow, Spark execution concepts, data-quality dimensions, and a tighter introduction. Your current notes address these subjects; retest them instead of assuming they remain absent.
- The [September 11 behavioral mock](https://docs.google.com/document/d/1heZoY9OqRi0U0RkSKmIzebX0WJPSh0rX5UY/edit) rated relevance, specificity, and problem-solving 5/5, collaboration 4/5, and clarity, results/impact, and composure 3/5. This is the strongest recent reason to increase spoken practice.
- The [February 10 modeling mock](https://docs.google.com/document/d/1znrJd2Fs_X0nexFqg5NCId0Oe3Xkx4HukJWfCwbfURQ/edit) emphasized asking clarifying questions and defining metrics precisely. It is an older signal to validate, not a current failing grade.
- The [January 27 mock](https://docs.google.com/document/d/10uvvBthWmWVZtB7I9xmvgh3obGqo1Hce/edit) emphasized explaining the approach, listening, and clean code. The [academy tracker](https://docs.google.com/spreadsheets/d/1PpGXwqU37C_z-0Q7r7ZoO2XM0siDOiFt/edit) records substantial completed curriculum and mocks. Completion marks do not measure current unaided accuracy.

## Specific issues to address first

### 1. Verify solutions before rehearsing them

In [[Python Algorithms Practice]], “Total Transactions Per User” initializes an empty `user_totals` dictionary and then iterates over that dictionary instead of the input transactions. It returns an empty result; its membership check also uses the wrong collection. Add a tiny input with repeated users and an expected output before treating it as a reference solution.

In [[SQL Algorithms Practice]], a tie-breaking example shows `ORDER BY order_date DESC order_id DESC`, missing a comma. “Latest record” exercises also need an explicit deterministic tie-break rule. [[Sample SQL Question]] describes the current month but uses a fixed July 2026 range. Parameterize the period when practicing and test month boundaries. These findings justify checking worked answers, not rejecting the whole collection.

### 2. Make project reliability demonstrable

The local Calendly `project_resources/02_silver_cleaning.py` sorts by ingestion time and uses `dropDuplicates`, then performs an unconditional matched update. Treat this as a candidate out-of-order/replay failure to reproduce: explicitly select the winning source version and prevent stale updates from replacing newer state. Spark's [dropDuplicates reference](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.dropDuplicates.html) documents duplicate removal, not a latest-version selection contract.

In `03_gold_analysis.py`, spend is selected for yesterday while meeting counts use event start dates; cost per booking falls back to spend when the booking count is zero. Define whether attribution is by booking creation date or meeting date, and use an explicit zero-denominator policy. The full-table overwrite with a one-day spend input also warrants a multi-day history-retention test. These are static observations, not claims about deployed behavior.

Tests already exist: Calendly includes Lambda/S3 and malformed-input tests; Wistia includes transformation, deduplication, and nested-action tests. Extend these with replay, out-of-order updates, empty input, rejected records, and reconciliation. Do not describe testing as absent.

### 3. Turn Airflow coverage into a small runnable demonstration

The guided healthcare DAG exists, but its local snapshot uses `days_ago` without an import, contains a bucket placeholder, and sets zero retries. This is an unfinished execution example, not a lack of orchestration knowledge. Use the existing project's compatible Airflow version and a small fixture dataset to demonstrate import, dependencies, retry, and date-specific reruns. Avoid making a framework upgrade or cloud deployment a prerequisite.

Airflow's [official best practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html) support repeatable task outputs, partition-specific reads/writes, and avoiding wall-clock-dependent critical computations. Apply those principles to one bounded example.

### 4. Validate modeling through actual queries

[[Data Modeling Practice Questions]] contains real worked cases, not just prompts. However, the Yelp example gives `dim_country` a country-code key while including city/state/postal attributes, which needs a consistent grain and key. The Instagram example associates a session with individual post/search/chat identifiers; clarify how multiple actions per session are represented. Rework two cases and prove them with queries, rather than adding ten more diagrams.

### 5. Reconcile stories and practice delivery

[[Project Leadership]] reports a 12 ms p99 reduction in its summary and roughly 5 ms in its result section. Verify the actual measurement and context; do not choose the more impressive number. If it cannot be verified, use a truthful qualitative result. [[Meeting Tight Deadline]] emphasizes a cache race-condition investigation; ensure the answer explicitly explains the deadline and prioritization when responding to a deadline prompt.

The [Calendly story](https://docs.google.com/document/d/1EoeCffgl7CWFd0D4jf488xIrFEq5mNNN/edit) mentions Streamlit reading checkpoint locations; reconcile that with actual table paths. Checkpoints track processing state and should not be described as the analytics dataset. Its `availableNow` execution should be explained accurately. The [Wistia story](https://docs.google.com/document/d/1SqS4wJFMD3lImtrFw4Sysd7rZI-VZYXu/edit) describes a scheduled batch variant; locate the separate Kinesis implementation you confirmed and clearly identify the two versions. Missing Kinesis in one snapshot does not mean you did not implement it.

## Overrepresented material and what to reduce

- **Broad AWS and distributed-systems reading.** [[AWS Services Review]] alone is about 42,000 words; [[Data Processing Systems (Spark)]] is about 23,000 and [[Distributed Systems Fundamentals]] about 13,000. Keep targeted review of services you used and concepts a posting requires. Pause new service surveys and advanced architecture collections.
- **Repeated references and worked solutions.** SQL/Python notes, original repositories, and course materials overlap. Select one primary reference per topic and spend saved time solving without it.
- **Detailed story scripts versus delivery practice.** You already have relevant examples. Rehearse short answers and follow-ups rather than writing more long narratives.
- **Advanced algorithms and unrelated AI material.** Keep common dictionary/set, sorting, traversal, and complexity skills. Defer deep competitive-programming work and GenAI/RAG reading unless a specific interview requires them. RAG is not implemented portfolio experience.
- **Backend story-defense topics as a general curriculum.** Redis, cache races, canaries, and concurrency in [[Topics to Review]] matter for your Amazon stories. They should not displace modeling, ETL reliability, and timed SQL.

“Overrepresented” means volume relative to your current preparation priorities. It does not mean these subjects are unimportant or that the volume reflects time you actually spent.

## Recommended allocation

These are personalized study allocations, not measured percentages of employer interview questions.

| Topic | Earlier baseline | Your next six weeks | Hours/week |
|---|---:|---:|---:|
| SQL | 30% | 25% | 7.5 |
| Python | 20% | 20% | 6 |
| Data modeling | 15% | 15% | 4.5 |
| Pipelines/ETL/data quality | 15% | 15% | 4.5 |
| Behavioral/project explanation | 10% | 15% | 4.5 |
| Data system design | 5% | 5% | 1.5 |
| Role tools/business context | 5% | 5% | 1.5 |
| **Total** | **100%** | **100%** | **30** |

The five-point shift from SQL to behavioral reflects strong prior SQL/technical feedback and the more recent delivery scores. SQL remains the largest allocation. Rebalance after two weeks using timed results. Continue applications while preparing; the six-week plan is not a prerequisite for applying.

## Source register

The existing [[Data Engineering Study Guide]] links the technical-note corpus. Behavioral notes, [[Topics to Review]], and the dashboard were also included. Supporting cloud sources include:

- [Notion Job Search Workspace](https://app.notion.com/p/3c2b22cf93008060b455f065e132caf7): targeting and preferences.
- [Behavioral summary](https://docs.google.com/document/d/1SyArzAEbj-Y08NF6VXRYQ1DAbX6kdiqFk8XWFfWOrK4/edit) and [modeling question framework](https://docs.google.com/document/d/1BsRqH0GOSZLB96oO3n7ag1uw05QQYbzX/edit).
- [CRM project story](https://docs.google.com/document/d/1E-97VbofU6-wd9S4sLOOyz-HSx2oKBMB/edit), alongside the Calendly/Wistia stories above.
- [AWS overview](https://docs.google.com/document/d/1GdtqiJvdyDUwtCDSUUsqdPxAs-WJH8P0ZHuklxVa9Iw/edit), [Spark optimization](https://docs.google.com/document/d/1Ng5xJuqn9r5CDbncB_d6696zhvm8ZcnJcRjKsfZvJoE/edit), and [Snowflake overview](https://docs.google.com/document/d/1swvNIqTzhePiySpQFYK3_AJQzQ1NmF0acfoMg427jcE/edit).
- [DE interview ebook](https://drive.google.com/file/d/1uV-7spRTp_xkPyKU642Ya8BnVjshiPuA/view), [DE System Design Playbook](https://drive.google.com/file/d/1yz7rvCq266a2_KxKjDWjAc5j5ZBTpAEc/view), [System Design Principles](https://drive.google.com/file/d/1XOn1kBRlnqEU1Ap45bfmkFC1349AutCd/view), and [Understanding ETL](https://drive.google.com/file/d/1UKbf0XH32P55Pb6iEYy_9y2SCmiWKLxS/view): accessible text/contents inspected, not every page audited.
- [Big Book of GenAI](https://drive.google.com/file/d/1XoFMUfBeTxjbjcKLJ6a_Iq6NjR9SGgJb/view) and [Big Book of Data Science](https://drive.google.com/file/d/1wrKk0zyaPJC_tMt6vTZXMfb9HCdW8Xua/view): inventory/context, lower priority for this cycle.

## Vault inventory at review

The following index records the notes present before these two reports were added. Navigation notes and templates are context, not additional practice evidence.

- [[00_daily_logs/Daily Review Template]]
- [[01_job_hunting/Interview and Job Search Advice]]
- [[02_technical_prep/AWS Interview Prep]]
- [[02_technical_prep/AWS Services Cheat Sheet]]
- [[02_technical_prep/AWS Services Review]]
- [[02_technical_prep/AWS Setup Walkthroughs]]
- [[02_technical_prep/Apache Airflow]]
- [[02_technical_prep/Apache Kafka]]
- [[02_technical_prep/DBT Review]]
- [[02_technical_prep/Data Engineering Foundations]]
- [[02_technical_prep/Databricks Review]]
- [[02_technical_prep/Git and GitHub]]
- [[02_technical_prep/Snowflake Review]]
- [[02_technical_prep/Spark Optimization Review]]
- [[02_technical_prep/Streamlit Example Project]]
- [[02_technical_prep/Streamlit Notes]]
- [[02_technical_prep/data_modeling/Data Modeling Cheat Sheet]]
- [[02_technical_prep/data_modeling/Data Modeling Fundamentals]]
- [[02_technical_prep/data_modeling/Data Modeling Interview Guidance]]
- [[02_technical_prep/data_modeling/Data Modeling Practice Questions]]
- [[02_technical_prep/python/Pandas DataFrame Fundamentals]]
- [[02_technical_prep/python/Pandas Interview Practice]]
- [[02_technical_prep/python/Pandas Practice Exercises]]
- [[02_technical_prep/python/Python Algorithms Practice]]
- [[02_technical_prep/python/Python Data Engineering Toolkit]]
- [[02_technical_prep/python/Python Fundamentals Practice]]
- [[02_technical_prep/python/Python Fundamentals]]
- [[02_technical_prep/snowflake/Snowflake Mini Project 1]]
- [[02_technical_prep/snowflake/Snowflake Mini Project 2]]
- [[02_technical_prep/snowflake/Snowflake Mini Project 3]]
- [[02_technical_prep/snowflake/Snowflake Mini Project 4]]
- [[02_technical_prep/snowflake/Snowflake Mini Project 5]]
- [[02_technical_prep/snowflake/Snowflake Worked Examples]]
- [[02_technical_prep/spark_databricks/Delta Lake Labs]]
- [[02_technical_prep/spark_databricks/Lakeflow Initial Setup]]
- [[02_technical_prep/spark_databricks/Lakeflow Pipeline Labs]]
- [[02_technical_prep/spark_databricks/Lakeflow Sales Data]]
- [[02_technical_prep/spark_databricks/PySpark and Databricks Labs]]
- [[02_technical_prep/sql/SQL Advanced Lesson Exercises]]
- [[02_technical_prep/sql/SQL Advanced Lessons]]
- [[02_technical_prep/sql/SQL Advanced Practice Problems]]
- [[02_technical_prep/sql/SQL Algorithms Practice]]
- [[02_technical_prep/sql/SQL Beginner Lesson Exercises]]
- [[02_technical_prep/sql/SQL Beginner Lessons]]
- [[02_technical_prep/sql/SQL Beginner Practice Problems]]
- [[02_technical_prep/sql/SQL Database Essentials]]
- [[02_technical_prep/sql/SQL Intermediate Lesson Exercises]]
- [[02_technical_prep/sql/SQL Intermediate Lessons]]
- [[02_technical_prep/sql/SQL Intermediate Practice Problems]]
- [[02_technical_prep/sql/SQL Interview Practice Problems]]
- [[02_technical_prep/sql/SQL Optimization Principles]]
- [[02_technical_prep/sql/SQL Problem-Solving Walkthroughs]]
- [[02_technical_prep/sql/SQL Study Workflow]]
- [[02_technical_prep/sql/Sample SQL Question]]
- [[02_technical_prep/system_design/Concurrency and Performance]]
- [[02_technical_prep/system_design/Data Architecture Worked Examples]]
- [[02_technical_prep/system_design/Data Processing Systems (Spark)]]
- [[02_technical_prep/system_design/Data Warehousing]]
- [[02_technical_prep/system_design/Distributed Systems Fundamentals]]
- [[02_technical_prep/system_design/Problem 1 - Data Lake Implementation for a Retail Chain]]
- [[02_technical_prep/system_design/Problem 2 - Real Time Fraud Detection Pipeline for a Fintech Company]]
- [[02_technical_prep/system_design/Problem 3 - Data Warehouse Modernization for a Healthcare Provider]]
- [[02_technical_prep/system_design/Sample System Design Question]]
- [[02_technical_prep/system_design/Solving System Design Interview Questions]]
- [[02_technical_prep/system_design/System Design Case Studies]]
- [[02_technical_prep/system_design/System Design Interview Prep]]
- [[03_behavioral/Behavioral Question Checklist]]
- [[03_behavioral/Bias For Action]]
- [[03_behavioral/Challenging Project]]
- [[03_behavioral/Choosing The Best Solution]]
- [[03_behavioral/Dive Deep]]
- [[03_behavioral/Influencing Without Authority]]
- [[03_behavioral/Learning From Mistakes]]
- [[03_behavioral/Meeting Tight Deadline]]
- [[03_behavioral/Ownership]]
- [[03_behavioral/Prioritization]]
- [[03_behavioral/Project Leadership]]
- [[03_behavioral/Resume Retrospective]]
- [[Data Engineering Study Guide]]
- [[Job Search Dashboard]]
- [[Obsidian Markdown Quick Reference]]
- [[Topics to Review]]
- [[Welcome]]
- [[_templates/Company Template]]
- [[_templates/STAR Behavioral Template]]
