# Answer Key — Practice Test 1

**Q1. Correct: C**

EventBridge rules can directly pattern-match on CloudTrail event fields including sourceIPAddress. Using EventBridge with Lambda provides real-time detection (under 1 minute latency), requires no infrastructure management, and allows custom logic to evaluate IP ranges. This is the most operationally efficient solution for this specific IP-based detection requirement.

---

**Q2. Correct: A**

An organization trail in the management account automatically applies to all accounts in the organization including future accounts. EventBridge rules matching StopLogging (cloudtrail:StopLogging) API calls provide real-time detection of attempts to disable logging. This centralized approach requires minimal configuration and automatically covers new accounts without additional deployment.

---

**Q3. Correct: C**

EventBridge rules trigger immediately when RunInstances API calls occur (near real-time, typically under 30 seconds). Lambda can call DescribeVolumes to check encryption status of all volumes attached during launch. This event-driven approach meets the 2-minute requirement and works for all instance types. Security Hub provides centralized finding management.

---

**Q4. Correct: D**

Amazon Detective automatically ingests and correlates CloudTrail, VPC Flow Logs, and GuardDuty findings across all regions into a unified behavior graph. Detective visualizes relationships between entities (users, IP addresses, S3 buckets) and identifies anomalous patterns. Detective is specifically designed for security investigations and provides time-based analysis up to 1 year with automatic correlation - ideal for investigating suspicious access patterns.

---

**Q5. Correct: B**

GuardDuty threat lists allow you to upload custom lists of malicious IP addresses and domain names stored in S3. GuardDuty automatically monitors network traffic and generates findings when communication with threat list entries is detected. Updating the S3 file automatically updates the threat list without service reconfiguration. This is the lowest maintenance option for detection (not prevention).

---

**Q6. Correct: C**

GuardDuty analyzes DNS logs (when VPC Flow Logs are enabled) and automatically detects communication with known malicious domains using AWS threat intelligence. GuardDuty generates findings like Backdoor:EC2/C&CActivity.B!DNS when instances query domains associated with C2 infrastructure. This works automatically for all instances with minimal latency (typically minutes) and requires no custom code or maintenance of threat feeds.

---

**Q7. Correct: B**

EventBridge rules can filter CloudTrail events in real-time using content-based filtering on the sourceIPAddress field with pattern matching. This provides near-immediate detection (typically within seconds to 1-2 minutes), includes all relevant event details (principal, action, source IP) in the event payload, and directly integrates with SNS for notifications. The rule pattern can use anything-but matching to exclude corporate CIDR ranges.

---

**Q8. Correct: B**

The first priority in credential compromise is containment - immediately disable the compromised access keys to prevent further unauthorized actions. This stops the active threat before investigation and remediation. AWS best practice for incident response follows: contain, investigate, eradicate, recover.

---

**Q9. Correct: A**

EventBridge + Lambda + SNS is the AWS-native, serverless approach with lowest operational overhead. EventBridge natively integrates with GuardDuty findings as event sources, Lambda provides custom remediation logic (security group isolation, EBS snapshots via API), and SNS handles notifications. All fully managed services with no infrastructure to maintain.

---

**Q10. Correct: B**

This approach preserves running state (required for memory capture), provides network isolation through security groups, and maintains access via SSM Session Manager which uses AWS Systems Manager endpoints and doesn't require inbound network access. Memory must be captured from a running system; stopping the instance loses volatile memory contents.

---

**Q11. Correct: A**

AWS WAF rate-based rules provide immediate, cost-effective mitigation with granular control. Rate limiting at 100 requests per 5 minutes (20 req/min) per IP allows legitimate users while blocking flood traffic. WAF on ALB costs $5/month + $1 per million requests, far cheaper than Shield Advanced ($3,000/month). Deployment takes minutes.

---

**Q12. Correct: A**

