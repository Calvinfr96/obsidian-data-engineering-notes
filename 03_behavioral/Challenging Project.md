---
category: 👥 Conflict / 📈 Impact
skills_highlighted: Cross-functional Communication, Optimization
---

# 🎭 STAR: Challenging Project

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** Our team was tasked with launching a new feature under an aggressive timeline. Shifting product requirements required a major architectural pivot. This caused frustration and a drop in team morale.
- **The Ultimate Outcome:** By improving communication and collaboration strategies, we successfully delivered the feature on time for a peak traffic event, with zero high-severity alarms post-launch.

---

## 🌌 1. Situation & Task

- Tell me about your most challenging project.
- Describe a time requirements changed unexpectedly.
- Tell me about a time you had to redesign an existing solution.
- Describe a project with significant ambiguity.
- Tell me about working across team boundaries.
- Tell me about coordinating multiple stakeholders during a complex project.

In my role as an SDE I at Amazon Ads, our team's service was responsible for delivering optimized ad creatives by retrieving the creative from an in-memory EC2 cache and related optimization assets from a separate ElastiCache instance. Attribution metadata was normally available directly from the bid response, so our service had very few external dependencies. Midway through the project, the product team updated the attribution model. Previously, all attribution data came from the current bid response. Under the new model, we also needed historical conversion data stored in another system. That fundamentally changed the architecture because our service now depended on another team's storage layer to retrieve additional attribution data before generating the final response.

The biggest challenge wasn't writing the code—it was that we no longer had a clear implementation path. We needed to understand what historical data was available, how it could be accessed, what latency guarantees existed, and how to integrate another team's service without jeopardizing our own latency requirements. Our original design assumed every piece of attribution information was available in the bid response. Once that assumption changed, we had to redesign the request flow to incorporate an additional storage dependency while still meeting our performance targets and the launch deadline.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

Even though I wasn't the project lead, I realized that waiting for someone else to coordinate the redesign would likely slow the project enough to put the launch window at risk. I took the initiative to bridge the gap. Because the change affected multiple work streams, I organized short design sessions between our engineers and the team responsible for the historical conversion store. I drove the discussions by documenting open architectural questions ahead of time and ensuring each meeting ended with clear owners and next steps. That prevented us from revisiting the same decisions and kept multiple engineers moving in parallel. It also allowed us to align on the interface, clarify ownership boundaries, and identify which assumptions in our original design were no longer valid. That gave everyone a shared implementation plan and prevented different engineers from making incompatible design decisions.

---

## 📊 3. Result & Data Engineering Pivot

Because we aligned early on interfaces and ownership, the team was able to continue implementation in parallel despite the architectural change. We delivered the attribution feature in time for the peak traffic event with no high-severity production issues after launch. I learned that architectural changes involving new data dependencies require alignment just as much as technical design. Investing time upfront to clarify interfaces and ownership saved us far more time during implementation.

---

## 4. Potential Followup Questions

1. “When the goals shifted mid-sprint, how did that impact the underlying data architecture or pipeline design, and how did you coordinate the technical pivot?”
	1. **Why They Ask:** They want to see if you understand the downstream data impacts of sudden changes, and if you can lead technical alignment among peers.
	2. **The Strategy:** Focus on data contracts, schema changes, or pipeline bottlenecks. Show that your collaboration directly prevented data corruption or broken pipelines.
	3. “The shift required us to ingest a completely new set of real-time ad interaction events that our original schema wasn't built to handle. To coordinate the pivot without breaking downstream analytics, I immediately drafted a updated schema proposal. I rallied the two other engineers working on the ingestion layer for a quick huddle. Together, we agreed on a temporary data contract to isolate the changes, ensuring the upstream pivot wouldn't corrupt our historical data lake tables. This quick alignment allowed us to rewrite the transformation logic in parallel rather than waiting on each other”
2. “You mentioned pair-programming and helping unblock junior peers. How did you balance helping them with delivering your own heavy engineering workload?”
	1. **Why They Ask:** As a Data Engineer, you have to manage massive individual data deliverables while supporting the team. They are testing your time management and prioritization.
	2. **The Strategy:** Frame your help not as a distraction, but as a strategic force-multiplier. Use Amazon terminology like "Bias for Action" balanced with "Deliver Results.”
	3. “I looked at it through the lens of overall team velocity. If my peers stayed blocked on their data pipelines, the entire feature would fail to launch, making my individual deliverables useless anyway. To balance both, I time-boxed our pairing sessions to the first hour of the day to tackle their toughest blockers. I also automated a set of local integration tests for our ETL scripts and shared it with them. This empowered them to debug their own schema mismatches safely without needing me for every issue, which freed up my afternoons to focus deeply on my own high-priority pipelines.”
3. “How did you ensure that rushing to meet this tight deadline didn't result in accumulating massive technical debt in your data pipelines?”
	1. **Why They Ask:** Rushing data projects often leads to sloppy code, lack of documentation, poor data quality checks, and hardcoded values. They want to know you don't cut corners on data quality.
	2. **The Strategy:** Prove that you insisted on the highest standards by implementing automated safeguards, even during a crunch.
	3. “We were moving fast, but we couldn't afford to compromise on data quality, especially in an advertising context where data directly impacts billing. While we did make some tactical compromises—like choosing a simpler batch processing approach over a complex streaming architecture to save time—I insisted that we could not skip data validation. I quickly added basic automated null-value and data-type assertions to our CI/CD pipeline. I also logged our architectural trade-offs in a centralized markdown file during our daily sync. This ensured that immediately after the peak traffic event, the team had a clear, pre-approved roadmap to go back and refactor the code.”