## Chapter 10: Full Practice Exam & Test-Taking Strategy — Practice Questions

**Q1.** You have 45 minutes remaining with 20 questions left (12 unanswered, 8 flagged). Domain breakdown shows: Incident Response 60%, Identity 80%, Infrastructure 50%, Data Protection 70%, Management 40%. What is the BEST test-taking strategy?

A) Review all 8 flagged questions first since you already spent time on them, then tackle the 12 new questions
B) Answer all 12 unanswered questions first (18 minutes), then spend remaining time on the 8 flagged questions, prioritizing Management and Infrastructure domains
C) Skip the flagged questions entirely, focus on the 12 unanswered questions, and return to flagged only if time permits
D) Spend equal time (2.25 min) on each remaining question regardless of domain performance

<details>
<summary>Answer</summary>

**Correct: B**

With 2.25 min/question available, answering unanswered questions first maximizes your score since flagged questions often indicate uncertainty. Prioritizing weak domains (Management 40%, Infrastructure 50%) on flagged review targets where you need points most. Unanswered questions = guaranteed 0 points; flagged = possible partial credit if educated guess.

**Why the others are wrong:**
- Reviewing flagged first wastes time on questions you're already uncertain about while leaving guaranteed-zero unanswered questions unaddressed
- Skipping flagged questions means abandoning 8 potential points when you have adequate time (2.25 min/question) to attempt them
- Equal time distribution ignores domain scoring - you need more correct answers in weak domains (Management, Infrastructure) to reach the 750 scaled score threshold

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A multiple response question asks: 'Which THREE security controls help prevent S3 data exfiltration?' with 6 options listed. You are certain about 2 answers (S3 Block Public Access, VPC Endpoints) but unsure about the third. What elimination strategy maximizes your probability of selecting the correct third answer?

A) Eliminate options that are detective controls rather than preventive controls, then select from remaining options
B) Choose the most complex-sounding option among the remaining 4 since AWS exam answers tend to favor comprehensive solutions
C) Select the option that involves IAM policies since identity controls appear in most S3 security questions
D) Pick the option mentioning the newest AWS service since exams test current best practices

<details>
<summary>Answer</summary>

**Correct: A**

The question specifically asks for PREVENTIVE controls. Elimination based on control category (preventive vs detective vs corrective) is the most reliable strategy. Detective controls (CloudTrail, GuardDuty, Macie) monitor but don't prevent. This narrows your 4 uncertain options significantly. Third answer is likely SCPs, S3 Bucket Policies, or PrivateLink - all preventive.

**Why the others are wrong:**
- Complexity bias is a test-taking fallacy - AWS exam correct answers are often the simplest solution that meets requirements, not the most complex
- IAM appears frequently but not in EVERY S3 question - this is confirmation bias. Resource-based policies, SCPs, or network controls may be the answer
- Recency bias (newest service) is unreliable - the exam tests foundational services heavily. S3 Block Public Access (2018) appears more than newer features

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** An ordering question shows 5 IAM policy evaluation steps and asks you to arrange them in the correct sequence: (A) Resource-based policies (B) SCPs (C) Session policies (D) Identity-based policies (E) Permissions boundaries. You must select the evaluation order. What is the CORRECT sequence?

A) B (SCPs) → E (Permissions boundaries) → C (Session policies) → D (Identity-based policies) → A (Resource-based policies)
B) B (SCPs) → E (Permissions boundaries) → D (Identity-based policies) → C (Session policies) → A (Resource-based policies)
C) D (Identity-based policies) → E (Permissions boundaries) → B (SCPs) → C (Session policies) → A (Resource-based policies)
D) A (Resource-based policies) → B (SCPs) → E (Permissions boundaries) → D (Identity-based policies) → C (Session policies)

<details>
<summary>Answer</summary>

**Correct: B**

IAM evaluation: 1) Deny by default 2) SCPs (organization boundary) 3) Permissions boundaries (maximum permissions) 4) Identity-based policies (what principal can do) 5) Session policies (temporary restriction) 6) Resource-based policies (who can access resource). For this question without resource policies in final position, B→E→D→C is correct. SCPs override everything, boundaries limit identity policies, session policies further restrict.