The finding specifically identifies instance credential exfiltration - temporary credentials from EC2 instance metadata service used outside AWS. This is the signature of SSRF attack exploiting IMDSv1. IMDSv2 requires PUT request headers and hop limit prevents forwarding. GuardDuty specifically detects this pattern when instance credentials appear from external IPs.

---

**Q13. Correct: A**

When MFA Delete is enabled and the MFA device is lost, AWS Support is the only path to disable the feature. This requires root account verification through alternative methods. Once disabled, versioned objects (including delete markers) remain accessible via standard S3 versioning API calls. After forensics, security should re-enable MFA Delete with a new device. This is documented AWS procedure for MFA device loss scenarios.

---

**Q14. Correct: A**

ACM Private CA with TLS certificates provides end-to-end encryption between EC2 instances at the application level, which is required for PCI DSS compliance. VPC Flow Logs with EventBridge can detect unencrypted traffic patterns, and this solution encrypts data in transit between all tiers with verifiable monitoring.

---

**Q15. Correct: C**

SCPs provide centralized, preventive controls at the organization level that cannot be overridden by local account administrators. An SCP applied to the development OU will block EC2 instance launches with public IPs across all development accounts while leaving production accounts unaffected, meeting the requirement for differentiated control.

---

**Q16. Correct: B**

Systems Manager State Manager provides continuous enforcement of desired configurations (CIS benchmarks) with automatic drift remediation. Combined with Patch Manager for automated patching and Security Hub as a central compliance dashboard that aggregates findings from multiple services, this provides the most complete automated solution for both configuration compliance and patch management.

---

**Q17. Correct: A**

This option combines both the preventive control (VPC endpoints to ensure traffic stays private) and the detective control (VPC Flow Logs with pkt-dstaddr field to verify no traffic goes to public IPs). The custom format with pkt-dstaddr is crucial because it shows the actual destination IP address, allowing verification that AWS service traffic goes to private VPC endpoint IPs rather than public service endpoints. Athena queries can definitively prove compliance by showing zero traffic to public AWS IP ranges.

---

**Q18. Correct: B**

Transit Gateway with central inspection VPC architecture is the AWS-recommended pattern for shared egress security services. Transit Gateway maintains network isolation between customer VPCs (no direct VPC-to-VPC routing) while allowing centralized traffic inspection. Network Firewall in the inspection VPC with managed threat intelligence feeds provides scalable, centrally managed domain and IP filtering for all 100+ customer VPCs with minimal ongoing management.

---

**Q19. Correct: C**

Gateway VPC endpoint for S3 with an endpoint policy that explicitly denies all buckets except company buckets provides network-level enforcement that works even if IAM credentials are compromised. The route table prioritizes VPC endpoint traffic, forcing all S3 requests through the endpoint. S3 bucket policies with aws:SourceVpce ensure buckets only accept requests from the specific VPC endpoint, creating defense-in-depth.

---

**Q20. Correct: B**

Systems Manager Session Manager provides browser-based or CLI shell access to EC2 instances without requiring open inbound SSH ports or bastion hosts. Session Manager logs complete session activity (including commands executed) to S3 and CloudWatch Logs for audit purposes. IAM policies and session tags can restrict access based on federated identity from IAM Identity Center. The instances only need outbound HTTPS (443) to Systems Manager endpoints, which can be restricted by source IP in security groups. This solution requires no agent installation beyond the SSM Agent (pre-installed on AWS-provided AMIs) and no changes to existing instance configurations.

---

**Q21. Correct: A**

Amazon ECR image scanning (enhanced scanning powered by Inspector) automatically scans on push, provides severity ratings (LOW, MEDIUM, HIGH, CRITICAL), integrates natively with EventBridge for automated workflows, allows Lambda to programmatically block deployments via CodePipeline stage actions or deployment gates, and supports SNS for security team notifications. This is a fully automated, AWS-native solution that specifically addresses the requirement to block deployments based on vulnerability severity and notify the security team without manual intervention.

---

**Q22. Correct: B**

