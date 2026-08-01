## Why Does Airflow Exist?

- Imagine your company has a nightly data pipeline. Every night:
	```
	Extract data
	
	↓
	
	Transform with Spark
	
	↓
	
	Load into Snowflake
	
	↓
	
	Generate reports
	
	↓
	
	Notify stakeholders
	```
	- This seems pretty straightforward, but how do you run it every night?
	- **Option 1: Cron**
		- Maybe you write: `0 1 * * *`.
		- Now the Spark job runs every night at 1AM.
		- Problem solved? Not really.
			- Suppose Snowflake is configured to load data from the Spark job every night at 1:30 AM.
			- If the job, which normally takes 30 minutes to run, all of a sudden takes 2 hours, the warehouse receives incomplete data.
	- **Option 2: Dependencies**
		- The only solid fix to this **orchestration** conundrum is to create a dependency.
		- Load data into Snowflake **after** the Spark job finishes successfully.
- **Airflow's Idea**:
	- The idea behind Airflow is to describe relationships between different jobs, instead of scheduling them by time.
	- Airflow figures out:
		- What can run
		- What must wait
		- What failed
		- What should retry
- **Airflow is an Orchestrator**:
	- Airflow doesn't process your data. It simply **orchestrates** the tools that do.
	- Think of it like a conductor leading an orchestra. The conductor doesn't know how to play every instrument. They simply coordinate everyone else.

### Scheduling vs. Orchestration
- **Scheduling** answers when something should run.
- **Orchestration** answers what should run, in what order, and under what conditions.
- Airflow schedules and orchestrates jobs. The orchestration is the bigger responsibility.
- **Airflow vs. Cron**:
	- Cron can:
		- Run a job at a certain time.
	- Airflow can:
		- Run a job at a certain time.
		- Manage dependencies
		- Retry failures
		- Track history
		- Alert failures
		- Visualize workflows
		- Backfill historical runs
		- Monitor execution
		- Run tasks in parallel
	- Cron simply wasn't designed for complex pipelines.

### Tradeoffs
- Airflow is very powerful, but it introduces:
	- Infrastructure
	- Metadata database
	- Scheduler
	- Workers
	- Monitoring
- For simple nightly script, Cron might be a better fit.
- For dozens or hundreds of dependent workflows, Airflow becomes extremely valuable.

