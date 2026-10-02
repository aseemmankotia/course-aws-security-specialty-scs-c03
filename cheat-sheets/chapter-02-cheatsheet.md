# Incident Response: Automated Playbooks & Forensics Cheat Sheet

## Automated Incident Containment

### EC2 Instance Isolation
| Method | Command | Impact | Preserves Memory? | Speed |
|--------|---------|--------|-------------------|-------|
| Security Group Replacement | `aws ec2 modify-instance-attribute --instance-id i-xxx --groups sg-isolation` | Single instance | ✅ Yes | <10 sec |
| NACL Deny Rule | `aws ec2 create-network-acl-entry --network-acl-id acl-xxx --rule-number 1 --protocol -1 --rule-action deny --cidr-block 10.0.1.5/32` | Subnet-wide | ✅ Yes | <10 sec |
| Stop Instance | `aws ec2 stop-instances --instance-ids i-xxx` | Single instance | ❌ No | 30-60 sec |
| Terminate | `aws ec2 terminate-instances --instance-ids i-xxx` | Single instance | ❌ No | <10 sec |

**Exam Rule:** Isolation SG = empty inbound/outbound rules. NACL requires TWO rules (inbound + outbound deny). Security groups are stateful, NACLs are stateless.

### EventBridge Incident Response Pattern
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": {
    "severity": [{"numeric": [">=", 7]}],
    "type": ["UnauthorizedAccess:*", "CryptoCurrency:*"]
  }
}
```
**Triggers:** GuardDuty findings ≥7 severity (High/Critical), specific threat types → Lambda for auto-remediation

## Forensic Evidence Collection

### EBS Snapshots for Forensics
| Action | Command | Notes |
|--------|---------|-------|
| Create Snapshot | `aws ec2 create-snapshot --volume-id vol-xxx --description "Forensic-incident-123"` | No downtime |
| Encrypted Snapshot | `aws ec2 create-snapshot --volume-id vol-xxx --kms-key-id arn:aws:kms:...` | Customer-managed KMS required for sharing |
| Share Cross-Account | `aws ec2 modify-snapshot-attribute --snapshot-id snap-xxx --attribute createVolumePermission --operation-type add --user-ids 123456789012` | Recipient needs KMS key access |
| KMS Key Sharing | `aws kms create-grant --key-id xxx --grantee-principal arn:aws:iam::123456789012:root --operations CreateGrant Decrypt` | Required for encrypted snapshots |

**Exam Trap:** Can't share snapshots encrypted with AWS-managed keys (aws/ebs). Must use customer-managed KMS key.

### Memory Capture via SSM Run Command
```bash
aws ssm send-command \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["sudo dd if=/dev/mem of=/tmp/memory.dump bs=1M","aws s3 cp /tmp/memory.dump s3://forensics-bucket/instance-i-xxx-$(date +%s).dump"]' \
  --targets "Key=instanceids,Values=i-xxx"
```
**Requirements:** SSM Agent installed, VPC endpoints (ssm, ec2messages, ssmmessages), S3 VPC endpoint, IAM role with s3:PutObject

## Credential Compromise Response

### IAM Access Key Compromise Actions
| Action | Command | When to Use |
|--------|---------|-------------|
| Deactivate Key | `aws iam update-access-key --user-name User --access-key-id AKIAI... --status Inactive` | During investigation (reversible) |
| Delete Key | `aws iam delete-access-key --user-name User --access-key-id AKIAI...` | After investigation complete |
| Session Revocation | Attach inline policy: `{"Effect":"Deny","Action":"*","Resource":"*","Condition":{"StringEquals":{"aws:userid":"AIDAI..."}}}` | Blocks all sessions immediately |
| Generate New Key | `aws iam create-access-key --user-name User` | After incident closed |

**Exam Key Concept:** `aws:userid` condition blocks ALL sessions from compromised identity (includes console, SDK, CLI). `aws:PrincipalArn` doesn't revoke active sessions.

### Credential Rotation Playbook
1. **Detect:** GuardDuty finding `UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration`
2. **Contain:** Deactivate key (Inactive status)
3. **Isolate:** Apply explicit Deny policy using `aws:userid` condition
4. **Investigate:** CloudTrail query: `LookupEvents --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=AKIAI...`
5. **Remediate:** Rotate key, update applications
6. **Recover:** Remove Deny policy, delete old key

## S3 Bucket Quarantine

### Bucket Policy - Complete Lockdown with Exception
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Principal": "*",
    "Action": "s3:*",
    "Resource": [
      "arn:aws:s3:::exposed-bucket",
      "arn:aws:s3:::exposed-bucket/*"
    ],
    "Condition": {
      "StringNotEquals": {
        "aws:PrincipalArn": "arn:aws:iam::123456789012:role/ForensicsRole"
      }
    }
  }]
}
```
**Result:** Denies all access except ForensicsRole. Explicit Deny overrides all Allows (including bucket owner).

## CloudTrail Log Integrity

| Feature | Purpose | Verification Command |
|---------|---------|---------------------|
| Log File Validation | SHA-256 hash chain (current log + previous hash) | `aws cloudtrail validate-logs --trail-arn arn:aws:cloudtrail:region:account:trail/name --start-time 2024-01-01T00:00:00Z` |
| Digest Files | Hourly hash manifest | Stored in S3: `s3://bucket/AWSLogs/account/CloudTrail-Digest/region/YYYY/MM/DD/` |
| S3 Object Lock | Prevents deletion/modification | Enable on log bucket: `aws s3api put-object-lock-configuration --bucket logs-bucket --object-lock-configuration '...'` |

**Exam Fact:** Digest files delivered every hour. Each digest contains hashes of all log files delivered in that hour + signature. Tampering breaks hash chain.

## Service Timing for Exam

| Detection/Response Method | Typical Time | Use Case |
|---------------------------|--------------|----------|
| EventBridge (CloudTrail events) | 5-15 seconds | API call detection (S3 public, IAM changes) |
| GuardDuty findings | Minutes to hours | Behavioral analysis (credential exfiltration, crypto mining) |
| AWS Config rules | Up to 24 hours | Compliance drift (resource configuration) |
| Lambda execution | 1-3 sec (cold start), <1 sec (warm) | Automated remediation |
| SSM Run Command | 5-30 seconds | Remote command execution (memory capture) |

**Exam Rule:** EventBridge + Lambda = fastest (<30 sec total). Config rules = compliance checking, not incident response speed.

## AWS Security Incident Response Service

**Automated Evidence Collection:**
- EBS snapshots (encrypted with customer KMS)
- VPC Flow Logs export to S3
- CloudTrail logs with integrity validation
- GuardDuty/Security Hub findings
- S3 access logs

**NOT Included (manual via SSM/custom tools):**
- Memory dumps
- Packet captures (requires Traffic Mirroring setup)
- Application logs

**Integration:** AWS Support Premium/Enterprise for escalation, Security Hub for finding aggregation, EventBridge for triggering

## Common Exam Traps

1. **SG Modification vs Replacement:** Modifying shared SG affects all instances. Use `modify-instance-attribute --groups` to replace SG on single instance.
2. **Access Key Deletion:** Deletes evidence. Use `--status Inactive` during investigation.
3. **NACL Isolation:** Requires BOTH inbound and outbound deny rules (stateless). Rule numbers matter (lowest = evaluated first).
4. **Snapshot Sharing:** AWS-managed encrypted snapshots can't be shared. Must use customer-managed KMS key.
5. **Session Revocation:** Deactivating access key doesn't revoke active sessions. Must use explicit Deny policy with `aws:userid` condition.