**Why the others are wrong:**
- Places session policies before identity-based policies - session policies can only restrict, not expand identity policies, so they evaluate AFTER
- Starts with identity-based policies - SCPs and permissions boundaries are evaluated BEFORE identity policies in the hierarchy
- Places resource-based policies first - resource policies are evaluated separately and can override denies in other policies, but organizational boundaries (SCPs) take precedence

</details>

---

**Q4.** A company must comply with PCI DSS requiring encryption key rotation every 365 days, audit logs retained for 7 years, and production database access restricted to on-call personnel only during incidents. Which combination of AWS services BEST meets ALL requirements?

A) AWS KMS automatic rotation, CloudTrail with S3 Glacier for log retention, IAM temporary security credentials via AWS STS
B) AWS KMS with manual annual rotation, CloudWatch Logs with 7-year retention, IAM roles with time-based conditions in policies
C) AWS CloudHSM with automated rotation, S3 with lifecycle policy to Glacier Deep Archive, AWS SSO with session duration limits
D) AWS Secrets Manager rotation, VPC Flow Logs with Athena retention, IAM Identity Center with attribute-based access control (ABAC)

<details>
<summary>Answer</summary>

**Correct: B**

KMS automatic rotation is 365 days but question states manual compliance verification; CloudWatch Logs supports 7-year retention (CloudTrail stores in S3 only 90 days default); IAM time-based conditions (aws:CurrentTime) restrict access to specific time windows matching on-call schedules. This explicitly addresses all three requirements with least complexity.

**Why the others are wrong:**
- CloudTrail to S3 Glacier works for retention but STS temporary credentials don't provide time-based access control - credentials work anytime within validity period (not incident-only)
- CloudHSM is overkill and 10x more expensive than KMS for key rotation; session duration limits don't restrict access to incident-time only
- Secrets Manager rotates secrets not KMS keys; VPC Flow Logs don't provide audit trails for API calls (wrong log type); ABAC doesn't inherently provide time-based restrictions

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** During the exam, you encounter a matching question: 'Match each AWS service to its primary security use case' with 6 services and 6 use cases. You confidently match 4 pairs but are uncertain about the remaining 2 (AWS Firewall Manager and AWS Network Firewall). Both remaining use cases are: (X) Centralized management of WAF rules across accounts (Y) Stateful layer 4-7 traffic filtering for VPC. What strategy should you use?

A) Match based on service name similarity - Firewall Manager sounds like management, Network Firewall sounds like filtering
B) Apply the 'most restrictive control' heuristic - assign the more specific use case (stateful filtering) to Network Firewall, and broader use case (centralized management) to Firewall Manager
C) Skip both matches and flag for review, focusing time on higher-point questions rather than 50/50 guesses
D) Match alphabetically - Firewall Manager to the first remaining use case, Network Firewall to the second

<details>
<summary>Answer</summary>

**Correct: B**

Service name analysis combined with control specificity works here: Network Firewall = specific network security appliance for stateful inspection at VPC level (layer 4-7). Firewall Manager = centralized policy management tool (manages WAF, Shield, Network Firewall across AWS Organizations). The heuristic 'specific service → specific use case, management service → broad use case' is reliable for AWS service matching.

**Why the others are wrong:**
- Name similarity alone is risky - while it works HERE, it fails with services like AWS Config (not configuration management) vs Systems Manager (is configuration management)
- Skipping guarantees 0 points when you can deduce the answer logically - matching questions often have 1-2 points per pair, worth attempting with educated reasoning
- Alphabetical matching is pure guessing (50% chance) when logical deduction (80%+ chance with service knowledge) is available

</details>

---

**Q6.** You finish the exam with 15 minutes remaining and have flagged 10 questions. Your mental tracking shows: 5 flags are multiple choice where you eliminated it down to 2 options, 3 flags are multiple response where you're unsure about 1 of the 3 required answers, 2 flags are scenario questions you didn't fully understand. How should you allocate your 15 minutes?

