## Threads

- Suppose you have a Java application that needs to:
	- Read a file
	- Call an external API
	- Write to a database
- Should the application perform these tasks sequentially? Not necessarily.
- A **thread** is the smallest unit of execution within a process.
	- Think of a process as a workshop. A thread would represent an individual worker.
	- One process can have multiple threads running **simultaneously**, just as a workshop can have multiple workers doing their jobs simultaneously.
- Example:
	```
	Process: Order Service
	
	├── Thread 1 → Handle Request A
	├── Thread 2 → Handle Request B
	├── Thread 3 → Write Logs
	└── Thread 4 → Background Cleanup
	```
	- All of these threads **share the same process** memory.
- Imagine an API request:
	```
	Receive Request
	↓
	Call Database (100 ms)
	↓
	Return Response
	```
	- During that 100 ms database call, the CPU is mostly waiting.
	- A single-threaded application can't do much else.
	- With multiple threads executing simultaneously, the CPU stays busy.
- **Threads vs. Processes**:
	- Processes:
		- Separate memory
		- More isolated
		- More expensive to create
		- Communication is slower
	- Threads:
		- Share memory
		- Faster communication
		- Lower overhead
		- Require synchronization
	- The shared memory is both the advantage **and** the danger. For example:
		- Suppose two threads share a simple variable: `int counter = 0;`
		- Thread A: `counter++;`
		- Thread B: `counter++;`
		- Without proper **synchronization** between threads, the result could be 1 instead of 2 because both threads may read the same original value before either writes the incremented result. This is an example of a **race condition**.
- **CPU-Bound vs. I/O-Bound Work**:
	- CPU-Bound:
		- Performance depends on CPU resources.
		- Adding more threads than CPU cores often provides little benefit because threads compete for the same processors.
		- Examples:
			- Image processing
			- Compression
			- Encryption
			- Heavy Calculations
	- I/O-Bound:
		- Performance is limited by waiting for external systems.
		- Multiple threads can improve throughput because while one thread waits, another can do useful work.
		- Examples:
			- Database queries
			- REST API calls
			- Reading files
			- Network requests
- **Thread Pools**:
	- Creating thousands of threads is expensive. Instead, most applications use a **thread pool**:
		```
		Incoming Requests
		
		↓
		
		Thread Pool
		
		↓
		
		Worker Threads
		```
	- Threads are reused instead of constantly created and destroyed.
	- Java's `ExecutorService` is a common example.
- **Thread Safety**:
	- A class is considered **thread-safe** if multiple threads can use it concurrently without corrupting shared state.
	- Common techniques include:
		- Immutable objects
		- Locks
		- Concurrent collections
		- Atomic operations

### Tradeoffs
- More Threads:
	- Pros:
		- Better utilization for I/O-bound workloads
		- Higher throughput
	- Cons:
		- Context-switch overhead
		- More memory usage
		- Synchronization complexity
		- Increased risk of race conditions and dead locks
- Fewer Threads:
	- Pros:
		- Simpler code
		- Lower overhead
	- Cons:
		- Lower throughput for I/O-bound workloads
		- Idle CPU while waiting for external systems

### Common Interview Questions
1. Does adding more threads always improve performance?
> 	No. It depends on the workload. For I/O-bound tasks, additional threads can improve throughput because some threads spend time waiting on external systems. For CPU-bound workloads, adding significantly more threads than available CPU cores often leads to increased context switching with little or no performance benefit.
2. Your application processes uploaded images. Each image requires:
	- Reading the file from cloud storage.
	- Applying an expensive image transformation.
	- Uploading the result back to storage.
	- Which parts of this workflow are **I/O-bound**, and which are **CPU-bound**? Would increasing the number of threads improve performance equally across all stages? Why or why not?
> 	Reading the file from cloud storage and uploading the result back to storage are I/O-bound while applying expensive image transformations is CPU-bound. Adding threads would primarily increase performance for the I/O-bound processes, since raw images can be read from storage while processed images are written back to storage. If each thread processes an independent image, synchronization is minimal because there is very little shared state. Synchronization becomes important only when threads need to coordinate access to shared resources.  Adding threads would also increase performance for the image transformation, but only up to a certain point. If more threads are added than there are CPU cores, threads compete for reasources and context switching adds additional strain on CPU.
	- If each thread is processing a **different image**, they usually don't share mutable state. No synchronization is needed because each thread owns its own work. Synchronization only becomes necessary when threads **share** resources, such as a cache, a queue, or a counter variable. 
3. A teammate says: "Our server has 8 CPU cores, so we should always run exactly 8 threads." Would you agree? How would your answer differ for:
	- A web API that spends most of its time waiting on databases?
	- A service performing heavy video encoding?
	- What tradeoffs would you discuss?
> 	A web API that spends most of its time waiting on databases would benefit from multiple threads, since multiple requests could be made concurrently, instead of sequentially. Video encoding is CPU-bound. I would generally size the thread count close to the number of available CPU cores. Adding significantly more threads than cores usually increases context-switch overhead without providing meaningful additional throughput.
4. If threads share memory, why not just create one thread per request?
	- Threads aren't free. Each thread consumes:
		- Memory (stack space)
		- Scheduling overhead
		- Context switching
	- Thousands of threads can overwhelm a server.
	- This is one reason modern servers often use:
		- Thread pools
		- Async I/O
		- Event loops

