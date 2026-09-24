---
tags: [interview-prep, study-plan]
created: 2026-09-24
weekly_hours: 30
weeks: 6
---

# Six Week Interview Study Plan

**Goal:** interview confidently for junior DE and adjacent data/software roles by improving timed execution, modeling precision, pipeline reliability, and spoken explanations. Based on [[Interview Preparation Audit]].

**Budget:** 30 hours/week, 180 hours total. Week 1 starts whenever you begin. No interview date was supplied. Continue applications during the plan; prior technical feedback already recommended applying. Projects are independent portfolio work. Use Amazon experience for professional examples, distinguish your implemented Kinesis work from batch variants, and never present unimplemented LLM/RAG features as completed.

## Weekly allocation and routine

| Topic | Hours | Share |
|---|---:|---:|
| SQL | 7.5 | 25% |
| Python | 6 | 20% |
| Data modeling | 4.5 | 15% |
| Pipelines/ETL/data quality | 4.5 | 15% |
| Behavioral/project explanation | 4.5 | 15% |
| Data system design | 1.5 | 5% |
| Role tools/business context | 1.5 | 5% |
| **Total** | **30** | **100%** |

Count each hour once. Project work uses these topic allocations; it is not extra. As an activity budget, aim for **6 hours focused review, 9 hours project work, and 15 hours drills, mocks, and correction** each week. These are two views of the same 30 hours. A project SQL query counts as SQL; recording its explanation counts as behavioral.

Suggested six-day schedule, five hours/day, with one day off:

| Day | Work blocks (hours) |
|---|---|
| Monday | SQL 2; Python 1; modeling 1; behavioral 1 |
| Tuesday | SQL 1.5; Python 1; pipelines 1.5; role context 1 |
| Wednesday | SQL 1.5; Python 1; modeling 1.5; behavioral 1 |
| Thursday | SQL 1; Python 1; pipelines 1.5; system design 1; role context 0.5 |
| Friday | SQL 1; Python 1; modeling 1; pipelines 1; behavioral 1 |
| Saturday | SQL 0.5; Python 1; modeling 1; pipelines 0.5; behavioral 1.5; system design 0.5 |

Breaks are outside study time. Move blocks to fit your life, preserving the weekly total. Use the tool/business block to inspect two live target postings and review only relevant requirements. Practice defining the business metric before proposing a tool.

For each week, choose roughly 8–12 SQL and 5–8 Python problems, including repeat attempts of mistakes. These are capacity guides, not quotas: correction and explanation matter more than new-question count. Use unseen variants, hide solutions, state assumptions aloud, and test empty input, nulls, duplicates, ties, and boundaries. Revisit errors after two days and seven days.

## Week 1 — Establish a baseline and repair reference errors

**Priority:** distinguish current skill gaps from old feedback and flawed reference answers.

**Review (6h):** [[SQL Optimization Principles]], selected window/date sections of [[SQL Algorithms Practice]], [[Python Data Engineering Toolkit]], [[Data Modeling Interview Guidance]], and [[Behavioral Question Checklist]]. Read the August technical and September behavioral feedback linked in the audit. Do not reread entire books.

**Drills and delivery (15h):** take a 45-minute mixed SQL baseline and a 45-minute Python transformation baseline without notes. Review every error. Do one 30-minute model from an unfamiliar business prompt. Record a 90-second introduction and two two-minute STAR answers; assess clarity, personal contribution, impact, and composure. Reserve follow-up time to explain technical choices conversationally.

**Project work (9h):** use the existing Calendly project as the main case study. Spend 3h defining a small synthetic fixture with known expected results; 3h making transformation checks runnable with the available environment; 3h documenting input/output grain, business questions, and current execution mode. Include duplicates, two event versions, late arrival, two dates, and zero bookings. If cloud access is unavailable, work locally and label cloud integration unverified.

**Deliverables / exit checks:**

- [ ] Baseline log records time, correctness, hints, and missing edge cases.
- [ ] Verify the transaction-total Python solution, SQL tie-break syntax, and fixed-month example identified in the audit; record corrected answers with expected results.
- [ ] Reconcile the 12 ms versus 5 ms leadership metric using evidence, or remove the unsupported number.
- [ ] Locate and label the Kinesis implementation separately from the scheduled batch story; keep LLM/RAG marked unimplemented.
- [ ] A reader can identify one row's meaning at each Calendly stage and reproduce the fixture checks.

## Week 2 — Define grain, joins, and business metrics precisely

