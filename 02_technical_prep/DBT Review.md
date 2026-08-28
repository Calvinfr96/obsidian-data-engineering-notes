# dbt + Jinja Interview Prep

> **Purpose:** Interview-focused notes for Data Engineering interviews. This consolidates the structured dbt/Jinja notes and the dbt interview Q&A, while refining several oversimplified statements in the source material.
>
> **Core mental model:** **dbt manages transformations as code; Jinja makes those transformations dynamic and reusable.** dbt compiles Jinja + SQL into SQL and executes that SQL against the target warehouse.

---

# 1. dbt in 60 Seconds

**dbt (data build tool)** is a transformation and analytics-engineering framework used primarily for the **T** in an ELT pipeline.

```text
Sources
   ↓
Ingestion / EL
(Fivetran, Airbyte, DMS, custom pipelines, etc.)
   ↓
Cloud Data Warehouse
(Snowflake / BigQuery / Redshift / etc.)
   ↓
dbt
(transform + test + document)
   ↓
Analytics / BI / ML / downstream consumers
```

### What dbt does

- Defines SQL transformations as **models**
- Builds dependency graphs between models
- Compiles Jinja + SQL into executable SQL
- Materializes models as views, tables, incremental tables, or ephemeral CTEs
- Runs data-quality tests
- Tracks sources and source freshness
- Documents data models
- Supports reusable macros and packages
- Supports CI/CD and scheduled production jobs

### What dbt does NOT primarily do

dbt is **not an ingestion tool**. It generally expects source data to already exist in the warehouse or supported data platform.

> **Interview answer:** "dbt is primarily a transformation and modeling layer for ELT. The source data is loaded into the warehouse by an ingestion system, and dbt then uses SQL, Jinja, tests, documentation, and dependency management to transform that raw data into analytics-ready datasets."

---

# 2. ETL vs. ELT

## ETL

```text
Extract → Transform → Load
```

Transformation occurs before data reaches the warehouse.

## ELT

```text
Extract → Load → Transform
```

Raw data is loaded first, then transformed using the warehouse's compute.

### Why modern stacks favor ELT

Cloud warehouses such as Snowflake provide scalable compute, so transformations can happen where the data already lives.

Benefits:

- Raw data becomes available quickly
- Transformations can be changed without re-running ingestion
- Warehouse compute handles large transformations
- SQL is accessible to analysts and engineers
- Transformations can be version controlled
- Testing and documentation can be managed alongside transformation code

### Where dbt fits

```text
Extract → Load → [dbt Transform]
```

---

# 3. dbt's Role in a Data Team

| Role | Primary responsibility |
|---|---|
| Data Engineer | Build ingestion, infrastructure, pipelines, platforms |
| Analytics Engineer | Transform/model warehouse data, test, document, serve business needs |
| Data Analyst | Analyze trusted datasets and produce business insights |

dbt is especially associated with the **analytics engineering** layer, although Data Engineers frequently use and maintain dbt projects.

---

# 4. dbt Project Structure

A typical project contains:

```text
my_dbt_project/
├── models/
│   ├── staging/
│   ├── intermediate/
│   └── marts/
├── snapshots/
├── seeds/
├── tests/
├── macros/
├── dbt_project.yml
├── packages.yml
└── README.md
```

