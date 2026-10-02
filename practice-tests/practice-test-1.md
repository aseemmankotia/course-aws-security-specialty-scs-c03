# Practice Test 1: AWS Certified Security Specialty (SCS-C03) Exam Prep

- **Questions:** 44
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-1.md`. Track results in `score-tracker.md`.

---

**Q1.** A security team needs to detect when an IAM user's access keys are used from an IP address outside their corporate IP range (10.0.0.0/8 and 172.16.0.0/12). The solution must generate alerts within 5 minutes and require minimal operational overhead. Which solution BEST meets these requirements?

A) Create a CloudWatch Log Insights query to analyze CloudTrail logs every 5 minutes, then use CloudWatch alarms to trigger SNS notifications when unauthorized IP addresses are detected
B) Enable GuardDuty and configure a custom threat list containing the corporate IP ranges, then create an EventBridge rule to trigger SNS notifications for UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS findings
C) Create an EventBridge rule that matches CloudTrail events where sourceIPAddress is not in the corporate ranges, then trigger a Lambda function to evaluate and send SNS notifications
D) Enable AWS Config with a custom rule that evaluates IAM access key usage against IP address ranges, then use Config conformance packs to trigger remediation workflows

---

**Q2.** A financial services company must detect and alert when someone attempts to disable CloudTrail logging in any AWS account across their organization. The security team needs a centralized solution that works for all current and future accounts. Which approach provides the MOST comprehensive detection coverage?

A) Enable CloudTrail in the organization management account with organization trail enabled, then create an EventBridge rule matching StopLogging API calls that triggers SNS notifications
B) Use AWS Config aggregator in the security account to monitor CloudTrail configuration changes, then create Config rules to detect when logging is disabled
C) Deploy a Lambda function via StackSets to all accounts that monitors CloudTrail status every 5 minutes and publishes metrics to CloudWatch in the security account
D) Configure GuardDuty in all accounts with S3 Protection enabled, which automatically detects CloudTrail logging disruptions and generates findings

---

**Q3.** A company's security policy requires detection of any EC2 instance launched without encryption enabled on all attached EBS volumes. The detection must occur within 2 minutes of instance launch and work for all instance types including those with ephemeral storage. Which solution meets these requirements with the LEAST complexity?

A) Create an AWS Config rule using the 'encrypted-volumes' managed rule with a trigger type of configuration changes, then use Config to automatically remediate non-compliant instances
B) Enable Amazon Inspector and configure an assessment template that runs every 5 minutes to check EBS volume encryption status across all EC2 instances
C) Create an EventBridge rule that matches RunInstances API calls, trigger a Lambda function to describe volumes and check encryption settings, then publish findings to Security Hub
D) Use GuardDuty with Runtime Monitoring enabled to detect unencrypted EBS volumes attached to EC2 instances and generate findings automatically

---

**Q4.** A security analyst needs to investigate potential data exfiltration. They suspect an attacker may have used compromised credentials to access S3 buckets and download sensitive data over the past 72 hours across multiple AWS regions. Which combination of AWS services provides the MOST efficient investigation approach?

A) Use CloudTrail Event History in each region to search for GetObject API calls, then manually correlate events across regions using the requestId field
B) Query CloudTrail logs stored in S3 using Athena with a partitioned table structure, filtering on eventName='GetObject' and eventTime within 72 hours, then analyze sourceIPAddress and userIdentity patterns
C) Enable VPC Flow Logs on all VPCs and use CloudWatch Logs Insights to search for outbound S3 traffic patterns over port 443
D) Use Amazon Detective to visualize S3 access patterns by querying the behavior graph for unusual GetObject activity correlated with authentication events

---

**Q5.** A company needs to detect when an EC2 instance in their VPC is communicating with known cryptocurrency mining pools. The security team has a list of 5,000 malicious IP addresses and domain names that are updated weekly. Which solution provides automated detection with the LEAST ongoing maintenance?

A) Enable VPC Flow Logs and stream them to CloudWatch Logs, then create metric filters for each of the 5,000 IP addresses and configure alarms for each metric
B) Enable GuardDuty and upload the IP addresses and domain names to a GuardDuty threat list in the S3 bucket, updating the S3 file weekly via automation
C) Create Network Firewall with a stateful rule group using domain list and IP set references, then configure CloudWatch alarms on rejected packet metrics
D) Deploy a Lambda function triggered by VPC Flow Logs that checks destination IPs against a DynamoDB table containing the threat list, then publishes findings to SNS

---

**Q6.** A security team discovered that an EC2 instance is making DNS queries to a domain associated with a known command-and-control (C2) server. They need to implement a detection mechanism that identifies similar behavior across all instances in the VPC, including future instances, with minimal latency. Which approach provides the MOST effective detection?

A) Enable VPC DNS query logging to CloudWatch Logs, create a subscription filter that sends logs to Lambda for analysis against a threat intelligence feed, then publish alerts to SNS
B) Configure Route 53 Resolver DNS Firewall with a managed domain list provided by AWS Threat Intelligence, then monitor CloudWatch metrics for blocked queries
C) Enable GuardDuty and ensure VPC Flow Logs are enabled, which allows GuardDuty to detect DNS queries to known C2 domains automatically
D) Deploy a transparent DNS proxy on EC2 instances using Systems Manager Run Command, configure it to log queries and check against threat feeds, then forward alerts to Security Hub

---

**Q7.** A security team needs to detect when IAM users are performing API calls from IP addresses outside their corporate network ranges (10.0.0.0/8 and 172.16.0.0/12). The solution must generate alerts within 5 minutes and provide details about the principal, action, and source IP. Which solution BEST meets these requirements?

A) Create a CloudWatch Logs Insights query to analyze CloudTrail logs every 5 minutes and send notifications via SNS when unauthorized IPs are detected
B) Create an EventBridge rule that filters CloudTrail events for sourceIPAddress not matching the corporate CIDR ranges, then sends to SNS
C) Enable GuardDuty and create a suppression rule for findings originating from corporate IP ranges, then configure SNS notifications for remaining findings
D) Create a CloudWatch Logs metric filter on CloudTrail logs matching IPs outside corporate ranges, set a CloudWatch alarm with a 5-minute period, and send to SNS

---

**Q8.** A security team discovers that an IAM user's access keys have been exposed in a public GitHub repository for the past 72 hours. CloudTrail logs show the keys were used to launch 50 EC2 instances in an unfamiliar region and create multiple S3 buckets. What should be the FIRST step in the incident response process?

A) Delete all EC2 instances and S3 buckets created by the compromised credentials
B) Disable the IAM user's access keys and rotate all credentials
C) Contact AWS Support to request an investigation and forensic analysis
D) Review CloudTrail logs to document all actions taken by the compromised credentials

---

**Q9.** A company's Security Operations Center (SOC) needs to automate incident response for GuardDuty findings classified as 'High' severity. The solution must isolate affected EC2 instances by modifying security groups, capture forensic snapshots, and notify the security team via email. Which solution provides this automation with the LEAST operational overhead?

A) Create an EventBridge rule triggered by GuardDuty findings, invoke a Lambda function to modify security groups and create snapshots, publish to SNS topic for notifications
B) Configure GuardDuty to send findings to Security Hub, use Security Hub automated response actions to modify security groups, manually create snapshots
C) Use AWS Systems Manager Incident Manager with runbooks to orchestrate security group changes, snapshot creation, and SNS notifications
D) Deploy a third-party SIEM solution that ingests GuardDuty findings and executes automated remediation scripts via API calls

---

**Q10.** During a security incident investigation, a forensics team needs to analyze the memory contents of a compromised EC2 instance running Amazon Linux 2023. The instance must remain running to preserve volatile data, but must be isolated from the network. Which approach allows memory capture while maintaining isolation?

A) Create an AMI of the running instance, launch a new instance from the AMI in an isolated VPC, then capture memory from the new instance
B) Attach a forensics security group that denies all inbound/outbound traffic except from a forensics workstation, use SSM Session Manager to connect and run memory capture tools
C) Stop the instance to create an EBS snapshot, analyze the snapshot using EC2 memory forensics tools in a separate forensics account
D) Detach the instance's network interface, attach a new network interface in an isolated subnet, then use EC2 Instance Connect to access the instance

---

**Q11.** A DDoS attack is targeting a public Application Load Balancer. CloudWatch metrics show request rates of 500,000 requests per second from 10,000 unique IP addresses across multiple geographic regions. The application normally handles 5,000 requests per second. AWS Shield Standard is active. What is the MOST cost-effective immediate mitigation while maintaining availability for legitimate users?

A) Enable AWS WAF on the ALB with a rate-based rule limiting requests to 100 per 5-minute period per IP address
B) Upgrade to AWS Shield Advanced to access DDoS Response Team (DRT) support and advanced mitigation
C) Create a CloudFront distribution in front of the ALB and enable AWS WAF with geo-blocking for attacking regions
D) Implement Amazon Route 53 health checks to failover to a static S3 website during the attack

---

**Q12.** A security analyst receives a GuardDuty finding 'UnauthorizedAccess:IAMUser/InstanceCredentialExfiltration.OutsideAWS' indicating that temporary credentials from an EC2 instance IAM role were used from an external IP address. CloudTrail shows API calls to s3:ListBuckets and s3:GetObject from the external IP. What is the MOST likely attack scenario and appropriate response?

A) The EC2 instance metadata service (IMDS) was exploited via SSRF vulnerability. Update to IMDSv2 and implement hop limit of 1, investigate application code for SSRF vulnerabilities
B) An employee is legitimately using AWS CLI from home with assumed role credentials. Verify the IP address belongs to a known employee and update security group rules
C) The IAM role trust policy was misconfigured to allow external account assumption. Review and restrict the trust policy to only allow EC2 service principal
D) Credentials were leaked through application logs to CloudWatch. Rotate the IAM role credentials and implement secrets scanning in the CI/CD pipeline

---

**Q13.** A company stores sensitive PII data in S3 encrypted with SSE-KMS using a customer managed key. After a security incident, forensics requires analyzing deleted objects from the past 30 days, but MFA Delete is enabled on the bucket and the original MFA device associated with the root account is lost. The bucket has versioning enabled. How can the security team retrieve the deleted object versions?

A) Contact AWS Support to disable MFA Delete, retrieve deleted versions using S3 API ListObjectVersions and GetObjectVersion, then re-enable MFA Delete with a new MFA device
B) Use S3 Batch Operations with Restore Object action to restore all deleted versions in bulk, bypassing MFA Delete requirements for automated operations
C) Access deleted object versions directly through AWS CLI using s3api list-object-versions and get-object commands with root account credentials without MFA
D) Enable S3 Access Logs to identify deleted object keys, then use AWS Backup to restore the objects from automated S3 snapshots

---

**Q14.** A financial services company runs a multi-tier application on Amazon EC2 instances across multiple Availability Zones. The application processes sensitive customer data and must comply with PCI DSS requirements. Security policies mandate that all network traffic between application tiers must be encrypted in transit, and the company needs to verify encryption is enforced. Which solution BEST meets these requirements with minimal operational overhead?

A) Deploy AWS Certificate Manager Private CA and configure EC2 instances to use TLS certificates. Enable VPC Flow Logs and create Amazon EventBridge rules to detect unencrypted traffic.
B) Implement AWS PrivateLink endpoints between tiers and enable AWS PrivateLink traffic encryption. Use VPC Flow Logs with Amazon Athena to verify all connections use PrivateLink.
C) Create a transit gateway and enable transit gateway network insights. Configure security groups to only allow HTTPS traffic and use AWS Config rules to verify security group compliance.
D) Deploy a Network Load Balancer with TLS listeners for each tier and enable TLS termination. Configure target health checks to verify TLS handshake success and use AWS Config to monitor NLB configurations.

---

**Q15.** A company uses AWS Organizations with multiple AWS accounts for development, staging, and production environments. The security team needs to prevent any EC2 instance in the development accounts from being launched with a public IP address, while production accounts must retain this capability for specific use cases. Which approach provides the MOST centralized and effective control?

A) Create an IAM permissions boundary in each development account that denies the ec2:AssociateAddress and ec2:RunInstances actions when PublicIp is true, and attach it to all IAM roles.
B) Use AWS Config rules with automatic remediation in development accounts to terminate any EC2 instance with a public IP address within 5 minutes of detection.
C) Implement a Service Control Policy (SCP) that denies ec2:RunInstances when the instance will have a public IP, and apply it to the development organizational unit.
D) Configure VPC settings in development accounts to disable auto-assign public IPv4 addresses on all subnets, and use AWS Config to detect manual associations.

---

**Q16.** An enterprise maintains a fleet of 500 Amazon EC2 instances that require consistent security hardening and patch management across Windows and Linux operating systems. The security team needs to ensure all instances comply with CIS benchmarks, automatically remediate drift, and generate compliance reports. Current manual processes are error-prone and time-consuming. Which solution provides the MOST automated and comprehensive approach?

A) Use AWS Systems Manager Patch Manager with maintenance windows to automate patching, and implement AWS Config conformance packs with CIS benchmark rules to detect drift and generate compliance reports.
B) Deploy AWS Systems Manager State Manager with associations that apply CIS benchmark configurations, use Patch Manager for automated patching, and leverage AWS Security Hub to aggregate compliance findings from both services.
C) Implement Amazon Inspector to continuously scan instances for CIS benchmark compliance and software vulnerabilities, and use AWS Systems Manager Run Command to execute remediation scripts when non-compliance is detected.
D) Create custom AMIs hardened to CIS benchmarks, use Auto Scaling groups to replace non-compliant instances, and implement AWS Systems Manager Inventory with AWS Config to track compliance status.

---

**Q17.** A healthcare organization processes Protected Health Information (PHI) in AWS and must demonstrate HIPAA compliance. Their VPCs contain a mix of EC2 instances, RDS databases, and Lambda functions. During a security audit, they need to prove that all network traffic destined for AWS services (S3, DynamoDB, Secrets Manager) never traverses the public internet. Which approach provides the MOST comprehensive verification of private connectivity?

A) Deploy VPC endpoints (Gateway and Interface) for all AWS services in use, then enable VPC Flow Logs with custom format including pkt-dstaddr fields and query using Amazon Athena to verify no traffic reaches public AWS service IP addresses.
B) Configure AWS Config rules to verify that S3 bucket policies deny requests that don't originate from specific VPC endpoints (aws:SourceVpce condition), and verify RDS instances are not publicly accessible.
C) Implement Network Firewall in each VPC with stateful rules to block all traffic to AWS service public IP ranges. Enable Network Firewall flow logs to verify no traffic reaches public AWS endpoints.
D) Enable VPC Flow Logs with default format and use CloudWatch Logs Insights to identify traffic patterns. Configure VPC endpoint policies to restrict access and monitor VPC endpoint CloudWatch metrics for usage verification.

---

**Q18.** A SaaS company runs containerized microservices on Amazon ECS with Fargate launch type across multiple VPCs. Each customer tenant is isolated in a separate VPC. The security team needs to implement centralized egress filtering to block connections to known malicious domains and IP addresses for all customer VPCs, while maintaining network isolation between tenants. The solution must scale to 100+ customer VPCs with minimal management overhead. Which architecture BEST meets these requirements?

A) Deploy AWS Network Firewall in a central egress VPC. Create VPC peering connections from each customer VPC to the central VPC, route all egress traffic through the Network Firewall, and configure stateful domain list and IP set rules.
B) Implement AWS Transit Gateway with a central inspection VPC containing Network Firewall. Attach all customer VPCs to the Transit Gateway, configure routing to direct internet-bound traffic through the inspection VPC, and use Network Firewall with managed threat intelligence feeds.
C) Deploy NAT Gateways in each customer VPC and configure them to use AWS Network Firewall endpoints. Create managed prefix lists containing malicious IPs and use VPC route tables to direct traffic through Network Firewall stateless rule groups.
D) Use AWS PrivateLink to share a centralized Network Firewall deployment across all customer VPCs. Configure each VPC's route table to send internet traffic to the PrivateLink endpoint and apply domain filtering rules at the Network Firewall.

---

**Q19.** A media company stores user-uploaded video files in Amazon S3 that are processed by EC2 instances in private subnets. Due to a recent security incident involving data exfiltration, the CISO mandates that EC2 instances must not be able to download S3 objects to unauthorized destinations outside the VPC, even if IAM credentials are compromised. The instances need read/write access to the company's S3 buckets but should be blocked from accessing any external S3 buckets. Which combination of controls MOST effectively enforces this requirement?

A) Create a VPC endpoint for S3 with an endpoint policy that allows access only to the company's S3 buckets. Remove all routes to the internet gateway and NAT gateway from the private subnet route tables. Configure S3 bucket policies with aws:SourceVpc condition.
B) Implement S3 Access Points for each authorized bucket with VPC-restricted access. Configure IAM roles for EC2 instances with permissions to access only these Access Points. Enable S3 Block Public Access at the account level.
C) Deploy a Gateway VPC endpoint for S3 and configure the endpoint policy to deny access to all S3 buckets except the company's buckets. Modify the route table to prioritize the VPC endpoint route. Add S3 bucket policies requiring aws:SourceVpce condition.
D) Configure EC2 instance IAM roles with explicit Deny statements for S3 actions on external buckets using NotResource element. Enable VPC Flow Logs to detect any S3 access attempts. Implement AWS Config rules to monitor IAM policy changes.

---

**Q20.** A company runs a fleet of Amazon EC2 instances that must be accessed by system administrators for maintenance tasks. Security policies require that SSH access be restricted to administrators connecting through the corporate VPN (IP range 10.50.0.0/16), and all SSH sessions must be logged with full command audit trails for compliance purposes. The solution must prevent direct internet SSH access and provide centralized session logging without requiring changes to existing EC2 instance configurations or installing additional agents. Which solution BEST meets these requirements?

A) Create a security group allowing SSH from 10.50.0.0/16, enable VPC Flow Logs to capture SSH traffic, configure EC2 instances to send auth.log files to CloudWatch Logs, and use CloudWatch Insights to query session data
B) Configure AWS Systems Manager Session Manager with IAM policies restricting access to users authenticated through AWS IAM Identity Center, enable session logging to S3 and CloudWatch Logs, and create a security group allowing HTTPS from 10.50.0.0/16 for Systems Manager connectivity
C) Deploy a bastion host in a public subnet with a security group allowing SSH from 10.50.0.0/16, configure the bastion to log all SSH sessions to CloudWatch Logs, and update EC2 security groups to allow SSH only from the bastion host's security group
D) Enable AWS CloudTrail data events for EC2 RunInstances API calls, create a security group allowing SSH from 10.50.0.0/16, configure auditd on each EC2 instance to capture command execution, and forward logs to a centralized syslog server

---

**Q21.** A healthcare technology company hosts a HIPAA-compliant application using Amazon ECS Fargate tasks that process Protected Health Information (PHI). The company's security team requires that container images must be automatically scanned for vulnerabilities before deployment, and any images with HIGH or CRITICAL vulnerabilities must be blocked from running in production. Failed scans should trigger automated notifications to the security team. The solution must integrate with the existing CI/CD pipeline using AWS CodePipeline. Which approach provides the MOST automated and secure solution?

A) Configure Amazon ECR image scanning on push, create an EventBridge rule that triggers on scan completion with HIGH/CRITICAL findings, invoke a Lambda function to update CodePipeline variables blocking deployment and send SNS notifications
B) Enable AWS Security Hub with ECR integration, configure Security Hub custom actions to block deployments, and use Security Hub findings for notifications
C) Implement a third-party scanning tool in CodeBuild, store results in S3, use S3 event notifications to trigger Lambda that parses results and updates deployment gates
D) Configure Amazon Inspector to continuously scan ECS tasks, create CloudWatch alarms for vulnerability findings, and manually review before each production deployment

---

**Q22.** A company has multiple AWS accounts managed through AWS Organizations. The security team needs to ensure that IAM users in the development account cannot create or modify IAM roles that have more permissions than their own user policies. Which solution provides the MOST restrictive control?

A) Implement a service control policy (SCP) that denies iam:CreateRole and iam:PutRolePolicy for all principals in the development account
B) Configure IAM permissions boundaries on all IAM users in the development account that limit the maximum permissions they can grant to roles
C) Create an IAM policy that denies iam:PassRole action for all users in the development account
D) Enable AWS CloudTrail and configure Amazon EventBridge rules to automatically delete any roles created with elevated permissions

---

**Q23.** A financial services application uses Amazon Cognito user pools for authentication. Regulatory requirements mandate that user session tokens must be revoked within 5 minutes if a user's account is disabled. The current configuration uses the default 1-hour access token validity. What is the MOST efficient solution to meet this requirement?

A) Reduce the access token validity period to 5 minutes in the Cognito user pool app client settings
B) Implement a Lambda function triggered by user disable events that calls the AdminUserGlobalSignOut API to revoke all tokens for the user
C) Configure Cognito user pool advanced security to automatically revoke sessions when accounts are disabled
D) Create an API Gateway authorizer that checks the user's account status in DynamoDB on every request

---

**Q24.** A company needs to grant a third-party vendor temporary access to read objects from a specific S3 bucket for 7 days. The vendor has their own AWS account. The security team requires the solution to automatically revoke access after 7 days with no manual intervention and wants to minimize long-term credential management. Which approach is MOST secure?

A) Create an IAM user in the company's account with read permissions to the S3 bucket, provide the access keys to the vendor, and set a calendar reminder to delete the user after 7 days
B) Configure an S3 bucket policy that allows the vendor's AWS account access and includes a Condition element with DateLessThan set to 7 days from now
C) Create an IAM role with read permissions to the S3 bucket, establish a trust relationship with the vendor's AWS account, and configure the role's maximum session duration to 7 days
D) Use AWS STS AssumeRole to generate temporary credentials for the vendor with a session duration of 7 days

---

**Q25.** A healthcare organization stores patient records in Amazon S3. Security policies require that only the data encryption service can decrypt objects, and individual IAM users or roles should never have direct decrypt permissions. Application servers need to read the encrypted objects. Which encryption configuration satisfies this requirement?

A) Use SSE-S3 with default AWS managed keys and grant s3:GetObject permission to application roles
B) Use SSE-KMS with a customer managed key and apply a key policy that only grants kms:Decrypt to the S3 service principal
C) Use SSE-C with client-provided encryption keys stored in AWS Secrets Manager
D) Use SSE-KMS with a customer managed key and grant kms:Decrypt via VIA grants that restrict usage to S3 operations only

---

**Q26.** A development team uses AWS CodeCommit for source control. The security team needs to ensure that developers can only push commits that are signed with their GPG keys, and any unsigned commits must be automatically rejected. Which solution implements this requirement?

A) Create a CodeCommit approval rule template that requires at least one approval from a designated approver group before merging unsigned commits
B) Configure AWS Config rules to detect unsigned commits and trigger a Lambda function to revert them automatically
C) Enable branch protection on all CodeCommit repositories and configure a pre-receive hook that validates GPG signatures on all commits
D) Implement an AWS Lambda function triggered by CodeCommit events that verifies commit signatures and uses Git commands to reject unsigned commits

---

**Q27.** A company operates a multi-region application with separate AWS accounts for each region. Users authenticate via SAML 2.0 federation with on-premises Active Directory. The security team reports that users sometimes receive 'AssumeRoleWithSAML' errors with the message 'Maximum session duration exceeded' when attempting to access resources in the Asia Pacific account, but not in other regional accounts. All IAM roles have identical policies. What is the MOST likely cause?

A) The SAML assertion's SessionDuration attribute in the Asia Pacific region exceeds the maximum session duration configured on the IAM role in that account
B) The Asia Pacific account has a service control policy (SCP) that restricts the maximum session duration for federated users
C) Clock skew between the on-premises SAML identity provider and AWS in the Asia Pacific region causes assertion timestamps to be invalid
D) The SAML trust relationship in the Asia Pacific account's IAM role has an incorrect SAML provider ARN

---

**Q28.** A financial services company requires that all API calls to their AWS resources be authenticated using temporary credentials that expire after 1 hour. The company has a mobile application with 2 million users who need to access Amazon S3 objects and invoke AWS Lambda functions. The solution must minimize credential management overhead and prevent long-term credential exposure. Which solution BEST meets these requirements?

A) Create an IAM user for each mobile app user and configure the mobile app to rotate access keys every hour using AWS Secrets Manager
B) Implement Amazon Cognito Identity Pools with an IAM role that has appropriate permissions, and configure the session duration to 1 hour
C) Use AWS STS AssumeRole with an external identity provider, storing the temporary credentials in Amazon DynamoDB with a 1-hour TTL
D) Configure AWS IAM Identity Center with SAML 2.0 federation and set the session duration to 1 hour for all users

---

**Q29.** A security engineer is implementing a permissions boundary for a development team's IAM roles. The company policy requires that developers cannot create IAM roles with more permissions than their own roles possess, and they must not be able to escalate privileges by modifying their own permissions boundary. The developers need to create IAM roles for Lambda functions. Which configuration BEST enforces this requirement?

A) Attach a permissions boundary to the developers' IAM roles with a Deny statement for iam:PutRolePermissionsBoundary on their own role ARN
B) Create an SCP that denies iam:CreateRole unless aws:RequestedRegion equals us-east-1 and attach it to the developers' OU
C) Attach a permissions boundary to the developers' IAM roles and use an IAM policy condition requiring iam:PermissionsBoundary in CreateRole actions to match the developers' boundary ARN
D) Enable AWS CloudTrail and create an EventBridge rule that triggers a Lambda function to automatically delete any IAM role created without a permissions boundary

---

**Q30.** A company uses AWS Organizations with multiple AWS accounts. The security team needs to enforce that all IAM roles in member accounts used for cross-account access must include an ExternalId condition and must only be assumable by principals from within the organization. An audit revealed that several roles lack these controls. Which action should the security team take to enforce both requirements?

A) Create an SCP that denies iam:CreateRole and iam:UpdateAssumeRolePolicy unless the trust policy includes both a Condition with sts:ExternalId and aws:PrincipalOrgID matching the organization ID
B) Create an SCP that denies sts:AssumeRole if aws:PrincipalOrgID does not match the organization ID, then enable AWS Config rule iam-policy-no-statements-with-admin-access for remediation
C) Enable AWS Config managed rule iam-role-managed-policy-check to detect roles without ExternalId, then create an SCP denying sts:AssumeRole from outside the organization
D) Create an SCP that denies sts:AssumeRole if the request lacks sts:ExternalId or if aws:SourceAccount is not within the organization's account list

---

**Q31.** A financial services company stores customer transaction data in Amazon S3. The company must ensure that all data is encrypted at rest and that they maintain full control over the encryption keys, including the ability to immediately revoke access. The solution must also provide detailed audit logs of all key usage. Which solution meets these requirements?

A) Use S3 default encryption with Amazon S3-Managed Keys (SSE-S3)
B) Use S3 encryption with AWS KMS Customer Managed Keys (SSE-KMS) and enable CloudTrail logging for the KMS key
C) Use S3 encryption with AWS KMS AWS-Managed Keys (SSE-KMS) and enable CloudWatch logging
D) Use client-side encryption with AWS Encryption SDK and store keys in AWS Secrets Manager

---

**Q32.** A healthcare application running on Amazon EC2 processes patient records containing Protected Health Information (PHI). The security team requires that all Amazon EBS volumes attached to these instances be encrypted, and they want to prevent any team from launching EC2 instances with unencrypted EBS volumes in the future. What is the MOST effective way to enforce this requirement?

A) Create an IAM policy that denies ec2:RunInstances unless the EBS volume is encrypted and attach it to all IAM users and roles
B) Enable EBS encryption by default in each AWS Region where the application runs
C) Implement AWS Config rule ebs-encrypted-volumes and configure automatic remediation to encrypt volumes
D) Use AWS Systems Manager to scan for unencrypted volumes daily and send alerts to the security team

---

**Q33.** A company uses Amazon RDS for PostgreSQL to store application data. The database contains sensitive customer information that must be encrypted at rest. The company's compliance framework requires that encryption keys be rotated every 90 days and that all key usage be auditable. The database is already in production with unencrypted data. Which approach satisfies these requirements with MINIMAL application downtime?

A) Create a snapshot of the unencrypted DB instance, copy the snapshot with encryption enabled using a KMS Customer Managed Key with automatic rotation, restore from the encrypted snapshot, and update the application endpoint
B) Enable encryption on the existing RDS instance using the modify-db-instance command with the --storage-encrypted parameter
C) Create a read replica of the DB instance with encryption enabled, promote the replica to primary, then update the application to use the new endpoint
D) Use AWS Database Migration Service (DMS) to migrate data from the unencrypted instance to a new encrypted RDS instance with continuous replication

---

**Q34.** A media company stores video files in Amazon S3. The company needs to provide secure, time-limited access to specific videos for premium subscribers without exposing the S3 bucket publicly or requiring AWS credentials. Videos range from 50MB to 5GB in size. Which solution provides the MOST secure access control?

A) Generate S3 presigned URLs with expiration times using AWS STS temporary credentials and distribute them to authenticated users
B) Enable S3 bucket ACLs and grant read permissions to authenticated users for specific objects
C) Use Amazon CloudFront with signed URLs or signed cookies and an Origin Access Control (OAC) to restrict S3 bucket access
D) Create an IAM role for each user and grant s3:GetObject permissions with resource-level conditions limiting access to specific prefixes

---

**Q35.** A company runs a multi-tenant SaaS application that stores customer data in Amazon DynamoDB. Each customer's data must be encrypted with a unique encryption key, and customers must be able to bring their own keys (BYOK). The solution must ensure that one customer cannot access another customer's data, even if application-layer access controls fail. Which solution meets these requirements?

A) Use DynamoDB encryption with AWS KMS and create a separate Customer Managed Key for each customer with key policies restricting access to specific IAM roles
B) Implement client-side encryption using AWS Encryption SDK with customer-provided keys stored in separate AWS Secrets Manager secrets per customer
C) Enable DynamoDB encryption with AWS-managed keys and implement attribute-level encryption using different KMS keys per customer
D) Use separate DynamoDB tables per customer, each encrypted with a different Customer Managed Key and table-level IAM policies

---

**Q36.** A research organization needs to share encrypted genomic data files (10-50GB each) stored in Amazon S3 with external collaborators at partner institutions. The data is encrypted using AWS KMS Customer Managed Keys. The collaborators have their own AWS accounts. The solution must minimize data transfer costs and ensure that only authorized collaborators can decrypt the data. What is the MOST cost-effective and secure solution?

A) Create IAM roles in the data owner's account with cross-account access for collaborators and grant decrypt permissions on the KMS key
B) Copy the encrypted objects to the collaborators' S3 buckets using S3 Cross-Region Replication and share the KMS key ARN
C) Update the KMS key policy to grant decrypt permissions to the external AWS accounts, configure S3 bucket policy for cross-account access, and have collaborators copy objects directly to their accounts
D) Decrypt the files, upload them to a new S3 bucket without encryption, and share presigned URLs with collaborators

---

**Q37.** A financial services company stores sensitive customer transaction data in Amazon S3. The company must ensure that all data is encrypted at rest and that they maintain full control over the encryption key lifecycle, including the ability to immediately revoke access to the data. The security team requires detailed audit logs of all key usage. The solution must meet compliance requirements for separation of duties between application teams and key administrators. Which encryption solution BEST meets these requirements?

A) Use Amazon S3 server-side encryption with Amazon S3 managed keys (SSE-S3) and enable CloudTrail logging for S3 bucket access
B) Use Amazon S3 server-side encryption with AWS KMS keys (SSE-KMS) with customer managed keys, enable KMS key rotation, and use CloudTrail to log KMS API calls
C) Use Amazon S3 server-side encryption with customer-provided keys (SSE-C) and store the encryption keys in AWS Secrets Manager with automatic rotation enabled
D) Use client-side encryption with AWS Encryption SDK and store master keys in AWS CloudHSM, configuring IAM policies to separate key administration from key usage

---

**Q38.** A healthcare organization runs a web application on Amazon EC2 instances that processes protected health information (PHI). The application connects to an Amazon RDS PostgreSQL database containing patient records. The security team needs to ensure that database credentials are never stored in application code or configuration files, are automatically rotated every 30 days, and that all credential retrieval attempts are logged. The solution must minimize application code changes. Which approach BEST meets these requirements?

A) Store database credentials in AWS Systems Manager Parameter Store as SecureString parameters, enable automatic rotation using Lambda functions, and configure the application to retrieve credentials using the AWS SDK with IAM role authentication
B) Configure the RDS database to use IAM database authentication, modify the application to generate authentication tokens using the AWS SDK, and use CloudTrail to log all authentication attempts
C) Store database credentials in AWS Secrets Manager, enable automatic rotation for RDS PostgreSQL, configure the application to retrieve credentials using the Secrets Manager API, and enable CloudTrail logging for Secrets Manager
D) Use AWS Certificate Manager to generate client certificates for database authentication, store the certificates in Amazon S3 with server-side encryption, and rotate certificates monthly using Lambda functions

---

**Q39.** A financial services company must demonstrate compliance with PCI DSS requirements for their AWS infrastructure. They need to implement automated compliance checking, generate evidence reports for auditors, and receive alerts when resources drift from compliant configurations. The solution must minimize operational overhead while providing continuous compliance visibility. Which AWS service BEST meets these requirements?

A) AWS CloudTrail with EventBridge rules to trigger Lambda functions that check compliance and send SNS notifications
B) AWS Config with conformance packs for PCI DSS, AWS Security Hub for centralized findings, and EventBridge for alerting
C) AWS Systems Manager Compliance with custom compliance rules and CloudWatch dashboards for visualization
D) AWS Audit Manager with PCI DSS framework assessment, automated evidence collection, and built-in reporting

---

**Q40.** An organization uses AWS Organizations with multiple AWS accounts across development, staging, and production environments. Security policies require that all S3 buckets must have versioning enabled and all EC2 instances must be launched from approved AMIs only. These policies must be enforced preventively across all accounts including future accounts. What is the MOST effective solution?

A) Create AWS Config rules in each account to detect non-compliant resources and use AWS Systems Manager Automation to remediate violations
B) Use Service Control Policies (SCPs) to deny s3:PutBucketVersioning with versioning disabled and deny ec2:RunInstances for non-approved AMIs
C) Implement AWS Control Tower with detective guardrails and AWS Config conformance packs deployed to all organizational units
D) Deploy AWS CloudFormation StackSets with AWS Config rules and AWS Lambda remediation functions to all accounts in the organization

---

**Q41.** A company needs to implement separation of duties for their AWS environment where no single person can both provision infrastructure and approve security changes. Database administrators should manage RDS instances but not modify security groups. Network administrators should manage VPCs and security groups but not access databases. What is the MOST effective approach to enforce this separation with least privilege?

A) Implement separate AWS accounts for each function using AWS Organizations, use cross-account IAM roles for necessary access, and apply SCPs to enforce organizational boundaries
B) Create separate IAM roles for each function, use IAM policy conditions with aws:PrincipalArn to prevent role assumption conflicts, and require MFA for sensitive actions
C) Use IAM permission boundaries on user roles to limit maximum permissions, implement resource tags for ownership, and use tag-based access control (TBAC) in IAM policies
D) Create IAM policies with explicit Deny statements for cross-functional actions, use IAM groups for job functions, and enable CloudTrail for audit logging

---

**Q42.** During a security audit, you discover that IAM users in the development account have been creating IAM roles with the AdministratorAccess managed policy attached. The development account is part of an AWS Organization. What is the MOST effective way to prevent this privilege escalation while allowing developers to create roles with appropriate limited permissions?

A) Create an SCP that denies iam:CreateRole and iam:AttachRolePolicy for any policy with 'AdministratorAccess' in the name
B) Use an IAM permission boundary policy that must be attached to all created roles, limiting maximum permissions to development-appropriate actions only
C) Enable AWS IAM Access Analyzer and configure findings to automatically delete roles with overly permissive policies
D) Implement AWS Config rules to detect roles with AdministratorAccess and use AWS Systems Manager Automation to remove the policy

---

**Q43.** A healthcare company must ensure that all AWS API calls across their multi-account organization are logged immutably for 7 years to meet HIPAA compliance requirements. The logs must be protected from deletion even by account administrators, and log integrity must be verifiable. Which solution provides these capabilities?

A) Enable CloudTrail in each account with logs sent to a central S3 bucket with versioning enabled and bucket policy denying all delete operations
B) Create an organization trail in AWS CloudTrail with S3 Object Lock in compliance mode, MFA delete enabled, and CloudTrail log file validation
C) Use Amazon CloudWatch Logs with log retention set to 2557 days, export to S3 with Glacier storage class, and enable S3 bucket versioning
D) Configure CloudTrail to deliver logs to CloudWatch Logs, create a Kinesis Data Firehose delivery stream to S3 with AWS Backup for long-term retention

---

**Q44.** Your company operates in multiple AWS regions and requires that all security findings from AWS Security Hub, GuardDuty, and Inspector be aggregated in a single account and region for the security team to review. New accounts and regions are regularly added to the organization. Which approach provides automated aggregation with LEAST operational overhead?

A) Enable GuardDuty, Inspector, and Security Hub delegated administrator in the security account, configure organization-wide enablement, and use Security Hub cross-region aggregation to a single region
B) Use EventBridge rules in each account to forward security findings to a central EventBridge bus, then Lambda functions to process and store findings in DynamoDB
C) Deploy CloudFormation StackSets to enable services in all accounts, configure each service to publish to SNS topics that forward to a central SQS queue for processing
D) Create AWS Config aggregators in the security account to collect findings from all accounts and use AWS Systems Manager Explorer for unified visibility

---

