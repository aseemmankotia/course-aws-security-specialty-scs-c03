## Chapter 2: Incident Response: Automated Playbooks & Forensics — Practice Questions

**Q1.** A security engineer receives an alert that an EC2 instance is communicating with a known command-and-control server. The engineer needs to immediately isolate the instance while preserving forensic evidence. Which approach BEST meets these requirements?

A) Terminate the instance immediately and create an AMI from the terminated instance for analysis
B) Use Lambda triggered by EventBridge to replace the instance's security group with one that denies all traffic, create an EBS snapshot, and tag the instance as quarantined
C) Stop the instance, detach the EBS volume, create a snapshot, then start the instance in an isolated VPC
D) Use AWS Systems Manager Session Manager to run 'iptables -P INPUT DROP' on the instance and notify the security team

<details>
<summary>Answer</summary>

**Correct: B**

Replacing security groups (not just modifying them) provides immediate network isolation without data loss. EBS snapshots capture block-level forensic evidence. Tagging enables tracking. Lambda automation ensures consistent response within seconds of detection. Instance remains running for memory capture.

**Why the others are wrong:**
- Terminating destroys memory state and running processes. AMIs can't be created from terminated instances. This violates evidence preservation.
- Stopping the instance loses volatile memory evidence. Moving to isolated VPC requires stop/start which destroys memory. Time-consuming manual process defeats automation requirement.
- iptables rules can be modified by attacker with root access. Session Manager requires network connectivity which may be compromised. Not automated via EventBridge.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An organization's incident response playbook requires capturing memory dumps from compromised EC2 instances running Amazon Linux 2. The instances are in private subnets with no internet access. Which solution enables automated memory capture?

A) Use SSM Run Command with SSM Agent to execute 'dd if=/dev/mem' and upload to S3 via VPC endpoint
B) Deploy EC2 Image Builder to create custom AMIs with memory capture tools pre-installed
C) Use Lambda to execute 'aws ec2 get-console-output' and store the output in CloudWatch Logs
D) Enable EC2 hibernation on all instances and use the hibernation file as the memory dump

<details>
<summary>Answer</summary>

**Correct: A**

SSM Run Command executes on instances via SSM Agent without SSH. Works in private subnets via VPC endpoints for SSM and S3. 'dd if=/dev/mem' or tools like LiME capture memory. S3 VPC endpoint enables upload without internet. Automatable via Lambda/EventBridge.

**Why the others are wrong:**
- Image Builder creates AMIs for future launches, doesn't capture memory from running compromised instances. Pre-installing tools doesn't trigger capture during incident.
- get-console-output only retrieves serial console output (last 64KB), not memory dumps. Console output is boot messages, not RAM contents.
- Hibernation saves memory to EBS for resume, but requires pre-enabling before compromise. Can't be enabled on running instances. Not designed for forensic capture.

</details>

---

**Q3.** A company discovers an IAM access key was exposed in a public GitHub repository 48 hours ago. CloudTrail shows the key was used to access S3 buckets and launch EC2 instances. What is the MOST comprehensive immediate response action?

A) Delete the IAM user immediately to prevent further access and preserve evidence for forensic analysis
B) Deactivate the compromised access key using 'aws iam update-access-key --status Inactive', review CloudTrail logs for unauthorized activity, and rotate to a new key after investigation
C) Create an SCP denying all actions for the compromised access key ID using the aws:userid condition key in the policy statement
D) Run 'aws sts get-session-token --serial-number' to revoke all active sessions associated with the compromised credentials and force re-authentication

<details>
<summary>Answer</summary>

**Correct: B**

Deactivating the access key immediately stops unauthorized access while preserving the IAM user and associated CloudTrail logs for forensic investigation. The 'aws iam update-access-key --status Inactive' command is the proper method to disable a specific access key. Reviewing CloudTrail logs helps identify the scope of the breach, and rotating credentials should occur after understanding what was compromised. This approach balances immediate threat containment with evidence preservation and proper incident response procedure.