- **models/** → SQL transformations
- **seeds/** → static CSV/reference data
- **snapshots/** → historical tracking of changing records
- **tests/** → data-quality assertions
- **macros/** → reusable Jinja/SQL logic
- **packages.yml** → external dbt packages
- **dbt_project.yml** → project configuration

---

# 5. Models

A **dbt model is a SQL file whose query defines a dataset that dbt materializes in the warehouse.**

```sql
SELECT
    customer_id,
    first_name,
    last_name
FROM {{ source('raw', 'customers') }}
```

A model is not necessarily a table. Depending on materialization, it can become a view, table, incremental table, ephemeral CTE, or database-native materialized view where supported.

---

# 6. Sources

A **source represents raw/pre-existing data that dbt does not create.**

```yaml
sources:
  - name: raw
    database: RAW_DB
    schema: PUBLIC
    tables:
      - name: customers
```

Reference it:

```sql
SELECT *
FROM {{ source('raw', 'customers') }}
```

### Why use `source()`?

Instead of hard-coding:

```sql
FROM RAW_DB.PUBLIC.CUSTOMERS
```

you centralize the physical location in YAML.

### Source vs. model

> **Source = data that exists before dbt.**

> **Model = dataset created/transformed by dbt.**

---

# 7. `ref()` and the dbt DAG

```sql
SELECT *
FROM {{ ref('stg_customers') }}
```

dbt uses `ref()` to:

1. Resolve the actual relation name
2. Build the dependency graph
3. Determine execution order

```text
source: raw_customers
          ↓
    stg_customers
          ↓
    int_customer_orders
          ↓
    fct_customer_sales
```

### `source()` vs. `ref()`

**`source()`** → references raw data declared as a source.

**`ref()`** → references another dbt model and creates a DAG dependency.

---

# 8. Common Model Layers

```text
Source
  ↓
Staging
  ↓
Intermediate
  ↓
Facts / Dimensions
  ↓
Marts
```

### Staging

Light transformations:

- Rename columns
- Cast types
- Standardize values
- Basic cleanup

### Intermediate

Reusable business logic:

- Joins
- Aggregations
- Complex transformations

### Fact

Business processes/events:

- Orders
- Transactions
- Sessions
- Payments

### Dimension

Business entities:

- Customers
- Products
- Employees
- Locations

### Mart

Business-facing datasets designed for analytics/BI.

---

# 9. Materializations

**Materialization determines how dbt persists a model in the warehouse.**

Core materializations:

1. `view`
2. `table`
3. `incremental`
4. `ephemeral`

dbt can also work with database-native materialized views where supported.

## View

```sql
{{ config(materialized='view') }}
SELECT ...
```

Good for lightweight transformations.

**Pros:** little duplicated storage; underlying data is queried directly.

**Cons:** complex/stacked views can be slow.

## Table

```sql
{{ config(materialized='table') }}
SELECT ...
```

Good when query performance matters and rebuilding is acceptable.

**Pros:** fast to query.

**Cons:** normal runs rebuild the table, which can be expensive for large datasets.

## Incremental

Build the table initially, then process only the rows selected by the incremental logic.

```sql
{{ config(materialized='incremental') }}

SELECT *
FROM {{ source('raw', 'events') }}

{% if is_incremental() %}
WHERE created_at > (
    SELECT MAX(created_at)
    FROM {{ this }}
)
{% endif %}
```

**Pros:** dramatically reduces work for large, mostly-append datasets.

**Cons:** more operational complexity; must handle late data, updates/deletes, unique keys, and schema/logic changes.

## Ephemeral

```sql
{{ config(materialized='ephemeral') }}
```

Not created as a database relation. dbt injects its SQL into downstream models, typically as a CTE.

**Pros:** reusable logic without another warehouse object.

**Cons:** cannot query directly; overuse can create large, difficult-to-debug compiled queries.

---

# 10. Materialization Decision Framework

| Materialization | Use when |
|---|---|
| View | Lightweight transformation; query frequency/complexity is low |
| Table | Query performance matters; rebuilding is acceptable |
| Incremental | Dataset is large and only a subset changes each run |
| Ephemeral | Small reusable logic doesn't need a physical relation |
| Materialized view | Database-native precomputation is appropriate |

> **Interview principle:** Choose based on data volume, transformation cost, query frequency, freshness requirements, and build cost—not because one materialization is universally "best."

---

# 11. Incremental Models

An incremental model is especially valuable when:

```text
10 TB existing data
+ 50 GB new/changed data
```

Instead of processing all 10 TB every run, the model can process only the relevant changes.

### Key configuration

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}
```

### `is_incremental()`

It is true when the model is configured as incremental, the target relation already exists, and the run is not a full refresh.

### `this`

`{{ this }}` refers to the current model's target relation.

```sql
SELECT MAX(updated_at)
FROM {{ this }}
```

---

# 12. Incremental Strategies

The strategy controls how dbt applies incremental changes to the target.

A common Snowflake strategy is:

```text
merge
```

Another is:

```text
delete + insert
```

### Merge

```text
Incoming row
    ↓
Does unique key exist?
   /  yes  no
 ↓    ↓
UPDATE INSERT
```

### Delete + Insert

```text
Delete matching target rows
          ↓
Insert incoming rows
```

Choose based on:

- Data volume
- Whether records change
- Reliability of the unique key
- Warehouse capabilities
- Performance requirements

### Important

A `unique_key` is dbt incremental configuration; it is not automatically a database-enforced primary-key constraint.

---

# 13. Incremental Model Pitfalls

A filter such as:

```sql
WHERE updated_at > (
    SELECT MAX(updated_at)
    FROM {{ this }}
)
```

can miss records.

Potential problems:

- Late-arriving records
- Multiple rows with the same timestamp
- Clock differences
- Updates to older records
- Deletes

A common mitigation is a lookback window:

```sql
WHERE updated_at >= (
    SELECT DATEADD(hour, -2, MAX(updated_at))
    FROM {{ this }}
)
```

Then use a reliable `unique_key`/merge strategy to prevent duplicates.

---

# 14. Full Refresh

Use a full refresh when the existing incremental target no longer matches the current model logic.

```bash
dbt run --full-refresh --select my_incremental_model
```

Typical reasons:

- Transformation logic changed
- Incremental filtering changed
- Historical data needs recalculation
- Existing incremental state is inconsistent
- Schema changes require rebuilding

> **Interview answer:** "Incremental models trade simplicity for performance. If the transformation logic changes in a way that affects historical rows, I'd full-refresh the model so the historical target is rebuilt consistently."

---

# 15. Snapshots

A dbt **snapshot preserves historical versions of changing records**.

They are useful for **slowly changing dimensions (SCDs)**.

```text
Customer
status = gold
    ↓
status changes
    ↓
Customer
status = platinum
```

A normal model may show only the current state. A snapshot lets you answer:

- What was the customer's status last month?
- When did the status change?
- What value did this attribute have at a previous point in time?

> **Interview answer:** "I'd use a dbt snapshot when I need to preserve historical versions of mutable source records rather than only keeping their current state."

---

# 16. Seeds

A **seed is static data stored in a CSV file and loaded into the warehouse by dbt.**

Good examples:

- Country codes
- Small lookup tables
- Mapping tables
- Static classifications

```text
seeds/
└── country_codes.csv
```

Run:

```bash
dbt seed
```

One seed:

```bash
dbt seed --select country_codes
```

Don't use seeds for large or frequently changing production datasets.

---

# 17. Tests

dbt tests are assertions about your data.

Classic generic tests:

### `unique`

No duplicate values.

### `not_null`

Column contains no nulls.

### `accepted_values`

Values must belong to an allowed set.

### `relationships`

Checks referential integrity.

```text
orders.customer_id
        ↓
customers.id
```

### Testing strategy

Test both:

**Sources**

- Expected key uniqueness
- Required fields
- Valid status values
- Basic relationships

**Models**

- Expected grain
- Business rules
- Referential integrity
- Valid outputs

A common baseline for primary-key-like columns is:

```text
unique
not_null
```

---

# 18. Custom Tests

A custom singular test is a SQL query that should return **zero rows**.

```sql
SELECT
    order_id,
    SUM(payment_amount) AS total_payment
FROM {{ ref('payments') }}
GROUP BY order_id
HAVING SUM(payment_amount) < 0
```

Mental model:

```text
0 rows → PASS
1+ rows → FAIL
```

### Custom generic test

Reusable parameterized tests can use Jinja:

```sql
{% test positive_value(model, column_name) %}

SELECT *
FROM {{ model }}
WHERE {{ column_name }} < 0

{% endtest %}
```

---

# 19. `dbt run`, `dbt test`, and `dbt build`

## `dbt run`

Builds selected models.

```bash
dbt run
dbt run --select customer
```

## `dbt test`

Runs tests.

```bash
dbt test
dbt test --select customer
```

## `dbt build`

Builds and tests selected resources while respecting dependencies. It can include models, tests, seeds, and snapshots.

> **Correction:** Don't describe `dbt build` as literally "four commands in one." The important idea is that it operates across supported resource types and the DAG, combining building and validation.

---

# 20. Model Selection

### One model

```bash
dbt run --select my_model
```

### Upstream dependencies

```bash
dbt run --select +my_model
```

### Downstream dependencies

```bash
dbt run --select my_model+
```

### Both directions

```bash
dbt run --select +my_model+
```

### Source and downstream

```bash
dbt run --select source:my_source+
```

Mental model:

```text
+model   → upstream
model+   → downstream
+model+  → both
```

---

# 21. Configurations

Model configuration can be embedded in SQL:

```sql
{{
    config(
        materialized='table',
        schema='analytics'
    )
}}
```

or applied at project/folder scope:

```yaml
models:
  my_project:
    staging:
      +materialized: view
```

Use broader configuration for defaults and model-level configuration for exceptions.

---

# 22. Custom Schemas

Models can be organized into separate schemas.

```sql
{{ config(schema='marketing') }}
```

Useful for:

- Separating staging and marts
- Organizing by domain
- Access control
- Keeping production objects understandable

Know your project's schema-generation behavior and environment strategy rather than assuming every dbt project names schemas identically.

---

# 23. Aliases

By default, dbt commonly uses the model filename as the database relation identifier.

Override it:

```sql
{{ config(alias='customer_summary') }}
```

This lets code and database naming conventions differ.

---

# 24. Hooks

Hooks execute SQL around model execution.

### Pre-hook

Runs before the model.

Possible uses:

- Setup
- Audit logging
- Permissions
- Session/object preparation

### Post-hook

Runs after model execution.

Possible uses:

- Audit logging
- Grants
- Cleanup
- Metadata updates

> Use hooks for SQL side effects around model execution; don't use them as a substitute for normal dbt dependencies or general-purpose orchestration.

---

# 25. Documentation

dbt supports documentation for:

- Sources
- Models
- Tables
- Columns
- Macros

```yaml
models:
  - name: customers
    description: "One row per customer."
    columns:
      - name: customer_id
        description: "Unique identifier for a customer."
```

Good documentation explains:

- What the dataset represents
- What the grain is
- What columns mean
- Where the data came from
- What transformations were applied

### Doc blocks

For reusable long descriptions:

```jinja
{% docs order_status %}
Detailed explanation of order statuses.
{% enddocs %}
```

Then reference the block from YAML.

---

# 26. Source Freshness

Source freshness asks:

> **"Is the upstream data arriving on time?"**

Example:

```yaml
freshness:
  warn_after:
    count: 6
    period: hour
  error_after:
    count: 12
    period: hour

loaded_at_field: _etl_loaded_at
```

This catches a problem that normal model execution may miss:

```text
dbt job succeeded
        ≠
data is fresh
```

---

# 27. Packages

Packages provide reusable dbt functionality.

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.3.3
```

Install:

```bash
dbt deps
```

Packages can provide:

- Macros
- Models
- Tests
- Utilities

Example:

```jinja
{{ dbt_utils.date_spine(
    datepart='day',
    start_date="cast('2025-01-01' as date)",
    end_date="cast('2026-01-01' as date)"
) }}
```

---

# 28. dbt Core, CLI, and Cloud

### dbt Core

The open-source dbt framework/runtime.

### dbt CLI

The command-line interface used to invoke dbt operations.

```bash
dbt run
dbt test
dbt build
dbt compile
dbt seed
```

### dbt Cloud

Managed platform capabilities around dbt development and operations, such as:

- Browser-based development
- Job scheduling
- CI/CD workflows
- Documentation/catalog
- Monitoring
- Collaboration

> **Interview answer:** "dbt Core is the open-source framework/runtime, the CLI is how I invoke dbt commands, and dbt Cloud is the managed platform that adds development, orchestration, collaboration, and operational capabilities."

---

# 29. Compilation

Conceptually:

```text
SQL + Jinja
     ↓
dbt compilation
     ↓
Pure SQL
     ↓
Warehouse
```

For example:

```sql
SELECT *
FROM {{ ref('customers') }}
```

becomes SQL referencing the resolved relation.

Useful command:

```bash
dbt compile
```

Compiled SQL can be inspected under:

```text
target/compiled/
```

### Why compilation matters

It lets you:

- Inspect generated SQL
- Debug Jinja
- Understand dependencies
- Verify macros
- See what the warehouse will execute

---

# 30. Debugging dbt

When a model fails:

1. Read the error.
2. Determine whether the issue is Jinja, SQL, permissions, a missing relation, warehouse execution, or a test.
3. Inspect compiled SQL.
4. Run the compiled SQL directly if useful.
5. Check upstream dependencies.
6. For failed tests, inspect the compiled test SQL and identify failing records.

A useful command when dependencies are missing:

```bash
dbt run --select +my_model
```

---

# 31. Jinja: Why It Exists in dbt

**Jinja is a templating language that adds programmatic behavior around SQL.**

SQL alone is declarative. Jinja provides:

- Variables
- Conditions
- Loops
- Reusable macros
- Dynamic SQL generation

### Mental model

> **SQL describes the transformation.**

> **Jinja generates or controls the SQL.**

---

# 32. Jinja Syntax

## Expression

```jinja
{{ variable }}
```

Used to output/render a value.

Examples:

```jinja
{{ ref('customers') }}
{{ source('raw', 'customers') }}
{{ my_macro('amount') }}
```

## Statement

```jinja
{% ... %}
```

Used for logic:

```jinja
{% if ... %}
{% for ... %}
{% set ... %}
```

## Comment

```jinja
{# comment #}
```

Jinja comments are removed during rendering.

### Easy memory trick

```text
{{ }} → output
{% %} → logic
{# #} → comment
```

---

# 33. Jinja Variables

```jinja
{% set threshold = 100 %}

SELECT *
FROM orders
WHERE amount > {{ threshold }}
```

Lists:

```jinja
{% set payment_methods = [
    'credit_card',
    'bank_transfer',
    'coupon'
] %}
```

Dictionaries:

```jinja
{% set customer = {
    'name': 'Calvin',
    'role': 'engineer'
} %}
```

Access:

```jinja
{{ customer['name'] }}
```

---

# 34. Jinja Conditionals

```jinja
{% if target.name == 'prod' %}

    -- production SQL

{% else %}

    -- development SQL

{% endif %}
```

Useful when SQL needs to vary by:

- Environment
- Incremental/full build
- Configuration
- Variables
- Runtime context

---

# 35. Jinja Loops

```jinja
{% for method in payment_methods %}

    SUM(
        CASE
            WHEN payment_method = '{{ method }}'
            THEN amount
            ELSE 0
        END
    ) AS {{ method }}_amount

{% endfor %}
```

Useful for generating repetitive SQL such as dynamic pivot expressions.

### `loop.last`

```jinja
{% for method in payment_methods %}
    ...
    {% if not loop.last %},{% endif %}
{% endfor %}
```

Avoids an invalid trailing comma.

---

# 36. Jinja Whitespace Control

```jinja
{%- ... -%}
```

The hyphens remove surrounding whitespace.

This mainly keeps compiled SQL clean; it normally does not change SQL semantics.

---

# 37. Macros

A **macro is reusable Jinja/SQL logic**, conceptually similar to a function.

Macros normally live in:

```text
macros/
```

Example:

```jinja
{% macro cents_to_dollars(column_name, decimals=2) %}
    ROUND({{ column_name }} / 100, {{ decimals }})
{% endmacro %}
```

Use:

```jinja
{{ cents_to_dollars('amount') }}
```

### Why macros?

They reduce duplicated SQL-generation logic and allow parameterization.

> **Interview answer:** "I use macros when the same SQL-generation logic appears in multiple models and is genuinely reusable or parameterizable."

---

# 38. Macro Arguments and Defaults

```jinja
{% macro cents_to_dollars(column_name, decimals=2) %}
    ROUND({{ column_name }} / 100, {{ decimals }})
{% endmacro %}
```

Call:

```jinja
{{ cents_to_dollars('amount') }}
```

or:

```jinja
{{ cents_to_dollars('amount', 4) }}
```

`decimals=2` is a default parameter.

---

# 39. dbt Functions Inside Jinja

A useful distinction:

```jinja
{{ ref('customers') }}
```

is not generic Jinja behavior. It is a **dbt-provided function exposed through Jinja**.

Other examples:

```jinja
{{ source(...) }}
{{ config(...) }}
{{ this }}
{{ target }}
{{ is_incremental() }}
```

Mental model:

```text
Jinja
  ↓
Templating language

dbt
  ↓
Provides context/functions/macros used inside Jinja
```

---

# 40. `run_query()`

`run_query()` lets a macro execute SQL against the warehouse and access the result from Jinja.

```jinja
{% set results = run_query("SELECT COUNT(*) FROM customers") %}
```

Useful for advanced dynamic macros.

### Caution

This introduces warehouse execution and side effects into macro logic, so use it deliberately.

---

# 41. `execute`

`execute` indicates whether dbt is in a phase where SQL execution is active.

A defensive pattern:

```jinja
{% if execute %}

    {% set results = run_query(...) %}

{% endif %}
```

Why? dbt may parse/compile code without actually executing the warehouse query.

---

# 42. `log()`

Macros can emit messages:

```jinja
{{ log('Starting transformation', info=True) }}
```

Useful for debugging and operational visibility.

---

# 43. `target`

`target` contains information about the active dbt target/environment.

Examples:

```jinja
{{ target.schema }}
{{ target.name }}
{{ target.role }}
```

Useful for environment-aware behavior.

---

# 44. `this`

`this` refers to the current model's target relation.

Especially important in incremental models:

```jinja
SELECT MAX(updated_at)
FROM {{ this }}
```

---

# 45. Jinja + Incremental Logic

```sql
SELECT *
FROM {{ source('raw', 'orders') }}

{% if is_incremental() %}

WHERE updated_at > (
    SELECT MAX(updated_at)
    FROM {{ this }}
)

{% endif %}
```

### Full run

```text
is_incremental() = false
        ↓
filter omitted
        ↓
process full source
```

### Incremental run

```text
is_incremental() = true
        ↓
filter included
        ↓
process selected records
```

---

# 46. Packages + Macros

Packages let teams reuse functionality written by others.

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.3.3
```

Then:

```bash
dbt deps
```

A package can provide macros, models, and tests that become available to the project.

---

# 47. Deployment and CI/CD

A common production workflow:

```text
Developer
   ↓
Feature branch
   ↓
Pull Request
   ↓
CI tests / build
   ↓
Merge to main
   ↓
Production job
   ↓
dbt build
   ↓
Production warehouse
```

### Development vs. production

Developers should generally work in isolated development schemas.

Production models should be built in production schemas using controlled deployment credentials.

Benefits:

- Developers can experiment safely
- Production consumers aren't interrupted
- Changes can be reviewed and tested
- Deployments become repeatable

---

# 48. Scheduling and Orchestration

dbt jobs can run:

- On a schedule
- Based on upstream job completion
- Through CI/CD
- Through external orchestration systems

Possible orchestrators include:

- dbt Cloud
- Airflow
- Dagster
- Prefect
- CI/CD systems

### Important interview point

dbt handles **transformation**, while a broader orchestrator may coordinate:

```text
Ingestion
  ↓
dbt
  ↓
Data quality
  ↓
Downstream processing
```

---

# 49. Data Lineage

Because dbt knows relationships created through:

```jinja
{{ ref(...) }}
```

and:

```jinja
{{ source(...) }}
```

it can build a DAG representing lineage.

```text
raw.orders
     ↓
stg_orders
     ↓
int_customer_orders
     ↓
fct_orders
     ↓
dashboard
```

Uses:

- Dependency understanding
- Impact analysis
- Debugging
- Selective execution
- Data discovery

---

# 50. Performance in dbt

dbt generally relies on the target warehouse for query execution.

Therefore, dbt performance often means:

> **How efficiently are we asking the warehouse to transform the data?**

Important techniques:

- Use incremental models for large changing datasets
- Avoid unnecessary view chains
- Choose materializations intentionally
- Filter early
- Select only needed columns
- Understand warehouse compute
- Inspect generated SQL
- Monitor model runtimes and warehouse usage

---

# 51. dbt + Snowflake Architecture

```text
                    ┌───────────────┐
                    │   Source      │
                    │ S3 / DB / API │
                    └───────┬───────┘
                            ↓
                    Ingestion Layer
                 Snowpipe / Fivetran / etc.
                            ↓
                    ┌───────────────┐
                    │ Raw Snowflake │
                    │    Tables     │
                    └───────┬───────┘
                            ↓
                         dbt
                            │
                  ┌─────────┴─────────┐
                  ↓                   ↓
             Staging              Tests/Docs
                  ↓
             Intermediate
                  ↓
           Facts / Dimensions
                  ↓
                Marts
                  ↓
             BI / Analytics
```

### Division of responsibility

**Snowflake**

- Storage
- Compute
- SQL execution
- Ingestion capabilities
- Security
- Recovery

**dbt**

- Transformation definitions
- Dependency management
- Testing
- Documentation
- Compilation
- Deployment workflows

**Jinja**

- Dynamic SQL generation
- Reusable logic
- Conditional/loop-based SQL generation

---

# 52. High-Value Interview Comparisons

## dbt vs. ETL tool

**ETL tool:** often extracts, transforms, and loads.

**dbt:** primarily transforms data already loaded into the warehouse.

## Source vs. model

**Source:** pre-existing raw data.

**Model:** dbt-created transformed dataset.

## `source()` vs. `ref()`

**`source()`** → raw source.

**`ref()`** → another dbt model + DAG dependency.

## Table vs. incremental

**Table:** rebuild the table.

**Incremental:** process selected changes after initial build.

## Incremental vs. snapshot

**Incremental:** efficient transformation/update strategy.

**Snapshot:** historical tracking of mutable records.

## Macro vs. model

**Model:** produces a warehouse dataset.

**Macro:** generates/reuses SQL/Jinja logic.

## Macro vs. package

**Macro:** reusable function-like logic.

**Package:** distributable collection of dbt functionality.

## `run` vs. `build`

**`dbt run`** → build models.

**`dbt build`** → build and test selected resources while respecting dependencies.

## Jinja vs. SQL

**SQL:** describes the transformation.

**Jinja:** generates/controls the SQL.

---

# 53. Common Interview Questions

### Q: What is dbt?

> "dbt is a transformation and modeling tool used primarily in ELT workflows. It lets teams define SQL transformations as code, build dependencies between models, test data quality, document datasets, and deploy transformations against a cloud data warehouse."

### Q: Why use dbt instead of disconnected SQL scripts?

> "dbt adds software-engineering practices around SQL: version control, dependency management, testing, documentation, reusable macros, environments, and deployment workflows."

### Q: Does dbt extract data?

> "Not primarily. dbt expects source data to already exist in the target data platform. In a typical ELT architecture, an ingestion system loads raw data and dbt handles transformation."

### Q: What is a dbt model?

> "A model is a SQL file that defines a transformation. dbt compiles the SQL and materializes its result according to configuration such as view, table, or incremental."

### Q: What is `ref()`?

> "`ref()` references another dbt model. dbt uses it to resolve the model's relation and automatically create a dependency in the DAG."

### Q: What is the difference between `source()` and `ref()`?

> "`source()` references raw data declared as a source, while `ref()` references another dbt model."

### Q: When would you use an incremental model?

> "I'd use one when the dataset is large enough that rebuilding it every run is expensive and only a subset of records changes. I'd define a reliable incremental filter and, for updates, usually a unique key and appropriate incremental strategy."

### Q: When would you NOT use incremental?

> "If the dataset is small, transformations are cheap, or the incremental logic would be more complex than the performance benefit justifies, I'd keep it as a table or view."

### Q: What are snapshots?

> "Snapshots preserve historical versions of mutable source records. They're useful for slowly changing dimensions where I need to know not just the current value, but how an attribute changed over time."

### Q: What are the four classic generic dbt tests?

> "`unique`, `not_null`, `accepted_values`, and `relationships`."

### Q: How does dbt determine execution order?

> "dbt builds a dependency graph from references such as `ref()`, then uses that DAG to determine execution order."

### Q: Why use Jinja?

> "Jinja allows me to generate dynamic SQL. I can use variables, loops, conditionals, and reusable macros rather than duplicating SQL."

### Q: What is a macro?

> "A macro is reusable Jinja/SQL logic, similar to a function. I use macros when the same SQL-generation logic appears across multiple models."

### Q: How do you debug compiled dbt SQL?

> "I'd run `dbt compile` or inspect the compiled SQL under `target/compiled`. That lets me see exactly what SQL dbt generated after resolving Jinja, refs, sources, and macros."

### Q: What is source freshness?

> "Source freshness checks whether upstream data is arriving within an expected time window. It's important because a dbt job can succeed technically while still producing stale data."

---

# 54. Scenario Questions

## Scenario 1 — Billion-row event table

> You have a 5-billion-row events table and only a few million new events arrive each day. Rebuilding takes two hours. What do you do?

Strong answer:

1. Consider an incremental model.
2. Identify a reliable timestamp/change column.
3. Determine whether events are append-only or mutable.
4. If mutable, define a reliable `unique_key`.
5. Choose an incremental strategy.
6. Handle late-arriving events with a lookback window if necessary.
7. Test duplicates and completeness.
8. Measure runtime and warehouse cost before/after.

## Scenario 2 — Customer status history

> A customer changes Bronze → Silver → Gold and the business wants historical status.

Answer:

> Use a dbt snapshot or another explicit SCD strategy because the requirement is historical state tracking.

## Scenario 3 — Repeated SQL logic

> Twenty models contain the same currency-conversion expression.

Answer:

> Create a macro if the logic is genuinely reusable and parameterizable.

## Scenario 4 — Slow BI dashboard

> A dashboard queries a model composed of several nested views.

Approach:

1. Inspect generated SQL and warehouse query profile.
2. Determine whether repeated view computation is expensive.
3. Consider materializing expensive logic as a table.
4. Consider incremental processing if only a small portion changes.
5. Validate the actual performance improvement.

## Scenario 5 — Bad upstream data

> The dbt model succeeds, but yesterday's source data never arrived.

Answer:

> Add source freshness checks and alert when the upstream source exceeds its warning/error threshold.

## Scenario 6 — Developer changes production data

> A developer needs to test a model change without affecting production.

Answer:

> Use an isolated development schema/environment and CI before merging to the production branch.

## Scenario 7 — Incremental model produces duplicates

Investigate:

1. Is the incremental filter correct?
2. Can records arrive late?
3. Can multiple records share the same timestamp?
4. Is the `unique_key` truly unique?
5. Is the incremental strategy appropriate?
6. Is a lookback window needed?
7. Are merges matching records correctly?

---

# 55. Important Interview Traps

### 1. dbt is not an ingestion tool

Don't claim dbt extracts data from APIs/databases and loads Snowflake.

### 2. `dbt build` isn't literally four commands

Say that it builds/tests supported resources while respecting dependencies.

### 3. A snapshot isn't simply an incremental table

Its purpose is historical tracking of changing records.

### 4. Incremental does not automatically mean "new rows only"

Incremental strategies can support updates depending on configuration.

### 5. `unique_key` is not automatically a database constraint

It is dbt incremental configuration.

### 6. Jinja doesn't execute SQL by itself

Jinja templates code. dbt supplies context/functions, and the resulting SQL is executed by the warehouse.

### 7. Macros don't normally create tables

Macros generate/reuse logic. Models represent datasets that dbt materializes.

### 8. Don't make everything incremental

Incremental logic adds complexity. Use it when the performance/cost benefit justifies it.

### 9. Don't make everything a table

Tables can improve query performance but may be expensive to rebuild.

### 10. Job success doesn't prove data freshness

Use source freshness checks to validate upstream timeliness.

---

# 56. Priority Study Order

## Tier 1 — Must Know

1. What dbt is
2. dbt's role in ELT
3. Models
4. Sources
5. `ref()` vs. `source()`
6. DAG/dependencies
7. Materializations
8. Incremental models
9. `is_incremental()` and `this`
10. Snapshots
11. Tests
12. `dbt run` vs. `dbt build`
13. Jinja syntax
14. Macros
15. `dbt compile`
16. Deployment/CI concepts

## Tier 2 — Strongly Recommended

17. Incremental strategies
18. `unique_key`
19. Seeds
20. Source freshness
21. Custom tests
22. Packages
23. Hooks
24. Custom schemas
25. Aliases
26. Documentation
27. Model selection syntax
28. Debugging compiled SQL

## Tier 3 — Advanced / Lower Priority

29. `run_query()`
30. `execute`
31. `target`
32. Code generation
33. Advanced macro patterns
34. dbt Cloud control-plane features

For a Data Engineering interview, **Tier 1 + scenario questions** are more valuable than memorizing every Q&A from the source PDF.

---

# 57. Final Interview Checklist

- [ ] What is dbt?
- [ ] Why is dbt associated with ELT?
- [ ] What does dbt actually execute?
- [ ] What is a dbt model?
- [ ] What is a source?
- [ ] `source()` vs. `ref()`
- [ ] How does dbt build the DAG?
- [ ] What is a materialization?
- [ ] View vs. table
- [ ] Table vs. incremental
- [ ] Incremental vs. snapshot
- [ ] What is `is_incremental()`?
- [ ] What is `{{ this }}`?
- [ ] What is `unique_key`?
- [ ] When should you full-refresh?
- [ ] What are snapshots used for?
- [ ] What are seeds?
- [ ] What are the four generic tests?
- [ ] How do custom tests work?
- [ ] `dbt run` vs. `dbt test` vs. `dbt build`
- [ ] How does dbt handle dependencies?
- [ ] How do you select upstream/downstream models?
- [ ] What is Jinja?
- [ ] `{{ }}` vs. `{% %}` vs. `{# #}`
- [ ] What are macros?
- [ ] Why use packages?
- [ ] What is `run_query()`?
- [ ] What is `execute`?
- [ ] What is `target`?
- [ ] How do you debug compiled SQL?
- [ ] What is source freshness?
- [ ] How would you deploy dbt safely?
- [ ] How does dbt fit into a Snowflake architecture?

---

# 58. One-Minute dbt Interview Answer

> "I think of dbt as the transformation and modeling layer in a modern ELT pipeline. The raw data is first ingested into a warehouse such as Snowflake, and dbt manages the SQL transformations that turn that raw data into staging, intermediate, fact, dimension, and mart models. One of the biggest benefits is that dbt treats transformations as software: models are version controlled, dependencies are represented through `ref()`, data quality is enforced through tests, and documentation can live alongside the code. For larger datasets, I can use incremental models so I don't rebuild the entire dataset on every run. Jinja and macros add reusable and dynamic SQL generation, while dbt's deployment and CI workflows allow those transformations to be tested before reaching production."

---

# 59. Final Mental Model

```text
Snowflake
   │
   │ storage + compute
   ↓
Raw Data
   │
   │ dbt
   ↓
┌──────────────────────────────────┐
│ Models                            │
│  ├── Staging                      │
│  ├── Intermediate                 │
│  ├── Facts / Dimensions           │
│  └── Marts                        │
│                                  │
│ Jinja → dynamic SQL              │
│ Macros → reusable SQL             │
│ Tests → data quality              │
│ Sources → raw-data metadata       │
│ ref() → dependencies / DAG        │
│ Snapshots → historical state      │
│ Incremental → efficient updates   │
└──────────────────────────────────┘
   ↓
Analytics / BI / ML
```

> **Snowflake provides the data platform and compute.**
>
> **dbt provides the transformation framework.**
>
> **SQL provides the transformation logic.**
>
> **Jinja makes that SQL dynamic and reusable.**
>
> **Tests provide data-quality checks.**
>
> **The DAG provides dependency management.**
>
> **CI/CD and deployment turn the project into a production data workflow.**
