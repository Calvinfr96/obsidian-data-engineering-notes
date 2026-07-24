---
category: 💥 Outage / 📈 Impact
skills_highlighted: (e.g., System Reliability, Cross-functional Communication, Optimization)
---

# 🎭 STAR: Learning From Mistakes

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** A straightforward change meant to reduce redundant cache warm-up requests ended up causing increased cache misses in one region. This was due to an overlooked / underestimated edge case.
- **The Ultimate Outcome:** I acknowledged my optimization introduced the regression, worked with my peers to roll back the deployment, analyzed logs to track the bug, and updated the logic accordingly. This restored normal cache behavior within the incident window.

---

## 🌌 1. Situation & Task

- Tell me about a mistake.
- Tell me about failure.
- Tell me about feedback.
- Tell me about learning.
- Tell me about accountability.
### Situation
While working on the Ads Creative Caching team, I was responsible for a service enhancement designed to reduce redundant cache warm-up requests. The change seemed relatively straightforward and passed our existing unit tests.

### Task
My responsibility was to deploy the optimization without affecting cache availability.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

I underestimated one edge case involving concurrent cache invalidation events. After deployment, we noticed that a small percentage of requests weren't triggering the expected warm-up, causing cache misses to increase in one region.

Once we identified the issue, I immediately acknowledged that my optimization introduced the regression. Seeing my deployment contribute to an incident was uncomfortable because I knew teammates had to respond after hours. To quickly mitigate the issue, I worked with the on-call engineer to roll back the deployment, analyzed logs to isolate the race condition, and wrote a fix that synchronized the update logic correctly.

I also added integration tests covering concurrent invalidation scenarios that weren't previously part of our test suite.

---

## 📊 3. Result & Data Engineering Pivot

We restored normal cache behavior within the incident window and successfully redeployed the corrected version after validating it in staging. The additional tests caught similar issues in later changes and reduced the likelihood of regressions. That experience permanently changed how I evaluate production risk. Today, whenever I make changes involving concurrency or distributed systems, I think beyond whether the code is correct in isolation and ask how it behaves under real production timing. I've started to identify high-risk assumptions before implementation and explicitly ask whether there are scenarios our tests don't cover.

---

## 4. Potential Followup Questions

1. Why didn't you catch the issue earlier?
	1. I relied primarily on unit tests and underestimated the complexity of concurrent execution. Looking back, integration tests simulating production behavior would have revealed the problem.
2. How did you know it was your change?
	1. We correlated the increase in cache misses with the deployment timeline, reviewed logs, and reproduced the issue in a staging environment.
3. What would you do differently today?
	1. I'd identify high-risk failure modes before implementation, include concurrency-focused integration tests, and use a staged rollout with monitoring before full deployment.
4. How did your teammates react?
	1. I communicated transparently, shared what I'd learned, and involved the team in reviewing the fix. The focus stayed on restoring service and improving the process rather than assigning blame.