## Async Processing

- Threads and asynchronous processing are similar and often work together, but they solve different problems.
- Suppose a web service receives a request:
	```
	Receive Request
	
	↓
	
	Call Database
	
	↓
	
	Return Result
	```
	- The database takes 200 ms to respond.
	- What is the CPU doing during those 200 ms? Pretty much zero. It's waiting.
- Traditional synchronous code looks like:
	```
	Call Database
	
	↓
	
	(wait...)
	
	↓
	
	Process Result
	
	↓
	
	Return Response
	```
	- Nothing else happens until the database responds.
	- The thread is **blocked**.
- Asynchronous code allows threads to perform other work while waiting:
	```
	Start Database Request
	
	↓
	
	Continue Doing Other Work
	
	↓
	
	Database Finishes
	
	↓
	
	Handle Result
	```
	- The thread doesn't spend 200 ms waiting. It can work on something else.
- Async programming is primarily about **not blocking while waiting**. It's not about using more CPU cores. That's what threads are for.
- **Threads vs. Asynch**:
	- Threads allow multiple pieces of code to execute simultaneously. They are useful for:
		- CPU work
		- Parallelism
	- Async processesing allows one thread to efficiently handle many waiting operations. This is useful for:
		- I/O
		- Network requests
		- Databases
	- Threads answer "How can I use more workers?"
	- Async processing answers "How can my workers avoid standing around waiting?"
- **Event Loop**:
	- Used by many async systems.
	- One thread. Many requests. No blocking.
	- Example:
		```
		Receive Request
		
		↓
		
		Start Database Query
		
		↓
		
		Start API Call
		
		↓
		
		Handle Another Request
		
		↓
		
		Database Completes
		
		↓
		
		Resume Processing
		```
- **Futures / Promises**:
	- Suppose you request `Customer Data`. Instead of waiting, you receive `Future<Customer>` or `Promise<Customer>`.
	- The result will arrive later. Your code continues running.
- **Async in Modern Languages**:
	- Python: `await database_query()`
	- Java: `CompletableFuture`
	- JavaScript: `async / await`
	- Different syntax. Same concept.
- Async doesn't always mean faster:
	- Suppose you're computing prime numbers. That's CPU work.
	- Making it async doesn't make the CPU faster.
	- Async mainly improves utilization while waiting on I/O.
- Connection to Kafka:
	- Kafka sends messages, continues processing, and waits for acknowledgements.
	- This is asynchronous and improves throughput.
- Connection to Airflow:
	- Sensors (Reschedule Mode) allow tasks to be conducted asynchronously.

### Tradeoffs
- Synchronous:
	- Pros:
		- Simple
		- Easy debugging
		- Easy reasoning
	- Cons:
		- Threads block
		- Lower throughput
- Asynchronous:
	- Pros:
		- Excellent for I/O
		- High throughput
		- Better resource utilization
	- Cons:
		- More complex
		- Harder debugging
		- Harder error handling
- When to Use Aync:
	- Good Candidates:
		- Database calls
		- HTTP requests
		- Reading files
		- Kafka producers
		- Cloud APIs
	- Poor Candidates:
		- Heavy image processing
		- Encryption
		- Video encoding
		- Matrix multiplication

### Common Interview Questions
1. When would you choose asynchronous programming instead of simply adding more threads?
> 	I'd use asynchronous programming for I/O-bound workloads where threads spend significant time waiting on external systems such as databases or network calls. Async allows a thread to continue doing useful work instead of blocking. For CPU-bound workloads, additional threads or processes are generally more appropriate because the goal is to increase parallel computation rather than hide waiting time.
2. Your service needs to retrieve data from:
	- A customer database
	- A payment service
	- A shipping service
	- Each request typically takes **150 ms**, and the requests are independent. Would you implement these calls synchronously or asynchronously? Why? How would that affect response time and resource utilization?
> 	I would implement these calls asynchronously because doing so frees the thread to perform other useful work rather than remaining blocked while waiting for the external service. This would significantly improve response time since each request can be made in parallel, instead of sequentially. Additionally, if any other required processes finish in less than 150 ms, then the service could potentially complete all other work while waiting for a response.
	- The thread doesn't necessary perform **computation** while waiting. It performs **other useful work**, such as serving other HTTP requests or processing other Kafka messages.
3. A teammate says: "Asynchronous programming is always faster than multithreading." Would you agree?Can you think of a workload where adding async provides little or no benefit, but adding CPU resources or worker threads would improve performance?How would you explain the difference?
> 	Asynchronous programming is only faster than multithreading for I/O-bound processes. CPU-bound process generally gain little to no benefit from asynchronous processing. Instead, performance improved by adding additional CPU resources or dividing existing CPU resources among multiple worker threads. Asynchronous processes wait for a response from an external service operating a different machine. Async improves scalability for I/O-bound workloads because threads don't remain blocked while waiting. Adding more threads can also improve throughput, but eventually thread creation, memory usage, and context switching become limiting factors. Async avoids much of that overhead.
	- Adding CPU or increasing thread count won't improve performance during the **waiting portion of an I/O operation**.
	- More threads **can** improve throughput for the other portions of the operation, but only up to a certain point.
	- Async often scales better because **one thread can manage many waiting operations**.