**Priority:** make SQL and modeling agree with the business question.

**Review (6h):** [[Data Modeling Cheat Sheet]], two cases from [[Data Modeling Practice Questions]], relevant dimensional sections of [[Data Warehousing]], and [[Pandas DataFrame Fundamentals]]. Review joins, aggregation, date handling, and conditional measures only where the baseline exposed errors.

**Drills and delivery (15h):** practice join fan-out, top-N with ties, retention/cohorts, and rates with zero denominators. In Python/Pandas, implement equivalent joins and aggregations and check duplicates. Redesign the Yelp grain/key example and one event/session model. For each, write the question, grain, cardinalities, and three proving queries before drawing more tables. Record two behavioral answers with the result stated clearly and supported honestly.

**Project work (9h):** spend 3h defining Calendly booking, meeting, spend, and attribution dates; 3h implementing or revising the fixture-backed metrics; 3h checking joins and documenting the zero-booking policy. Decide whether spend is evaluated against booking creation or meeting date and state the limitation of that choice.

**Deliverables / exit checks:**

- [ ] Each table has an explicit grain, key, and relationship cardinality.
- [ ] Two model exercises have executable queries demonstrating that joins do not inflate counts.
- [ ] Zero-booking cost is intentionally handled, and date-boundary examples match the stated metric definition.
- [ ] Explain one business outcome in plain language, including what the portfolio dataset cannot establish.
- [ ] Reassess allocations from Weeks 1–2: move up to 3h toward the weakest observed skill, taking those hours from the strongest category. Keep 30h total and regular behavioral practice.

## Week 3 — Prove replay and incremental correctness

**Priority:** demonstrate what happens when data arrives twice, late, or out of order.

**Review (6h):** selected deduplication/window sections of [[Data Processing Systems (Spark)]], [[Delta Lake Labs]], [[Spark Optimization Review]], and CDC/idempotency sections in your existing references. Be able to explain source event time, ingestion time, and processing time separately.

**Drills and delivery (15h):** practice latest-state SQL with deterministic tie-breaking and Python event parsing/grouping. Explain Spark lazy evaluation, transformations/actions, driver/executors, partitions, and shuffles using one example. Give a 20-minute design walkthrough focused on duplicate delivery and partial failure. Rehearse a failure/learning story and a disagreement story, including follow-ups.

**Project work (9h):** spend 3h reproducing the Calendly ordering/replay risks from the audit; 3h implementing an explicit winning-version policy and guarded update in the applicable code; 3h parameterizing the run date and checking that rerunning one date preserves another date's gold results. Keep changes small and test the existing architecture before adding services.

**Deliverables / exit checks:**

- [ ] Replaying the same fixture does not change expected row counts or totals.
- [ ] An older event cannot overwrite the chosen newer state; equal timestamps have a documented rule.
- [ ] A late event changes only the intended result, according to the declared business policy.
- [ ] Two-day history remains available after a one-day rerun.
- [ ] Test output and README explain guarantees and limits; avoid unsupported “exactly once” claims.

## Week 4 — Demonstrate orchestration and data quality

**Priority:** retest the concrete Airflow and quality gaps raised by the technical coach.

**Review (6h):** [[Apache Airflow]], relevant [[Data Engineering Foundations]] material, and [Airflow task best practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html). Use the project's compatible version; do not make upgrading Airflow the task.

**Drills and delivery (15h):** SQL reconciliation queries; Python validation and error handling; explain completeness, uniqueness, validity, consistency, freshness, and accuracy with a specific check or limitation. Practice troubleshooting a stale dashboard from source through output. Record an ownership answer and a deadline answer that actually states the deadline and trade-off.

**Project work (9h):** adapt the existing guided healthcare Airflow example, using a small fixture rather than building a second large project. Allocate 2h to resolving import/configuration issues; 3h to date-specific tasks and safe reruns; 2h to a controlled transient failure/retry; 2h to quality checks and evidence. Use a local sink if cloud credentials or paid resources are unavailable, and document that boundary.

**Deliverables / exit checks:**

- [ ] DAG imports in its documented environment and has explicit dependencies.
- [ ] Retry recovers from the controlled failure without duplicate final output.
- [ ] A selected historical date can be rerun without using today's date as the data-selection rule.
- [ ] At least four meaningful quality checks include an intentional failing fixture and an actionable failure message.
- [ ] Explain what checks cannot prove—for example, format validity does not establish real-world accuracy.

