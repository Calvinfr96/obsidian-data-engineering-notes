---
category: 👥 Conflict / 📈 Impact
skills_highlighted: Cross-functional Communication, Optimization
---

# 🎭 STAR: Prioritization

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** I needed to grant a major advertising client secure access to encrypted zip code information while working on operational issues during my on-call rotation.
- **The Ultimate Outcome:** By switching all in-person or virtual meetings to asynchronous forms of communication, as well as affectively time-boxing my engineering work and on-call duties, I was able to complete all sprint deliverables while reducing operational overhead for the team.

---

## 🌌 1. Situation & Task

Tell me about a time when you had two deadlines at the same time. How did you manage the situation?

While working at Amazon Ads, engineers were expected to continue delivering sprint commitments even while serving as the primary on-call engineer. During one rotation, I was nearing the end of a feature that had to be completed before the sprint closed when we began receiving several production issues that required immediate investigation. I had to decide how to keep the project on track without compromising our production support responsibilities.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

I first assessed whether each incoming ticket required immediate attention or could safely wait. High-severity production incidents always took priority because of their customer impact, while lower-priority operational work was scheduled into dedicated blocks later in the day.

To protect development time, I grouped feature work into uninterrupted focus periods instead of constantly context switching. I also moved status updates with our TPM to Slack so I wouldn't lose momentum or miss pages while waiting for meetings.

Whenever I was interrupted by a production incident, I documented my current progress before switching tasks. That made it much easier to resume feature development once the incident was resolved.

---

## 📊 3. Result & Data Engineering Pivot

By isolating my focus, I managed to complete all of my deliverables on time. Concurrently, I maintained 100% availability for high-severity incidents and cleared approximately 20 backlog tickets, significantly reducing our pipeline’s operational debt. This experience taught me that effective prioritization isn't about multitasking—it's about making deliberate decisions based on customer impact. By protecting focused development time while responding immediately to high-severity incidents, I was able to meet my sprint commitments without sacrificing production reliability.

---

## 4. Potential Followup Questions

1. “What was the most challenging ticket you handled out of those 20 backlog tickets, and how did you balance it with the project?”
	1. “The most impactful ticket I tackled from that backlog was investigating a recurring high-severity issue where ad creatives were missing from our edge cache. Historically, whenever this happened, the on-call engineer would get paged and manually run a cache warmer to pull the data from the database. It was a temporary fix that drained the team's time. Because this fell into my dedicated on-call block, I used that time to partner with the database platform team to find the root cause. We discovered a race condition during the initial bulk-load phase where certain creative records weren't fully committed before the cache initialization triggered. I coordinated a fix to synchronize the loading sequence, ensuring the cache warmer only initialized after a successful DB commit confirmation. By fixing this root cause during my allocated ticket hours, we completely eliminated that specific string of high-severity pages. This actually _saved_ me time later in the week, allowing me to focus entirely on the multi-region KMS project without unexpected interruptions.”
2. “How did your TPM and the advertiser react when you shifted all live meetings to asynchronous communication (Slack/Email)?”
	1. Frame it as a win-win. Explain that you proactively communicated _why_ you were doing it—to protect the project timeline from sudden meeting dropouts if you got paged. Because you were highly responsive and clear over Slack, the TPM appreciated the transparency, and it actually allowed everyone to document technical decisions better.
3. “If a high-severity page took up your entire afternoon, how did you catch up on the project work without missing the deadline?”
	1. Show your adaptiveness. Explain that if an incident completely wiped out a day, you would re-evaluate the project's remaining sub-tasks, identify if anything could be parallelized, or communicate immediately with the TPM if a minor milestone needed to shift by a day. You managed expectations dynamically rather than suffering in silence.