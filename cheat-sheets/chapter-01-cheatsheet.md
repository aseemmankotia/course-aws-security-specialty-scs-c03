# SCS-C03 Chapter 1 Cheat Sheet: Multi-Account Threat Detection & OCSF

## GuardDuty Finding Severity
| Severity Label | Numeric Range | Response SLA |
|---|---|---|
| Low | 0.1 - 3.9 | Document, review periodically |
| Medium | 4.0 - 6.9 | Investigate within 24 hours |
| High | 7.0 - 8.9 | Immediate containment + investigation |

**Exam Tip:** Severity 7.0+ requires immediate response. Questions test appropriate response based on severity.

## GuardDuty Multi-Account Architecture
| Approach | Management | Member Enablement | Findings Visibility |
|---|---|---|---|
| Delegated Administrator | Centralized via Org admin | Automatic for Org accounts | Visible in admin account |
| Individual Detectors | Per-account | Manual per account | Per-account only |

**Exam Pattern:** Multi-account aggregation requirement → Delegated admin + Security Hub

## CloudTrail Lake Query Syntax (Presto SQL)
```sql
-- Correct array access for AssumeRole events
SELECT element_at(resources, 1).ARN as role,
       userIdentity.principalId as caller,
       COUNT(*) as total
FROM cloudtrail
WHERE eventName = 'AssumeRole'
GROUP BY 1, 2
ORDER BY total DESC
```

**Common Mistakes:**
- ❌ `resources[0].arn` (wrong syntax)
- ❌ `requestParameters.roleArn` (wrong field)
- ✅ `element_at(resources, 1).ARN` (correct)

**Exam Trap:** Arrays in Presto start at index 1, use `element_at()` function

## Security Service Comparison
| Service | Primary Function | Data Sources | Use Case |
|---|---|---|---|
| GuardDuty | Threat detection | VPC Flow, CloudTrail, DNS logs | Automated threat detection |
| Security Hub | Finding aggregation | GuardDuty, Inspector, Macie, 3rd party | Centralized security posture |
| Detective | Investigation/analysis | VPC Flow, CloudTrail, GuardDuty | Relationship analysis, root cause |
| Security Lake | OCSF log aggregation | All AWS sources + custom | Historical analysis, correlation |

## OCSF (Open Cybersecurity Schema Framework)
**What it is:** Standard schema for normalizing security logs across sources

**AWS Support:**
- VPC Flow Logs: supports OCSF format (network class)
- Security Lake: stores all logs in OCSF format
- Security Hub: ASFF format with OCSF field mapping

**Exam Scenario:** Multi-cloud log correlation OR third-party SIEM integration → OCSF format required

## EventBridge Patterns for Security Automation

**Security Hub CRITICAL findings:**
```json
{
  "source": ["aws.securityhub"],
  "detail-type": ["Security Hub Findings - Imported"],
  "detail": {
    "findings": {
      "Severity": { "Label": ["CRITICAL"] }
    }
  }
}
```

**GuardDuty High severity:**
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [{ "numeric": [7, 8.9] }]
  }
}
```

## Detective Behavior Graph
**Retention:** Up to 1 year
**Auto-ingested sources:** VPC Flow Logs, CloudTrail, GuardDuty (no manual configuration)
**Use case:** Visualize entity relationships and behavior patterns
**NOT for:** Real-time alerting (use GuardDuty/Security Hub instead)

## Common Finding Types & Response
| Finding | Severity | Immediate Action |
|---|---|---|
| InstanceCredentialExfiltration.OutsideAWS | High | Isolate instance + revoke credentials |
| PortProbeUnprotectedPort | Low | Review SG rules, document if expected |
| UnauthorizedAccess:IAMUser/MaliciousIPCaller | Medium-High | Review CloudTrail for actions taken, rotate keys if confirmed |
| CryptoCurrency:EC2/BitcoinTool | High | Isolate instance, investigate for compromise |

## Service Limits & Key Numbers
- **CloudTrail Lake retention:** 7 days - 7 years
- **Detective data retention:** Up to 1 year
- **Security Hub finding retention:** 90 days
- **GuardDuty S3 finding delay:** 6 hour intervals for S3 data events analysis

## Exam Gotchas
1. **GuardDuty Malware Protection:** Detection only, doesn't prevent attacks
2. **Security Hub Custom Actions:** Require manual trigger, not automated
3. **CloudTrail Lake:** Cannot query data beyond retention period
4. **Detective:** For investigation/analysis, not real-time detection
5. **OCSF:** New in C03, expect questions on when OCSF format is required
