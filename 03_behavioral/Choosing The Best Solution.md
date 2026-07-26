---
category: 💥 Outage / ⚙️ Scale / 👥 Conflict / 📈 Impact
skills_highlighted: (e.g., System Reliability, Cross-functional Communication, Optimization)
---

# 🎭 STAR: Choosing The Best Solution

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** Highly popular creatives caused isolated traffic spikes in a single shard within an AWS region. These traffic spikes caused latency to rise above our strict SLAs. We needed to redesign how our request assignment and caching strategies
- **The Ultimate Outcome:** The routing redesign reduced shard hotspotting, lowered peak shard CPU utilization by roughly 20%, and improved p99 latency during holiday traffic without requiring us to expand the EC2 fleet.

---

## 🌌 1. Situation & Task

Tell me about a time when you were faced with a problem that had a number of possible solutions. What was the problem and how did you determine the course of action? What was the outcome of that choice?

### Situation
As an SDE I at Amazon Ads, my team maintained the low-latency distributed servers responsible for serving ad creatives immediately after a bid was won. Our infrastructure consisted of EC2 clusters spread across 4 shards per AWS region, utilizing localized in-memory caching for raw creatives and ElastiCache for optimization assets.

### Task
We faced a critical issue where viral or highly popular creatives caused severe p99 latency spikes and CPU starvation on specific shards. Because our routing logic was tied strictly to the Creative ID, all concurrent global traffic for a popular ad hit the exact same shard, creating massive hotspots and devastating our post-bid delivery latency SLA. We needed to completely redesign our request assignment and caching strategy. To determine the best course of action, our team needed to evaluate solutions across three vectors: routing architecture, concurrency bottlenecks, and runtime execution. I investigated the routing architecture.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

Initially, I considered simply increasing the number of shards, but that wouldn't eliminate hotspots because requests for the same creative would still hash to a single shard. We also discussed replicating popular creatives across every shard, but that would significantly increase memory usage and cache synchronization complexity. I ultimately favored changing the routing strategy because it addressed the root cause of the hotspotting problem without increasing infrastructure costs or significantly complicating the cache architecture. After investigating the routing patterns, I proposed changing the routing key from Creative ID to Campaign ID because it would distribute traffic more evenly while preserving cache locality. After discussing the proposal with senior engineers and validating the design, I implemented the routing changes and partnered with the team on a gradual production rollout.

---

## 📊 3. Result & Data Engineering Pivot

The routing redesign reduced shard hotspotting, lowered peak shard CPU utilization by roughly 20%, and improved p99 latency during holiday traffic without requiring us to expand the EC2 fleet. This experience reinforced that system design decisions should be driven by data access patterns and workload characteristics rather than assumptions. Understanding how requests flowed through the system allowed us to solve the bottleneck with a design change instead of simply scaling hardware.

---

## 4. Potential Followup Questions

1. “When you shifted the sharding key to Advertiser/Campaign ID, didn't you just move the hotspot problem from the creative level to the advertiser level? What happens if a massive brand like Nike launches a global campaign?”
	1. **Key Strategy:** Acknowledge the trade-off. Show how your multi-layered defense (Salted Hashing) caught what the new routing key couldn't.
	2. **Your Answer:** "That was a critical risk we accounted for. Sharding by Advertiser ID was chosen to optimize cache pre-fetching and keep asset portfolios localized, which solved the _average_ case beautifully. However, for massive tier-1 advertisers like Nike, we knew an entire shard could still get overwhelmed.
	3. **The Guardrail:** This is exactly why we layered **Two-Tier Sharding (Salted Hashing)** on top of the new routing key. Our dynamic monitoring system identified the top 5–10% hottest Campaign/Advertiser IDs in real-time.
	4. **The Execution:** When a specific campaign crossed our traffic threshold, the routing tier dynamically appended a salt suffix to it, overriding the default shard assignment and spreading that single massive advertiser's traffic evenly across all 4 shards. The routing key gave us predictability, and salted hashing gave us elasticity."