**Why the others are wrong:**
- Deleting the IAM user immediately may destroy valuable forensic evidence and could disrupt legitimate services if the user has multiple access keys or is used by applications. Evidence preservation is a critical component of incident response, and immediate deletion prevents thorough investigation of the breach scope.
- SCPs (Service Control Policies) apply at the organization/OU level and cannot target specific access key IDs. The aws:userid condition refers to the unique IAM user ID, not individual access keys. Additionally, SCPs are not the appropriate tool for emergency credential revocation - they're designed for preventive guardrails, not incident response.
- The 'aws sts get-session-token' command is used to obtain temporary credentials for MFA authentication, not to revoke sessions. AWS does not provide a direct API call to revoke all sessions; session revocation is achieved by changing credentials, attaching explicit deny policies with a date condition, or deleting/disabling the principal. This command would not address the compromised long-term access key.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** During forensic analysis of a compromised EC2 instance, a security analyst needs to share an EBS snapshot with a third-party forensics firm's AWS account while maintaining chain of custody. What is the MOST secure approach?

A) Create an unencrypted snapshot, share it via snapshot permissions to the forensics account, and provide SHA-256 hash of snapshot
B) Create an encrypted snapshot using customer-managed KMS key, modify key policy to grant forensics account kms:CreateGrant and kms:Decrypt, share snapshot with cross-account permissions, and provide snapshot ID and timestamp in audit log
C) Copy the snapshot to S3 as a file using 'aws ec2 export-snapshot', enable S3 Object Lock compliance mode, then grant forensics account s3:GetObject via bucket policy
D) Use AWS Backup to create a recovery point, share the recovery point with forensics account via AWS Organizations, and enable CloudTrail logging

<details>
<summary>Answer</summary>

**Correct: B**

Encrypted snapshots prevent unauthorized access during transit. Customer-managed KMS key allows granular access control. KMS key policy modifications are logged in CloudTrail. Cross-account snapshot sharing is native AWS feature with audit trail. Snapshot ID + timestamp establish chain of custody. kms:CreateGrant allows forensics account to use snapshot.

**Why the others are wrong:**
- Unencrypted snapshots expose evidence to anyone with snapshot permissions. SHA-256 hash proves integrity but doesn't prevent unauthorized access. No encryption violates security best practices.
- EC2 doesn't support native 'export-snapshot' command. Snapshots can't be directly copied to S3 as files. This solution is technically impossible.
- AWS Backup recovery points are for backup/restore, not forensic sharing. Organizations-based sharing requires forensics firm to be in same organization (unlikely). Doesn't address encryption or access control specifics.

</details>

---

**Q5.** A company's incident response playbook states that when an S3 bucket is accidentally set to public, automated containment must occur within 60 seconds. Which solution meets this requirement?

A) Create EventBridge rule matching 's3:PutBucketAcl' and 's3:PutBucketPolicy' events, trigger Lambda to apply bucket policy with explicit Deny for Principal '*', and add 'aws:AccountNotEquals' condition
B) Enable S3 Block Public Access at account level and use AWS Config rule 's3-bucket-public-read-prohibited' with auto-remediation
C) Use GuardDuty S3 Protection findings to trigger SNS notification, which invokes Lambda to modify bucket policy
D) Create SCP that prevents 's3:PutBucketAcl' and 's3:PutBucketPolicy' actions for all principals

<details>
<summary>Answer</summary>

**Correct: A**

EventBridge detects S3 API calls in near-real-time (seconds). Lambda executes in <5 seconds. Explicit Deny overrides all allows. aws:AccountNotEquals allows own account access while blocking external. Meets 60-second requirement through event-driven architecture.

**Why the others are wrong:**
- Block Public Access prevents public access but is preventive control, not incident response. Config rules evaluate periodically (up to 24 hours) or on changes, too slow for 60-second SLA. Doesn't meet timing requirement.
- GuardDuty S3 Protection detects unusual access patterns, not configuration changes. Detection occurs after hours/days of machine learning analysis. SNS adds latency. Far exceeds 60-second requirement.
- SCP prevents future misconfigurations but doesn't remediate existing public buckets. Prevention ≠ incident response. Doesn't meet 'containment' requirement for already-public buckets.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** An EC2 instance in a public subnet is compromised and needs immediate network isolation. The instance must remain running for memory forensics. Current security group allows SSH (port 22) and HTTPS (port 443). Which combination of actions provides FASTEST isolation while maintaining instance state?

A) Create NACL rule denying all traffic for the instance's subnet, then create EBS snapshot
B) Modify existing security group to remove all inbound and outbound rules
C) Use 'aws ec2 modify-instance-attribute' to replace security group with isolation-sg (no rules), and add NACL Deny rule for instance's IP
D) Move instance to private subnet using 'aws ec2 modify-instance-attribute --subnet-id'

<details>
<summary>Answer</summary>

**Correct: C**

