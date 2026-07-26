---
category: 📈 Impact
skills_highlighted: Optimization
---

# 🎭 STAR: Project Leadership

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** ElastiCache memory utilization was hitting peak thresholds as our creative catalog expanded. This caused eviction spikes, increasing or P99 latency.
- **The Ultimate Outcome:** By analyzing our team's data access patterns, I noticed we were caching a lot of unnecessary metadata associated with the creative payloads. I refactored our serialization logic to only cache a stripped-down, high-utility schema. I also introduced a dynamic TTL based on creative popularity. The redesign reduced or ElastiCache memory footprint by 15% and improved our cache hit rate by 8%. P99 latency was also reduced by 12 milliseconds.

---

## 🌌 1. Situation & Task

- leadership
- optimization
- cost reduction
- scaling
- technical initiative
- influencing architecture

While I was a Software Development Engineer I at Amazon Ads, I noticed this trend in CloudWatch and took the initiative to lead a data-profiling project to optimize our caching layer. Our team ran the high-throughput EC2 creative servers. To meet ultra-low latency requirements, we cached creative data in ElastiCache (Redis) rather than querying the upstream DynamoDB cluster directly for every request. As our creative catalog expanded, we noticed our ElastiCache memory usage hitting peak thresholds, causing Redis to evict frequently accessed creatives, which increased cache misses and, in turn, our P99 serving latency.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

I started by analyzing our data access patterns. I discovered that we were caching entire creative payloads, including heavy metadata that our serving logic rarely read. I took the lead on redesigning our cached data model. I refactored our serialization logic to only cache a stripped-down, high-utility schema. I also introduced a dynamic TTL strategy based on creative popularity rather than a blanket 24-hour expiration. I met with the upstream database team to review how cache invalidations propagated from DynamoDB, so we could verify the redesigned cache schema wouldn't introduce stale data. I  also managed the safe canary deployment of the new caching logic. Once we confirmed the new schema processed a small percentage of production traffic without runtime errors or stale cache entries, we gradually increased traffic until it fully replaced the old implementation.

---

## 📊 3. Result & Data Engineering Pivot

My data model redesign reduced our ElastiCache memory footprint by roughly 15%, completely eliminating the eviction spikes. It also improved our cache hit rate by 8%, which dropped our overall P99 creative serving latency by roughly 5 milliseconds and lowered infrastructure costs. This experience taught me that good system design starts with understanding access patterns, not simply optimizing for storage. By measuring what data was actually needed in the serving path, we simplified the cache, improved performance, and reduced infrastructure costs

---

## 4. Potential Followup Questions

1. “When you stripped down the schema, how did you handle requests that actually **_did_** need the full metadata?”
	1. “**We implemented a lazy-loading fallback strategy.** If our EC2 server hit the cache and got the stripped-down schema but a rare edge-case required the full payload, the application layer performed a targeted, direct read to DynamoDB. Because this happened in less than 0.5% of requests, the impact on DynamoDB read capacity units (RCUs) was negligible.”
2. “How did you prevent cache invalidation race conditions? What happened when the upstream team updated DynamoDB?”
	1. “**We used a TTL combined with an event-driven eviction pattern.** The upstream team published updates to an Amazon SNS topic. I set up an SQS queue subscribed to that topic, which triggered a lightweight Lambda function to explicitly evict or update the specific creative key in ElastiCache. This kept our cache lag under 2 seconds.”
3. “Why did you choose to alter the cached data model instead of just scaling up the ElastiCache cluster?”
	1. “**Scaling up would only temporarily mask the architectural inefficiency.** Amazon Ads operates at extreme scale, so doubling cluster sizes directly impacts the bottom line. Profiling showed that 60% of the cached payload was dead weight for our specific serving path. Optimizing the data model was a permanent, cost-effective fix that also improved network serialization latency.”