4. Can asynchronous code still use multiple threads?
	- Absolutely. Modern applications commonly use both. The two concepts aren't mutually exclusive.
	- Example:
		```
		Thread Pool
		
		├── Thread 1
		│      ├── Async DB Call
		│      ├── Async HTTP Call
		│      └── Async Kafka Send
		│
		├── Thread 2
		│      ├── Async DB Call
		│      └── Async API Call
		│
		└── Thread 3
		```
		- Each thread uses async to avoid blocking.
		- Multiple threads allow the application to utilize multiple CPU cores and handle more concurrent work.
		- Modern servers often combine:
			- Thread pools
			- Asynch I/O
			- Non-blocking networking

## Deadlocks

- **Race conditions** happen because there's **too little synchronization**.
- **Deadlocks** happen because there's **too much synchronization**.
- Suppose two threads need access to two shared resources:
	- Resource A: Cache
	- Resource B: Database Connection
	- Thread 1:
		```
		Acquire Lock A
		
		↓
		
		Acquire Lock B
		```
	- Thread 2:
		```
		Acquire Lock B
		
		↓
		
		Acquire Lock A
		```
	- Timeline (Thread 1):
		```
		Lock A ✓
		
		(waiting for Lock B...)
		```
	- Timeline (Thread 2):
		```
		Lock B ✓
		
		(waiting for Lock A...)
		```
	- Thread 1 can't continue until Thread 2 releases B.
	- Thread 2 can't continue until Thread 1 releases A.
	- Neither thread can make progress. This is a **deadlock**.
- A deadlock occurs when two or more threads permanently wait for one another to release resources.
	- Nothing crashes.
	- Nothing throws an exception.
	- Everything simply stops making progress.
- Deadlocks are so dangerous because applications can appear perfectly healthy while trapped in one.
	- The CPU utilization may be low, but the threads are stuck waiting indefinitely.
	- This can cause production systems to become partially or completely unavailable.

### Four Conditions for a Deadlock
1. **Mutual Exclusion**: A resource can only be used by one thread at a time.
2. **Hold and Wait**: A thread holds one resource while waiting for another.
3. **No Preemption**: Resources cannot be forcefully taken away.
4. **Circular Wait**: Threads form a cycle where each waits for the next.
- Break any one of these conditions and the deadlock cannot occur.

### Preventing Deadlocks
1. **Acquire Locks in a Consistent Order**:
	- Suppose every thread is configured to do:
		```
		Lock A
		
		↓
		
		Lock B
		```
	- And never:
		```
		Lock B
		
		↓
		
		Lock A
		```
	- Now, the circular wait can't happen.
	- This is the preferred solution.
1. **Minimize Lock Scope**:
	- Don't hold locks longer than necessary.
	- Bad:
		```
		Acquire Lock
		
		↓
		
		Read Database
		
		↓
		
		Call API
		
		↓
		
		Write File
		
		↓
		
		Release Lock
		```
	- Good:
		```
		Acquire Lock
		
		↓
		
		Update Shared State
		
		↓
		
		Release Lock
		
		↓
		
		Call API
		```
	- Hold locks **only while accessing the shared resource**.
1. **Use Timeouts**:
	- Instead of waiting forever: `tryLock(5 seconds)`
	- If the lock isn't acquired:
		- Retry
		- Log an error
		- Backoff
	- The application continues instead of freezing.
2. **Reduce Shared State**:
	- Sometimes, the best lock is no lock.
	- These approaches reduce contention altogether:
		- Immutable objects
		- Message queues
		- Actor models
		- Partitioned ownership

### Deadlock vs. Race Condition
- **Race Condition**:
	- Problem: Two threads modify shared data simultaneously.
	- Result: Incorrect data.
- **Deadlock**:
	- Problem: Threads wait on each other forever.
	- Result: No progress.

### Modern Systems
- One reason technologies like Kafka are so popular is that they often reduce the need for shared mutable state.
- Instead of :
	```
	Thread A
	
	↓
	
	Shared Object
	
	↑
	
	Thread B
	```
- You get:
	```
	Producer
	
	↓
	
	Kafka
	
	↓
	
	Consumer
	```
- Threads communicate through messages instead of shared memory.
- This results in reduced need for locks and a lower potential for deadlocks.

### Tradeoffs
- Heavy Locking:
	- Pros:
		- Prevents race conditions
		- Protects shared state
	- Cons:
		- Contention
		- Deadlocks
		- Lower throughput
- Minimal Locking:
	- Pros:
		- Better scalability
		- Higher throughput
	- Cons:
		- Greater risk of race conditions if shared state isn't designed carefully.

### Common Interview Questions
1. How would you prevent deadlocks?
> 	I'd start by ensuring locks are always acquired in a consistent order to eliminate circular wait. I'd also keep critical sections as small as possible, avoid holding locks while performing slow operations such as network or database calls, and use lock timeouts or retry mechanisms where appropriate. When possible, I'd reduce shared mutable state altogether through immutable data structures or message-passing architectures.
2. Suppose two services both need to update:
	- A customer's account balance
	- Their transaction history
	- Each operation requires locking both resources. How would you design the locking strategy to minimize the risk of deadlocks? Would lock ordering help? Would timeouts be useful?
