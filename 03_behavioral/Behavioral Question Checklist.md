## Refining Questions

1. [ ] **Add context about decision making**
	- Interviewers don't primarily hire people for their technical expertise. They hire people because **they make good engineering decisions.**
	- For example, instead saying "I implemented X", say "I considered, A, B, and C. I rejected A because... I rejected B because..."
2. [ ] **Emphasize how you used data to drive decisions**
	- For example, correlating logs, analyzing metrics, identifying patterns across incidents, or balancing performance, reliability, and cost.
3.  [ ] **Don't discuss the actions your team took. Specifically mention what YOU did**
	- Make sure to use "I", not "we" or "my team".
4.  [ ] **Add Uncertainty**
	- Great behavioral stories aren't obvious. They usually involve some degree of uncertainty. Specifically mention when you weren't sure about something, had conflicting signals, had incomplete data, or had to make a decision with incomplete information.
5. [ ] **Add tradeoffs**
	- Data engineers constantly need to make tradeoffs. For example:
		- Performance vs. consistency
		- Latency vs. cost
		- Batch vs. streaming
		- Availability vs. correctness
6. [ ] **Add non-technical lessons you learned**
	- For example, instead of saying "I learned concurrency needs integration tests", say "I learned that optimization work deserves the same level of rollout planning as customer-facing features."
7. [ ] **Don't fabricate or embellish details you don't remember**
	- Interviewers are usually better at detecting a reconstructed story than people expect.
	- If details are fabricated or embellished, the conversation often starts to feel unnatural.
	- For example, instead of saying:
		- "I considered three different caching strategies and ultimately selected Protobuf because..."
	- If you don't remember the discussion, say something like:
		- "At the time, the team agreed that the biggest bottleneck was memory usage, so we focused on reducing the cache footprint rather than scaling the cluster. I don't remember every alternative we discussed, but that was the reasoning behind the direction we took."
	- When you don't remember each step you took in an investigation, explain your thought process. For example:
		- "Once I saw the timing correlation across multiple incidents, synchronization became the simplest explanation that fit the evidence. Before changing anything, I partnered with the database team to validate that hypothesis."
		- This works better than mentioning every step in detail, such as mentioning specifically that you investigated logs, timestamps, and commit records

## Telling Your Story

If an interviewer asks you to tell them about the project your most proud of, tell them about [[Meeting Tight Deadline]]. It demonstrates debugging, data analysis, ownership, cross-team collaboration, decision-making under ambiguity, and measurable operational impact. If an interviewer says, "Tell me about the project you're most proud of," that's the story I'd lead with. It naturally showcases many of the skills companies look for in Data Engineers.

When telling your stories during an interview, try to use this flow to stay on track:
1. **State the problem clearly.**
2. **Mention one or two alternatives you considered.**
3. **Explain why you chose your approach.**
4. **Describe your implementation.**
5. **End with a lesson that's broader than the specific technology.**

That structure naturally showcases judgment, which is what behavioral interviewers are evaluating.

Don't memorize paragraphs.

Instead, memorize each story at three levels.

### Level 1 (30 seconds)

Be able to answer:

> "Tell me about a time you..."

in about 30 seconds.

This is your elevator pitch.

---

### Level 2 (2-3 minutes)

This is what you'll normally give.

Situation

↓

Task

↓

Action

↓

Result

---

### Level 3 (10+ minutes)

This is where interviewers spend most of the interview.

Once they find an interesting story, they'll stop asking behavioral questions and start digging.

Your follow-up sections are already heading in this direction.
## Topics to Review

1. Redis / ElastiCache
2. DynamoDB consistency and transactions
3. CloudWatch Logs Insights
4. KMS and IAM policies
5. Race conditions and concurrency
6. Caching strategies and cache invalidation
7. Serialization formats (JSON vs. Protobuf, if you mention them)
8. Canary deployments
9. Ad Attribution

### Tier 1 (I would absolutely know these)

These come up in multiple stories.

- Redis / ElastiCache
    - eviction policies
    - TTL
    - cache hit ratio
    - cache warming
    - cache invalidation
    - hot keys
