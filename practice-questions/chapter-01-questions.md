## Chapter 1: Detection: Multi-Account Threat Detection & OCSF Logging — Practice Questions

**Q1.** A security team needs to aggregate GuardDuty findings from 50 AWS accounts across multiple regions into a single location for analysis. The solution must preserve original finding metadata and enable custom threat correlation rules. Which approach BEST meets these requirements?

A) Configure GuardDuty in each account to send findings to a central S3 bucket using CloudTrail, then use Athena to query the aggregated logs
B) Enable GuardDuty delegated administrator in AWS Organizations with automatic member account enablement, then use Security Hub to aggregate findings
C) Create EventBridge rules in each account to forward GuardDuty findings to a central SNS topic, then process with Lambda
D) Use CloudWatch Logs subscription filters to forward GuardDuty findings to a central CloudWatch log group

<details>
<summary>Answer</summary>

**Correct: B**

Security Hub with GuardDuty delegated administrator provides native multi-account aggregation, preserves full finding metadata including OCSF-normalized fields, supports custom insights for correlation, and automatically enables GuardDuty across member accounts. This is the AWS-native solution designed for exactly this use case.

**Why the others are wrong:**
- GuardDuty findings don't flow through CloudTrail - CloudTrail logs API calls, not GuardDuty findings. This approach misunderstands the data flow.
- While EventBridge can forward findings, it requires custom processing logic for aggregation and correlation, lacks native finding normalization, and doesn't provide the query/insight capabilities of Security Hub.
- CloudWatch Logs subscription filters add complexity and cost, don't preserve GuardDuty's structured finding format, and lack the correlation and compliance mapping features of Security Hub.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An organization detects a GuardDuty finding with severity 8.0 classified as 'UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS'. What is the MOST appropriate immediate response action?

A) Rotate the IAM user's access keys and enable MFA
B) Isolate the EC2 instance by changing its security group to deny all traffic and revoke the instance profile's temporary credentials
C) Create a CloudTrail Lake query to identify all API calls made with the compromised credentials
D) Enable GuardDuty S3 Protection to prevent data exfiltration

<details>
<summary>Answer</summary>

**Correct: B**

Severity 8.0 is High (7.0-8.9 range). InstanceCredentialExfiltration means EC2 instance credentials are being used outside AWS, indicating active credential compromise. Immediate containment requires isolating the compromised instance AND revoking the temporary credentials from the instance profile, which prevents further malicious API calls.

**Why the others are wrong:**
- The finding indicates INSTANCE credentials (temporary credentials from instance profile), not IAM user access keys. Rotating IAM user keys doesn't address the actual compromised credentials.
- While forensic analysis via CloudTrail Lake is important, it's an investigative action, not containment. The compromised credentials are actively being used - investigation comes after containment.
- S3 Protection is a GuardDuty feature for detecting S3 threats, but doesn't stop an active credential compromise. This is reactive monitoring, not incident response containment.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A company must query CloudTrail logs across 20 accounts to identify all 'AssumeRole' API calls where the source IP is outside known corporate IP ranges and the session duration exceeds 8 hours. The query must complete in under 30 seconds. Which solution meets these requirements?

A) Enable CloudTrail Lake, create an event data store aggregating all accounts, then use SQL query: SELECT * FROM cloudtrail WHERE eventName='AssumeRole' AND sourceIPAddress NOT IN (...) AND sessionDuration > 28800
B) Stream CloudTrail logs from all accounts to a central S3 bucket, use Athena to query with: SELECT * FROM cloudtrail_logs WHERE eventname='AssumeRole' AND sourceipaddress NOT LIKE '10.%' AND requestparameters.durationseconds > 28800
C) Configure CloudWatch Insights queries across all account log groups with filter pattern: { $.eventName = "AssumeRole" && $.sourceIPAddress != "10.*" }
D) Use AWS Config aggregator to track IAM role usage and create a custom Config rule to evaluate AssumeRole session duration

<details>
<summary>Answer</summary>

**Correct: A**

CloudTrail Lake provides sub-30-second query performance on multi-account CloudTrail data using purpose-built query engine. The event data store aggregates trails from multiple accounts. Correct SQL syntax for sessionDuration filtering (8 hours = 28800 seconds) and sourceIPAddress NOT IN for IP exclusion.

**Why the others are wrong:**
- Athena on S3 works but typically cannot guarantee sub-30-second performance on large CloudTrail datasets, lacks the optimized indexing of CloudTrail Lake, and session duration is in requestParameters.durationSeconds not a top-level field requiring JSON parsing.
- CloudWatch Insights doesn't natively ingest CloudTrail events (requires log delivery to CloudWatch), filter patterns don't support complex numeric comparisons like session duration, and cross-account querying is complex and slow.
- AWS Config tracks resource configuration changes, not runtime API call patterns. Config rules evaluate resource compliance, not analyze API call metadata like source IP and session duration.

