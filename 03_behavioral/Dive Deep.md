---
category: 👥 Conflict
skills_highlighted: Optimization
---

# 🎭 STAR: Dive Deep

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** Our team was struggling to effectively plan our sprints. This caused our planning meetings to constantly run past the scheduled time slot. It also caused tasks to constantly spill over into subsequent sprints.
- **The Ultimate Outcome:** Working with our team, I analyzed how sprints were currently being planned and redesigned the way tasks were broken down and how their difficulty and expected duration was estimated. I also redesigned the planning meetings to encourage more active engagement from developers.

---

## 🌌 1. Situation & Task

- redesigned process
- improved efficiency
- identified root cause
- operational excellence
- continuous improvement
### Situation
At Amazon Ads, our team was struggling with sprint planning efficiency. Looking back at three months of sprint metrics, I found two key insights. Our planning meetings were running longer than scheduled and tasks regularly spilled over into subsequent sprints.

### Task
I realized our planning strategy didn’t account for real-world variables. We were planning for 100% capacity, completely ignoring unexpected production bugs, code reviews, and administrative overhead. Additionally, we lacked a strict 'Definition of Ready,' leading to poor task breakdowns and inaccurate, time-based estimations. I took the initiative to analyze these systemic bottlenecks and design a more predictable sprint framework.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

I proposed a complete overhaul of our sprint process across three pillars:
1. **Process Separation:** I separated backlog refinement from sprint planning because we were spending too much planning time discussing work that wasn't ready to be estimated. By the time planning began, every item already had clear acceptance criteria and technical understanding.
2. **Capacity Calibration:** Planning around ideal conditions made our commitments unreliable. I suggested moving away from estimating tasks primarily in hours and instead relying on relative complexity points to better account for uncertainty and seniority bias. I also capped our sprint velocity based on the lowest velocity of our previous three sprints.
3. **Developer Autonomy:** I decentralized the meetings. After the manager stated the business goals, I had developers drive the meeting and share screens rather than the Scrum Master. I wanted developers to estimate and commit to their own work rather than having estimates driven by the Scrum Master or manager. That created better ownership and surfaced implementation concerns earlier. I also introduced a quick mid-sprint refinement session for proactive re-estimation.
4. To get buy-in from my manager, I proposed a low-risk, phased rollout: we would test the strategy on a single sprint, and if satisfied, extend it to two additional sprints to clearly isolate and gauge its impact on team productivity.

---

## 📊 3. Result & Data Engineering Pivot

The phased trial was highly successful and became our permanent workflow. By the end of the three-sprint pilot, we **reduced sprint planning time by 20 minutes** per session, and our **on-time task completion rate rose by 30%**. More importantly, the team became much more predictable. Because we planned around realistic capacity rather than ideal capacity, stakeholders had greater confidence that sprint commitments would actually be delivered. Furthermore, developers reported feeling greater ownership over sprint commitments because they participated directly in planning and estimation.

---

## 4. Potential Followup Questions

1. “How did you handle resistance from the team or stakeholders when you proposed this?”
	1. **The Strategy:** Acknowledge that change is hard. Explain that you didn't force compliance; instead, you used data, empathy, and a low-risk trial to lower their guard.
	2. “Initially, there was some pushback from a few senior engineers who felt that moving backlog refinement to a separate meeting just added _more_ meetings to their calendar. I listened to their concerns and validated that calendar fatigue is real.To mitigate this, I explained that by separating the two, we would actually shorten the overall time spent in meetings because we wouldn’t be context-switching mid-planning. I asked them to give it just a one-sprint trial as an experiment. Once they saw they could get back to coding 20 minutes faster and didn't have to debate ticket requirements during planning, they completely bought in.”
2. “How did you enforce the strict 'Definition of Ready' without becoming a bottleneck or micromanager?”
	1. **The Strategy:** Show that you scaled the standard by building a shared checklist rather than acting as a solo gatekeeper.
	2. “I knew that if I was the only person checking tickets, the process would fail. Instead, I collaborated with the team to define a clear, 3-point automated checklist in our ticketing system for our 'Definition of Ready' (e.g., clear acceptance criteria, identified dependencies, and a visual mockup or architectural design attached).During our new backlog refinement meetings, any ticket that didn't hit those three marks was immediately flagged and pushed back to the product manager or tech lead to fill in before planning. This made the standard objective and community-enforced, rather than subjective.”
3. “What would you do if a sprint still had major spill-over even under your new framework?”
	1. **The Strategy:** Treat failure as a data point. Show that you would look at the root cause (metrics) rather than panicking or abandoning the system.
	2. “If we still faced chronic spill-over, I would immediately dive deep into our mid-sprint refinement data. I’d look to see if the spill-over was caused by an external dependency blocker, an unexpected high-severity production sev-bug, or an underestimation of complexity.If it was an underestimation, it means our relative point system needs calibration, and I would use the retro to compare that ticket against our baseline tasks. If it was an external blocker, I would look at updating our 'Definition of Ready' to ensure cross-team dependencies must be signed off _before_ a ticket can enter the sprint.”
4. “Why did you choose a 3-sprint window to calculate the velocity cap instead of a longer or shorter historical average?”
	1. **The Strategy:** Prove you didn't pick an arbitrary number. Explain the engineering logic behind choosing a 3-sprint trailing window for software teams.
	2. “I chose a 3-sprint window because it perfectly captures our team's _current_ operational reality. A single sprint is too volatile—an engineer taking a vacation or a single major bug can skew the data. Conversely, a 6-month average fails to account for recent changes, like a team member switching projects or new onboarding overhead.A 3-sprint window (roughly 6 weeks of data) provides enough history to smooth out anomalies while remaining highly sensitive to the team's immediate, realistic bandwidth.”