### Strategies For Answering Unknown Questions
1. Use this when the interviewer gets stuck on a low-level implementation detail (like a specific library feature or code syntax) and you want to pull back to the system architecture.
	1. **The Pivot Phrase:** _"That’s a great low-level detail. In our specific implementation, we handled that by... [1-2 sentences]. But zooming out to the broader system design, the real bottleneck we were trying to solve was..."_
	2. **Example in Action:** "That’s a great point about Caffeine's internal thread pool. In our implementation, we relied on its default ForkJoinPool to handle those evictions asynchronously. But zooming out to the broader system design, the real bottleneck we were trying to solve wasn't just eviction speed—it was ensuring that our thread pools never blocked incoming live ad traffic. That lock-free architecture is what ultimately allowed us to drop our p99 latency by 45%."
2. Use this if they ask about a niche edge case or an operational detail that you didn't personally work on, or that your team decided to handle manually.
	1. **The Pivot Phrase:** _"For that specific scenario, our team actually relied on a separate infrastructure layer, so I didn't deep-dive into that exact configuration. However, on the application side where I focused, the primary safeguard I built was..."_
	2. **Example in Action:** "For that specific scenario regarding cross-region replication delays in ElastiCache, our dedicated DevOps team actually managed the global replication lag parameters, so I didn't deep-dive into those specific cluster tweaks. However, on the application side where I focused, the primary safeguard I built was the Single-Flight pattern. This ensured that even if a replica lag occurred, our EC2 shards wouldn't spam the primary database with duplicate queries."
3. Use this when the conversation becomes purely theoretical, and you want to ground the interviewer back in your real-world results and business impact.
	1. **The Pivot Phrase:** _"We actually debated that exact theoretical trade-off during our design review. Ultimately, our engineering decisions were guided by our 'North Star' requirement of keeping latency under [X] ms. That's why we prioritized..."_
	2. **Example in Action:** "We actually debated that exact theoretical trade-off between G1GC tuning and Shenandoah during our design review. Ultimately, our engineering decisions were guided by our team's North Star requirement of preserving our low-latency ad-serving SLA. That's why we prioritized the Shenandoah switch, which completely eliminated the random stop-the-world pauses and brought our shard outages down to zero."
4. Use this when the interviewer isolates just one of your six measures (e.g., just the GC switch) and you want to remind them that this was a holistic, multi-layered solution.
	1. **The Pivot Phrase:** _"The garbage collection change was definitely a huge win, but it didn't work in isolation. It was actually designed to complement the..."_
	2. **Example in Action:** "The garbage collection change was definitely a huge win for managing our memory heaps, but it didn't work in isolation. It was actually designed to complement our application-level changes. Because the Single-Flight pattern dramatically reduced the number of duplicate objects being created in the first place, it gave the Shenandoah GC a highly predictable workload to clean up in the background.”

### Followup Questions For Interviewer
1. **The "Hook" Technique:** Before asking the question, connect it back to your conversation. For example: _"We talked a lot today about my experience managing shard hotspots at Amazon Ads. I'm curious, how does_ **_this_** _team handle data skewedness and hot keys at your scale?"_
2. **Take Notes:** When they answer, visibly nod or write a quick note. It shows you are genuinely engaged in their technical engineering challenges.
3. **Questions About Scale and Hotspots:** Use these if the team handles massive, unpredictable traffic spikes.
	1. “Given the scale this team operates at, how do you handle data skewedness or 'hot keys' in your distributed caches? Do you lean toward application-layer fixes like salted hashing, or do you solve it at the infrastructure tier?”
	2. “As the business grows, what is the current architectural bottleneck your team is actively trying to solve? Is the primary constraint network I/O, database write locks, or compute limits during peak traffic?”
4. **Questions About Low Latency and Performance:** Use these if the team is highly focused on strict SLAs or real-time processing.
	1. “Maintaining tight p99 latency SLAs is always a balancing act. How does this team approach the trade-offs between local in-memory caching for raw performance versus distributed caching for data consistency?”
	2. “When rolling out major runtime optimizations—like changing a garbage collection strategy or a low-level cache implementation—what does the benchmarking and canary testing pipeline look like here to ensure zero regression?”
5. **Questions About Engineering Culture and Design:** Use these to show you care about code quality, system evolution, and team collaboration.
	1. “In my past role, solving our shard hotspot issue required deep collaboration across different layers of our stack. How does this team handle cross-component architectural changes? Is it driven by individual SDE initiatives, or a centralized design review board?”
	2. “How does the team balance micro-optimizations (like code-level concurrency patterns) with long-term infrastructure scaling? How do you decide when a system needs code refactoring versus a complete architectural redesign?”