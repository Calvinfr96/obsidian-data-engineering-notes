## Simple Framework

1. **Clarify the requirements** (functional and non-functional).
2. **Identify the data sources** and classify them as streaming or batch.
3. **Design the ingestion layer** for each type of source.
4. **Store data in a layered data lake** (Bronze → Silver → Gold).
5. **Describe transformations** and how data flows between layers.
6. **Enable analytics and ML** for different user groups.
7. **Address governance, security, monitoring, and orchestration.**
8. **Explain how each design decision solves a business problem.**
	- Can be done while doing steps 1 - 7. You don't need to wait until the end.

## Step 1: Don't jump into drawing

- The biggest mistake candidates make is immediately saying: "We'll use S3, Glue, Kinesis..."
	- Interviewers don't care about the solution yet.
	- **Start by understanding the problem and clarifying requirements**.
		- Just like in an SQL interview where you ask clarifying questions **first**, do the same in a System Design interview.

## Step 2: Break the architecture into layers

- Never think about individual AWS services first.
- Think about **layers** first.
- A good answer looks like:
	```
	Data Sources
	
	↓
	
	Ingestion
	
	↓
	
	Storage
	
	↓
	
	Transformation
	
	↓
	
	Analytics
	
	↓
	
	Monitoring/Security
	```

## Step 3: Identify the data sources

- Identify the data sources given in the problem statement.
- Once all of the data sources are identified categorize them. The two main types of sources are:
	- Streaming sources
	- Batch sources
- Depending on how you classify them, you'll need **one or more** ingestion patterns.
- Example Sources:
	- Point of Sail (POS)
	- Mobile App
	- E-Commerce
	- Enterprise Resource Planning (ERP)
	- Customer Relationship Management (CRM)
- Example Classifications:
	- Streaming (Near Real-Time):
		- POS
		- Mobile App
		- E-Commerce
	- Batch (Daily Exports)
		- ERP
		- CRM

## Step 4: Design the ingestion layer

- Separate batch ingestion ingestion from streaming ingestion if there is more than one type of source.
- Streaming Example:
	```
	POS
	App
	E-Commerce
	
	↓
	
	API Gateway
	
	↓
	
	Lambda
	
	↓
	
	Kinesis
	
	↓
	
	Firehose
	```

## Step 5: Design the batch layer

- Batch Example:
	```
	ERP
	
	↓
	
	Glue
	
	↓
	
	S3
	```

## Step 6: Design the data lake

- Don't dump all of the data into one folder.
- Use the Medallion architecture to separate data based on its purpose and level of transformation.
- Bronze: Raw data
	- Replay
	- Debugging
	- Auditing
- Silver: Cleaned data
	- Remove duplicates
	- Normalize schema
- Gold: Business ready
	- Create analytical dashboards and reports

## Step 7: Define the analytics layer

- Note the possible analytics tools you can use to utilize the processed data.
- Choose the best one and justify the decision **based on business requirements**.

## Step 8: Define governance

- State how the pipeline will perform data governance tasks such as:
	- Encryption
	- Masking
	- RBAC
	- Auditing

## Step 9: Define orchestration

- Define how the pipeline will use automated orchestration to prevent unnecessary human intervention.

## Step 10: Define CI / CD

- Define which tools will be used to enable continuous integration and deployment.
- This ensures the pipeline is maintained **through code**, not manually.

## Step 11: Tie the architecture decisions back to the business problems

- This is a step many candidates forget.
- Every technology should map back to a business requirement. It shouldn't just be a "nice to have."

## Step 12: Explain tradeoffs

- If multiple technologies were considered for the same business requirement, explain the tradeoffs of using one vs the other.