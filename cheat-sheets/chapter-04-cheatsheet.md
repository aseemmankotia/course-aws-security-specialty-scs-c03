# IAM Policy Evaluation Logic Cheat Sheet

## Policy Evaluation Order (Memorize This)

| Priority | Policy Type | Effect |
|----------|-------------|--------|
| 1 | **Explicit Deny** | Overrides everything - game over |
| 2 | **Service Control Policies (SCP)** | Organization/OU boundaries |
| 3 | **Resource-based policies** | S3 bucket policies, SNS topic policies, etc. |
| 4 | **Permissions boundaries** | Maximum ceiling for user/role |
| 5 | **Session policies** | For STS temporary credentials |
| 6 | **Identity-based policies** | User/role/group policies |

**Default: DENY** - Requests denied unless explicitly allowed

## Decision Flow Algorithm

```
1. Is there an explicit DENY anywhere?
   YES → Request DENIED (stop here)
   NO → Continue

2. Is there an SCP DENY or lack of SCP ALLOW?
   YES → Request DENIED
   NO → Continue

3. For resource-based policy evaluation:
   - If resource policy allows → Request ALLOWED (for cross-account)
   - If no resource policy → Continue

4. Check permissions boundary (if exists):
   - Does boundary allow action? NO → DENIED
   - YES → Continue

5. Check session policy (if using STS):
   - Does session allow action? NO → DENIED
   - YES → Continue

6. Check identity policy:
   - Does identity policy allow? YES → ALLOWED
   - NO → DENIED (implicit deny)
```

## Cross-Account Access Requirements

| Scenario | Source Account Needs | Target Account Needs |
|----------|---------------------|---------------------|
| **S3 Bucket Access** | Identity policy: `s3:GetObject` on `arn:aws:s3:::bucket/*` | Bucket policy: Allow principal from source account |
| **Role Assumption** | Identity policy: `sts:AssumeRole` on target role ARN | Trust policy: Allow source principal in role's trust relationship |
| **SNS Topic Publishing** | Identity policy: `sns:Publish` on topic ARN | Topic policy: Allow source principal |

**Exam Gotcha:** Both sides must allow. Missing target resource policy = Access Denied.

## Policy Type Behaviors

| Policy Type | Can Grant Permissions? | Can Deny Permissions? | Scope |
|-------------|------------------------|----------------------|-------|
| **Identity-based** | ✅ Yes | ✅ Yes | Attached to users/roles/groups |
| **Resource-based** | ✅ Yes | ✅ Yes | Attached to resources (S3, SNS, SQS, etc.) |
| **Permissions Boundary** | ❌ NO - only restricts | ✅ Yes | Maximum ceiling for user/role |
| **Session Policy** | ❌ NO - only restricts | ✅ Yes | Passed during STS AssumeRole |
| **SCP** | ❌ NO - only restricts* | ✅ Yes | Organization/OU level |

*SCPs use deny-list (allow all except X) or allow-list (deny all except X) approach

## Common CloudTrail Error Codes

| errorCode | errorMessage Contains | Likely Cause |
|-----------|----------------------|-------------|
| `AccessDenied` | "not authorized to perform" | Check for explicit deny or missing allow |
| `AccessDenied` | Cross-account scenario | Missing resource policy in target account |
| `AccessDenied` | STS AssumeRole | Missing trust policy in target role |
| `InvalidClientTokenId` | "security token" | Credentials invalid/expired |
| `UnauthorizedOperation` | EC2 specific | Missing EC2 permissions |

## Permissions Boundary Examples

**Example: Maximum S3 Read-Only**
- Identity policy: `s3:*`
- Boundary: `s3:Get*`, `s3:List*`
- **Effective:** Only Get and List (intersection)

**Example: Maximum Regional Restrictions**
- Identity policy: `ec2:*`
- Boundary: Allow ec2:* with condition `aws:RequestedRegion = us-east-1`
- **Effective:** EC2 actions only in us-east-1

## Session Policy Behavior

```python
# Assume role with session policy
sts.assume_role(
    RoleArn='arn:aws:iam::TARGET:role/MyRole',
    SessionPolicy=json.dumps({
        "Version": "2012-10-17",
        "Statement": [{
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::specific-bucket/*"
        }]
    })
)
```

**Effective Permissions:** INTERSECTION of (Role's identity policy AND Session policy)

## IAM Policy Simulator Use Cases

- Test new policies before production deployment
- Troubleshoot "Why was this denied?"
- Validate SCPs, boundaries, and session policies
- Test specific action + resource combinations

**Exam Tip:** Simulator is for TESTING permissions. Access Analyzer is for finding EXTERNAL resource sharing.

## Quick Troubleshooting Checklist

1. ✅ Check CloudTrail for `errorCode` and `errorMessage`
2. ✅ Look for **explicit deny** in ANY policy type
3. ✅ If in Organization: Check **SCPs** at OU and Org root
4. ✅ If cross-account: Verify **both** identity policy AND resource policy
5. ✅ If using STS: Check **session policy** isn't over-restricting
6. ✅ Check for **permissions boundary** limiting max permissions
7. ✅ Use **IAM Policy Simulator** to test combined effect

## Exam Traps to Watch For

⚠️ **"User has s3:* in identity policy but can't delete buckets"** → Check for SCP or boundary restricting

⚠️ **"Cross-account access fails with identity policy in place"** → Missing resource policy in target account

⚠️ **"Assumed role can't access resource"** → Session policy or role's identity policy missing permission

⚠️ **"Access denied despite explicit allow"** → There's an explicit deny somewhere overriding it