Permissions boundaries define the maximum permissions that an identity-based policy can grant to an IAM entity. When applied to users, they prevent those users from creating roles (or other entities) with permissions that exceed the boundary, directly addressing the requirement to prevent privilege escalation.

---

**Q23. Correct: B**

AdminUserGlobalSignOut immediately invalidates all tokens (access, refresh, and ID tokens) for a specific user, meeting the 5-minute requirement. This can be triggered automatically when an account is disabled, providing immediate token revocation without reducing token validity for all users.

---

**Q24. Correct: B**

S3 bucket policies with time-based conditions using aws:CurrentTime and DateLessThan automatically enforce access expiration without manual intervention. This eliminates the need for credential management and ensures access automatically terminates after the specified date.

---

**Q25. Correct: D**

KMS grants with encryption context conditions allow you to restrict kms:Decrypt permission so it only works when used through S3 service operations (via the encryption context). Application roles can read S3 objects without having direct decrypt permissions - KMS validates the operation came from S3 via encryption context.

---

**Q26. Correct: C**

CodeCommit pre-receive hooks run server-side before accepting pushed commits. They can validate GPG signatures and reject the entire push if any commit is unsigned, preventing unsigned commits from ever entering the repository. This is the only preventive control that blocks unsigned commits at push time.

---

**Q27. Correct: A**

When using AssumeRoleWithSAML, if the SAML assertion includes a SessionDuration attribute that exceeds the maximum session duration setting on the IAM role, AWS rejects the assume role request with this specific error. Since this only affects one region's account and all roles are supposed to be identical, the Asia Pacific account's role likely has a lower maximum session duration setting.

---

**Q28. Correct: B**

Amazon Cognito Identity Pools is specifically designed for mobile and web applications with large user bases. It automatically provides temporary AWS credentials through STS, integrates with external identity providers (Google, Facebook, etc.), supports unauthenticated access if needed, and allows role-based access control. Session duration can be configured from 15 minutes to 12 hours (1 hour meets the requirement). This eliminates credential management overhead and provides automatic credential rotation.

---

**Q29. Correct: C**

This solution uses permissions boundaries correctly to prevent privilege escalation. By requiring iam:PermissionsBoundary in the CreateRole condition, developers MUST attach the same permissions boundary (or a stricter one) to any new roles they create. The permissions boundary on the developers' roles defines the maximum permissions they can grant, and by forcing them to use that same boundary on created roles, you ensure created roles cannot exceed the developers' own permissions. This is the correct implementation of delegated administration with boundaries.

---

**Q30. Correct: A**

This SCP enforces both requirements at role creation/modification time by denying IAM operations that would create or update roles unless the trust policy includes the required ExternalId condition AND restricts access to principals within the organization (via aws:PrincipalOrgID). This preventive control ensures compliance before non-compliant roles are created, which is more effective than detective or reactive approaches.

---

**Q31. Correct: B**

SSE-KMS with Customer Managed Keys provides full control over encryption keys including the ability to disable or delete keys immediately to revoke access. CloudTrail automatically logs all KMS key usage including encrypt/decrypt operations, providing detailed audit logs. This meets all requirements for control and auditing.

---

**Q32. Correct: B**

Enabling EBS encryption by default at the Region level ensures that all new EBS volumes are automatically encrypted without requiring any additional configuration. This is the most effective preventive control because it works at the infrastructure level and requires no user action, eliminating human error. Once enabled, it applies to all volumes created by any user or service.

---

**Q33. Correct: A**

Creating an encrypted snapshot copy is the standard AWS-recommended approach for encrypting existing RDS instances. You can specify a KMS Customer Managed Key with automatic 90-day rotation enabled, snapshot copy is fast, and restore creates a new encrypted instance. Downtime is limited to the time needed to switch the application endpoint. KMS automatically logs all key usage in CloudTrail.

---

**Q34. Correct: C**