> 	To prevent deadlocks, I would design the two services to acquire the necessary locks in a consistent order to prevent circular waiting. I would also ensure each service only holds locks when necessary and set timeouts for how long a service waits for a lock. Finally, I would design the two services in a manner that reduced shared state as much as possible, using immutable objects and message queues. These strategies would help minimize the chance of a deadlock and the impact of a deadlock if one managed to occur.
3. A teammate says: "To prevent race conditions, let's just put one giant lock around the entire application." Would you agree? What problems could this create? How would you balance correctness with performance?
> 	Putting one giant lock around an entire application would be inefficient because it would greatly reduce the ability of multiple threads to operate concurrently. Striking the right balance between heavy and minimal locking involves designing a system to only use shared state when absolutely necessary. When shared state is necessary, ensuring threads acquire locks in a consistent order, only hold on to locks when necessary, and don't wait for locks indefinitely reduces the potential performance impacts of a deadlock.
4. If avoiding shared state is so beneficial, why don't we eliminate it completely?
	- Sometimes, shared state simply can't be eliminated.
	- Certain resources are inherently shared:
		- Account balances
		- Inventory counts
		- Configuration
		- Caches
		- Connection pools
	- The goal isn't to eliminate all shared state. It's to minimize it and synchronize access only where necessary.

## Observability

- Suppose it's 2:00 AM and your on-call pager goes off. The alert says:
	```
	API Latency
	
	↑
	
	4x Higher Than Normal
	```
	- Where do you start investigating?
	- Without good **observability**, you're guessing.
	- With good observability, you have the information needed to investigate.
- Observability is the ability to understand the internal state of a system by examining the information it produces.
- The core pillars of observability are:
	- Metrics
	- Logs
	- Taces
- **Metrics**:
	- Metrics answer: "Is something wrong?"
	- Examples:
		- CPU utilization
		- Memory utilization
		- Request latency
		- Error rate
		- Kafka consumer lag
		- Spark executor memory
		- Cache hit ratio
	- Metrics are numerical values over time.
- **Logs**:
	- Logs answer: "What happened?"
	- Example:
		```
		ERROR
		
		Failed to deserialize Kafka message
		
		Topic:
		Orders
		
		Partition:
		7
		```
	- Logs provide context. They're usually your first stop after an alert fires.
- **Traces**:
	- Traces answer: "Where is the time being spent?"
	- Suppose a user request flows through:
		```
		API
		
		↓
		
		Authentication
		
		↓
		
		Order Service
		
		↓
		
		Payment Service
		
		↓
		
		Database
		```
	- A trace shows:
		```
		API
		10 ms
		
		↓
		
		Auth
		5 ms
		
		↓
		
		Order
		30 ms
		
		↓
		
		Payment
		220 ms
		
		↓
		
		DB
		15 ms
		```
	- Now you immediately know the payment service is the bottleneck.
- **Putting Them Together**:
	- Metric: `Increasing latency`
	- Logs: `Database timeout`
	- Trace: `95% of time spent waiting on DB`
	- Using all three gives you the full picture of what the issue is. None of them alone can help solve a problem.
- **Golden Signals**:
	- Google's Site Reliability Engineering (SRE) book popularized four key metrics:
		- **Latency**: How long a request takes.
			- Example: P99 latency increasing from 45 ms to 200 ms.
		- **Traffic**: How much work the system is doing.
			- Examples:
				- Requests/sec
				- Messages/sec
				- Queries/sec
		- **Errors**: What is wrong with the system
			- Examples:
				- HTTP 500s
				- Failed Kafka writes
				- Failed Spark jobs
		- **Saturation**: How close the system is to capacity
			- Examples
				- CPU
				- Memory
				- Disk
				- Queue depth

### Observability Tools
- **Dashboards**:
	- Good dashboards combine the following metrics:
		```
		CPU
		
		Memory
		
		Latency
		
		Errors
		
		Traffic
		```
	- One screen. Fast diagnosis.
- **Alerts**:
	- Good alerts are **actionable**.
	- Good example: P99 Latency and Error Rate trends for the past 10 minutes.
	- Bad example: CPU > 20%
- **Composite Alarms**:
	- Composite alarms only alert when multiple metrics breach SLA concurrently.
	- Example: Instead of alerting on CPU **or** latency **or** memory, a composite alarm **combines** signals.
		- Page only if: High Latency **and** High Error Rate.
	- This results in fewer false alarms and more actionable alerts.
- **Log Search**:
	- Instead of manually reading 1000s of log lines, log search enables searching of logs using **structured querires**.
	- Example: Find:
		- All errors
		- Specific request IDs
		- A customer ID
		- a time window
	- This is what makes tools such as CloudWatch Log Insights, Splunk and ElasticSearch so powerful.
- **Distributed Tracing**:
	- Modern systems involve:
		```
		Gateway
		
		↓
		
		Service A
		
		↓
		
		Service B
		
		↓
		
		Database
		```
	- Without tracking, you only know how long the request took, not where the bottleneck is.
	- Tracing reveals exactly which service consumed the time.