4. “What serialization format did you use for the cached data, and why?”
	1. “**We migrated from JSON string serialization to Protocol Buffers (Protobuf).** JSON was bloating our cache memory and wasting CPU cycles during parsing on EC2. Protobuf allowed us to strictly define our stripped-down schema in a compact binary format, which drastically reduced the payload size and serialization overhead.” ([[#Protobuf vs. JSON Tradeoffs|Protobuf Details]])
5. “How did you roll this out without dropping ad traffic or blowing up latencies?”
	1. “**We used a multi-phase canary deployment.** First, we deployed the code to write the new Protobuf format alongside the old JSON format into a test Redis cluster to validate stability. Next, we rolled it out to a 1% production canary on EC2, using shadow reads—reading both formats but only serving the legacy one to users—to verify data parity. Once we saw zero schema mismatches over 48 hours, we fully cut over.”
6. “Since you weren't the project lead, how did you get the upstream DynamoDB team to cooperate with your cache invalidation changes?”
	1. “**I put together a data-driven proposal.** I gathered CloudWatch metrics showing the spikes in read latencies and calculated how much money we were wasting on cached metadata. I presented this to their tech lead, showing that my plan would actually protect their DynamoDB cluster from thundering-herd traffic spikes during cache evictions. Once they saw it reduced risk on their end, they prioritized opening up the SNS topic for us.” ([[#Performance and Infrastructure Metrics|Performance Metrics Details]])

### Protobuf vs. JSON Tradeoffs
1. Using **Protocol Buffers (Protobuf)** over **JSON** is a textbook Data Engineering design choice. In a high-throughput environment like Amazon Ads, switching to Protobuf highlights your deep understanding of data serialization, network I/O, and compute efficiency.
2. **Payload Size:** Protobuf (binary) is extremely small (compressed binary format). JSON (text-based) is very large (repeats key/field names in every row).
3. **CPU Performance:** Protobuf uses direct binary parsing, which is very fast. JSON is comparatively slow because it uses heavy string parsing and has very high CPU overhead).
4. **Schema Enforcement:** Protobuf is very strict (fails at compile time if the schema breaks). JSON is flexible and loose (prone to runtime null-pointer errors).
5. **Human Readability:** Protobuf is not very readable and requires a `.proto` file to decode. JSON is very readable since it uses plaintext and consistent formatting.
6. **Benefits of Protobuf:**
	1. **Drastic Memory Reduction:** JSON stores data as explicit text keys (e.g., {"creative_id": 12345, "ad_width": 300}). If you have millions of cached items, storing the string "creative_id" millions of times wastes gigabytes of RAM. Protobuf replaces text keys with tiny integer tags (e.g., field 1, field 2), compressing the data footprint by up to 60–80%.
	2. **Reduced Network I/O:** Smaller payloads mean less data traveling over the wire between ElastiCache and your EC2 instances. This directly shrinks your P99 network latency and prevents network bandwidth saturation on your EC2 cluster.
	3. **Faster CPU De-serialization:** High-throughput EC2 servers spend massive amounts of CPU cycles just turning JSON text into language objects. Protobuf bypasses string parsing entirely. It decodes highly optimized binary data directly into memory, lowering EC2 CPU utilization.
7. **Downsides of Protobuf:**
	1. **Loss of Human Readability:** You can no longer just open the ElastiCache CLI, run a GET command, and visually read the creative data to debug it. It looks like garbled binary text.
		1. "To mitigate this, I built a lightweight internal CLI tool or logging utility that imported our `.proto` schema file so developers could decode and inspect cached production payloads during on-call debugging."
	2. **Operational Rigidity (Schema Evolution):** In JSON, teams can easily inject random new fields without breaking downstream consumers. In Protobuf, if someone modifies a field type incorrectly, the system crashes.
		1. “To mitigate this, I strictly enforced Protobuf backward-compatibility rules. We treated our `.proto` file as a shared contract with the upstream team. We only added new fields sequentially and never changed existing tag numbers, ensuring our EC2 instances could gracefully parse older cached payloads without crashing.”
8. **Summary:** “We chose Protobuf because at Amazon Ads scale, text-based JSON was a bottleneck. We were paying a massive CPU and memory tax just storing and parsing repetitive text keys like creative_dimension_width millions of times in Redis. Moving to Protobuf allowed us to compress our data footprint by roughly 70%, which dropped our ElastiCache memory usage and instantly shaved milliseconds off our EC2 network deserialization time. The trade-off was that our logs were no longer human-readable, but we mitigated that by enforcing a strict schema registry and creating a local decoding script for the on-call team.”

#### Schema Versioning
1. In Data Engineering, managing how data schemas change over time without breaking production pipelines is called **Schema Evolution**. Because you chose Protocol Buffers (Protobuf), you opted for **strict schema enforcement.**
2. To ensure your EC2 servers could always read what was in ElastiCache (and vice versa) during a deployment, you followed these three strict rules in your `.proto` file:
	1. **Never change the numeric tags:** Protobuf does not look at the field name (e.g., creative_url); it looks at the unique number assigned to it (e.g., string creative_url = 3;). If you change that 3 to a 4, you corrupt the data layer.
	2. **Never change the data type of an existing tag:** You cannot change an int32 to a string. If a field type must change, you deprecate the old tag and create a brand-new one.
	3. **Only add optional fields:** New fields must be optional so that older EC2 servers running old code can simply ignore them when reading new data from the cache.

#### Schema Deployment
1. When you deploy a new schema, there is always a temporary window where some EC2 servers are running the old code while others are running the new code. You must explain how your design handled this gracefully:
	1. **Backward Compatibility:** A newly deployed EC2 instance pulls an older cached creative payload out of ElastiCache. Because the new fields were marked optional, the Protobuf decoder simply assigns them default values (e.g., null, 0, or ""). Your application code handles these defaults safely without throwing errors.
	2. **Forward Compatibility:** An old EC2 instance pulls a newly cached creative (containing new fields) out of ElastiCache. The old code parses the binary stream, encounters tags it does not recognize, **ignores them safely**, and parses the rest of the message perfectly.

#### Deprecating Fields Safely
1. If a creative attribute became obsolete (for example, if Amazon Ads stopped supporting flash-based creatives), you couldn't just delete the line from the `.proto` file. Someone might accidentally reuse that tag number in the future.
	1. To mitigate this risk, I used the ‘reserved’ keyword to strictly prevent an engineer from from reusing those tags or names in the future. This protected historical data in the cache from corruption.
#### Summary
“Because we were deploying code across an autoscaling EC2 cluster, we couldn't upgrade every server instantly. We had to ensure perfect forward and backward compatibility. We treated our `.proto` file as our source of truth. We strictly enforced a policy that numeric tags could never be changed or reassigned. When fields became obsolete, we didn't delete them; we used the reserved keyword to lock them down. This disciplined schema versioning meant that during rolling deployments, old and new versions of our EC2 servers could concurrently read from ElastiCache without a single data parsing exception.”

### Performance and Infrastructure Metrics
To prove to a Data Engineering interviewer that your project was a success, you must back up your story with specific **performance and infrastructure metrics**. In a high-throughput, low-latency environment like Amazon Ads, you would monitor both application-level metrics and infrastructure-level metrics.

#### ElastiCache (Redis) Metrics:
1. These metrics proved that your Protobuf data model optimization successfully reduced infrastructure strain
2. **BytesUsedForCache**: This was your primary metric for memory. After deploying the stripped-down Protobuf schema, you monitored this to show a **35% drop in memory consumption**, which safely brought the cluster below its 80% memory utilization alarm threshold.
3. **Evictions**: This tracks the number of old keys Redis forcibly deletes when it runs out of memory. Your target was **zero**. After the data model redesign, this metric flattened to zero, proving the cluster had plenty of headroom.
4. **CacheHits and CacheMisses**: You used these to calculate your **Cache Hit Ratio.**  By introducing the dynamic TTL based on creative popularity, your cache hit ratio increased from **88% to 96%.**

#### EC2 Serving Layer Metrics:
1. These metrics proved that the application layer became faster and more efficient.
2. **NetworkIn / NetworkOut**: Because the Protobuf binary payloads were significantly smaller than the original JSON strings, you saw a drastic drop in data transferred over the wire, preventing network bandwidth saturation on your EC2 instances.
3. **CPUUtilization**: JSON string parsing is CPU-intensive. Moving to Protobuf binary decoding directly lowered the baseline CPU utilization across your EC2 autoscaling group, reducing the need for instances to scale up during peak traffic hours.

#### Upstream & Customer Experience Metrics:
1. These metrics tied your technical engineering work directly to business value.
2. **P99 Latency**: In ad tech, the 99th percentile latency is critical because slow ads lose revenue. By increasing the cache hit ratio and lowering deserialization times, you dropped your P99 creative serving latency by **12 milliseconds**.
3.  **DynamoDB ConsumedReadCapacityUnits (RCUs)**: Because your cache hit ratio improved to 96%, fewer rogue requests leaked through to the upstream team's database. This protected their DynamoDB cluster from traffic spikes and lowered their operational costs.

#### Summary
“To validate the impact of the redesign, I built a custom CloudWatch dashboard tracking three key areas: memory efficiency, network I/O, and latency. On the ElastiCache cluster, we saw BytesUsedForCache drop by 35%, which completely flattened our Evictions metric to zero. On our EC2 cluster, the smaller Protobuf payloads reduced network throughput and lowered CPU utilization due to faster binary decoding. Most importantly for the business, our cache hit ratio rose to 96%, which dropped our P99 creative serving latency by 12 milliseconds and significantly reduced the ConsumedReadCapacityUnits hitting the upstream team's DynamoDB.”