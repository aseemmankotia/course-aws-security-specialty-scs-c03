# SCS-C03 Test-Taking Strategy Cheat Sheet

## Exam Format Numbers
| Metric | Value | Strategy |
|--------|-------|----------|
| Total time | 170 minutes | Bank time early: 2 min/question pace |
| Total questions | 65 questions | Flag questions >3 minutes |
| Time per question | 2.6 min average | Complex scenarios: 4-5 min budgeted |
| Passing score | 750/1000 scaled | ~72% correct (adjusted for difficulty) |
| Multiple choice % | 70% of exam | Fastest points - prioritize these |
| Multiple response % | 20% of exam | 2-3 correct answers required |
| Ordering/Matching % | 10% of exam | 5% each - know evaluation orders |

## Domain Weight Distribution
| Domain | Weight | Study Priority If Below 72% |
|--------|--------|-----------------------------|
| Incident Response | 24-26% | CRITICAL - highest weight |
| Data Protection | 22-24% | CRITICAL - second highest |
| Identity & Access | 20-22% | HIGH |
| Infrastructure Security | 18-20% | MEDIUM |
| Management & Governance | 16-18% | MEDIUM |

## Time Management Matrix
| Scenario | Action | Time Allocation |
|----------|--------|------------------|
| First pass through exam | Answer confident questions, flag uncertain | 110-120 minutes (40 min banked) |
| 45 min remaining, 20 questions left | Answer all unanswered first, then flagged | 2-min pace on new, 3-min on flagged |
| 15 min remaining, 10 flagged | Review 50/50 guesses first, skip unclear scenarios | 1.5 min max per flag |
| <5 min remaining | Ensure ALL questions answered (random guess if needed) | No blanks - scaled scoring penalizes omissions |

## Question Type Strategies

### Multiple Choice (1 correct answer)
**Elimination Priority:**
1. Eliminate technically impossible answers (wrong service, exceeds limits)
2. Eliminate answers that don't meet ALL requirements
3. Between 2 options: choose the one that explicitly addresses constraint keywords

### Multiple Response (2-3 correct)
**When certain about N answers:**
- 3/3 certain → answer immediately
- 2/3 certain → eliminate wrong CATEGORY (preventive vs detective), guess from remaining
- 1/3 certain → flag for review, attempt other questions first
- 0/3 certain → educated guess based on most common answer patterns (IAM, encryption, logging)

### Ordering Questions
**Common sequences to memorize:**
- IAM evaluation: SCPs → Permissions boundaries → Identity policies → Session policies → Resource policies
- Incident response: Detect → Analyze → Contain → Eradicate → Recover
- S3 encryption priority: Bucket key → SSE-KMS → SSE-S3
- CloudFront evaluation: WAF → Lambda@Edge → Origin request

### Matching Questions
**Deduction heuristic:**
- Specific service name → specific/technical use case
- Manager/Hub/Orchestrator service → centralized/management use case
- Example: Security Hub (management) = centralized findings; GuardDuty (specific) = threat detection

## Scenario Question Keyword Analysis

| Keyword Pair | Indicates Conflict | Correct Answer Must Address |
|--------------|-------------------|-----------------------------|
| "least cost" + "high availability" | Budget vs resilience | Multi-AZ with minimal redundancy (not multi-region) |
| "prevent" + "break-glass" | Security vs emergency access | Control with emergency bypass (e.g., SCP + emergency role) |
| "automatic" + "approval required" | Automation vs governance | Workflow with auto-trigger, manual approval gate |
| "real-time" + "least overhead" | Speed vs simplicity | Managed service with native integration (not custom) |
| "compliance" + "existing apps" | Standards vs legacy | Retrofittable control (SCPs, not app refactor) |

## Flag & Review Decision Tree
```
Question taking >3 minutes?
├─ YES → Flag it
└─ NO → Continue
    |
    Eliminated to 2 options?
    ├─ YES, 70%+ confident → Answer now
    └─ YES, 50/50 guess → Flag it
        |
        Multiple response with <2 known answers?
        ├─ YES → Flag it
        └─ NO → Answer with best guess
```

## Common Wrong Answer Patterns (Elimination Fast-Track)
1. **Overengineered solutions** - Custom Lambda when native service exists
2. **Incomplete solutions** - Addresses only 1 of 2 explicit requirements
3. **Wrong control type** - Detective control when preventive required
4. **Deprecated services** - EC2-Classic, CloudHSM Classic (not CloudHSM)
5. **Impossible configurations** - S3 bucket in different region than KMS key (not supported)

## Score Improvement Strategy
**If practice domain scores show:**
- **All domains 65-75%:** Focus weak domains (below 72%) until threshold, maintain strong
- **1-2 domains <60%:** Allocate 70% study time to weakest domains - these drag scaled score
- **1 domain <50%:** Deep dive that domain exclusively until 65%+, risk of auto-fail
- **All domains >75%:** Practice exam simulation for timing, flag discipline

## Final 15-Minute Review Triage
1. **Quick wins (6-8 min):** Multiple choice flagged with 50/50 eliminated
2. **Medium wins (4-5 min):** Multiple response with 2/3 known answers
3. **Long shots (only if time):** Unclear scenario questions
4. **Never:** Leave questions blank - always guess if out of time
