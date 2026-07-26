---
category: 👥 Conflict / 📈 Impact
skills_highlighted: Cross-functional Communication, Optimization
---

# 🎭 STAR: Influencing Without Authority

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** Ticket resolution was being slowed down by manually looking through error logs to find the root cause. Additionally, our infrastructure and alarm configuration caused repetitive tickets that related to the same issue, causing unnecessary stress and distraction for on-call engineers.
- **The Ultimate Outcome:** Using CloudWatch Composite Alarms and CloudWatch Log Insights, I was able to eliminate redundant alarms and significantly reduce mitigation time for high-severity tickets.

---

## 🌌 1. Situation & Task

- Tell me about influencing someone.
- Tell me about disagreeing with your manager.
- Tell me about advocating for an idea.
- Tell me about improving operations.
- Tell me about influencing a decision.
- Tell me about improving an operational process.
- Tell me about using data to identify a problem.
- Tell me about persuading a stakeholder.
- Tell me about a time your idea wasn't initially accepted.
- Tell me about balancing short-term priorities with long-term improvements.
### Situation
I was a software engineer working at Amazon Ads handling on-call rotations. Looking through our team's incident history, I analyzed the timestamps between when SEV2 tickets were acknowledged and when they were mitigated. That analysis revealed two operational bottlenecks slowing incident resolution. Engineers spent too much time manually digging through raw CloudWatch logs for generic errors. Additionally, The 4-shard regional infrastructure caused a "ticket storm" (bursts of duplicate, overlapping alarms) during an incident.

### Task
I first wanted to determine whether this was a process problem or a tooling problem. I reviewed our team's SOPs and found they were already thorough and up to date, which suggested the bottleneck was the tooling rather than the documentation. Next, I wanted to research and implement **CloudWatch Log Insights** to accelerate log analysis and **CloudWatch Composite Alarms** to deduplicate the ticket storms. My manager agreed that this was an issue, but not one that could currently be prioritized. They insisted that the team could not divert resources away from high-priority sprint deliverables. I wasn't the team's tech lead or manager, so I knew I couldn't simply decide to do the work. My goal was to make the cost of trying the idea small enough that it was easy to approve.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

I didn't just accept the "no" or start an argument; I used reasoned data and empathy. I explained that reducing Time-to-Mitigate (TTM) directly protects ad revenue and customer experience. I also highlighted how fixing this would reduce on-call engineer burnout and drastically lower the barrier to entry and stress for new hires onboarding onto their first rotation. I intentionally proposed a one-day proof of concept because I knew asking for a full sprint commitment would likely be rejected. A small experiment lowered the cost of saying yes.

---

## 📊 3. Result & Data Engineering Pivot

My manager agreed to the one-day spike. Within that day, I successfully built a prototype of the composite alarm and a reusable Log Insights query. I proved the value, my manager decided to prioritize it for the following sprint, and it ultimately reduced duplicate alarms, shortened the initial investigation time for SEV2 incidents, and gave the team enough confidence to prioritize the work in the following sprint.

---

## 4. Potential Followup Questions

1. “How exactly did you design the Composite Alarm? What was the underlying rule logic?”
	1. Explain that you combined the alarms from the 4 regional shards using AND / OR logic so that if all 4 shards threw the same error, it rolled up into a single high-severity ticket instead of 4 separate ones
2. “What were the sample CloudWatch Log Insights queries looking for? How did they save time?”
	1. Explain that you wrote parsed queries targeting specific error codes or request IDs, allowing engineers to filter millions of logs in seconds rather than manually scrolling through standard streams.
3. “Why did the infrastructure have 4 shards per region in the first place? Did your changes affect blast radius?”
	1. Clarify that the sharding was for horizontal scaling and fault tolerance. Your changes did not touch the infrastructure itself; they only improved the _observability layer_ on top of it.
4. “What would you have done if your manager said no to the one-day compromise?”
	1. Emphasize respect for hierarchy and commitments. Say you would have accepted the decision for that sprint, documented the data bottlenecks during your next shift, and brought it up again during a retrospective or sprint planning when capacity opened up.
5. “How did your teammates react to this change once it was implemented?”
	1. Mention that the team was incredibly grateful. You can note that the next person on rotation explicitly mentioned how much easier it was to pinpoint errors using the saved queries you built.
6. “In hindsight, do you think your manager was right to push back initially?”
	1. Validate your manager's perspective. Say, _"Yes, absolutely. As a manager, his job was to protect the team's committed deliverables. It taught me that when proposing operational fixes, I always need to present them alongside a risk-mitigation strategy for our current sprint.”_
7. “Do you have an estimate of how much Time-to-Mitigate (TTM) was reduced?”
	1. You don't need exact decimal points, but provide a realistic estimate. _"Before the change, parsing logs manually could take 15 to 20 minutes per incident. With the Log Insights queries, we cut that initial triage time down to under 5 minutes."_
8. “How many duplicate tickets were eliminated by the composite alarms?”
	1. “Instead of getting 4 separate alarms for a single regional shard issue, it consolidated them into 1, effectively cutting our incident ticket volume by roughly 75% during a region-wide event.”