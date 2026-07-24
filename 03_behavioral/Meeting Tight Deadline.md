---
category: 💥 Outage / 📈 Impact
skills_highlighted: System Reliability, Optimization
---

# 🎭 STAR: Meeting Tight Deadline

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** Our team faced recurring high-severity production issues related to ad creatives missing from our in-memory cache. The usual fix was to simply run the cache warmer to retrieve the missing creatives, but this wasn't a permanent solution. We needed to identify the root cause and implement a lasting solution.
- **The Ultimate Outcome:** By analyzing database commit records with cache initialization events, we missing creatives were caused by a lack of synchronization between these two events. Sometimes, the cache would initialize before a database commit had completed. By synchronizing these two events, we completely resolved the issue and freed on-call engineers to work on more important tasks.

---

## 🌌 1. Situation & Task

- toughest bug
- root cause
- used data
- ownership
- technical challenge
- analytical thinking
- solving ambiguity
- operational excellence
- data-driven decision making
- What project are you most proud of?
### Situation
While I was a Software Development Engineer I at Amazon Ads, we had a recurring high-severity production issue where ad creatives would occasionally be missing from our edge cache. Every incident triggered an on-call page, and the standard response was to manually run a cache warmer to repopulate the cache from the database. Although this restored service, it didn't address the underlying problem, and the issue kept recurring. Since these were customer-impacting incidents, I used my scheduled on-call project block to investigate and permanently resolve the issue before it generated additional customer-facing incidents.

### Task
My goal was to identify the root cause and implement a permanent solution before the issue generated additional customer-facing incidents and engineering overhead.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

Initially, I was able to rule out deployment regressions because none of the incidents corresponded with recent deployments. I ruled out cache evictions by reviewing cache-related metrics and TTL configurations. I also ruled out bad data because we never received reports of malformed creatives. Furthermore, manually running the cache warmer always resolved the issue.

Rather than accepting the manual workaround, I analyzed incident data from previous pages, comparing timestamps from application logs, cache initialization events, and database commit records. Looking across multiple incidents, I noticed that every failure shared the same sequence of events: cache initialization completed before the corresponding creative records finished committing to the database. That timing pattern explained why rerunning the cache warmer always succeeded—the data existed by then. I partnered with the database platform team to validate this hypothesis and confirm it was a race condition during the initial bulk-load process.

Based on this analysis, I proposed changing the cache initialization strategy so that the cache warmer would only execute after receiving confirmation that the database transaction had successfully committed. We coordinated the implementation across both teams and validated the solution in a staging environment before deployment.

---

## 📊 3. Result & Data Engineering Pivot

The fix permanently eliminated that category of high-severity pages, removing the need for repeated manual cache warming and reducing operational burden on the on-call rotation. Beyond improving system reliability, it freed engineers to focus on higher-value work instead of repeatedly responding to the same production issue. This experience reinforced that operational excellence isn't about creating better runbooks—it's about eliminating the need for the runbook in the first place by fixing the underlying system.

---

## 4. Potential Followup Questions

1. How did you determine it was a race condition?
	1. **What they're testing:** Root-cause analysis and debugging methodology.
	2. **Strong Answer:** "I started by reviewing several incidents instead of treating each one independently. I compared application logs, cache initialization timestamps, and database commit logs for successful and failed cases. The failures consistently showed that cache initialization sometimes started milliseconds before the corresponding creative records were fully committed. Since the issue was intermittent and timing-dependent, that pointed toward a race condition rather than corrupted data or cache eviction."
2. "What data did you analyze?"
	1. I relied on operational data rather than business metrics. Specifically, I analyzed:
		1. Application logs
		2. Cache initialization timestamps
		3. Database commit timestamps
		4. Incident history from previous on-call pages
		5. Deployment history to rule out recent code changes
	2. Correlating those data sources let me identify a consistent sequence of events that explained why the failures occurred.
3. Why wasn't the manual cache warmer sufficient?
	1. The cache warmer restored service, but it treated the symptom instead of the underlying issue. Every occurrence still required an engineer to respond, investigate, and manually repopulate the cache. That increased operational costs and delayed resolution. I wanted to eliminate the recurring work by fixing the underlying synchronization issue.
4. How did you prioritize this over your other work?
	1. I considered both customer impact and engineering cost. This issue repeatedly generated high-severity pages and interrupted the on-call rotation. Even though each incident could be resolved manually, the cumulative operational burden was high. Eliminating recurring incidents had a much larger long-term impact than completing another feature ticket.
5. How did you validate your hypothesis?
	1. **This is a very common technical followup question**
	2. Once we believed the issue was caused by cache initialization occurring before database commits, I worked with the database platform team to reproduce the timing scenario in a staging environment. We instrumented additional logging around commit completion and cache initialization to verify the ordering. After implementing the synchronization change, we confirmed the cache was never initialized before a successful commit.
6. What alternatives did you consider?
	1. We considered making the cache warmer retry failed records, but that would have hidden the underlying synchronization problem. We also discussed periodically rescanning the database for missing creatives, but that would have increased unnecessary database load. Synchronizing cache initialization with successful commits addressed the root cause with minimal operational overhead.
7. What was your specific contribution?
	1. **This is a very common technical followup question.** Interviewers care a lot about distinguishing **your work** from the team's work.
	2. I owned the investigation. I analyzed the incident data, identified the correlation between commit timing and cache initialization, proposed the synchronization strategy, coordinated with the database platform team to validate the hypothesis, and drove the implementation and testing. The platform team helped validate behavior on the database side, while I owned the application-side changes.
8. What was challenging about working with another team?
	1. The challenge was that neither team had complete visibility into the end-to-end workflow. I understood the cache initialization process, while the platform team understood transaction commit behavior. We established a shared timeline of events using logs from both systems, which allowed us to identify the race condition together.
9. How would you prevent this from happening again?
	1. Besides implementing the synchronization fix, I'd add monitoring that compares the number of committed creative records with the number successfully loaded into the cache. Any discrepancy would trigger an alert before customers were affected. I'd also document the dependency between database commits and cache initialization so future changes don't unintentionally reintroduce the issue.
10. If you had more time, what would you improve?
	1. I'd build automated monitoring around cache consistency and create dashboards tracking cache hit rates, initialization failures, and missing creative counts. That would allow the team to detect anomalies proactively rather than relying solely on customer-facing incidents.
11. You said you leveraged data. How is this different from simply debugging code?
	1. I approached it as a data analysis problem rather than a code review. Instead of inspecting the implementation first, I collected operational data from multiple systems—logs, timestamps, commit events, and incident history—to identify patterns. The evidence pointed to a timing issue, which guided the debugging effort. That's very similar to data engineering work: collecting data from different sources, identifying patterns, forming hypotheses, and using those insights to drive technical decisions.
12. How did you know it wasn't a database performance issue instead of a race condition? Why did you believe synchronization was the right fix?
	1. The database itself wasn't failing or timing out. The evidence showed the records were eventually written successfully because rerunning the cache warmer consistently worked. The failures only occurred when cache initialization happened before those writes completed, which pointed to an ordering problem rather than a persistence problem.