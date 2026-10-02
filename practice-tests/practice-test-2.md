# Practice Test 2: AWS Certified Security Specialty (SCS-C03) Exam Prep

- **Questions:** 44
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-2.md`. Track results in `score-tracker.md`.

---

**Q1.** A financial services company runs a multi-account AWS environment with AWS Organizations. Security requires automated detection of IAM policies that grant overly permissive cross-account access. The solution must identify when principals from external accounts are granted access to sensitive S3 buckets, and must differentiate between trusted partner accounts and unknown external accounts. Which solution provides the MOST comprehensive detection capability?

A) Enable AWS IAM Access Analyzer with organization-level analysis, create archive rules for trusted external accounts, and configure EventBridge rules to trigger SNS notifications for active findings with 'isPublic: true' status
B) Use AWS Config with the 's3-bucket-policy-grantee-check' managed rule, specify trusted account IDs as parameters, and enable AWS Security Hub to aggregate cross-account policy violations
C) Enable AWS IAM Access Analyzer for each account, define a zone of trust that includes the organization, create filters for external access findings, and integrate findings with Security Hub for centralized review
D) Implement a Lambda function triggered by CloudTrail 's3:PutBucketPolicy' events that parses bucket policies for cross-account principals, compares against a DynamoDB table of approved accounts, and publishes violations to SNS

---

**Q2.** A company's security team needs to detect potential credential compromise by monitoring for unusual API call patterns. They want to identify when IAM credentials are used from geographic locations inconsistent with normal usage, AND when credentials show signs of automated scanning behavior (rapid successive API calls to enumerate resources). Current CloudTrail logs are stored in S3. The detection must occur within 15 minutes of the activity. Which approach provides detection for BOTH scenarios with minimal operational overhead?

A) Enable Amazon GuardDuty, which analyzes CloudTrail events for anomalous behavior including impossible travel, credential exfiltration patterns, and reconnaissance activity, and publishes findings to EventBridge within 5-15 minutes
B) Create an Amazon Athena query scheduled via Lambda every 10 minutes to analyze CloudTrail logs for unusual source IPs and high-frequency API calls, store baselines in DynamoDB, and trigger SNS when thresholds are exceeded
C) Enable AWS Security Hub and activate the 'AWS Foundational Security Best Practices' standard, which includes automated checks for unusual IAM activity patterns based on CloudTrail analysis
D) Configure CloudTrail Insights to detect unusual API call volume patterns, and create a separate EventBridge rule with pattern matching for 'sourceIPAddress' changes from CloudTrail events to detect geographic anomalies

---

**Q3.** An enterprise security team manages 150 AWS accounts. They need to detect when EC2 instances in production VPCs communicate with known malicious IP addresses or domains. The solution must work for instances without public IPs that route through NAT Gateways, must not require agent installation, and must detect both inbound and outbound connections. Current VPC Flow Logs are enabled and sent to a central S3 bucket. Which solution provides the required detection capability?

A) Configure Amazon GuardDuty in the centralized security account with VPC Flow Logs and DNS logs as data sources, enable GuardDuty across all organization accounts using delegated administrator, which will analyze traffic patterns against threat intelligence feeds
B) Create an Athena query against VPC Flow Logs to join flow records with a threat intelligence feed stored in S3, run the query every 30 minutes via Lambda, and publish matches to SNS for alerting
C) Deploy AWS Network Firewall in each production VPC with Suricata-compatible rules containing known malicious IPs and domains, configure alert-only mode to avoid blocking legitimate traffic, and send alerts to CloudWatch Logs
D) Enable Amazon Detective and configure it to analyze VPC Flow Logs, CloudTrail, and GuardDuty findings to build behavioral graphs that identify communications with malicious endpoints

---

**Q4.** A healthcare company must detect when S3 objects containing protected health information (PHI) are unintentionally exposed. Objects are tagged with 'DataClassification: PHI' upon creation. The company needs to detect: (1) when PHI objects are made public via ACLs or bucket policies, (2) when PHI objects are shared with unauthorized AWS accounts via cross-account access, and (3) when bucket encryption settings for PHI buckets are disabled. Detection must be continuous and automated. Which combination of services provides complete coverage for all three scenarios?

A) Enable Amazon Macie to continuously monitor S3 for sensitive data exposure and policy changes, configure S3 Event Notifications for PutBucketPolicy and PutBucketEncryption events to trigger Lambda functions that check PHI tags, and use AWS Config s3-bucket-public-read-prohibited and s3-bucket-server-side-encryption-enabled rules
B) Use AWS IAM Access Analyzer to detect external access to S3 resources, enable AWS Config with s3-bucket-public-read-prohibited, s3-bucket-server-side-encryption-enabled, and s3-bucket-acl-prohibited rules filtered to buckets with PHI tags, and configure EventBridge to monitor Config compliance changes
C) Enable Amazon GuardDuty S3 Protection which detects suspicious access patterns and policy changes, combine with Amazon Macie sensitive data discovery jobs scheduled daily to identify public exposure of PHI data, and use CloudTrail to monitor encryption changes
D) Deploy AWS Config rules for public access and encryption settings on PHI buckets, use IAM Access Analyzer for cross-account access detection, and enable Amazon Macie for automated discovery of PHI exposure through object-level scanning and policy analysis

---

**Q5.** A security operations team receives AWS Security Hub findings from GuardDuty, IAM Access Analyzer, and Macie across 80 accounts. They need to automatically suppress findings that match specific criteria: (1) GuardDuty 'Recon:EC2/PortProbeUnprotectedPort' findings from their approved vulnerability scanning CIDR ranges, (2) IAM Access Analyzer findings for S3 buckets intentionally shared with a specific partner account, and (3) Macie findings for objects in a bucket used for security testing. The suppression must not delete findings but must prevent them from appearing in active findings and affecting compliance scores. What is the MOST operationally efficient approach?

A) Create Security Hub custom actions that trigger Lambda functions to update finding workflow status to 'SUPPRESSED' based on the specific criteria, then use EventBridge rules to automatically invoke these actions when matching findings arrive
B) Configure suppression rules in each source service: create GuardDuty suppression rules for the specific finding type and IP criteria, create IAM Access Analyzer archive rules for the trusted account, and configure Macie allow lists for the testing bucket
C) Use Security Hub automation rules to automatically update finding workflow status and add notes for findings matching the criteria patterns, configured with filters for finding type, resource identifiers, and source account
D) Enable AWS Config aggregator to collect findings, create AWS Config rules with exemption logic for the specified scenarios, and configure Security Hub to only ingest findings that pass Config rule evaluation

---

**Q6.** A company runs containerized applications on Amazon ECS Fargate across multiple accounts. Security requires detection of container images with known vulnerabilities before deployment AND runtime detection of containers exhibiting malicious behavior. The solution must integrate with existing CI/CD pipelines (AWS CodePipeline/CodeBuild), provide findings in Security Hub, and detect runtime threats like cryptocurrency mining or unexpected network connections. Which combination meets all requirements?

A) Enable Amazon Inspector for container image scanning in ECR repositories, configure CodeBuild to pull Inspector findings and fail builds for critical/high vulnerabilities, enable GuardDuty ECS Runtime Monitoring for Fargate tasks to detect runtime threats, integrate both with Security Hub
B) Implement Aqua Security or Prisma Cloud container security platform with both image scanning integrated into CodeBuild and runtime agents deployed as Fargate sidecars for behavioral detection, forward findings to Security Hub via EventBridge custom event bus
C) Use Amazon ECR image scanning with scan-on-push enabled, create a CodeBuild step that queries scan results via ECR API and fails on vulnerabilities, enable AWS Config with ecs-task-definition-user-for-host-mode-check rule, and use CloudWatch Container Insights for runtime monitoring
D) Enable Amazon Inspector for ECR scanning and ECS task scanning, configure EventBridge rules to block task launches for images with critical vulnerabilities, deploy GuardDuty ECS Runtime Monitoring, and use Security Hub for centralized findings aggregation

---

**Q7.** A security team discovered that an IAM role in their production AWS account has been assumed from an IP address in a different geographic region than their corporate network. The role has permissions to read S3 buckets containing PII. CloudTrail logs show the AssumeRole API call succeeded using valid temporary credentials, but the MFA serial number field is null. The security team needs to detect similar incidents in real-time and automatically revoke active sessions. Which solution provides the MOST comprehensive detection and response?

A) Create an EventBridge rule that triggers on AssumeRole API calls where MFA is not used, invoke a Lambda function to call GetSessionToken with a DenyAll policy, and send notifications to SNS
B) Configure a CloudWatch Logs Insights query to parse CloudTrail logs for AssumeRole events without MFA, schedule it to run every 5 minutes via Lambda, and use IAM Access Analyzer to identify the role's permissions
C) Create a GuardDuty custom threat list containing approved IP ranges, enable GuardDuty findings for UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS, and configure EventBridge to trigger a Lambda function that attaches an explicit deny policy to the role and revokes active sessions using the IAM DeleteSessionToken API
D) Use AWS Config to create a custom rule that evaluates IAM role trust policies for missing MFA conditions, configure automatic remediation using Systems Manager Automation to update the trust policy, and enable Security Hub to aggregate findings

---

**Q8.** A security team discovers that an IAM role's credentials have been exposed in a public GitHub repository for approximately 72 hours. CloudTrail logs show the credentials were used to launch 50 EC2 instances across multiple regions, create several S3 buckets, and modify security group rules. The role has AdministratorAccess policy attached. What is the MOST effective immediate response to contain this incident?

A) Attach an explicit Deny all actions policy to the compromised IAM role and rotate the role's credentials
B) Delete the compromised IAM role immediately and create a new role with identical permissions for legitimate services
C) Add a aws:SourceIp condition to the role's trust policy restricting access to corporate IP ranges only
D) Revoke all active sessions for the IAM role using AWS STS, then attach an SCP denying all actions for that role

---

**Q9.** During incident response, a security analyst needs to preserve forensic evidence from a potentially compromised EC2 instance running in a production VPC. The instance must remain running for business continuity, but all existing EBS snapshots are over 48 hours old. Compliance requires evidence preservation within 15 minutes of detection and maintaining chain of custody. Which approach BEST satisfies these requirements?

A) Create an EBS snapshot of all attached volumes, tag with incident ID and timestamp, copy snapshots to a separate forensics account using encrypted snapshot copy, then create AMI from snapshots
B) Use SSM Run Command to execute a memory dump to S3 with Object Lock enabled, create EBS snapshots with encryption, enable EBS snapshot archive tier for long-term retention
C) Enable termination protection, isolate instance in forensic security group allowing only SSM access, create encrypted EBS snapshots, share snapshots to forensics account with KMS key grants, document actions in incident ticket
D) Detach the root volume, attach forensically-sound replacement volume, create snapshot of detached volume with encryption and cross-region replication to forensics account

---

**Q10.** A company's security team receives an Amazon GuardDuty finding: 'UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS' indicating that temporary credentials from an EC2 instance profile are being used from an external IP address in a different country. CloudTrail shows API calls to DescribeInstances, DescribeSecurityGroups, and ListBuckets from this external IP. What is the MOST comprehensive containment strategy?

A) Immediately rotate all IAM access keys in the account, enable MFA delete on all S3 buckets, and create a VPC endpoint for EC2 to prevent external API calls
B) Revoke the active IAM role session using AWS CLI revoke-sessions command, modify the role's trust policy to add aws:SourceIp condition matching the VPC CIDR, enable IMDSv2 requirement on all instances, review and rotate instance profile if necessary
C) Terminate the affected EC2 instance, delete the instance profile role, create a new restrictive role with only required permissions, enable VPC Flow Logs and GuardDuty S3 protection
D) Create an SCP that denies all actions when aws:SourceIp does not match corporate ranges, attach explicit Deny policy to the compromised role, enable CloudTrail Insights for anomaly detection

---

**Q11.** An organization detects unusual AWS API activity in their CloudTrail logs showing AssumeRole calls from a federated identity that successfully assumed an administrative role, but the source IP address is from a known malicious TOR exit node. The federated identity uses SAML 2.0 integration with their on-premises Active Directory. Investigation reveals the SAML assertion appears valid with correct attributes. What is the MOST likely attack vector and appropriate response?

A) The SAML signing certificate has been compromised; immediately rotate the SAML provider certificate in AWS IAM, force re-authentication for all federated sessions, and conduct forensic analysis of the identity provider infrastructure
B) An attacker has stolen valid Active Directory credentials and is federating through the legitimate SAML flow; reset the affected user's AD password, revoke all AWS federated sessions for that user, implement Conditional Access policies in the IdP requiring trusted locations
C) The AWS SAML provider trust policy has been modified to accept assertions from an unauthorized IdP; review and restore the SAML provider configuration, implement SCP to prevent unauthorized SAML provider modifications, enable CloudTrail log file validation
D) Session token hijacking has occurred after legitimate authentication; immediately disable the SAML provider in IAM, force all users to re-authenticate through a clean authentication path, implement AWS STS session tagging with source IP validation

---

**Q12.** A security team is investigating a suspected data exfiltration incident. VPC Flow Logs show 500GB of data transferred from a private subnet to an unknown external IP over port 443 during a 3-hour window. The subnet hosts RDS databases and application servers with no direct internet access (NAT Gateway in public subnet). CloudTrail shows no unauthorized API activity. What is the MOST effective forensic investigation approach to identify the exfiltration source and method?

A) Query VPC Flow Logs using Amazon Athena to identify which ENI initiated the connections, cross-reference ENI ID with EC2 instances, analyze CloudWatch Logs from the identified instances for application-level evidence, capture network traffic using Traffic Mirroring to the destination IP if still active
B) Enable GuardDuty S3 protection and Malware Protection, create EBS snapshots of all instances in the subnet, analyze snapshots using forensic AMI in isolated VPC, review Security Group rules for recent modifications
C) Review NAT Gateway CloudWatch metrics for top talkers, enable VPC Flow Logs at subnet level with additional metadata fields, implement AWS Network Firewall with IDS mode to detect data exfiltration patterns, analyze RDS audit logs for data access
D) Use AWS Detective to visualize the network connections and identify the source, enable VPC Traffic Mirroring on the entire subnet to capture packets, deploy a network forensics appliance to analyze mirrored traffic in real-time

---

**Q13.** During a security incident, an analyst discovers that an attacker has gained access to an AWS account and created multiple IAM users with programmatic access. The attacker then deleted CloudTrail trails, disabled GuardDuty, and modified Config rules. However, AWS Config recorded configuration changes before being disabled. The company has AWS Organizations with CloudTrail configured at the organization level logging to a separate security account. What is the BEST approach to reconstruct the complete attack timeline and identify all compromised resources?

A) Use AWS Config advanced queries to retrieve all configuration changes during the incident window, correlate with organization CloudTrail logs in the security account using Amazon Athena, use GuardDuty archived findings if available, generate a timeline using CloudTrail Insights anomaly detection
B) Restore the deleted CloudTrail trail from S3 versioning if enabled, use CloudTrail Lake to query event history across the organization, cross-reference IAM Access Analyzer findings for unusual access patterns, use AWS Security Hub to aggregate findings
C) Query the organization-level CloudTrail logs in the security account using Athena with specific filters for IAM, CloudTrail, GuardDuty, and Config API events, correlate with AWS Config timeline to identify resource state changes, use EventBridge event history for recently emitted events, validate findings against Cost and Usage Reports for unexpected resource creation
D) Use AWS Systems Manager Session Manager logs to identify which instances the attacker accessed, review VPC Flow Logs for network evidence, enable CloudTrail again and use CloudTrail lookup-events with --max-results parameter to retrieve deleted events, analyze S3 server access logs for CloudTrail bucket access patterns

---

**Q14.** A financial services company runs a multi-tier application on Amazon ECS with AWS Fargate. The application processes sensitive customer data and must comply with PCI DSS requirements. Security auditors have identified that container images are being pulled from a public ECR repository and that runtime protection is insufficient. The security team needs to implement controls that prevent deployment of vulnerable images and detect anomalous runtime behavior. Which solution should the security engineer implement to address both vulnerabilities?

A) Enable Amazon Inspector for container image scanning, configure AWS Systems Manager Patch Manager to automatically remediate vulnerable packages in running containers, and use AWS Security Hub to aggregate findings across all containers
B) Migrate container images to a private ECR repository with scan-on-push enabled using Amazon ECR enhanced scanning, enable Amazon GuardDuty Runtime Monitoring for ECS Fargate, and configure EventBridge rules to trigger Lambda functions that prevent task updates when CRITICAL vulnerabilities are detected or suspicious runtime activity occurs
C) Implement AWS Signer for container image signing with automatic verification at deployment time, enable ECR image tag immutability to prevent overwrites, and deploy Amazon Detective to analyze container network traffic patterns for anomaly detection
D) Enable Amazon ECR basic scanning with scan-on-push, create an AWS Config rule to check for vulnerability findings, and configure AWS WAF with managed rule groups to protect containerized applications from runtime attacks

---

**Q15.** A company's security team is designing network segmentation for a new VPC that will host production workloads across three tiers: web (public subnet), application (private subnet), and database (isolated private subnet). The application tier must initiate outbound HTTPS connections to third-party APIs, but the database tier must have zero internet connectivity. The security team wants to enforce these requirements at the network layer with defense in depth. An auditor has requested proof that the database tier cannot reach the internet even if security group rules are misconfigured. What is the MOST secure architecture to meet these requirements?

A) Deploy a NAT Gateway in the public subnet, route application tier traffic through the NAT Gateway, use separate route tables for each tier with the database tier route table containing only local VPC routes, and enable VPC Flow Logs to prove no internet traffic from database tier
B) Use AWS Network Firewall in the public subnet with strict egress filtering rules, route all outbound traffic through the firewall, implement network ACLs denying all outbound traffic from database subnet IP ranges, and configure GuardDuty VPC protection to detect unauthorized internet access attempts
C) Create a VPC with three subnets: public subnet with NAT Gateway, application private subnet with route to NAT Gateway, database isolated subnet with no route table entry for 0.0.0.0/0, enable both network ACLs (denying internet on database subnet) and security groups, and implement VPC Flow Logs with Athena queries to audit zero internet traffic
D) Deploy VPC endpoints for all required AWS services in the database subnet, use AWS PrivateLink for third-party API access from application tier, remove all internet gateway routes from database subnet route table, and configure AWS Config rules to alert on any route table modifications

---

**Q16.** A healthcare organization runs a HIPAA-compliant application on EC2 instances behind an Application Load Balancer. The application must enforce end-to-end encryption from the client browser to the application tier. The security team has implemented ACM certificates on the ALB for TLS termination, but a compliance audit has flagged that this creates an unencrypted segment between the ALB and EC2 instances within the VPC. The auditor requires proof of encryption for the entire path. Backend instances are in an Auto Scaling group and cannot use static IP addresses. What solution provides end-to-end encryption while maintaining operational scalability?

A) Enable end-to-end encryption by configuring the ALB target group protocol to HTTPS, install self-signed certificates on EC2 instances, configure ALB health checks to use HTTPS protocol with certificate validation disabled, and use AWS Certificate Manager Private CA to issue certificates to backend instances via AWS Systems Manager distributor
B) Implement Network Load Balancer instead of ALB with TLS passthrough configured, terminate TLS directly on EC2 instances using ACM certificates attached to elastic network interfaces, configure target group to use TLS protocol, and enable connection draining to maintain sessions during scaling events
C) Configure the ALB listener to use HTTPS and the target group protocol to HTTPS, deploy ACM certificates to EC2 instances using AWS Secrets Manager with automatic rotation, configure ALB to perform backend authentication using the ACM public certificate, and enable access logs to prove end-to-end encryption
D) Configure ALB target group protocol to HTTPS, use AWS Certificate Manager Private CA to issue certificates to backend instances, install certificates on EC2 instances via user data scripts that retrieve certificates from ACM Private CA, configure ALB health checks to use HTTPS with SNI, and enable connection logs showing TLS handshake success

---

**Q17.** A media company stores millions of video files in Amazon S3 across three accounts within an AWS Organization: production account (main storage), analytics account (ML processing), and archive account (cold storage). The security team needs to implement a solution where objects stored in the production S3 bucket are automatically encrypted with a customer-managed KMS key, objects copied to the analytics account use a different KMS key for that account, and objects moved to archive account are encrypted with S3-managed keys to reduce costs. Cross-account access must follow least privilege. S3 Replication is being used. What is the correct KMS key policy and S3 configuration?

A) Create a KMS key in the production account with a key policy granting s3.amazonaws.com encrypt permissions, enable S3 bucket replication with replica configuration specifying SSE-KMS encryption, create separate KMS keys in analytics and archive accounts with cross-account principals allowing the production account's S3 service to use them, and configure S3 Lifecycle policies to transition objects to SSE-S3 in the archive account
B) Create customer-managed KMS keys in production and analytics accounts, configure S3 bucket key feature on all buckets, grant the destination bucket permission to use the source KMS key in production account, enable S3 replication with SourceSelectionCriteria for SSE-KMS encrypted objects, set ReplicaModifications in replication configuration to enable encryption-configuration-override, and use S3 Batch Operations to re-encrypt archive objects with SSE-S3
C) Enable S3 bucket replication with SSE-KMS as default encryption in all three buckets, create a multi-region KMS key with replicas in all three regions, grant cross-account access using resource-based policies on S3 buckets instead of KMS key policies, configure S3 Intelligent-Tiering to automatically move objects between storage classes while maintaining encryption settings, and use VPC endpoints for S3 to control cross-account access
D) Configure production bucket with SSE-KMS using customer-managed key, grant the S3 replication role decrypt permissions on source key and encrypt permissions on destination keys via KMS key policies, enable S3 Replication with encryption override specifying destination SSE-KMS keys for analytics and SSE-S3 for archive, configure ReplicationConfiguration with SourceSelectionCriteria for replica modification support, and implement S3 Object Lock governance mode to prevent encryption changes

---

**Q18.** A financial institution is migrating a legacy three-tier application to AWS that currently runs on physical servers behind a hardware firewall. The application tier makes database queries that include customer SSNs and credit card numbers in the query strings (legacy design that cannot be immediately refactored). The CISO requires that this sensitive data in transit between application and database tiers must be protected from potential VPC Flow Log exposure and from AWS administrators. The database runs on RDS PostgreSQL. Which combination of controls provides defense-in-depth protection for this data in transit?

A) Enable RDS encryption at rest with AWS managed KMS key, configure application to connect to RDS using SSL/TLS with certificate verification, disable VPC Flow Logs on the subnets containing application and database tiers, implement AWS PrivateLink endpoints for RDS connectivity, and enable RDS Performance Insights with encryption
B) Configure RDS to require SSL/TLS connections by setting rds.force_ssl=1 parameter, use IAM database authentication instead of passwords, enable VPC Flow Log encryption with a customer-managed KMS key that AWS administrators cannot access, configure security groups to restrict traffic only between application and database security groups, and enable RDS Enhanced Monitoring with CloudWatch Logs encryption
C) Enable SSL/TLS for RDS connections with certificate verification, configure VPC Flow Logs to use maximum aggregation interval (10 minutes) to reduce log verbosity, implement Network Access Control Lists blocking all traffic except application-to-database on port 5432, use AWS Secrets Manager to rotate database credentials daily, and enable RDS audit logging with log exports to encrypted S3
D) Require SSL/TLS for RDS connections using rds.force_ssl parameter and verify certificates in application, deploy application tier in private subnets without NAT Gateway, implement AWS Network Firewall between application and database subnets with TLS inspection disabled, use VPC endpoints for AWS services to prevent internet traffic, and configure RDS to use customer-managed KMS key with key policy restricting AWS service principals

---

**Q19.** A SaaS company provides a multi-tenant application where each customer's data is stored in separate S3 buckets following the pattern: company-data-{customer-id}. The application runs on EC2 instances using an IAM role. A security review identified that the current IAM role policy grants s3:* permissions to arn:aws:s3:::company-data-*, allowing any application instance to access any customer's data. The security team must implement least privilege so each application instance can only access S3 objects for the customer ID it's serving, but the solution must scale to thousands of customers without creating thousands of IAM roles. Customer ID is determined at instance launch time via a tag. What is the MOST scalable solution?

A) Create a single IAM role with a policy using the aws:RequestedRegion condition key to limit S3 access based on the region tag applied to instances, implement AWS Organizations SCPs to enforce per-customer data boundaries, use S3 bucket policies on each customer bucket granting access only to instances tagged with the matching customer ID, and enable S3 Access Points with VPC network origin controls
B) Implement an IAM policy for the instance role using the aws:PrincipalTag condition key to match against the customer-id instance tag, construct the resource ARN dynamically using the policy variable ${aws:PrincipalTag/customer-id} in the S3 resource specification (arn:aws:s3:::company-data-${aws:PrincipalTag/customer-id}/*), ensure instances are tagged at launch with the customer-id tag, and enable AWS Config to detect instances without required tags
C) Create AWS IAM roles per customer and use EC2 instance metadata service version 2 (IMDSv2) to dynamically assume the correct customer role at runtime, implement an AWS Lambda function that monitors instance launches via EventBridge and automatically attaches the correct IAM role based on customer ID tag, configure Session Manager instead of SSH for instance access to ensure role assumption audit trails, and enable CloudTrail data events for all S3 access
D) Deploy AWS Systems Manager Session Manager with RunAs capability to execute S3 API calls under dynamic IAM credentials, create an IAM policy with the s3:ExistingObjectTag condition key requiring objects to be tagged with the same customer-id as the instance, implement S3 Object Lambda access points to filter S3 responses based on instance tags, and use VPC endpoints for S3 with endpoint policies restricting bucket access patterns

---

**Q20.** A financial services company runs a multi-tier application on AWS with strict data residency requirements. The application uses Amazon RDS for PostgreSQL in us-east-1, and they need to ensure that all automated backups, snapshots, and read replicas remain within the US region boundaries. The security team has discovered that enabling cross-region automated backups could violate compliance requirements. Database administrators need the ability to create manual snapshots but must be prevented from copying them to non-US regions. Which solution will enforce these requirements while maintaining operational flexibility for the DBA team?

A) Configure RDS backup retention to 0 days and rely solely on AWS Backup with resource-based policies that restrict backup vault access to US regions only
B) Implement an SCP that denies rds:CopyDBSnapshot and rds:CopyDBClusterSnapshot actions when aws:RequestedRegion is outside us-east-1, us-east-2, us-west-1, and us-west-2, while allowing all other RDS snapshot operations
C) Create an IAM permissions boundary for database administrators that allows rds:CreateDBSnapshot but denies rds:CopyDBSnapshot to all non-US regions using a condition key on aws:RequestedRegion
D) Enable RDS encryption with a KMS key that has a key policy restricting kms:CreateGrant to US regions, preventing snapshot copies to other regions through encryption controls

---

**Q21.** A healthcare SaaS provider operates a VPC with application servers in private subnets that must access both AWS services (S3, DynamoDB, Secrets Manager) and on-premises LDAP servers via AWS Direct Connect. The security architecture requires that traffic to AWS services never traverse the internet or Direct Connect gateway, while LDAP queries must route through the Direct Connect private VIF. Current configuration shows intermittent LDAP authentication failures when S3 VPC endpoints are enabled. Network ACL logs indicate that return traffic from on-premises LDAP (TCP port 389 responses) is being blocked. Which solution resolves the LDAP connectivity issue while maintaining the requirement that AWS service traffic remains within the AWS network?

A) Modify the private subnet route table to include a specific route for the on-premises LDAP CIDR range pointing to the Virtual Private Gateway, ensuring it has higher priority than the VPC endpoint routes
B) Configure Gateway VPC endpoints for S3 and DynamoDB with an endpoint policy that denies access from the on-premises CIDR range, then add a route to the Virtual Private Gateway for on-premises traffic
C) Replace Gateway VPC endpoints with Interface VPC endpoints for all services and enable private DNS, which allows proper routing of both AWS service traffic and Direct Connect traffic simultaneously
D) Add a network ACL rule in the private subnet allowing inbound TCP port 389 from the on-premises CIDR range with a rule number lower than 100, and verify the outbound rule allows ephemeral ports 1024-65535

---

**Q22.** A financial services company uses AWS Organizations with multiple AWS accounts. The security team needs to enforce that all IAM users across all accounts must use hardware MFA devices (not virtual MFA) and rotate their access keys every 60 days. The solution must generate compliance reports showing which users are non-compliant. Which approach provides the MOST operationally efficient solution?

A) Create an SCP that denies all actions unless aws:MultiFactorAuthPresent is true and aws:MultiFactorAuthAge is less than 60 days. Use AWS Config aggregator with custom Config rules to check MFA device type and access key age across all accounts.
B) Deploy AWS IAM Access Analyzer to all accounts and create custom findings for MFA type and key rotation. Use EventBridge to aggregate findings in a central security account.
C) Use AWS Config with managed rules iam-user-mfa-enabled and access-keys-rotated deployed via AWS CloudFormation StackSets. Create a custom Config rule using a Lambda function to verify hardware MFA specifically. Aggregate compliance data using Config aggregator.
D) Implement AWS Security Hub with CIS AWS Foundations Benchmark controls for MFA and key rotation. Create a Lambda function triggered by Security Hub findings to verify hardware MFA tokens and generate reports.

---

**Q23.** A company's application running on EC2 instances needs to access objects in an S3 bucket and send messages to an SQS queue. The security team requires that credentials never be stored on the instances, and they must implement least privilege access. An IAM role named AppRole exists with the following trust policy:

{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole",
    "Condition": {
      "StringEquals": {"aws:SourceAccount": "123456789012"}
    }
  }]
}

The instances are launched but cannot access S3 or SQS. What is the MOST likely cause?

A) The IAM role needs permissions policies attached granting s3:GetObject and sqs:SendMessage actions, and the EC2 instances must have an instance profile attached that references the AppRole.
B) The trust policy condition aws:SourceAccount does not apply to EC2 service principals. Remove the Condition block from the trust policy.
C) The EC2 instances must be tagged with a specific tag key-value pair that matches a condition in the IAM role's permissions policy.
D) The instances need both an instance profile AND explicit IAM user credentials stored in ~/.aws/credentials file with the same permissions as the role.

---

**Q24.** A startup is designing a multi-tenant SaaS application where each customer's data is isolated in separate DynamoDB tables with customer-specific prefixes (customer-A-data, customer-B-data, etc.). The application uses Amazon Cognito User Pools for authentication. The security architect needs to ensure that authenticated users can ONLY access their own customer's DynamoDB tables using temporary credentials. Which solution implements the MOST secure fine-grained access control?

A) Configure Cognito Identity Pools with an IAM role that includes a policy using dynamodb:LeadingKeys condition with ${cognito-identity.amazonaws.com:sub} variable to restrict access to items based on user ID.
B) Create an IAM role for each customer and configure Cognito Identity Pools to use role-based access control with rules that map custom:customer_id attribute to the appropriate customer role. Use policy variables in the role's DynamoDB policy with a Condition on dynamodb:LeadingKeys matching the customer prefix.
C) Use Cognito Identity Pools with an enhanced authentication flow and IAM role with a policy that grants DynamoDB access where the Resource ARN uses the ${aws:PrincipalTag/customer_id} variable to match table names.
D) Configure Cognito User Pools to include a customer_id custom attribute. Create a Lambda@Edge function that validates tokens and uses STS AssumeRole to generate temporary credentials scoped to the customer-specific DynamoDB table before forwarding requests to the application.

---

**Q25.** A company uses AWS IAM Identity Center (formerly AWS SSO) integrated with Azure AD as the identity provider. The security team needs to implement a solution where users in the 'DatabaseAdmins' Azure AD group can assume an IAM role with administrative access to RDS across 15 AWS accounts in an Organization, but ONLY when they authenticate from the corporate network (IP range 203.0.113.0/24). The solution should minimize ongoing operational overhead. What is the MOST appropriate approach?

A) Create a permission set in IAM Identity Center with AdministratorAccess AWS managed policy attached. Map the permission set to the DatabaseAdmins group. Add an inline permissions boundary to the permission set that denies all actions when aws:SourceIp is not in 203.0.113.0/24.
B) Create a permission set in IAM Identity Center with a custom inline policy granting RDS administrative actions. Add a Condition block to the policy using IpAddress with aws:SourceIp key for 203.0.113.0/24. Assign the permission set to the DatabaseAdmins group across all 15 accounts.
C) Create an SCPs that enforces IP-based access restrictions for RDS actions. Create a permission set in IAM Identity Center with RDS administrative permissions and assign it to the DatabaseAdmins group.
D) In each of the 15 AWS accounts, manually create an IAM role with RDS administrative permissions and a trust policy that includes an IpAddress condition for 203.0.113.0/24. Configure Azure AD to pass network location as a SAML attribute that IAM evaluates during AssumeRoleWithSAML.

---

**Q26.** An enterprise application uses IAM roles with temporary credentials to access AWS services. The security team conducted a review and found that several EC2 instances are using credentials that have been active for 8+ hours, violating the company's 4-hour maximum session policy. The IAM role has the following configuration:

Maximum session duration: 4 hours
Permissions: AmazonS3ReadOnlyAccess managed policy

The instances are running a Java application that creates an AmazonS3Client once at startup. What is the MOST likely cause and solution?

A) The application is not refreshing the credentials from instance metadata. Modify the application to use AWS SDK's default credential provider chain which automatically rotates credentials before expiration, or explicitly refresh credentials every 3.5 hours.
B) The Maximum session duration setting only applies to console sessions, not instance profile credentials. Create a custom AWS Config rule that monitors credential age and triggers Lambda to restart instances every 4 hours.
C) EC2 instance profile credentials ignore role session duration settings and use a fixed 6-hour expiration. Modify the trust policy to add a Condition with aws:TokenIssueTime to enforce the 4-hour limit.
D) The AmazonS3ReadOnlyAccess managed policy overrides session duration limits for backwards compatibility. Replace it with a custom inline policy with identical permissions but without the session extension.

---

**Q27.** A financial institution needs to grant a third-party auditing firm temporary access to read CloudTrail logs and AWS Config snapshots stored in an S3 bucket in the company's AWS account. The access must expire after exactly 7 days, require MFA authentication, and prevent the auditor from deleting or modifying any data. The auditor has their own AWS account. Which solution meets ALL requirements with the LEAST operational complexity?

A) Create an IAM role with a trust policy that allows the auditor's AWS account to assume it with an external ID and MFA requirement. Attach a permissions policy granting s3:GetObject and s3:ListBucket for the audit bucket. Manually delete the role after 7 days.
B) Create an IAM user in the company's account with access keys, attach a permissions policy for read-only S3 access with an explicit Deny for delete/modify actions, and enable MFA for the user. Use IAM Access Analyzer to set expiration, then delete the user after 7 days.
C) Create an S3 bucket policy that grants the auditor's AWS account principal read-only access with Condition blocks requiring MFA (aws:MultiFactorAuthPresent) and enforcing access expiration using DateLessThan with aws:CurrentTime set to 7 days from grant date. Use s3:GetObject and s3:ListBucket with explicit Deny for s3:DeleteObject and s3:PutObject.
D) Create an IAM role with a trust policy allowing the auditor's account to assume it with external ID. Attach AWS managed policy SecurityAudit. Configure the role with a maximum session duration of 7 days (604800 seconds) and add a Condition to the trust policy requiring MFA using aws:MultiFactorAuthPresent.

---

**Q28.** A financial services company uses AWS Organizations with multiple accounts. The security team needs to enforce that all IAM users across all accounts must use hardware MFA tokens (not virtual MFA) and rotate access keys every 60 days. The solution must generate automatic compliance reports and prevent non-compliant users from accessing resources. Which solution provides the MOST operationally efficient enforcement?

A) Create an SCP that denies all actions unless aws:MultiFactorAuthPresent is true and aws:TokenIssueTime is within 60 days. Deploy AWS Config rules to detect virtual MFA and generate reports.
B) Use IAM Access Analyzer to identify users without hardware MFA. Create EventBridge rules to disable access keys older than 60 days. Generate monthly compliance reports using AWS Config.
C) Implement a combination of SCPs to deny actions without MFA, AWS Config conformance packs with remediation to enforce hardware MFA requirements, and IAM Access Analyzer for access key age monitoring with automated SNS notifications.
D) Deploy an identity-based policy to all accounts requiring hardware MFA using aws:MultiFactorAuthAge and aws:credentials:age conditions. Use AWS Security Hub with custom Config rules for hardware MFA detection and IAM credential reports parsed by Lambda for enforcement.

---

**Q29.** An enterprise application running on EC2 instances needs to access objects in an S3 bucket and send messages to an SQS queue. The security team requires that credentials never be stored on the instances, all actions must be logged with the assumed role identity visible in CloudTrail, and the solution must support credential rotation without application downtime. The application currently uses an IAM user with access keys stored in environment variables. What is the MOST secure solution?

A) Create an IAM role with trust policy allowing ec2.amazonaws.com. Attach policies for S3 and SQS access. Associate the role with the EC2 instance profile. Update application to use AWS SDK default credential provider chain.
B) Store IAM user credentials in AWS Secrets Manager with automatic rotation enabled. Update application code to retrieve credentials from Secrets Manager at startup. Create CloudTrail data events to log S3 and SQS access.
C) Create an IAM role for EC2 with required permissions. Use AWS STS AssumeRole API calls from the application to get temporary credentials. Cache credentials and refresh before expiration using STS GetSessionToken.
D) Migrate IAM user credentials to AWS Systems Manager Parameter Store with SecureString type. Create an IAM role for EC2 that can read the parameter. Update application to retrieve credentials from Parameter Store using instance profile credentials.

---

**Q30.** A company operates a multi-tenant SaaS application where each customer's data is isolated in separate S3 bucket prefixes (s3://app-data/customer-A/, s3://app-data/customer-B/). The application uses Amazon Cognito for user authentication. The security architect needs to ensure that authenticated users can only access their own customer's data prefix, the solution must scale to 10,000+ customers without creating individual IAM roles per customer, and it must prevent privilege escalation between customers. Which approach satisfies these requirements?

A) Create a single IAM role for authenticated users with an S3 bucket policy that uses cognito-identity.amazonaws.com:sub as a condition key to match the S3 prefix. Configure Cognito Identity Pool to pass custom customer_id claim as a session tag.
B) Create an IAM role for Cognito authenticated users with S3 permissions restricted using the ${aws:PrincipalTag/customer_id} variable in the resource ARN. Configure Cognito to include customer_id as a session tag during AssumeRoleWithWebIdentity. Use attribute-based access control (ABAC) policies.
C) Use S3 Access Points with separate access point policies for each customer prefix. Create a Lambda authorizer that maps Cognito user identity to the appropriate access point. Configure the application to use the customer-specific access point URL.
D) Implement S3 bucket policies with a Deny statement unless s3:prefix matches the Cognito identity ID. Create a DynamoDB table mapping Cognito identity IDs to customer prefixes. Use a Lambda@Edge function to validate access before S3 requests.

---

**Q31.** A financial services company stores customer transaction data in Amazon S3. They must comply with PCI DSS requirements mandating that encryption keys are rotated every 90 days and that the company maintains full control over key material generation and rotation. The solution must provide detailed audit trails of all key usage. Which approach meets these requirements with the LEAST operational overhead?

A) Use SSE-S3 with automatic key rotation enabled and AWS CloudTrail for auditing
B) Use SSE-KMS with customer managed keys (CMK), enable automatic key rotation, and use AWS CloudTrail integrated with CloudWatch Logs for monitoring key usage
C) Use SSE-C where the application generates and rotates keys every 90 days, storing keys in AWS Secrets Manager with rotation enabled
D) Use AWS CloudHSM to generate encryption keys, implement custom rotation logic in AWS Lambda triggered every 90 days, and use CloudHSM audit logs

---

**Q32.** A healthcare application processes Protected Health Information (PHI) using Amazon EMR clusters. The data engineering team needs to ensure that data at rest on EMR cluster nodes is encrypted, data in transit between nodes is encrypted, and encryption keys are rotated quarterly. The security team requires that decryption of data should be impossible if an EBS volume is detached and attached to another instance. Which configuration satisfies all requirements?

A) Enable EMR security configuration with at-rest encryption using EBS encryption with AWS managed keys, in-transit encryption using TLS, and implement a quarterly Lambda function to re-encrypt all EBS volumes with new KMS keys
B) Enable EMR security configuration with at-rest encryption using LUKS with keys stored in AWS Secrets Manager, in-transit encryption enabled, and configure Secrets Manager automatic rotation every 90 days
C) Enable EMR security configuration with at-rest encryption using EBS encryption with customer managed KMS keys, in-transit encryption enabled, enable automatic KMS key rotation, and attach EBS volumes with encrypted snapshot option
D) Enable EMR security configuration with at-rest encryption using EMRFS with SSE-KMS, in-transit encryption using TLS, and enable KMS automatic key rotation with EBS encryption using the same CMK

---

**Q33.** A company uses AWS Certificate Manager (ACM) to manage SSL/TLS certificates for their Application Load Balancers. They need to implement a solution where certificates for on-premises servers are generated using the same private certificate authority (CA) to maintain a unified PKI infrastructure. The private CA must have intermediate CA certificates that chain to a root CA managed by the company's security team in an isolated AWS account. Which solution meets these requirements?

A) Create an ACM Private CA in the application account with a root CA type, export the CA certificate, and have the security team import it into their account as a subordinate CA
B) Create an ACM Private CA in the security team's account as a root CA, create a subordinate CA in the application account, share the subordinate CA using AWS RAM, and issue certificates from the subordinate CA
C) Create an ACM Private CA in the security team's account as a root CA, create a subordinate CA in the application account using a CSR signed by the root CA, and use the subordinate CA to issue certificates for both ACM and on-premises use
D) Create an ACM Private CA in the application account, configure it to use an external CA certificate from the security team's on-premises HSM, and issue certificates directly from this CA

---

**Q34.** A data analytics team processes sensitive customer data using AWS Glue ETL jobs that read from Amazon S3, transform data, and write to Amazon Redshift. The security team mandates that Glue must not have persistent access to decrypt S3 data - decryption permissions should only be available during active job execution. Additionally, all encryption keys must be different for S3, Redshift, and Glue's own metadata. Which configuration meets these requirements?

A) Use three separate KMS customer managed keys for S3, Redshift, and Glue Data Catalog. Create an IAM role for Glue with time-based conditions that restrict kms:Decrypt permissions to working hours only. Use VPC endpoints for all services.
B) Use three separate KMS customer managed keys. Configure the Glue job IAM role with kms:Decrypt permission on the S3 key. Use KMS key policies with kms:EncryptionContext conditions that only allow decryption when aws:PrincipalArn matches the specific Glue job ARN and aws:RequestedRegion matches the deployment region.
C) Use three separate KMS customer managed keys. Create an AWS Lambda function that temporarily grants the Glue IAM role kms:Decrypt permissions on the S3 KMS key at job start and revokes them at job completion using IAM policy modification. Trigger Lambda via EventBridge rules monitoring Glue job state changes.
D) Use three separate KMS customer managed keys. Configure KMS key policy for the S3 key to grant kms:Decrypt only when aws:PrincipalArn matches the Glue service role and kms:ViaService equals s3.amazonaws.com. Use Glue job bookmarks to ensure keys are not cached between runs.

---

**Q35.** A company stores application logs in Amazon S3 with a lifecycle policy that transitions objects to S3 Glacier Flexible Retrieval after 90 days and to Glacier Deep Archive after 1 year. The compliance team requires that all log data, regardless of storage class, must be encrypted with keys that can be immediately disabled in case of a security incident, and that any key disablement must prevent ALL future access to the data even if objects are restored from Glacier. The current encryption uses SSE-KMS with a customer managed key. What additional configuration is required to meet the immediate disable requirement?

A) Enable S3 Object Lock in compliance mode with a 7-year retention period and use SSE-KMS with automatic key rotation disabled so the key can be immediately deleted
B) Modify the KMS key policy to include a 'Deny' statement that can be quickly enabled via a manual policy update, and configure S3 Event Notifications to trigger a Lambda function that denies the key policy when a security incident is detected
C) Configure the KMS customer managed key with imported key material instead of AWS-generated material, allowing immediate deletion of key material while maintaining the key ID, which prevents decryption of all objects across all storage classes
D) Enable S3 Bucket Key with SSE-KMS to reduce the number of KMS requests, then configure AWS Config rules to automatically disable the KMS key when unauthorized access patterns are detected in CloudTrail logs

---

**Q36.** A company uses Amazon Macie to discover and classify sensitive data in S3 buckets. They have configured custom data identifiers to detect proprietary product codes with the pattern 'PROD-[A-Z]{3}-[0-9]{4}'. Security findings show that Macie is generating false positives by matching test data files that contain similar patterns but with an additional '-TEST' suffix (e.g., 'PROD-ABC-1234-TEST'). The company needs to reduce false positives while ensuring actual product codes are still detected. Data is stored in both CSV and JSON formats. What is the MOST effective solution?

A) Modify the custom data identifier regex to 'PROD-[A-Z]{3}-[0-9]{4}(?!-TEST)' using a negative lookahead assertion, and configure maximum match distance of 50 characters to improve detection accuracy
B) Create two custom data identifiers: one with 'PROD-[A-Z]{3}-[0-9]{4}' and another with 'PROD-[A-Z]{3}-[0-9]{4}-TEST', then configure suppression rules to ignore findings from the second identifier
C) Use managed data identifier for credentials instead of custom identifier, create an S3 bucket prefix structure that separates test and production data (e.g., s3://bucket/test/ and s3://bucket/prod/), and configure Macie job scope to exclude the test prefix
D) Modify the custom data identifier to use keywords proximity with required keywords 'production' or 'customer' within 20 characters of the pattern match, and enable ignore words list containing 'test', 'sample', 'demo'

---

**Q37.** A financial services company stores encrypted customer data in Amazon S3. They use AWS KMS customer managed keys with automatic key rotation enabled. The security team has discovered that a former employee who left 8 months ago still has decrypt permissions through an IAM role that was never updated. The company needs to ensure this former employee cannot decrypt any data encrypted after their departure, while maintaining access for current employees. Which solution provides the MOST secure long-term approach?

A) Disable the compromised IAM role and create a new KMS key. Re-encrypt all S3 objects using the new key with a lifecycle policy that automatically migrates objects over 30 days old.
B) Manually rotate the KMS key immediately, update the key policy to remove the compromised role, and use S3 Batch Operations to re-encrypt all objects with the new key material.
C) Add a condition to the KMS key policy using kms:EncryptionContext to require a time-based attribute, then update application code to include timestamps in encryption context for all new encryptions.
D) Enable S3 Object Lock in compliance mode on the bucket, revoke the IAM role's permissions, and configure a bucket policy that denies GetObject requests from the compromised role's ARN.

---

**Q38.** A healthcare organization runs a HIPAA-compliant application that processes protected health information (PHI) across multiple AWS accounts in an AWS Organizations structure. They use AWS Secrets Manager to store database credentials that are rotated every 30 days. The security audit revealed that secrets are being accessed from an Amazon EC2 instance in a non-production account that should not have PHI access. The SecOps team needs to implement controls that prevent cross-account secret access while maintaining legitimate use cases for shared infrastructure secrets. Which solution BEST achieves this requirement?

A) Enable AWS CloudTrail data events for Secrets Manager in all accounts, create an Amazon EventBridge rule that triggers on GetSecretValue API calls, and use AWS Lambda to automatically revoke IAM permissions when cross-account access is detected.
B) Create a centralized Secrets Manager secret in the production account, use AWS Resource Access Manager (RAM) to share secrets with authorized accounts, and implement VPC endpoint policies that restrict access to specific VPC endpoints in authorized accounts.
C) Configure Service Control Policies (SCPs) that deny secretsmanager:GetSecretValue for secrets with a specific tag (PHI=true) unless the request comes from principals in accounts within the production OU, AND implement Secrets Manager resource policies that use aws:SourceVpce condition to restrict access only from specific VPC endpoints.
D) Implement Secrets Manager resource policies on all PHI-related secrets that explicitly deny access using aws:PrincipalOrgID condition key for accounts not in the production OU, but allow cross-account access for secrets without the PHI tag to support shared infrastructure use cases.

---

**Q39.** A financial services company must comply with PCI DSS requirements and needs to demonstrate continuous compliance of their AWS environment. They have 50+ AWS accounts under AWS Organizations. The security team needs to automatically detect when S3 buckets storing cardholder data have public access enabled, when EC2 instances processing payments are missing required security patches, and when IAM policies violate least privilege principles. The solution must provide automated remediation and generate compliance reports for auditors. Which combination of services BEST meets these requirements?

A) Enable AWS Security Hub with PCI DSS standard, use AWS Config managed rules for S3 and IAM, deploy AWS Systems Manager Patch Manager for EC2, and use Security Hub custom actions with EventBridge to trigger Lambda functions for automated remediation
B) Deploy AWS Audit Manager with PCI DSS framework, enable AWS Config conformance packs for all accounts, use AWS Systems Manager State Manager for patch compliance, and configure AWS Config remediation actions with SSM Automation documents
C) Implement AWS CloudTrail with CloudWatch Logs Insights for S3 API monitoring, use Amazon Inspector for EC2 vulnerability scanning, deploy Service Control Policies to prevent IAM policy violations, and create custom Lambda functions for compliance reporting
D) Enable Amazon GuardDuty for threat detection, configure AWS Config rules in the management account, use EC2 Image Builder for compliant AMIs, and implement AWS Control Tower detective controls with AWS Lambda for remediation

---

**Q40.** A healthcare organization processes PHI across multiple AWS accounts and must enforce that all data at rest is encrypted using FIPS 140-3 validated cryptographic modules. They need to prevent any team from launching resources that don't meet this requirement, even if they have AdministratorAccess permissions. The solution must apply to EBS volumes, S3 buckets, RDS databases, and DynamoDB tables. Which approach provides the STRONGEST enforcement?

A) Create SCPs that deny ec2:RunInstances without encrypted EBS, s3:CreateBucket without default encryption, rds:CreateDBInstance without storage encryption, and dynamodb:CreateTable without encryption at rest, then attach to all OUs
B) Deploy AWS Config rules checking for encryption across all services, configure automatic remediation to enable encryption, and use EventBridge to notify security team of violations
C) Implement IAM permission boundaries on all roles requiring aws:SecureTransport condition and kms:EncryptionContext for all KMS operations, blocking unencrypted resource creation
D) Enable AWS CloudFormation Guard in all accounts with policy-as-code rules requiring encryption, configure Service Catalog to only allow compliant products, and use CloudFormation StackSets hooks to validate templates

---

**Q41.** A security team manages 200 AWS accounts and needs to implement a solution where security-critical IAM roles (incident-response-role, security-audit-role, break-glass-admin-role) are identical across all accounts and cannot be modified by account administrators. The roles must automatically update when the central security team modifies the master template. If an account administrator attempts to modify or delete these roles, the change must be automatically reverted within 5 minutes. Which solution accomplishes this with the LEAST operational overhead?

A) Create the IAM roles using CloudFormation StackSets with SELF_MANAGED permissions, enable termination protection, configure EventBridge rules in each account to detect IAM changes via CloudTrail, and trigger Lambda functions to revert unauthorized modifications
B) Define the roles in AWS Control Tower Account Factory customizations, enable drift detection, and configure Service Catalog to redeploy the baseline when drift is detected
C) Use CloudFormation StackSets with SERVICE_MANAGED permissions (AWS Organizations integration), deploy the roles with stack instance retention policy set to RETAIN, enable automatic deployment, and configure StackSets to use OVERRIDE mode on the next deployment cycle
D) Create the IAM roles in the management account, use AWS RAM to share the roles with all member accounts, implement SCPs to prevent iam:DeleteRole and iam:UpdateRole on these specific role names, and schedule Lambda to sync roles nightly

---

**Q42.** A company's security governance requires that all AWS API calls be logged and retained immutably for 7 years for forensic analysis. The logs must be protected from deletion or modification by any user, including the root account. The solution must support querying logs for security investigations and meet SEC 17a-4(f) compliance requirements for immutability. Log volume is approximately 2TB per month. Which solution BEST meets these requirements in the most cost-effective manner?

A) Enable CloudTrail Organization trail logging to S3 with S3 Object Lock in Compliance mode with 7-year retention, enable S3 Glacier Flexible Retrieval storage class transition after 90 days, use S3 MFA Delete, and integrate with Amazon Athena for querying
B) Configure CloudTrail to log to CloudWatch Logs with 7-year retention period, export to S3 using subscription filters, enable S3 Versioning with MFA Delete, and use CloudWatch Logs Insights for querying
C) Enable CloudTrail logging to S3 with SSE-KMS encryption, configure S3 Lifecycle to move logs to S3 Glacier Deep Archive after 30 days, implement Vault Lock on Glacier vaults with 7-year retention policy, and use S3 Select for querying
D) Set up CloudTrail to deliver logs to S3 with S3 Object Lock in Governance mode, create an SCP denying s3:DeleteObject on the log bucket, enable AWS Backup for S3 with 7-year retention, and use Amazon Athena for analysis

---

**Q43.** A security architect needs to design a cross-account access strategy for a multi-account AWS environment where developers in the Development account (111111111111) need read-only access to production CloudWatch metrics and logs in the Production account (222222222222), but should NOT be able to access any production resources, modify any settings, or assume any production IAM roles. The solution must support automatic access for new developers added to the Development account's "Developers" IAM group and must enforce that all access is logged with the developer's identity (not a shared role identity). Which solution BEST meets these requirements?

A) In Production account: Create an IAM role with CloudWatch read permissions and a trust policy allowing the Development account. In Development account: Grant Developers group sts:AssumeRole permission for the Production role. Configure CloudTrail in both accounts to log AssumeRole events.
B) In Production account: Create a resource-based policy on CloudWatch Log Groups allowing read access from Development account principals with aws:PrincipalOrgID condition. Enable CloudWatch cross-account observability. In Development account: Grant Developers group cloudwatch:GetMetricData and logs:FilterLogEvents permissions with resource ARNs from Production account.
C) Configure AWS IAM Identity Center (successor to AWS SSO) with a permission set containing CloudWatch read-only policies, assign the permission set to the Developers group for the Production account, and enable CloudTrail integration to capture user-level access.
D) In Production account: Enable CloudWatch cross-account sharing and create a CloudWatch cross-account sink with resource share via AWS RAM allowing the Development account. In Development account: Grant Developers group permissions to access shared CloudWatch resources and configure CloudTrail to log RAM resource access.

---

**Q44.** A regulated company must ensure that any AWS service supporting their production workloads complies with their corporate security standards before the service can be used. They need to prevent developers from launching new AWS services not yet approved by the security team, while allowing use of approved services. The approval process includes security review, compliance validation, and cost analysis, taking 2-4 weeks per service. Currently approved services: EC2, S3, RDS (PostgreSQL only), Lambda, CloudWatch, VPC. The solution must work across 75 AWS accounts. Which approach provides the MOST restrictive and maintainable enforcement?

A) Create an SCP with an explicit Deny for all AWS services using "NotAction" listing approved services, then add conditions to restrict RDS to PostgreSQL engines only (rds:DatabaseEngine condition). Attach to Production OU. Maintain a central document of approved services and update SCP when new services are approved.
B) Implement a default-deny SCP that denies all actions ("Action": "*"), then create explicit Allow SCPs for each approved service (ec2:*, s3:*, etc.) with RDS limited to PostgreSQL via condition. Attach both policies to Production OU in the correct precedence order.
C) Deploy AWS Config rules in all accounts checking for unapproved service usage, configure automatic remediation to terminate non-compliant resources, and use EventBridge to alert security team. Create IAM permission boundaries limiting users to approved services.
D) Use Service Control Policies with explicit Allow statements for approved services only, include a condition on RDS actions requiring engine type equals postgresql, and add a baseline Deny statement for NotAction: [approved service list]. Configure AWS Config to monitor for violations.

---

