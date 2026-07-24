---
category: 📈 Impact
skills_highlighted: System Reliability, Cross-functional Communication
---

# 🎭 STAR: Ownership

## 📌 Elevator Pitch (The 30-Second Summary)
*A brief summary to center your memory before speaking.*
- **Context:** Working as a Software Engineer at Amazon Ads.
- **The Core Problem:** A major advertising client needed access to customer info that was encrypted due to privacy regulations.
- **The Ultimate Outcome:** Working with our organization's TPM, I found a way to grant the client secure access to the information, which helped them execute their advertising campaigns more effectively.

---

## 🌌 1. Situation & Task

Tell me about a time when you went over and above your job responsibility to help the company?
### Situation
In my previous role as an SDE I at Amazon Ads, a major automotive advertiser urgently needed access to geographic targeting data, which was encrypted using AWS KMS, in order to execute a highly targeted, region-specific advertising campaign.

### Task
My core responsibility was simply to maintain the existing data pipeline. However, the data was strictly encrypted using single-region AWS KMS keys, which complicated securely supporting workloads that operated across multiple AWS regions.

---

## 🛠️ 2. Action (The Engineering Deep-Dive)

Instead of telling the client it wasn't possible, I took the initiative to redesign the architecture. First, I analyzed their regional data requirements and audited our encryption pipeline. Originally, I considered sticking to the original, single-region key infrastructure that was currently in place. After evaluating both approaches, I concluded that continuing with single-region keys would increase operational complexity as more advertisers required multi-region access. Additionally, it would complicate maintenance of IAM and key policies. I proposed and implemented a migration from single-region keys to AWS Multi-Region KMS keys. To ensure absolute data security, I collaborated closely with a Technical Program Manager (TPM) to architect strict IAM roles and least-privilege Key Policies, allowing the advertiser secure decryption capabilities without exposing adjacent data. Finally, to ensure this wasn't just a one-off fix, I wrote a comprehensive Standard Operating Procedure (SOP) document outlining how to onboard new or existing clients who needed access to this information.

---

## 📊 3. Result & Data Engineering Pivot

The redesign successfully unblocked the client, enabling them to launch their multi-region targeted campaign on schedule. Furthermore, the SOP I created reduced the engineering onboarding time for future advertisers with similar multi-region requirements by **several days to a few hours**, transforming a complex security bottleneck into a scalable self-service process for the team. This experience reinforced that ownership doesn't end when the immediate customer problem is solved. By designing a scalable architecture and documenting the onboarding process, I reduced future operational effort while keeping customer data secure.

---

## 4. Potential Followup Questions

1. “Why did you choose Multi-Region KMS keys instead of replicating the data or using cross-region key replication manually?”
	1. “Manually replicating data or keys creates a massive maintenance overhead and increases the risk of split-brain scenarios or synchronization lag. AWS Multi-Region keys allow the same key material to exist in multiple regions. This meant we could decrypt the data locally in the client's target regions with minimum latency, zero data duplication costs, and completely consistent envelope encryption.”
2. Why did Multi-Region KMS solve the problem?
	1. The immediate issue wasn't encryption itself—it was that our existing key management approach assumed workloads stayed within a single AWS region. Multi-Region KMS let us replicate cryptographic keys across regions while maintaining consistent key material, which simplified our architecture and reduced the operational overhead of managing separate keys and policies."
3. “How did you ensure compliance and data security when granting an external advertiser decryption access?”
	1. “Security was the highest priority. The IAM roles and Key Policies were scoped down strictly to the specific resource ARNs and client principals. We used condition keys to enforce that decryption requests only originated from their specific VPCs and AWS accounts. Furthermore, we enabled comprehensive AWS CloudTrail logging on the KMS keys, so every single decryption event was fully auditable by our security team.”
4. “What was the biggest challenge you faced during this process?”
	1. “The biggest challenge was aligning the security requirements of Amazon Ads with the client's aggressive campaign timeline. Navigating the legal and security governance framework for data sharing takes time. I overcame this by proactively looping in the TPM early, presenting a fully mapped-out IAM policy architecture on day one, which sped up the security review and approval process significantly.”
5. “If you had to build this system again from scratch today, what would you do differently?”
	1. “While the SOP was a great step for scaling, if I did it today, I would completely automate the infrastructure provisioning. I would abstract the KMS multi-region key setup, IAM roles, and policy generation into an Infrastructure as Code (IaC) template using AWS CDK or Terraform. This would turn the onboarding process into a simple, single-click CI/CD pipeline deployment instead of a manual SOP.”