### Connections to ETL Tools
- **Connection to Spark**:
	- The Spark UI is really an observability tool. It shows:
		- Stage durations
		- Task durations
		- Shuffle sizes
		- Memory usage
		- Failed tasks
- **Connection to Kafka**:
	- Kafka metrics include:
		- Consumer lag
		- Messages/sec
		- Partition throughput
		- Broker CPU
- **Connection to Airflow**:
	- Airflow provides:
		- DAG execution history
		- Task durations
		- Retry counts
		- Failed tasks

### Tradeoffs
- Too Few Metrics:
	- Pros:
		- Simple
	- Cons:
		- Poor visibility
		- Slower debugging
- Too Many Metrics:
	- Pros:
		- Lots of data
	- Cons:
		- Expensive
		- Noisy
		- Hard to find **useful** information
- The goal is **meaningful** observability—not collecting every possible metric.

### Common Interview Questions
1. What would you monitor in a production data pipeline?
> 	I'd monitor pipeline success rates, execution time, row counts, data freshness, error rates, CPU and memory utilization, throughput, and any system-specific metrics such as Kafka consumer lag or Spark stage duration. I'd also ensure structured logging and distributed tracing are available so engineers can quickly investigate failures rather than simply detect them.
2. Your Kafka-based data pipeline suddenly starts falling behind. Consumer lag is increasing, but producers are operating normally. Walk me through how you would use the following to identify the root cause:
	- Metrics
	- Logs
	- Traces
	- What specific information would you look for at each stage?
> 	I would use metrics to confirm producers are operating normally.  I would also use logs to identify any silent errors that could be occurring. If producers are healthy but consumer lag is increasing, I'd investigate whether consumers are CPU-bound, blocked on downstream systems such as databases, or experiencing processing errors. Metrics help identify which consumers are falling being, Logs help identify any particular specific errors or warnings, and traces help identify the specific bottleneck in the consumer workflow that is contributing to the lag.
	- Expand on *which* metrics you'd identify during your root cause analysis:
		- Producer metrics:
			- Messages/sec
			- Producer latency
			- Error rate
		- Consumer metrics:
			- Consumer lag
			- Messages/sec
			- Processing latency
		- Infrastructure metrics:
			- CPU
			- Memory
			- Network utilization
		- Spark metrics (if applicable):
			- Task duration
			- Shuffle size
3. A teammate says: "We already collect CPU and memory metrics, so we don't really need logs." Would you agree? Can metrics alone explain _why_ a system is failing? How do metrics, logs, and traces complement one another during an incident?
> 	CPU and memory metrics can only identify that there is an issue. These metrics are purely numerical and don't describe what the actual issue is. Logs can provide more insight by giving detailed info, warning, and error messages throughout a program's execution. Traces complement logs by breaking down which components consume the most resources or have the highest latency.
4. Suppose your dashboard shows high CPU utilization, but customer latency is completely normal. Would you immediately page the on-call engineer?
	- Probably not. This is exactly why **composite alarms** are valuable. High CPU alone isn't always customer-impacting.
	- Maybe:
		- Traffic increased as expected.
		- Auto-scaling is handling the load.
		- Latency remains low.
		- Error rates remain low.
	- Paging someone at 2 AM would just create alert fatigue.
	- A much better alert would be:
		```
		CPU > 90%
		
		AND
		
		P99 latency increasing
		
		AND
		
		Error rate increasing
		```

## Cache Hit Ratio

- Suppose your service receives 100 requests:
	- 95 requests are served directly from the cache.
	- 5 requests must query the database.
	- The cache hit rate in this scenario is 95%.
- The cache hit ratio is the percentage of requests that can be served directly from cache instead of the backing data store.
	- Formula:
		```
		Cache Hits
		───────────────
		Total Requests
		```
- Caches exist because they help reduce strain on a database system.
	- Instead of querying the database for each request, commonly used items are cached. Unlike databases, caches are optimized for fast and efficient reads.
	- A **cache hit** occurs when a requested item already exists in the cache and can be served without calling the database.
	- A **cache miss** occurs when a requested item does not exist in the cache and needs to be retrieved from the database.
- Cache hit ratio is a very important metric because it directly impacts request latency. A lower cache hit ratio results in higher average request latency. It also puts excessive strain on database CPU resources and can lead to cascading failures.
- **Common Causes of Low Cache Hit Ratio**:
	- Poor Cache Key Design:
		- If a cache key is too unique, nothing gets reused.
		- Cache hit ratio remains low.
	- Small Cache:
		- If a cache is too small, it fills up quickly.
		- Older entries are constantly evicted.
		- Popular data disappears.
		- Cache misses increase.
	- Short TTL:
		- A short TTL can force every request to hit the database because entries are constantly evicted before being reused.
		- Cache hit ratio falls.
	- Poor Access Pattern:
		- Imagine every request is for a completely different object.
		- Even a perfect cache won't help reduce strain on a database or decrease request latency.
