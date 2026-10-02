# IAM Troubleshooting & Least Privilege Cheat Sheet

## CloudTrail Error Code Patterns

| Error Code | Meaning | Common Causes |
|------------|---------|---------------|
| AccessDenied | Explicit Deny in policy | SCP deny, identity policy deny, permissions boundary deny, resource policy deny |
| UnauthorizedOperation | Missing Allow in policy | Action not in identity policy, SCP doesn't allow, permissions boundary doesn't allow |
| InvalidPermission.NotFound | Missing dependent permission | iam:PassRole missing when creating resources with roles |

## Policy Evaluation Logic

1. **Explicit Deny** (anywhere) → Request **DENIED**
2. **SCP Allow** → Continue to step 3 (if SCP doesn't allow → denied)
3. **Permissions Boundary Allow** → Continue to step 4 (if exists and doesn't allow → denied)
4. **Identity-based OR Resource-based Allow** → Request **ALLOWED**

**Default:** Everything denied unless explicitly allowed.

## IAM Access Analyzer

### Finding Types
- **External Access:** Resource accessible outside zone of trust
- **Public Access:** Resource accessible by internet
- **Cross-account Access:** Resource shared with specific AWS account

### Policy Generation from CloudTrail
- **Analysis Period:** Up to 90 days of CloudTrail events
- **Output:** Least-privilege policy based on actual API calls
- **Includes:** Actions, resources, conditions used
- **Best Practice:** Analyze 90 days for comprehensive coverage

### Archive Rules
- **Purpose:** Suppress expected external access findings
- **Criteria:** Principal (account/org/federated user), action, resource
- **Result:** Finding archived but remains accessible
- **Use Case:** Trusted partner accounts, cross-account backups

## IAM Policy Simulator Limitations

| Evaluates | Does NOT Evaluate |
|-----------|-------------------|
| Identity-based policies | Resource-based policies on target (S3 bucket policy) |
| Permissions boundaries (if specified) | SCPs (unless included) |
| Inline and managed policies | VPC endpoint policies |
| Condition blocks | Session policies (STS) |

**Exam Trap:** Simulator shows Allow but real request denied → check bucket policy.

## Service Control Policies (SCPs)

### Core Rules
- **Only Deny matters:** Allow statements have no effect
- **Cannot be bypassed:** Even account root cannot override SCP deny
- **Scope:** OU or account level
- **Common Patterns:**

```json
// Block region
{"Effect":"Deny","Action":"*","Resource":"*",
 "Condition":{"StringNotEquals":{"aws:RequestedRegion":["us-east-1","us-west-2"]}}}

// Block root user
{"Effect":"Deny","Action":"*","Resource":"*",
 "Condition":{"StringLike":{"aws:PrincipalArn":"arn:aws:iam::*:root"}}}

// Block specific action
{"Effect":"Deny","Action":"ec2:CreateInternetGateway","Resource":"*"}
```

## Least Privilege Policy Construction

### NotAction Patterns
```json
// Allow everything except dangerous actions
{"Effect":"Allow","NotAction":["iam:*","organizations:*"],"Resource":"*"}

// Two-statement pattern for conditional restrictions
[
  {"Effect":"Allow","NotAction":"ec2:RunInstances","Action":"ec2:*"},
  {"Effect":"Allow","Action":"ec2:RunInstances",
   "Condition":{"StringEquals":{"ec2:InstanceType":["t3.micro","t3.small"]}}}
]
```

### Deny Patterns
```json
// Explicit deny for specific condition
{"Effect":"Deny","Action":"ec2:RunInstances",
 "Condition":{"StringNotEquals":{"ec2:InstanceType":["t3.micro","t3.small"]}}}

// Time-based restrictions
{"Effect":"Deny","Action":"*","Resource":"*",
 "Condition":{"DateGreaterThan":{"aws:CurrentTime":"2026-12-31T23:59:59Z"}}}
```

## iam:PassRole Requirements

**When Required:**
- ec2:RunInstances (passing instance profile)
- lambda:CreateFunction (passing execution role)
- rds:CreateDBInstance (passing monitoring role)
- ecs:CreateService (passing task role)

**Policy Format:**
```json
{"Effect":"Allow","Action":"iam:PassRole",
 "Resource":"arn:aws:iam::123456789012:role/SpecificRole"}
```

**Exam Tip:** PassRole should specify exact role ARN, not wildcard.

## S3 Block Public Access

| Level | Can Override? | Scope |
|-------|---------------|-------|
| Organization | No | All accounts/buckets |
| Account | Yes (if not org-level) | All buckets in account |
| Bucket | Yes (if not account/org-level) | Specific bucket |

**Strongest Protection:** Organization-level (cannot be overridden).

## Resource Control Policies (RCPs)

- **Purpose:** Control which principals can access resources in your organization
- **Perspective:** Resource-centric (vs SCP identity-centric)
- **Use Case:** Prevent external access to organization resources
- **Introduced:** 2025
- **Exam Note:** Newer feature, may appear in scenario questions about org-wide resource restrictions
