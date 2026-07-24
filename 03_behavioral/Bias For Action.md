---
category: 📈 Impact
skills_highlighted: Optimization
---

# 🎭 STAR: Bias For Action

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** One of our critical servers was experiencing minor, but regular latency spikes during high-throughput traffic windows.
- **The Ultimate Outcome:** Found and corrected the source of the latency spikes, reducing latency and stabilizing CPU utilization during peak traffic hours.

---

## 🌌 1. Situation & Task

- Tell me about a time you took a risk at work?
- Tell me about a calculated risk.
- Tell me about bias for action.
- Tell me about improving performance.
- Tell me about making a difficult technical decision.
- Tell me about innovation.
### Situation
 At Amazon Ads, I was responsible for maintain the downstream server that serves ad creatives instantly after a winning auction bid. Because this server was a critical part of the bidding flow, our p99 latency SLA was incredibly strict. I noticed that a specific object serialization process in our payload handling was causing minor but regular latency spikes during high-throughput traffic windows, threatening our SLA margins. Several engineers had noticed the issue before, but because it wasn't customer-visible and touching the serialization layer was risky, everyone understandably avoided changing it.

### Task
The safe choice was to leave the legacy serialization logic alone, as it was technically stable. However, leaving the code unchanged meant we had almost no performance headroom as traffic continued to grow. Even though customers weren't yet affected, I believed we'd eventually hit a scaling limit if we didn't address it proactively. I wanted to refactor this logic to a more efficient memory-allocation pattern, but the risk was severe: any bug or regression in this critical path could cause malformed creative payloads, dropped ad renders, and immediate revenue loss.
 
---

## 🛠️ 2. Action (The Engineering Deep-Dive)

To mitigate this technical risk, I didn't rush the deployment. First, I profile-tested the new serialization logic in a local benchmark environment to prove the CPU savings. Before starting the work, I shared my benchmark results and rollout plan with my manager and the senior engineer on the team. Once we agreed on the mitigation strategy, I moved forward with the implementation. Instead of a standard deployment, I set up a shadow canary routing system. This allowed the new implementation to process a duplicate stream of live production traffic, comparing the payloads and latency metrics in real-time without actually serving the results to end-users. Once I verified zero mismatches over a 48-hour period, gradually increased the percentage of production traffic served by the new implementation.

---

## 📊 3. Result & Data Engineering Pivot

The calculated risk paid off. We successfully shaved 6 milliseconds off our p99 latency and stabilized our CPU utilization during peak traffic hours. This was important because we had extremely tight latency budgets. Every millisecond of latency directly affected auction responsiveness. This experience taught me that technical debt isn't always about messy code. Sometimes it's accepting a known performance bottleneck because changing it feels risky. I learned that the right approach isn't avoiding risk—it's reducing risk through careful benchmarking, shadow testing, and gradual rollout.

---
## 4. Potential Followup Questions

1. “How did you determine that a 48-hour shadow test was sufficient?”
	1. “48 hours allowed the code to experience two full diurnal traffic cycles, capturing both the lowest troughs and the highest peak traffic windows of the ad server.”
2. “What was your rollback plan if the canary failed after going live?”
	1. “I kept the legacy code path active behind a dynamic configuration change (or feature flag). Turning it off took seconds and required zero re-deployments.”
3. “What did you sacrifice to get that 6ms latency improvement?”
	1. “The new serialization strategy used slightly more memory to save CPU Utilization. This was an acceptable tradeoff considering memory utilization was always well within safe limits.”
4. “Why did you choose shadow routing instead of just writing more unit and integration tests?”
	1. “High-scale production traffic has unpredictable variations, network jitter, and payload sizes that synthetically generated unit and integration tests simply cannot accurately replicate.”
5. “What would you have done if your manager told you the risk wasn't worth the sprint time?”
	1. “I would present the data (the local benchmarks showing the CPU spikes) to show the long-term risk of inaction, but ultimately respect the prioritization if the team had a more critical fire to fight.”
6. “If you could do this project again, what would you do differently?”
	1. “I would automate the payload mismatch verification process using scripts rather than manually checking the comparison logs during the shadow phase.”