- DynamoDB
    - consistency
    - transactions
    - partition keys
    - throughput
    - why cache instead of querying DynamoDB directly
- CloudWatch
    - Logs Insights
    - dashboards
    - metrics
    - alarms
    - composite alarms
- Race conditions
- Canary deployments
- Shadow traffic
- Feature flags
- Rollback strategies

### Tier 2 (Important)

These appear in several stories.

- Protobuf vs JSON
- serialization
- schema evolution
- backward compatibility
- monitoring
- observability
- p99 latency
- CPU utilization
- memory utilization
- cache hit rate

### Tier 3 (Worth reviewing)

- IAM
- AWS KMS
- least privilege
- AWS Multi-Region KMS
- SNS
- SQS
- Lambda
- CloudTrail
- Terraform / CDK (since you mention doing this differently today)

## Practice defending decisions

The strongest follow-up questions aren't

> "What is Redis?"

They're

> "Why?"

For every story, ask yourself:

Why this?

instead of

that?

For example...

---

Bias for Action

Why shadow traffic instead of just integration tests?

Why 48 hours?

Why accept higher memory usage?

Those are all already documented in your notes.

---

Meeting Tight Deadline

Why was synchronization better than retries?

Why wasn't this a database bug?

Why did you rule out deployment regressions first?

Excellent additions in V2.

---

Ownership

Why Multi-Region KMS?

Why not replicate data?

Why not use Terraform originally?

Again, already covered.

---

Project Leadership

Why redesign the schema instead of buying a larger Redis cluster?

That is exactly the kind of question I'd ask.

## Practice saying "I don't know"

This is probably the most underrated interview skill.

Many candidates think:

> "If I don't know, I fail."

Actually, experienced interviewers usually care much more about **how you reason** than whether you know one obscure fact.

If someone asked me:

> "Exactly how does Redis implement its eviction clock?"

I would not want you to bluff.

I'd say something like:

> "I don't remember the implementation details of Redis's eviction algorithm well enough to answer accurately. What I can explain is why we chose that eviction policy and how it affected our workload..."

Notice what happened.

You

- admitted the limit,
- answered honestly,
- shifted back to something you genuinely know.

That is much stronger than guessing.

### 1. You genuinely don't know

Just say so.

For example:

> "I don't know enough about that internal implementation to answer confidently."

That's perfectly acceptable.

---

### 2. You understand the concept but not the implementation

This is probably where most of your questions will land.

Example:

> "I don't remember exactly how Redis implements that internally, but the behavior we relied on was..."

That's a strong answer.

---

### 3. It wasn't your responsibility

Example:

> "The infrastructure team managed that configuration. My responsibility was the application layer..."

Interviewers hear this all the time.

They don't expect one engineer to own every part of a large production system.

---

Another good template is:

> "I wasn't responsible for that particular layer, so I don't want to speculate. On the application side, where I was responsible, we..."

You already started incorporating this style into your [[Choosing The Best Solution#Strategies For Answering Unknown Questions|Strategies For Answering Unknown Questions]] and I think it's exactly the right mindset.

### Don't bluff

This is the biggest piece of advice I'd give.

Amazon interviewers—and experienced engineers in general—can usually tell when someone is guessing.

Suppose someone asks:

> "How does the Redis eviction clock algorithm work internally?"

These are two very different answers:

❌

> "I think Redis probably..."

versus

✅

> "I actually don't know the implementation details of Redis's eviction algorithm. We treated it as a configurable service. What I did spend time analyzing was how different eviction policies affected our cache hit rate and latency."

The second answer is far more credible.

## Things to Avoid

Some of the "Potential Followup Questions" drift into system design rather than recollection.

For example:

> "How would you design..."

> "What if..."

Those are perfectly reasonable interview questions, but don't feel obligated to answer as though those design choices were part of your original project.

It's completely acceptable to distinguish between what you actually implemented and what you would choose today:

> "At the time, we implemented X because it fit our constraints. If I were designing the system today, I'd also consider Y because..."

That separation makes your answers more credible.