modify-instance-attribute with --groups replaces security groups without stopping instance. Empty security group (isolation-sg) denies all traffic immediately. NACL adds defense-in-depth at subnet level for instance's IP. Both actions preserve instance state and memory. Can be automated via Lambda in <10 seconds.

**Why the others are wrong:**
- NACL rules affect entire subnet, would isolate all instances in subnet, not just compromised one. Overly broad impact. NACL rules are stateless (require explicit in/out rules) and have order dependencies.
- Modifying existing security group affects all instances using that SG. If SG is shared, legitimate instances lose connectivity. Doesn't isolate single instance. Changing shared resources violates blast radius principle.
- Can't change subnet of running instance. Requires stop/start which destroys memory state. Violates 'remain running for forensics' requirement. This command fails.

</details>

---

**Q7.** A security team needs to implement automated credential rotation when IAM access keys are used from unauthorized geographic locations. GuardDuty generates 'UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration' findings. Which solution provides automated response with least operational overhead?

A) Create EventBridge rule matching GuardDuty finding type, trigger Step Functions workflow that: 1) calls IAM PutUserPolicy to Deny all actions for 24 hours, 2) uses SNS to notify user to rotate keys, 3) creates support ticket
B) Enable AWS Security Hub automatic action to invoke Lambda when finding arrives, Lambda executes: DeleteAccessKey, CreateAccessKey, SendEmail with new key
C) Use EventBridge rule to trigger Lambda that: 1) deactivates access key via UpdateAccessKey, 2) attaches explicit Deny policy to user using aws:userid condition, 3) sends SNS notification with rotation instructions, 4) logs to S3 with timestamp
D) Configure GuardDuty auto-remediation in finding preferences to automatically rotate credentials using AWS Secrets Manager

<details>
<summary>Answer</summary>

**Correct: C**

UpdateAccessKey with Inactive status immediately stops key use while preserving for audit. Explicit Deny on aws:userid blocks all sessions even with active key. SNS provides notification. S3 logging establishes chain of custody. Doesn't auto-create new key (security risk). User/security team controls rotation. Automated containment, manual recovery.

**Why the others are wrong:**
- PutUserPolicy with time-based Deny (24 hours) is not standard IAM feature. Requires complex policy logic. SNS email with manual rotation defeats 'automated' requirement. Support ticket creation adds unnecessary overhead and delay.
- DeleteAccessKey destroys only credential evidence before investigation. CreateAccessKey and emailing new key in plaintext is massive security violation. Automated key creation without human verification risks compromise escalation. Security Hub 'automatic action' is not a feature.
- GuardDuty doesn't have 'auto-remediation' or 'finding preferences' for credential rotation. Secrets Manager rotates database/service credentials, not IAM access keys. This solution is fictional.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** An organization uses AWS Security Incident Response to manage security events. A critical incident requires preserving EC2 instance state, network traffic captures, and CloudTrail logs for legal investigation with evidence integrity verification. Which capabilities does AWS Security Incident Response provide?

A) Automated EBS snapshot creation, VPC Flow Logs export to S3 with SSE-KMS, CloudTrail log file integrity validation, and case management with access audit trail
B) Integration with AWS Support for Premium Support case escalation, CloudWatch Logs Insights queries for threat hunting, and automated EC2 instance quarantine via security group replacement
C) Built-in digital forensics tools for memory analysis, network packet capture via Traffic Mirroring, and blockchain-based evidence chain of custody
D) Automated incident response runbooks, AWS Systems Manager Incident Manager integration, and AWS Audit Manager compliance report generation

<details>
<summary>Answer</summary>

**Correct: A**

AWS Security Incident Response (launched 2024) provides guided incident response with automated evidence collection: EBS snapshots for forensics, VPC Flow Logs export, CloudTrail log file validation (SHA-256 + digital signatures), and case management UI with access logging. These features support legal evidence requirements.

**Why the others are wrong:**
- While AWS Support integration exists, CloudWatch Logs Insights is separate service. Security group quarantine is customer-built automation (Lambda/EventBridge), not built-in Security Incident Response feature. Partial credit for support integration.
- AWS Security Incident Response doesn't include memory analysis tools (customer uses SSM + custom tools). Traffic Mirroring is separate VPC feature. Blockchain chain of custody is not implemented (uses CloudTrail + S3 Object Lock instead).
- Incident Manager is separate service for operational incidents (outages), not security incidents. Audit Manager is for compliance audits, not incident response. Runbooks exist but this answer conflates multiple unrelated services.

</details>

---