</details>

---

**Q4.** A security engineer needs to analyze network traffic patterns to detect lateral movement between EC2 instances in different VPCs. The solution must identify unusual connection patterns and aggregate flows for analysis. Which combination provides the MOST comprehensive detection capability?

A) Enable VPC Flow Logs for all VPCs, publish to CloudWatch Logs, create CloudWatch Insights queries for connection patterns, use Detective for relationship graphing
B) Enable VPC Flow Logs with OCSF format, aggregate in S3, query with Athena using OCSF network class fields, visualize with QuickSight
C) Configure AWS Network Firewall with intrusion detection, enable flow logs, use GuardDuty to analyze traffic patterns
D) Use Traffic Mirroring to send packet captures to a central analysis instance running open-source IDS, correlate with VPC Flow Logs

<details>
<summary>Answer</summary>

**Correct: A**

VPC Flow Logs capture connection metadata. Detective automatically ingests VPC Flow Logs and creates behavior graphs showing relationships and communication patterns between EC2 instances across VPCs, enabling visual analysis of lateral movement. CloudWatch Insights provides fast querying of flow data. This combination provides both detailed flow analysis and relationship visualization without custom development.

**Why the others are wrong:**
- While OCSF format provides normalized schema, this approach requires significant custom development for pattern analysis and relationship detection. Athena + QuickSight provide querying/visualization but lack Detective's ML-driven behavior graph and automated relationship mapping for lateral movement detection.
- Network Firewall provides perimeter security and basic IDS, but doesn't analyze traffic patterns for lateral movement between instances. GuardDuty analyzes VPC Flow Logs but focuses on known threats (malicious IPs, port scanning), not anomalous connection patterns indicating lateral movement.
- Traffic Mirroring captures full packets (expensive, high volume), requires custom IDS configuration and maintenance, and correlation with Flow Logs adds complexity. Over-engineering for lateral movement detection when Detective provides this capability natively.

</details>

---

**Q5.** An organization using OCSF-normalized logs from GuardDuty, CloudTrail, and VPC Flow Logs needs to create a correlation rule that triggers when a High-severity GuardDuty finding occurs followed by unusual API activity from the same identity within 10 minutes. The rule must execute custom remediation logic. Which solution is MOST operationally efficient?

A) Create two EventBridge rules: one for GuardDuty findings (severity ≥7.0), one for CloudTrail events, use Step Functions to correlate events within 10-minute window and trigger remediation Lambda
B) Send all OCSF-normalized logs to Security Lake, create a correlation rule using Security Lake's query engine, trigger Lambda via EventBridge when correlation matches
C) Use Detective to create a finding group that includes GuardDuty and CloudTrail events, configure Detective to trigger EventBridge when finding group criteria match
D) Stream all logs to a SIEM with native OCSF support, create correlation rule in SIEM, use SIEM webhook to trigger AWS Lambda for remediation

<details>
<summary>Answer</summary>

**Correct: B**

Security Lake is AWS's purpose-built service for OCSF-normalized security data aggregation and correlation. It natively ingests GuardDuty, CloudTrail, and VPC Flow Logs in OCSF format, provides query engine for correlation rules with time-based logic, and integrates with EventBridge for automated response. This is the AWS-native solution designed specifically for OCSF log correlation.

**Why the others are wrong:**
- Requires custom development of correlation logic in Step Functions, manual management of state/timing across multiple EventBridge rules, doesn't leverage OCSF normalization (operates on native AWS event formats), and adds operational complexity.
- Detective creates behavior graphs and finding groups for investigation but doesn't support custom correlation rules with time-based logic or automated response triggering. Detective is for human analysis, not automated correlation and remediation.
- Adds third-party SIEM licensing cost, requires webhook integration back to AWS, introduces additional latency, and doesn't leverage AWS-native OCSF handling. Operationally inefficient compared to Security Lake which is designed for this use case.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A CloudTrail Lake query returns 10,000 rows of AssumeRole events. The security team needs to identify which roles were assumed most frequently and by which IAM identities. Which SQL query CORRECTLY aggregates this data?

A) SELECT requestParameters.roleArn, userIdentity.principalId, COUNT(*) as assume_count FROM cloudtrail WHERE eventName='AssumeRole' GROUP BY requestParameters.roleArn, userIdentity.principalId ORDER BY assume_count DESC LIMIT 100
B) SELECT resources[0].arn, userIdentity.arn, COUNT(*) FROM cloudtrail WHERE eventName='AssumeRole' GROUP BY 1,2 ORDER BY 3 DESC LIMIT 100
C) SELECT responseElements.assumedRoleUser.arn as role_arn, userIdentity.principalId as caller, COUNT(*) as count FROM cloudtrail WHERE eventName='AssumeRole' GROUP BY role_arn, caller ORDER BY count DESC LIMIT 100
D) SELECT element_at(resources, 1).ARN, userIdentity.principalId, COUNT(*) as total FROM cloudtrail WHERE eventName='AssumeRole' GROUP BY 1, 2 ORDER BY total DESC LIMIT 100