## Week 5 — Warehouse history, performance, and portfolio evidence

**Priority:** connect modeling, implementation, and a credible project walkthrough.

**Review (6h):** targeted SCD/history sections of [[Data Warehousing]], [[DBT Review]], [[Snowflake Review]], and one relevant [[Snowflake Worked Examples]] exercise. Review execution-plan fundamentals from [[SQL Optimization Principles]].

**Drills and delivery (15h):** effective-dated joins, changing dimension attributes, cumulative measures, and incremental aggregates. Model one SCD2 example and answer a historical as-of question. Inspect a query plan before suggesting an optimization. Run a behavioral mock with interruptions and follow-ups; assess clarity and impact separately from technical detail.

**Project work (9h):** extend an existing warehouse exercise, rather than starting a new platform. Spend 3h on a small SCD2 fixture and historical query; 2h on uniqueness/non-overlap/reconciliation tests; 2h on one measured query-plan experiment; 2h replacing a generic pipeline README with setup, architecture, example data, tests, and limitations. Use an already available local SQL engine if needed; distinguish local SQL validation from untested Snowflake integration.

**Deliverables / exit checks:**

- [ ] Unchanged input does not create a new dimension version; changed attributes close/open versions correctly.
- [ ] There is at most one current row per business key and no overlapping validity intervals.
- [ ] Historical fact joins choose the expected dimension version.
- [ ] Any performance claim names dataset size, environment, and measurement; report an inconclusive result honestly.
- [ ] Give a five-minute project walkthrough covering the problem, your work, data model, failure handling, tests, and limitations.

## Week 6 — Simulate interviews and close the largest remaining gaps

**Priority:** convert preparation into repeatable interview performance.

**Review (6h):** only the error log, concise cheat sheets, verified stories, and relevant requirements from active applications. No new broad courses, service surveys, or LLM/RAG features.

**Drills and delivery (15h):** run two interview simulations on separate days, each including timed SQL/Python, a modeling or pipeline scenario, and behavioral follow-ups. Review recordings or ask a partner for feedback. Practice asking clarifying questions, stating assumptions, explaining an approach, and checking the result. Spend remaining drill time on the two most persistent errors.

**Project work (9h):** spend 3h rerunning the fixture and failure cases from previous weeks; 3h making setup/results reproducible from the README; 3h preparing the concise demo and evidence index. Document unresolved problems instead of hiding them. Avoid expansion into another project.

**Deliverables / exit checks:**

- [ ] Complete two independent timed sets with clear reasoning and tested results.
- [ ] Explain one model and one pipeline in 20–30 minutes each, including grain, business metrics, backfill, and failure handling.
- [ ] Deliver a 90-second introduction and two-minute stories, then handle follow-ups without inventing details.
- [ ] Project notes distinguish implemented, tested locally, tested in cloud, and proposed behavior.
- [ ] Pick the next week's top two priorities from measured errors and actual interview feedback.

## Scorecard and adjustment rules

These are personal practice targets, not employer cutoffs or guarantees:

| Area | Useful evidence of progress |
|---|---|
| SQL | On two different 45-minute sets of three mixed questions, at least two fully correct unaided; clearly explain and retest every miss, including edge cases |
| Python | Complete two appropriately scoped transformation tasks in 45 minutes with correct expected outputs and relevant edge checks; repeat on another set |
| Modeling | In 30 minutes, clarify the metric, state grain/keys/cardinalities/history, and show how the requested query works |
| Pipelines | Reproduce replay, out-of-order, quality-failure, and rerun cases and explain the observed outcome |
| Behavioral | Two recordings or peer reviews show concise structure, clear personal contribution, truthful impact, and calm follow-ups; aim for at least 4/5 on the same rubric used by your coach |

If a check fails, use the next week's review/correction blocks to address it; do not add hours beyond 30. Reduce new-problem volume before dropping error review. If an interview is scheduled, use the role-context block to tailor the next week to its stated format while keeping core SQL/Python practice.

Copy this log into [[00_daily_logs/Daily Review Template]] or a dated study note:

| Date | Topic / problem | Minutes | Unaided result | Edge case or explanation missed | Next action | Retest +2 days | Retest +7 days |
|---|---|---:|---|---|---|---|---|
| | | | | | | | |

For project experiments, add the input fixture, expected output, observed output, and reproducible command or test name. Keep repository changes in the repository. Save standalone local exports in Documents; keep these study notes in this vault as requested.