CloudFront with signed URLs/cookies and OAC provides the most secure solution: OAC ensures the S3 bucket is never publicly accessible and only CloudFront can access it, signed URLs/cookies provide time-limited access control, CloudFront caching reduces S3 costs and improves performance for large video files, and you can implement additional security controls at the CloudFront level (geo-restrictions, WAF, etc.).

---

**Q35. Correct: B**

Client-side encryption with AWS Encryption SDK ensures data is encrypted before it reaches DynamoDB, providing defense-in-depth. Using customer-provided keys (BYOK) stored in Secrets Manager with separate secrets per customer enables unique keys per tenant. Even if application access controls fail or DynamoDB/KMS is compromised, data remains encrypted with keys the attacker cannot access. This provides cryptographic isolation between tenants.

---

**Q36. Correct: C**

Updating the KMS key policy to grant kms:Decrypt to external accounts and configuring S3 bucket policy for cross-account GetObject allows collaborators to copy encrypted objects directly to their accounts. S3 automatically re-encrypts with the destination account's key during copy. This is cost-effective (collaborators pay for their copy, no data egress within same region if applicable) and secure (data remains encrypted in transit and at rest).

---

**Q37. Correct: D**

CloudHSM provides full customer control over encryption keys with the ability to immediately revoke access by deleting keys from the HSM. It provides detailed audit logs, meets compliance requirements for key ownership and separation of duties (HSM administrators vs application users), and supports client-side encryption for complete control over the encryption process before data reaches S3.

---

**Q38. Correct: C**

Secrets Manager provides native automatic rotation for RDS credentials with built-in Lambda rotation functions for PostgreSQL. It logs all secret access via CloudTrail, requires minimal application changes (just API calls to retrieve secrets), and handles the complete credential lifecycle including rotation coordination with the database.

---

**Q39. Correct: D**

AWS Audit Manager is purpose-built for continuous compliance auditing with pre-built frameworks including PCI DSS. It automatically collects evidence from AWS services, generates audit-ready reports, maps controls to compliance requirements, and reduces manual effort in compliance demonstration. It directly addresses all requirements: automated checking, evidence reports, and continuous visibility with minimal overhead.

---

**Q40. Correct: B**

Service Control Policies (SCPs) provide preventive controls at the organization level that block actions before they occur. SCPs can deny the ability to disable versioning and restrict EC2 launches to approved AMIs only. SCPs automatically apply to all accounts in the organization including future accounts, enforcing compliance before violations occur rather than detecting and remediating after.

---

**Q41. Correct: C**

Permission boundaries combined with TBAC provides the strongest separation of duties within a single account. Permission boundaries set the maximum permissions any user in a role can have (preventing privilege escalation), while TBAC allows granular resource-level access control based on ownership tags. This creates a defense-in-depth approach where users can only access resources tagged for their function AND cannot exceed their boundary limits. This is more practical than multi-account for most organizations while still maintaining strict separation.

---

**Q42. Correct: B**

IAM permission boundaries are specifically designed to prevent privilege escalation by setting the maximum permissions that can be granted to an identity. By requiring all created roles to have a permission boundary attached, developers can create roles but those roles cannot exceed the permissions defined in the boundary, even if AdministratorAccess is attached. This is preventive and allows legitimate role creation.

---

**Q43. Correct: B**

An organization trail automatically logs all accounts in the organization. S3 Object Lock in compliance mode provides true immutability where even the root user cannot delete objects for the specified retention period. MFA delete adds additional protection against deletion. CloudTrail log file validation enables cryptographic verification of log integrity. This combination meets all requirements: immutability, 7-year retention, protection from admin deletion, and integrity verification.

---

**Q44. Correct: A**

Security Hub delegated administrator enables automatic enablement of Security Hub across all organization accounts. Security Hub natively aggregates findings from GuardDuty and Inspector. Cross-region aggregation feature consolidates all findings from multiple regions to a single region automatically. This is native AWS functionality requiring minimal configuration and automatically includes new accounts/regions added to the organization.

---