### Mental Model
- Connection to Previous Topics:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	
	↓
	
	Dashboards
	```
	- Airflow sits above all of these systems and coordinates them.

### Common Interview Questions
1. Why use Airflow instead of cron?
> 	Cron schedules independent jobs based on time, but it has no understanding of dependencies or workflow state. Airflow orchestrates entire pipelines by managing task dependencies, retries, monitoring, alerts, and execution history. It's better suited for complex ETL and data engineering workflows where multiple tasks depend on one another.
2. Your company currently uses Cron to schedule a nightly ETL process:
	```
	Extract
	
	↓
	
	Spark
	
	↓
	
	Snowflake
	
	↓
	
	Reporting
	```
	- The Spark job occasionally takes much longer than expected, causing downstream jobs to fail. How would Airflow improve this workflow? What advantages would it provide over cron?
> 	Since the execution time of the Spark job can vary wildly, a simple Cron job could not reliably schedule each individual task. Providing generous time gaps between each task wastes time and delays reporting. Using Airflow allows each task to be orchestrated so that a task doesn't begin until its dependent tasks have completed successfully.
1. A teammate says: "We already have Spark. Why do we need Airflow? Spark can run jobs by itself." How would you respond? What roles do Spark and Airflow each play in a modern data platform?
> 	I would mention that Spark is primarily a transformation tool. It still needs to extract data from a source and write it to a data store such as Snowflake. Coordinating these tasks using time alone would be unreliable since each task doesn't finish in a fixed amount of time. Airflow provides orchestration between tasks, so that one task doesn't begin until its dependent tasks have completed successfully.
	- Don't forget to mention that Airflow doesn't **replace** Spark, it **uses** it. Spark acts as one step in a workflow.
2. If Spark already has retry mechanisms, why does Airflow also support retries?
> 	They operate at different levels. Spark retries failed tasks within a Spark job, such as re-executing a failed partition after an executor failure. Airflow retries entire workflow tasks. For example, if a Spark job fails because a source database was temporarily unavailable, Airflow can retry launching the entire Spark job after a delay.

## Directed Acyclic Graphs (DAGs)

- Suppose your pipeline looks like:
	- Process Customers:
		```
		Extract Customers
		
		↓
		
		Transform Customers
		
		↓
		
		Load Customers
		```
	- Process Orders:
		```
		Extract Orders
		
		↓
		
		Transform Orders
		
		↓
		
		Load Orders
		```
	- Generate Reports:
		```
		Generate Reports
		```
- Airflow describes this pipeline using a DAG.
- A DAG is a graph that describes tasks and the dependencies between them.
	- Each node represents a task.
	- Each arrow represents a dependency.
	- Example:
		```
		Extract Customers
		
		        ↓
		
		Transform Customers
		
		        ↓
		
		Load Customers
		
		                ↘
		
		                 Generate Report
		
		                ↗
		
		Load Orders
		
		        ↑
		
		Transform Orders
		
		        ↑
		
		Extract Orders
		```
		- **Directed**: Every arrow has a direction that represents a dependency.
		- **Acyclic**: Airflow **strictly** forbids cycles because they make it impossible which task should start first.
		- **Graph**: A workflow isn't always one straight line. Sometimes, tasks branch.
	- **Parallel Execution**:
		- Parallel execution is one of the biggest advantages of DAGs. When two pipelines don't depend on each other, Airflow can run them simultaneously. This is much faster.
	- **Dependencies**:
		- Dependencies answer: "What must finish before this task can begine?"
		- A task will not start if any of its dependent tasks fail.
	- When airflow executes a DAG, it independtly keeps track of which tasks succeeded (task state) so it doesn't need to retry them unless explicitly told to do so.
	- **DAGs Are Declarative**:
		- Instead of telling Airflow when to run a task, you describe that task's dependencies. Airflow figures out the rest from there.

### Tradeoffs
- More Dependencies:
	- Pros:
		- Safer execution
		- Clear worflow
		- Prevents invalid runs
	- Cons:
		- Less parallelism
		- Longer total runtime
- Fewer Dependencies:
	- Pros:
		- More concurrency
		- Faster pipelines
	- Cons:
		- Greater risk of tasks executing before prerequisites are complete
- Dependencies should only be declared when they're **required**.

### Mental Model
- Connection to Previous Topics:
	```
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	
	↓
	
	dbt
	
	↓
	
	Dashboard
	```
	- Each of these systems perform work.
	- The DAG defines:
		- When each task starts
		- What it depends on
		- What can run simultaneously

### Common Interview Questions
1. Why must Airflow DAGs be acyclic?
> 	Cycles create circular dependencies where tasks wait on one another indefinitely, making it impossible for the workflow to begin. Airflow requires DAGs to be acyclic so that every task has a well-defined execution order.
2. Suppose you have the following workflows:
	```
	Extract Customers
	
	↓
	
	Transform Customers
	
	↓
	
	Load Customers
	```
	and
	```
	Extract Orders
	
	↓
	
	Transform Orders
	
	↓
	
	Load Orders
	```
	finally
	```
	Generate Daily Report
	```
	- Which tasks can run in parallel? Which tasks must wait for others? Why is this a good fit for a DAG?
> 	The Customers and Orders workflow can run in parallel since one doesn't depend on the other. However the daily report generation workflow must wait for the Customers and Orders workflows to complete before it begins. This is a good fit for a DAG because it determines dependencies between tasks, executes them in the proper order, and allows for parallel execution of unrelated tasks. Running independent tasks in parallel reduces the total pipeline runtime because Airflow doesn't serialize work unnecessarily.
1. A teammate suggests adding this dependency:
	```
	Task A
	
	↓
	
	Task B
	
	↓
	
	Task C
	
	↓
	
	Task A
	```
	- They say: "It's fine because eventually they'll all finish." How would you explain why Airflow rejects this workflow? Why are cycles fundamentally incompatible with DAG execution?
> 	I would explain that Airflow strictly prohibits cycles because they create circular dependencies, making it impossible for Airflow to determine where a workflow should begin.
	- Airflow validates DAGs and checks for cycles **before** they're scheduled. If you try to add a task to an existing DAG and that task creates a cycle, it will be rejected.

## Tasks & Operators

- Imagine a simple pipeline:
	```
	Extract
	
	↓
	
	Transform
	
	↓
	
	Load
	```
	- The DAG describes the workflow.
	- What does the `Extract` task actually do?
	- This is where operators come into play.
- An **operator** defines what work should be performed. Think of an operator like a template.
- Examples:
	- Execute Python code
	- Run a Bash command
	- Submit a Spark job
	- Execute SQL
	- Send an email
	- Call an API
- A **task** is an instance of an operator within a DAG.
- **Common Operators**:
	- `PythonOperator`: Runs Python code.
		- Example Use Cases:
			- Data validation
			- Small transformations
			- Calling REST APIs
			- Simple business logic
	- `BashOperator`: Runs shell commands.
		- Example: `spark-submit...` or `aws s3 cp...`
	- SQL Operators:
		- Execute SQL Against:
			- Snowflake
			- PostgreSQL
			- BigQuery
			- Redshift
	- Spark Operators:
		- Instead of processing data inside Airflow, Airflow submits a Spark job.
	- Email Operators:
		- Useful For:
			- Success notifications
			- Failure reports
			- Daily reports
	- Every task in a DAG uses the operator best suited for its work.
- **Should One Task Do Everything?**
	- Suppose someone writes a single task that:
		- Downloads data
		- Runs Spark
		- Loads Snowflake
		- Refreshes dashboards
	- Technically possible. Not a good idea.
	- This forces successful work to be rerun in the event of task failure and subsequent retry.
- **Task Granularity**:
	- Too Few Tasks:
		- Hard to monitor.
		- Hard to retry.
		- Poor visibility.
	- Too Many Tasks:
		- More scheduling overhead.
		- More complex DAGs that are harder to understand.
		- Increased operational complexity.
	- **Good Rule of Thumb**: A task should perform **one logical unit of work**.

### Mental Model
- Connection to Previous Topics:
	```
	Python
	
	↓
	
	Spark
	
	↓
	
	Parquet
	
	↓
	
	Snowflake
	
	↓
	
	dbt
	```

### Common Interview Questions
1. Why shouldn't an Airflow task perform an entire ETL pipeline?
> 	Tasks should represent logical units of work. Breaking a pipeline into separate tasks improves observability, allows independent retries, makes dependencies explicit, and avoids rerunning successful work after a failure. Extremely fine-grained tasks, however, introduce unnecessary scheduling overhead, so task granularity should balance maintainability and operational efficiency.
> 	Yes, I would recommend breaking the single task into multiple tasks. This improves observability, allows for parallel work (when tasks are independent) and independent retries, makes dependencies, more explicit, and avoids reruning successful work after failure. However, breaking the task up into too many tasks increases operational complexity and introduces unnecessary scheduling overhead.
2. Your teammate creates one Airflow task that:
	- Downloads files
	- Runs Spark
	- Loads Snowflake
	- Refreshes dashboards
	- Sends a notification
	- Would you recommend breaking this into multiple tasks? Why or why not? What benefits would that provide?
> 	I would not agree unless it was a very complex SQL statement that performed expensive transformations. Simpler SQL statements should be combined into single units of work and run as one task. This improves maintainability, reduces complexity, and reduces scheduling overhead.
3. Suppose one Airflow task takes two hours to run. Should you split it into multiple tasks?
	- The answer ultimately depends on:
		- Is it one logical unit of work?
		- Can portions run independently?
		- Would splitting improve retries?
		- Would splitting improve observability?
		- Are there natural dependency boundaries?

## Scheduling, Retries & Catchup

- Imagine a simple pipeline:
	```
	Download Data
	
	↓
	
	Spark Job
	
	↓
	
	Load Snowflake
	
	↓
	
	Refresh Dashboard
	```
	- Suppose the pipeline should run once a day.
	- What should happen if the pipeline fails one day? Should tomorrow's run proceed? Should yesterday's run be retried?
	- Should both happen?
- **Scheduling**:
	- In Airflow, scheduling primary answers **when a DAG should run**.
		- This can be Every day/week/month at a certain time or it can be once every hour/day/week.
		- When the schedule time arrives, Airflow creates a **DAG run**. The DAG is the blueprint. The DAG run is the execution of the blueprint.
	- When a failure occurs, Airflow retries the execution instead of failing the entire pipeline immediately. No manual intervention is required.
	- Retries don't happen immediately. Airflow gives the downstream service time to recover before attempting a retry.
		- Due to the possibility of retries, it's important to consider **idempotency** in each step of an Airflow DAG to eliminate the possibility of duplicate effects, such as loading the same data more than once.
		- When airflow retries tasks in a DAG, it does not retry the entire DAG. It only retries failed tasks.
	- After a certain number of retries, Airflow gives up. This is because repeated failures indicate a **permanent** failure, not a **transient** failure.
- **Catchup**:
	- Enabled: Suppose you create a DAG with a start date of July 1st and a daily schedule. If the DAG starts running on July 10th, it automatically creates runs for July 1st through July 9th in order to catch up.
	- Disabled: Airflow only creates a run for the current day.
	- Catchup is useful in scenario where missing data due to transient outages is unacceptable, such as processing financial transactions.
- **Backfill**:
	- Suppose a bug corrupts data for a DAG run on March 15th. A backfill is performed when you intentionally rerun the data for that day with the updated logic.
	- It's the **manual** reprocessing of updated data.
- **Catchup vs. Backfill**:
	- A catchup is the **automatic** scheduling of missed intervals.
	- A backfill is the **intentional** rerun of historical intervals.

### Tradeoffs
- More Retries:
	- Pros:
		- Recover from temporary failures.
		- Less manual interventions.
	- Cons:
		- Longer pipeline execution.
		- Delayed failure notification.
		- Wasted resources for permanent failures.
- Catchup Enabled:
	- Pros:
		- Complete historical data.
		- No missing intervals.
	- Cons:
		- Can overwhelm downstream systems.
		- Many historical DAG runs may execute simultaneously.
- Catchup Disabled:
	- Pros:
		- Simpler.
		- Good for non-historical workflows.
	- Cons:
		- Missed runs remain missing **unless manually backfilled**.

### Mental Model
- Connection to Previous Topics:
	```
	Retries
	      │
	      ▼
	Transient Failures
	      │
	      ▼
	Idempotent Tasks
	      │
	      ▼
	Catchup
	      │
	      ▼
	Backfills
	      │
	      ▼
	Reliable Pipelines
	```

| Earlier Topic            | Airflow Equivalent |
| ------------------------ | ------------------ |
| Kafka retries            | Task retries       |
| Kafka idempotency        | Idempotent tasks   |
| Checkpointing            | DAG/task state     |
| At-least-once processing | Task reruns        |
| Fault tolerance          | Retry & recovery   |

### Common Interview Questions
1. Why should Airflow tasks be idempotent?
> 	Airflow may retry failed tasks or rerun historical DAGs during catchup or backfills. If a task isn't idempotent, rerunning it could produce duplicate or inconsistent data. Designing tasks to be idempotent makes retries and historical reprocessing safe.
2. Your Airflow DAG runs every night to load sales data into Snowflake. The Snowflake task fails because Snowflake is temporarily unavailable. How would you configure retries? Why is task idempotency important in this situation? What could happen if the load task simply performed `INSERT` statements on every retry?
> 	I would configure the retries to use exponential backoff and fail after 5 attempts. This would give Snowflake time to recover, instead of hammering it with repeated retry requests while the service is unavailable. Failing after a finite number of retries prevents wasting resources on excessive retries. Exponential backoff increases the delay between each successive retry, increasing the likelihood of success. Idempotency is important is important in this situation because failing to implement it correctly could result in loading duplicate data. A `MERGE` would be a more appropriate implementation than an `INSERT`.
3. Your daily reporting DAG has been disabled for five days while the team fixed a bug. After deploying the fix, a teammate enables **catchup**. Another teammate suggests running a **backfill** instead. What is the difference between catchup and backfill? In what situations would you choose one over the other?
> 	The primary difference between a catchup and a backfill is that a catchup is automatic, while a backfill is manual. A catchup would be appropriate if you wanted to rerun the DAG for the entire five-day period automatically. A backfill would be appropriate if you only wanted to rerun the DAG for a specific day or days during the outage period.
4. Would you always enable catchup?
> 	Not necessarily. Business requirements determine which implementation is correct. For situations such as processing, financial transactions, missing data is unacceptable and catchup would be an appropriate choice to ensure all data is processed. For situations such as daily email delivery, where missing data isn't as critical, a backfill may be a more appropriate implementation.

## Sensors

- Many people think Airflow is only used to **execute and orchestrate** tasks. In reality, many workflows spend just as much time **waiting** as they do **working**. That's exactly what sensors a re for.
- Imagine the following pipeline:
	```
	Vendor uploads CSV
	
	↓
	
	Spark Transformation
	
	↓
	
	Snowflake Load
	
	↓
	
	Dashboard Refresh
	```
	- How do you know when the vendor uploaded the file?
- A **Sensor** is a special Airflow task that waits for a condition to become true before allowing downstream tasks to execute.
	- Examples:
	- File exists in S3
	- Kafka topic receives data
	- Database table is updated
	- Another DAG finishes
	- HTTP endpoint becomes available
- **Common Sensor Types**:
	- `FileSensor`: Waits for a file upload. For example, a CSV file being uploaded to an S3 bucket.
	- `ExternalTaskSensor`: Waits for another DAG. For example, if there was a `MarketingPipeline` and a `CustomerPipeline`, marketing data depends on customer data. Instead of guessing when the `CustomerPipeline` finishes, wait for it.
	- `SQLSensor`: Waits until a query returns a desired result. For example:
		```sql
		SELECT COUNT(*)
		FROM sales;
		```
		- The censor could only allow the task to continue when `COUNT > 0`.
	- `HTTPSesnor`: Waits for an API.
- **Sensor Modes**:
	- Poke Mode: Sensor periodically checks on the status of a condition.
		- **Continues occupying** the worker.
		- Good for short waits.
	- Reschedule Mode: Sensor periodically checks on the status of condition.
		- **Releases** the worker in between checks.
		- Good for long waits.
		- Allows the worker to perform other tasks while waiting.

### Tradeoffs
- Pros:
	- Enables event-driven workflows.
	- Avoids unnecessary failures.
	- Better resource untilization.
	- More reliable pipelines.
- Cons:
	- Poorly configured sensors can delay workflows.
	- Long waits can consume resources (when using Poke Mode).

### Mental Model
- Connection to Previous Topics:
	```
	Vendor Upload
	
	↓
	
	Sensor
	
	↓
	
	Spark
	
	↓
	
	Snowflake
	
	↓
	
	Dashboard
	```
	- The DAG still defines dependencies.
	- The Sensor simply introduces a dependency on an **external event** instead of another Airflow task.

|Spark/Kafka Concept|Airflow Equivalent|
|---|---|
|Backpressure|Waiting for prerequisites|
|Retries|Task retries|
|Resource utilization|Reschedule mode|
|Operational efficiency|Sensors instead of failed retries|

### Common Interview Questions
1. Why use a Sensor instead of scheduling a job every hour?
> 	A Sensor allows the workflow to wait for a specific condition, such as a file arriving or another pipeline completing, before executing downstream tasks. This avoids unnecessary failures, retries, and wasted compute resources that would occur if jobs ran before their prerequisites were ready.
2. Your pipeline processes a daily CSV file uploaded by a third-party vendor. The vendor usually uploads the file around **2:00 AM**, but occasionally it's delayed until **4:00 AM**. Would you schedule the Spark job for a fixed time or use a Sensor? Which type of Sensor would you choose? Would you configure it in **poke** mode or **reschedule** mode? Why?
> 	Since the daily file upload arrives at varying times, scheduling a Spark job would not be efficient. Using a File Sensor to detect a file upload would allow the Spark job to execute when the file actually arrives, instead of attempting an execution and hoping it has arrived. This also reduces end-to-end latency because the Spark job can start as soon as the file arrives rather than waiting until the next scheduled execution. Since the arrival times vary over a couple of hours instead of a couple of minutes, using Reschedule mode would be ideal since it frees workers to work on other tasks in between status checks.
3. A teammate suggests replacing every Sensor with repeated retries: "If the file isn't there, just let the Spark task fail and retry until it eventually succeeds." Would you agree? What are the operational drawbacks of using retries instead of Sensors in this situation?
> 	I would not agree. Repeated retries waste resources and can potentially overwhelm downstream systems. Using Sensors is a much better approach because they wait for a condition to be true before executing a task. Sensors are especially efficient when used in Reschedule Mode because they release the worker to perform other tasks in between resources. Repeated retries also leads to noisy alarms and can give the false impression that Spark is unstable. This reduces adequate observability.
4. Would you ever use a Sensor to wait for five days?
	- Probably not. This is primarily a workflow design issue. If you're waiting 5 days, you should try the following instead:
		- Trigger the DAG when the event occurs.
		- Use an event-driven architecture.
		- Break the workflow into separate DAGs.
		- Rethink the dependency.
	- Sensors are great for waiting on expected external events—but extremely long waits often indicate a different orchestration strategy would be better.

## Cross Communications (XComs)

- Suppose Task A downloads a file:
	```
	Download File
	
	↓
	
	Transform File
	```
	- How does the Transform task know:
		- where the file was downloaded?
		- what its name is?
		- whether it was compressed?
	- Hardcoding these values is impractical.
	- A better approach would to allow tasks to communicate with one another.
- XComs allow one Airflow task to pass **small pieces of information** to another task. Examples include:
	- File name
	- S3 path
	- Row count
	- Record ID
	- Timestamp
	- Status flag
- XComs should **not** be used to share large amounts of data between tasks. They are stored in Airflow's metadata database and are intended for **metadata** not datasets.
	- For example, a Spark job writes a dataset to S3 in Parquet format. An Extract task can share the S3 file location with a Transform task so it can load the file and perform its work.
	- Other bad examples of data that should not be stored in XComs are Spark DataFrames. This is what storage systems are meant for.
- XComs should not be used for **everything** because small amounts of data across hundreds of DAGS and thousands of runs can quickly add up, overwhelming Airflow's metadata database.
	- This can have the following negative consequences:
		- Queries slow down.
		- Scheduling slows down.
	- XComs should remain lightweight and should only be used when necessary.

### Tradeoffs
- Pros:
	- Easy task communication.
	- Dynamic workflows.
	- Lightweight metadata.
- Cons:
	- Not suitable for large data.
	- Stored in a metadata database.
	- Excessive use hurts Airflow performance.

### Mental Model
- Connection to Previous Topics:
	```
	Spark
	
	↓
	
	Write Parquet
	
	↓
	
	S3
	```
	- In this pipeline, Airflow uses XComs to pass the S3 URL between tasks, not the Parquet file.
	- The Parquet file is written to and stored in S3. It isn't passed between tasks.

### Common Interview Questions
1. Can I pass a Spark DataFrame using XCom?
> 	Technically some Airflow configurations can serialize complex objects, but it's not a recommended design. XComs are intended for small pieces of metadata. Large datasets should be stored in systems like S3, HDFS, or a database, while the XCom contains only a reference such as a file path or table name.
2. Your Spark task writes a **2 TB Parquet dataset** to S3. The next task needs to process that dataset. What would you pass through an XCom? What would you store in S3? Why?
> 	I would pass the S3 URI throw XCom so the next task can independently retrieve the file from S3. The previous task should write the data to S3 instead of storing it in Airflow's metadata database. Large XComs increase the size of Airflow's metadata database, which can degrade scheduler performance, slow the UI, and make the orchestration platform itself less reliable.
3. A teammate proposes storing an entire pandas DataFrame in an XCom because: "It avoids writing temporary files." Would you agree? What operational problems could this create? What architecture would you recommend instead?
> 	I would not agree. While storing it in XCom might avoid writing a temporary file, the DataFram would still need to be stored in Airflow's metadata database. Depending on the size of the DataFrame, this could have a significant impact on job performance. Instead of passing the DataFrame to the next task through XCom, one task should write the DataFrame to storage while the other reads it from storage. This also increases fault tolerance since storage locations such as S3 are less volatile than a metadata database.
4. If all you're passing around are file paths, why use XCom at all? Why not hardcode the S3 path?
> 	Hardcoding reduces flexibility. File names often include execution dates, partition values, or unique identifiers generated at runtime. XComs allow upstream tasks to communicate those dynamically generated values to downstream tasks without tightly coupling the workflow.

## Executors

- Suppose you have a DAG with three tasks: Task A, Task B, and Task C. Which component runs the tasks?
	- The scheduler decides **when** to run tasks.
	- The executor **executes** tasks.
- An Executor determines:
	- Where tasks run.
	- How many tasks run simultaneously.
	- How work is distributed.
- **Sequential Executor**:
	- This is the simplest type of executor.
	- Only one task executes at a time.
	- Useful mostly for:
		- Development
		- Learning Airflow
		- Local testing
	- Not useful in production because it eliminates parallelism, significantly increasing execution time.
- **Local Executor**:
	- Capable of running multiple tasks simultaneously on the same machine.
	- Much better resource utilization.
	- Useful mostly for:
		- Small production deployments
		- Medium-sized teams
		- Moderate workloads
- **Celery Executor**:
	- Distributes tasks across multiple workers.
	- Allows Airflow to scale horizontally.
	- Benefits:
		- Useful when one worker is busy, but another one is capable of executing a pending task.
		- Increases scalability.
	- Tradeoffs:
		- Requires:
			- Celery
			- Message broker (Redis/RabbitMQ)
			- Multiple worker nodes.
		- More infrastructure overhead.
- **Kubernetes Executor**:
	- Instead of permanent workers, each task gets its own container.
	- When a task is finished, the container disappears.
	- Benefits:
		- Excellent isolation
		- Independent scaling
		- Allows for different dependencies per task
	- Tradeoffs:
		- Requires Kubernetes.
		- Higher operational complexity.
		- Container startup latency.

### Comparing Executors
- Sequential:
	- Pros:
		- Simple
		- Easy setup
	- Cons:
		- No parallelism
- Local:
	- Pros:
		- Parallel tasks
		- Simple infrastructure
	- Cons:
		- Limited to one machine
- Celery:
	- Pros:
		- Horizontal scaling
		- Mature
		- Ideal for production workloads
	- Cons:
		- More infrastructure
		- Worker management
- Kubernetes:
	- Pros:
		- Maximum scalability
		- Isolation
		- Elastic resources
	- Cons:
		- Most operational complexity
		- Requires Kubernetes expertise

### Choosing an Executor
- Small ETL team:
	- Sequential would be too limiting.
	- Local is probably enough.
- Hundreds of concurrent DAGs:
	- Local becomes a bottleneck.
	- Celery or Kubernetes make more sense.
- Need different software stacks per task:
	- Kubernetes has a major advantage.

### Mental Model
- Connection to Previous Topics:
	- Spark
		```
		Driver
		
		↓
		
		Executors
		```
	- Airflow
		```
		Scheduler
		
		↓
		
		Executor
		
		↓
		
		Workers
		```
	- Different technologies.
	- Similar architectural idea: A coordinator distributes work to execution resources.

### Common Interview Questions
1. Why would a company move from the Local Executor to the Celery Executor?
> 	The Local Executor is limited by the resources of a single machine. As the number of concurrent DAGs and tasks grows, a distributed executor like Celery allows Airflow to distribute work across multiple workers, increasing throughput and improving fault tolerance.
2. Your company currently runs Airflow with the **Local Executor**. Over the past year, the number of DAGs has increased significantly. During peak hours:
	- Hundreds of tasks are queued.
	- CPU utilization on the Airflow machine stays near 100%.
	- Other machines in the cluster are mostly idle.
	- Would changing executors help? If so, which executor would you recommend? Why? What tradeoffs would you discuss?
> 	Switching to the Celery executor in this scenario would significantly improve parallelism, as the Celery executor is capable of distributing tasks across multiple machines. Those symptoms suggest the bottleneck isn't Airflow scheduling—it's that task execution is limited to a single machine. Since there are unused resources elsewhere in the cluster, moving to the Celery Executor allows Airflow to utilize those machines. While the Celery executor is ideal for workloads that require horizontal scaling, it requires more infrastructure and worker management than a Local executor.
3. A teammate says: "The Kubernetes Executor is always the best choice because Kubernetes is modern." Would you agree? What factors would influence your choice between the Celery Executor and the Kubernetes Executor?
> 	While Kubernetes is modern, this does not universally make the best choice for all workloads. The right executor depends on the business requirements. Kubernetes should only be chosen over Celery when the business requires greater scalability than Celery can provide and/or needs to run workloads with varying dependencies.
4. If Kubernetes is more scalable, why do many companies still use the Celery Executor?
> 	Because scalability isn't the only consideration. Kubernetes introduces additional operational complexity and requires expertise in managing a Kubernetes cluster. For many organizations, the Celery Executor provides sufficient scalability with a simpler operational model. If the workload doesn't require per-task container isolation or elastic scaling, Kubernetes may not provide enough additional value to justify the added complexity.
	- Technology choices are **business decisions**, not popularity contests.

## Troubleshooting and System Design

- Supposed you get paged and the alert says: "The daily ETL pipeline is delayed."
	- The mistake many engineers make is trying to **immediately solve the issue**, instead of properly diagnosing it first.
	- Experienced engineers first ask: "Where is the bottleneck?"
- **Scenario 1: Tasks Stuck in Queued**:
	- You open the Airflow UI and see:
		```
		Scheduled
		
		↓
		
		Queued
		
		↓
		
		Queued
		
		↓
		
		Queued
		
		↓
		
		Queued
		```
		- Nothing is actually running.
		- "Queued" means the scheduler wants the task to run, but the executor hasn't started it yet.
	- Possible Causes:
		- No Available Workers:
			- New tasks must wait.
			- Metrics:
				- Worker utilization
				- Running task count
				- Queue length
		- Executor Capacity:
			- When using a Local Executor, queued tasks are expected.
			- Once CPU and/or memory become a bottleneck, the executor can't start additional work.
		- Resource Limits:
			- Kubernetes refuses to create new pods.
			- Celery workers have reached their concurrency limits.
	- Investigation Strategy: Check The Following:
		1. Scheduler healthy?
		2. Executor healthy?
		3. Worker utilization?
		4. Queue length?
		5. Infrastructure limits?
- **Scenario 2: Task Keeps Failing After Retries**:
	- This is typically **not** indicative of a simple transient failure. It's more likely:
		- Bad credentials
		- Code bug
		- Invalid SQL
		- Missing file
		- Schema mismatch
	- Retries won't fix this.
	- Investigation Strategy: Check The Following:
		1. Task logs
		2. Error messages
		3. Deployment history
		4. Recent configuration changes
- **Scenario 3: One DAG Starves Everything Else**:
	- Imagine if one huge DAG started, forcing all of the other smaller DAGs to wait for resources to free up.
	- Possible Solutions:
		- Pools
		- Priority weights
		- More workers
		- Separate queues
	- The ultimate goal is to prevent one pipeline from monopolizing resources.
- **Scenario 4: Thousands of DAG Runs Suddenly Appear**:
	- Investigation Strategy: Check The Following:
		- DAG start date
		- Catchup setting
		- Recent deployment changes
- **Scenario 5: Sensors Exhaust Workers**:
	- Suppose your DAG is configured with 500 `FileSensor`s that each wait for 6 hours in Poke Mode.
	- These sensors occupy all of the workers and nothing runs.
	- A better solution would be to use Reschedule Mode.
- **Scenario 6: Scheduler Becomes Slow**:
	- Symptoms:
		- DAGs appear late
		- UI becomes sluggish
		- Metadata queries slow
	- Investigation Strategy: Check The Following:
		- Huge XComs?
		- Millions of task instances?
		- Large metadata database?
		- Remember:
			- Airflow is an orchestrator, not a data warehouse.
- **End-to-End Troubleshooting**:
	- Suppose:
		```
		Kafka
		
		↓
		
		Spark
		
		↓
		
		Parquet
		
		↓
		
		Snowflake
		
		↓
		
		Dashboard
		```
		- If the dashboard is stale, where is the issue?
		- It could be:
			- Airflow
			- Spark
			- Kafka
			- Snowflake
			- S3
			- Network
		- Airflow is only designed to **report failure**. Airflow itself is almost never the issue.
- **System Design Example**:
	- Suppose you're designing a daily pipeline with the following requirements:
		- Download vendor files.
		- Transform with Spark.
		- Load Snowflake.
		- Run dbt.
		- Notify analysts.
	- A good Airflow design might look like:
		```
		FileSensor
		      │
		      ▼
		Download
		      │
		      ▼
		Spark Transform
		      │
		      ▼
		Load Snowflake
		      │
		      ▼
		Run dbt
		      │
		      ▼
		Notify
		```
	- Design Choices:
		- FileSensor in **reschedule** mode.
		- Spark, Snowflake, and dbt as separate tasks.
		- Idempotent loads using `MERGE`.
		- XComs for file paths and row counts.
		- Exponential backoff on retries.
		- Catchup enabled only if historical processing is required.

### Mental Model
- Connection to Previous Topics:
	```
	Vendor
	     │
	     ▼
	FileSensor
	     │
	     ▼
	Airflow DAG
	     │
	     ▼
	Spark
	     │
	     ▼
	S3
	     │
	     ▼
	Snowflake
	     │
	     ▼
	dbt
	     │
	     ▼
	Dashboard
	```
	- Airflow orchestrates **every step**, but owns very little data itself.
- Think of Airflow as an airport control tower. It doesn't fly planes. It coordinates:
	- When planes take off.
	- Which runway they use.
	- What must happen before landing.
	- How to respond when weather causes delays.
- If flights are delayed, the tower isn't always the cause—it may simply be coordinating around problems elsewhere in the system.

### Common Interview Questions
1. How do you troubleshoot an Airflow pipeline?
> 	I start by identifying where execution has stopped. If tasks are queued, I investigate the executor, worker capacity, and infrastructure. If tasks are failing, I examine logs to determine whether the failure is transient or permanent. I also verify dependencies, recent deployment changes, downstream system health, and scheduler behavior before deciding whether the issue is with Airflow itself or another component of the pipeline.
2. Your Airflow UI shows hundreds of tasks in the **Queued** state, but very few are actually running. Walk me through your investigation. What components would you check? What possible root causes would you consider? How would your investigation differ if you were using the **Local Executor** versus the **Celery Executor**?
> 	First, I'd verify that the Airflow scheduler is healthy and actively scheduling tasks before focusing on the executor. Next, I would check how many available workers there were. If there were no available workers, I'd look at task execution time and try to determine if the tasks are running efficiently. If tasks were taking significantly longer than usual, I'd look at the underlying logic to try and identify bottlenecks. If task performance wasn't an issue, I'd consider adding more workers. Next, I'd check executor capacity. If a Local Executor was being used, then queued tasks must wait. If a Celery executor was being used, I'd check if workers have reached their concurrency limits.
	- The scheduler typically isn't the issue, but it's still worth checking before diving deeper.
3. A team reports:
	- The DAG is slow.
	- Spark jobs are healthy.
	- Kafka has no consumer lag.
	- Snowflake queries are fast.
	- Airflow's metadata database has grown dramatically over the past year.
	- Many tasks pass large objects through XComs.
	- What is your leading hypothesis? What changes would you recommend? Why is this an example of violating Airflow's intended architecture?
> 	My leading hypothesis would be that Airflow is being slowed down by excessive metadata overhead, due to the large objects being passed through XComs. I would recommend one task writing these large objects to a storage layer, sending the location to the next task through XComs, and having that task read the object from the storage layer. This violates Airflow's intended architecture because Airflow is primarily an orchestrator that can manage reasonable amounts of data needed to coordinate tasks. It is not meant to act as a storage layer.
4. If the metadata database is already huge, is deleting old XComs enough?
	- Not necessarily. Deleting old XComs helps, but the following should also be investigated:
		- Why are we generating huge XComs?
		- Why are we storing datasets in Airflow?
		- Can we redesign the pipeline?
	- Deleting XComs solves the symptoms, not the disease.