<details>
<summary>Answer</summary>

**Correct: D**

CloudTrail Lake uses Presto/Trino SQL syntax. For AssumeRole events, the assumed role ARN is in resources array (not requestParameters.roleArn). element_at(resources, 1).ARN extracts the first resource ARN (Presto function for array access). userIdentity.principalId identifies the caller. GROUP BY 1,2 (positional grouping) is valid Presto syntax. ORDER BY total DESC shows most frequently assumed roles first.

**Why the others are wrong:**
- requestParameters.roleArn is not the correct CloudTrail field for assumed role - it's in the resources array. This query will return empty or incorrect results.
- resources[0].arn uses bracket notation which is invalid in Presto SQL (CloudTrail Lake query engine) - must use element_at(). userIdentity.arn doesn't exist (correct field is principalId or arn depending on identity type). Will cause syntax error.
- responseElements.assumedRoleUser.arn contains the ARN of the assumed role session (temporary credentials), not the role itself. This query groups by session ARNs (which are unique per assume action) rather than role ARNs, defeating the purpose of aggregation.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** GuardDuty generates a finding 'Recon:EC2/PortProbeUnprotectedPort' with severity 2.0 for an EC2 instance. The instance is running a public-facing web application. What is the BEST course of action?

A) Immediately isolate the instance by removing it from the load balancer and changing security group rules to deny all traffic
B) Review the security group rules to ensure only required ports (80, 443) are open, verify the source IPs in the finding against known scanners, document as expected behavior if legitimate
C) Enable GuardDuty Malware Protection for the instance to prevent exploitation
D) Terminate the instance and restore from a known-good AMI snapshot

<details>
<summary>Answer</summary>

**Correct: B**

Severity 2.0 is Low (0.1-3.9 range). PortProbeUnprotectedPort indicates reconnaissance scanning of open ports, which is common internet background noise for public-facing services. For a web application, port scanning is expected. Appropriate response is to verify security group rules are correct (only necessary ports exposed), check if IPs are known scanners, and document. No immediate containment needed for Low severity reconnaissance against a public service.

**Why the others are wrong:**
- Severe overreaction to a Low severity finding. Port scanning is reconnaissance, not active exploitation. Isolating a public-facing web application disrupts legitimate business operations for a routine internet scan.
- GuardDuty Malware Protection scans EC2 instances and EBS volumes for malware - it doesn't prevent port scanning or exploitation. This is the wrong service for the threat indicated.
- Extreme overreaction. Terminating and rebuilding for a port scan (reconnaissance activity) is unnecessary and disruptive. Port scanning indicates someone is looking for vulnerabilities, not that compromise has occurred.

</details>

---

**Q8.** A company needs to forward Security Hub findings with CRITICAL severity to a ticketing system via webhook within 2 minutes of detection. The solution must include finding details in OCSF format for compatibility with their SOAR platform. Which implementation is MOST reliable?

A) Create EventBridge rule matching Security Hub findings with Severity.Label='CRITICAL', target Lambda that transforms finding to OCSF and calls webhook with retry logic
B) Configure Security Hub custom action to trigger Lambda when analyst marks finding as critical, Lambda transforms to OCSF and sends to webhook
C) Use Security Hub integration with Security Lake to normalize findings to OCSF, create EventBridge rule on Security Lake events, invoke Lambda for webhook delivery
D) Configure Security Hub to send findings to SNS topic, subscribe Lambda to SNS to transform to OCSF, use SQS for guaranteed webhook delivery

<details>
<summary>Answer</summary>

**Correct: A**

EventBridge rules on Security Hub findings trigger in real-time (sub-minute latency), can filter by Severity.Label='CRITICAL' natively, and invoke Lambda reliably. Lambda can transform Security Hub's ASFF format to OCSF and implement retry logic for webhook calls. This is the simplest, most reliable architecture for real-time automated response. Security Hub findings already include OCSF-mapped fields in recent versions.

**Why the others are wrong:**
- Custom actions require manual analyst triggering - not automated. Defeats the 2-minute SLA requirement. Custom actions are for human-initiated workflows, not automated alerting.
- Adds unnecessary complexity by routing through Security Lake first. Security Hub findings can be consumed directly via EventBridge with lower latency. Security Lake adds storage and query capabilities but increases latency for real-time alerting (findings written to S3, then queryable).
- Over-engineered solution. SNS → Lambda → SQS → webhook adds multiple hops and latency. While SQS provides guaranteed delivery, it doesn't help if the webhook itself fails. Lambda with retry logic (direct or using EventBridge retry policy) is simpler and faster than SNS+SQS chain.

</details>

---