- **Eviction Policies**:
	- Least Recently Used (LRU):
		- Most common.
		- Objects not accessed recently are evicted first.
	- Least Frequently Used (LFU):
		- Frequently accessed objects remain.
		- Useful when popular objects stay popular over long periods.
	- First In First Out (FIFO):
		- Simple.
		- Less common for application caches.

### Monitoring Cache Performance
- A good engineer doesn't just monitor cache hit ratio.
- They also monitor:
	- Cache misses
	- Evictions
	- Cache memory utilization
	- Average lookup latency
	- Database query rate
- For example, when cache hit ratio drops significantly, an engineer might ask the following questions:
	- Did TTL change?
	- Is memory full?
	- Are eviction rates increasing?
	- Did request patterns change?
	- Was a deployment made?
	- Is cache warming failing?
- Good engineers also monitor cache metrics **in the appropriate context**. For example, if cache hit ratio is 99.9%, but the 0.1% of cache misses represent the largest customer accounts, that's a serious problem.

### Tradeoffs
- Increasing Cache Size:
	- Pros:
		- Higher hit ratio
		- Lower database load
		- Lower latency
	- Cons:
		- More memory
		- Higher cost
		- Longer warm-up
- Cache Warming:
	- Pros:
		- Fewer cold misses
		- Better startup latency
	- Cons:
		- Additional infrastructure
		- Warm-up time
- Large Cache:
	- Pros:
		- High hit ratio
		- Lower latency
		- Lower database load
	- Cons:
		- Higher memory utilization
		- Longer cache warm-up
		- Potentially stale data
- Small Cache:
	- Pros:
		- Lower cost
		- Simpler
	- Cons:
		- More misses
		- More database load
		- Increased latency

### Common Interview Questions
1. Your cache hit ratio suddenly dropped from 98% to 70%. What would you investigate?
> 	I'd first verify whether request patterns changed or whether a recent deployment affected caching behavior. I'd review cache memory utilization, eviction rates, TTL configuration, cache warming, and cache key design. I'd also monitor backend database traffic and latency because a lower hit ratio often shifts load onto downstream systems.
2. Your service's cache hit ratio suddenly drops from **97% to 75%** after a deployment. Database CPU utilization also increases significantly. Walk me through your investigation. What metrics would you examine? What are your leading hypotheses?
> 	First, I'd look for significant changes in access pattern or a recent deployment that affected caching behavior. I would check evictions, cache memory utilization, and database query rate to see if the cache is filling up too quickly and frequently evicting items. I would also check cache-warming metrics to see if the cache is being adequately prepared to handle incoming requests. My leading hypothesis would be requests for recently added data that the cache warmer did not add to the cache yet, or a recently popular dataset may have been evicted under an LRU policy and is now being reloaded into the cache.
	- An LFU eviction policy is actually **less likely** to evict a popular item that hasn't been requested recently because it's based on **frequency**. An LRU eviction policy gives an item a certain amount of time to be requested before evicting it. An LFU policy evicts items based on how popular they are, regardless of how recently they were accessed.
	- Since the question specified the cache hit ratio occurred **after a deployment**, other hypotheses should include:
		- Cache key generation changed.
		- Serialization format changed.
		- TTL configuration changed.
		- Cache invalidation bug.
		- Cache warmer not running.
3. A teammate says: "Our cache hit ratio is already 99%, so there's no reason to look at any other cache metrics." Would you agree? Can you think of situations where a high hit ratio might still hide performance problems? What additional cache metrics would you monitor?
> 	Although cache hit rate of 99% implies the cache is healthy and operating efficiently, other metrics memory utilization and lookup time need to checked to ensure the cache isn't quickly filling up and that that items are being retrieved efficiently. Database query rate should also be checked to ensure the 1% of cache misses isn't putting an excessive burden on the database.
4. Suppose increasing the cache size improves the hit ratio from 97% to 99%, but memory utilization rises from 40% to 95%. Would you keep the larger cache?
	- It depends. First, the following should be evaluated:
		- Is memory pressure causing GC pauses?
		- Is the additional 2% hit ratio materially reducing backend load?
		- Are there eviction spikes?
		- Is latency improving enough to justify the memory cost?

## Serialization Overhead

- **Serialization isn't free**. Moving data between systems almost always requires converting it into a format that can be transmitted or stored. That conversion consumes CPU, memory, and time.
- Suppose your application has:
	```java
	Customer {
	    id = 100
	    name = "Alice"
	    balance = 250
	}
	```
	- Can this object be sent directly over the network? No.
	- First it must be converted into a format that both systems understand.
	- For example:
		```json
		{
		  "id":100,
		  "name":"Alice",
		  "balance":250
		}
		```
	- This conversion is called **serialization**.
- On the receiving side:
	```json
	{
	  "id":100,
	  "name":"Alice",
	  "balance":250
	}
	```
	- becomes `Customer`. This is called **deserialization**. Every distributed system performs this constantly.
- Serialization and deserialization occur almost everywhere. For example:
	- REST APIs (JSON)
	- Kafka messages
	- Spark shuffles
	- Database drivers
	- Redis
	- gRPC
	- Parquet
	- Avro
	- Protobuf
- Serialization becomes increasingly expensive as the number of objects that need to be serialized and deserialized increase.
- Serialization consumes:
	- CPU
	- Memory allocations
	- Garbage collection
	- Network bandwidth