A) Spend 3 minutes re-reading each scenario question thoroughly, then 1 minute each on the multiple choice and response questions
B) Review all 5 multiple choice flags first (6 minutes), then 3 multiple response (6 minutes), skip scenarios if no time remains
C) Allocate time proportionally: 45 seconds per multiple choice (3.75 min), 1.5 minutes per multiple response (4.5 min), 3.5 minutes per scenario (7 min)
D) Focus only on the 3 multiple response questions (5 minutes each) since they're worth more points than single-answer questions

<details>
<summary>Answer</summary>

**Correct: B**

Multiple choice with 2 eliminated options = 50% guess probability = highest expected value per minute. Multiple response with 2 known correct answers = reasonable chance to deduce third. Scenario questions you didn't understand are lowest probability - re-reading rarely helps if the scenario was unclear initially. Maximize expected points: high-confidence revisions first, low-confidence questions only if time permits.

**Why the others are wrong:**
- Spending 3 minutes per unclear scenario is inefficient - if you didn't understand it initially, more time rarely clarifies without additional knowledge
- Proportional allocation wastes time on low-probability questions (unclear scenarios) - expected value should drive allocation, not equal distribution
- Multiple response questions are NOT worth more points - SCS-C03 uses scaled scoring where each question type contributes to domain scores equally, not weighted by difficulty

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** After completing a practice exam, your domain scores are: Incident Response 68%, Identity 82%, Infrastructure 71%, Data Protection 65%, Management 58%. The passing scaled score is 750 (roughly 72% correct). Which study strategy will MOST efficiently improve your exam readiness?

A) Focus 70% of remaining study time on Management (58%) and Data Protection (65%), 30% on maintaining strong domains
B) Distribute study time equally across all 5 domains to ensure balanced knowledge
C) Deep-dive Identity (82%) to achieve 90%+ mastery, leveraging your strength to offset weaknesses
D) Focus exclusively on Management until it reaches 72%, then move to Data Protection

<details>
<summary>Answer</summary>

**Correct: A**

Scaled scoring requires passing threshold across weighted domains. Management (24-26% of exam) and Data Protection (22-24%) are high-weight domains where you're below passing. Marginal improvement in weak, high-weight domains yields more scaled score points than perfecting already-strong domains. 70/30 split ensures weak domains reach threshold while preventing strong domains from degrading.

**Why the others are wrong:**
- Equal distribution is inefficient - you're wasting time on Identity (already 10 points above threshold) when Management needs 14 points to pass
- Perfecting strong domains has diminishing returns - going from 82% to 90% in Identity (already passing) doesn't help your scaled score as much as bringing Management from 58% to 72%
- Exclusively focusing on one domain risks degradation in other areas and doesn't account for Management's high exam weight - you need both domains above threshold

</details>

---

**Q8.** A scenario question states: 'A financial services company requires that all administrator actions on production systems be approved by a second administrator before execution, with a complete audit trail. The solution must prevent privilege escalation and support emergency break-glass procedures.' The question then lists 4 solutions involving combinations of AWS Organizations SCPs, IAM policies, AWS Systems Manager Session Manager, AWS CloudTrail, and custom Lambda functions. What is the MOST effective elimination strategy?

A) Eliminate any solution that doesn't mention CloudTrail since audit trail is explicitly required
B) Eliminate solutions using Lambda functions first since AWS native services are preferred over custom code
C) Identify the keywords 'approved by second administrator' and 'break-glass' then eliminate solutions that don't address BOTH dual-control AND emergency access
D) Eliminate the longest, most complex solution first since AWS best practices favor simplicity

<details>
<summary>Answer</summary>

**Correct: C**

The question contains TWO critical constraints that create tension: dual approval (restricts access) AND break-glass (emergency bypass). Solutions must address BOTH. Many wrong answers will solve one but not the other. Keywords: 'approved by second' → requires approval workflow (not just logging); 'break-glass' → requires emergency bypass mechanism. Eliminate options missing either component.

**Why the others are wrong:**
- CloudTrail provides audit trails but most answers will include it - it's not a differentiator. The approval workflow and break-glass mechanism differentiate correct from incorrect answers
- Lambda functions are often required for approval workflows - eliminating custom code is a weak heuristic. AWS doesn't have a native dual-approval service, so custom orchestration may be necessary
- Simplicity bias fails on security compliance questions - dual-control systems are inherently complex. The correct answer will likely be one of the longer options implementing approval workflow

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