- Sometimes, serialization itself can become a bottleneck.
- **JSON vs. Binary Formats( Avro, Protobuf)**:
	- JSON:
		- Pros:
			- Human readable
			- Easy debugging
			- Widely supported
		- Cons:
			- Large payload
			- Slower parsing
			- More CPU
	- Binary:
		- Pros:
			- Smaller payloads
			- Faster serialization
			- Faster deserialization
			- Machines generally prefer binary
		- Cons:
			- Hard to inspect manually
			- Requires schemas and tools
- **Spark Example**:
	- Every time Spark performs a shuffle, every partition is:
		- Serialized
		- Sent over the network
		- Deserialized
	- Large shuffles can lead to a lot of serialization overhead.
	- This is one reason shuffles are so expensive.
- **Kafka Example**:
	- When a producer publishes a message to a Kafka topic, it is serialized first.
	- When a consumer reads a message from a Kafka topic, it is deserialized first.
	- This occurs with **every message**.

### Reducing Serialization Overhead
1. **Smaller Objects**:
	- Don't serialize fields you don't need.
	- This results in less data and less CPU strain.
2. **Binary Formats**:
	- Use Avro or Protobuf instead of JSON, when appropriate.
3. **Avoid Repeated Serialization**:
	- If the same object needs to be serialized repeatedly, cache the serialized object so it can be reused, if appropriate.
	- This saves costs associated with serializing the object each time it is needed.
4. **Avoid Repeated Spark Shuffles**:
	- Every shuffle requires serialization.
	- Reducing unnecessary shuffles reduces serialization overhead too.
5. **Compress Wisely**:
	- Compression reduces network traffic, but increases CPU.
	- Careful compromise is required.

### Tradeoffs
- JSON:
	- Pros:
		- Readable
		- Easy debugging
	- Cons:
		- Larger
		- Slower
- Binary:
	- Pros:
		- Fast
		- Compact
	- Cons:
		- Harder debugging
		- Schema management

### Mental Model
- Connection to Previous Topics:
	```
	Application
	
	↓
	
	Kafka
	
	↓
	
	Spark
	
	↓
	
	Delta Lake
	
	↓
	
	Snowflake
	```
	- Data crosses multiple boundries.
	- Every boundary is a potential serialization point.
	- Understanding this helps explain why distributed systems incur overhead beyond just computation.

### Common Interview Questions
1. Why might serialization become a performance bottleneck?
> 	Serialization converts in-memory objects into a transferable format, which consumes CPU, memory allocations, and often additional network bandwidth. In distributed systems that move large volumes of data—such as Spark shuffles, Kafka messaging, or REST APIs—the cumulative cost of repeated serialization and deserialization can significantly impact throughput and latency. Reducing payload size, minimizing unnecessary serialization, and choosing efficient binary formats can improve performance.
2. Your team notices that a Spark job spends much more time in the shuffle stage than expected. CPU utilization is also unusually high during the shuffle. How could serialization overhead contribute to this problem? What optimizations would you investigate?
> 	Serialization overhead contributes to the problem because shuffling requires serialization and deserialization. During a shuffle, Spark must serialize records before sending them across the network and deserialize them on the receiving executors. If a large amount of data is shuffled, the CPU may spend a significant amount of time performing serialization rather than executing transformations. Optimizations would need to involve reducing unnecessary shuffling. Potential optimizations include adjusting the partitioning strategy and using broadcast joins.
	 - Besides reducing unnecessary shuffling, other avenues of investigation would include:
		- Are we serializing unnecessarily large objects?
		- Can we project only the required columns before the shuffle?
		- Are we using an efficient serialization format?
		- Is data skew causing one executor to serialize far more data than others?
3. A teammate says: "JSON is easier to debug, so we should always use JSON instead of Avro or Protobuf." Would you agree? What tradeoffs would you discuss? When might a binary format be the better engineering choice?
> 	While JSON is easier to debug, the decision of which serialization format to use ultimately depends on the business requirements. If debugging is more of a priority than preserving CPU, memory, and network bandwidth, then JSON is an excellent choice. If cost optimization and efficiency are higher priorities than readability and debugging, Avro or Porotobuf would be a better choice. Business requirements also determine where each format is most appropriate. JSON could be used in development environments while Avro or protobuf is used in production.
4. If Avro is smaller and faster than JSON, why don't we always use Avro?
	- Performance isn't the only concern. JSON provides advantages such as:
		- Human readability
		- Simpler debugging
		- Broad language support
		- No schema registry required
	- For small internal tools or low-volume APIs, those advantages may outweigh the performance benefits of Avro.

## Network Latency

- Suppose your application needs consumer data. There are several viable options for retrieving the data:
	- Option 1:
		```
		Application
		
		↓
		
		Local Memory
		
		↓
		
		2 ns
		```
	- Option 2:
		```
		Application
		
		↓
		
		Redis
		
		↓
		
		1 ms
		```
	- Option 3:
		```
		Application
		
		↓
		
		Database
		
		↓
		
		40 ms
		```
	- Option 4:
		```
		Application
		
		↓
		
		Remote API
		
		↓
		
		250 ms
		```
- The farther data needs to travel, the longer it takes to arrive.
- Network latency is the time required for data to travel between two systems.
- This includes:
	- Physical travel
	- Network routing
	- Serialization
	- Encryption
	- Connection overhead
- **Latency exists even if the network has plenty of bandwidth**.
- **Latency vs. Bandwidth**:
	- Latency measures how long it takes for the **first byte** to arrive.
	- Bandwidth measures how much data can be transmitted per second.
- **Why Network Calls Are Expensive**:
	- Suppose a database query takes 40 ms.
	- Now imagine your API makes 20 database calls.
	- The latency is now much higher than a single query.
	- This is why reducing the **number of round trips** is often as important as making individual queries faster.
- **N+1 Query Problem**:
	- Suppose you need to retrieve data for 100 customers.
	- If each customer requests data separately, it results in 100 different database calls.
	- A more optimal approach would be to JOIN the queries or use batch queries.
	- This results in far fewer network roundtrips.
- **Connection to Caching**:
	- Caching doesn't just reduce database load.
	- Caching also reduces network latency by minimizing the number of network roundtrips associated with each database call.
- **Connection to Async**:
	- Suppose each API call takes 100 ms.
	- Three sequential calls would take 300 ms.
	- Three asynchronous calls would take 100 ms.
	- Async **hides** latency, it doesn't reduce or eliminate it.
- **Connection to Serialization**:
	- Every network request typically requires a serialization cycle.
	- Network latency is just one aspect of request time.
	- Serialization adds additional overhead.
- Examples:
	- Spark:
		- During each shuffle, executors send data across the network.
		- More shuffling means:
			- More serialization
			- More network latency
			- More waiting
	- Kafka:
		- As events move from producer to broker to consumer, each step involves network communication.
		- Replication adds even more network traffic.
		- This is one reason `acks=all` has higher latency than `acks=1`.

### Common Ways to Reduce Network Latency
1. Reduce Round Trips:
	- Instead of 100 requests, only send 1 request.
	- Use **batch operations** to reduce the number of unique requests that need to be made.
2. Cache Frequently-Accessed Data:
	- Avoids remote calls altogether.
3. Move Computer Closer to Data:
	- Instead of downloading data and processing it locally, process the data closer to where it is stored.
4. Compress Large Payloads:
	- Less data moves across the network.
	- Tradeoff: Higher CPU cost.
5. Use Efficient Serialization:
	- Smaller payloads means less transmission time.
6. Parallelize Independent Calls:
	- Use async programming to process multiple requests simultaneously.
	- Only wait once for all requests.

### Tradeoffs
- Lots of Small Requests:
	- Pros:
		- Simple API
	- Cons:
		- Higher cumulative latency
- Larger Batched Requests:
	- Pros:
		- Fewer network trips
		- Better throughput
	- Cons:
		- Larger payloads
		- More memory

### Observability
- How do you know network latency is the problem?
- Metrics:
	- Request latency
	- Network utilization
- Logs:
	- Connection timeouts
	- Retries
- Traces:
	- Exactly which services are waiting
- Observability ties directly into troubleshooting network issues.

### Common Interview Questions
1. How would you reduce latency in a distributed system?
> 	I'd first identify where latency is occurring using metrics, logs, and distributed tracing. Common optimizations include reducing unnecessary network round trips through batching, caching frequently accessed data, choosing efficient serialization formats, parallelizing independent I/O operations with asynchronous processing, and minimizing large Spark shuffles or unnecessary service-to-service communication. The right optimization depends on where the latency originates.
2. Your microservice currently makes **15 sequential REST API calls** to retrieve customer information. Average response time is **2.5 seconds**. How would you investigate whether network latency is the primary bottleneck? What optimizations would you consider? How would you decide between batching requests, asynchronous processing, or caching?
> 	I would examine the bottleneck by reviewing performance metrics for the microservice as well as network latency. If CPU and memory utilization, as well as processing latency is high in the microservice, but network latency is low, the bottleneck is with the microservice. If network latency is high, the bottleneck is the sequential REST API calls being made. Reducing network latency could be done by batching the 15 API calls into one API call to reduce the number of network roundtrips, asynchronous processing to process the 15 API calls simultaneously, or caching to potentially reduce the number of API calls if the 15 API calls include repeated requests for the same resource. Deciding which optimization is best depends on the current architecture and business requirements.
3. A teammate says: "Our network is already 10 Gbps, so network latency can't possibly be our performance problem." Would you agree? How do latency and bandwidth differ? Can a high-bandwidth network still experience poor application performance?
> 	High network bandwidth only determines how much data can moved across a network per second. It doesn't determine how quickly data moves. High-bandwidth applications can still experience poor performance if network calls aren't optimized using methods such as asynchronous processing, batch requests, or caching.
4. You reduced network latency by 50%, but overall request latency only improved by 5%. Why?
	- Network latency wasn't the dominant bottleneck.
	- Maybe:
		- Database queries dominate.
		- CPU serialization dominates.
		- Lock contention dominates.
		- Cache misses dominate.
	- This is where **observability** becomes critical. You optimize the largest bottleneck—not just the easiest one to find.