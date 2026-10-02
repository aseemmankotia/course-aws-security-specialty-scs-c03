## Chapter 8: Data Protection: Encryption at Rest, in Transit, and Data Classification — Practice Questions

**Q1.** A company stores RDS PostgreSQL database credentials in AWS Secrets Manager with automatic rotation enabled every 30 days. After rotation, the application begins failing with authentication errors for approximately 2 minutes before recovering. The security team requires maintaining automatic rotation. What is the MOST operationally efficient solution?

A) Reduce rotation frequency to 90 days to minimize disruption
B) Enable multi-user rotation strategy and update application to use both AWSCURRENT and AWSPREVIOUS secret versions
C) Configure Lambda rotation function to wait 5 minutes before updating database password
D) Store credentials in Systems Manager Parameter Store SecureString instead and rotate manually

<details>
<summary>Answer</summary>

**Correct: B**

Multi-user rotation creates a new user before disabling the old one, allowing both AWSCURRENT and AWSPREVIOUS versions to work during rotation. Application should attempt AWSCURRENT first, fall back to AWSPREVIOUS. This eliminates the authentication window without changing rotation frequency.

**Why the others are wrong:**
- Reducing frequency doesn't eliminate the problem, just makes it less frequent. Security requirement is 30 days.
- Adding wait time extends the vulnerable period and doesn't solve the authentication gap. Rotation should complete quickly.
- Parameter Store doesn't support automatic rotation. Secrets Manager is correct service for automated credential rotation.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An application running on EC2 instances needs to retrieve database connection strings and API keys. The values change monthly and must be encrypted at rest. The solution should minimize cost while meeting encryption requirements. Which approach should be used?

A) Store values in Secrets Manager with automatic rotation enabled
B) Store values in Systems Manager Parameter Store as SecureString parameters using default aws/ssm KMS key
C) Store values in S3 bucket encrypted with SSE-KMS and grant EC2 IAM role s3:GetObject permission
D) Store values in DynamoDB table with encryption at rest enabled using AWS owned keys

<details>
<summary>Answer</summary>

**Correct: B**

Parameter Store SecureString with default aws/ssm KMS key provides encryption at rest at no cost (no Secrets Manager charges, no KMS key charges for AWS managed keys). Perfect for configuration data that doesn't need automatic rotation. Standard parameters are free, advanced parameters cost $0.05 per parameter per month.

**Why the others are wrong:**
- Secrets Manager costs $0.40 per secret per month plus $0.05 per 10,000 API calls. Automatic rotation isn't needed here - values change manually monthly. Unnecessarily expensive.
- S3 is not designed for small configuration values. Additional complexity with object retrieval, versioning management. Parameter Store is purpose-built for this.
- DynamoDB is database service, not configuration management. More complex to query single values, IAM permissions more complex, AWS owned keys don't allow key policy control.

</details>

---

**Q3.** A company needs TLS certificates for 50 internal microservices running in EKS clusters across multiple AWS accounts. Services communicate via internal Application Load Balancers. Certificates must be issued from a company-controlled certificate authority and rotated every 90 days. What is the MOST secure and cost-effective solution?

A) Create ACM Private CA in shared services account, issue certificates via ACM, attach to ALBs, enable auto-renewal
B) Use ACM to request public certificates from Amazon's CA for each service using DNS validation
C) Create ACM Private CA, export private key, use external automation to issue certificates and upload to ACM
D) Deploy OpenSSL-based CA on EC2, manually generate certificates, import to ACM in each account

<details>
<summary>Answer</summary>

**Correct: A**

ACM Private CA provides company-controlled root CA for internal PKI. ACM-issued certificates from Private CA auto-renew before expiration. Single Private CA ($400/month) can issue unlimited certificates. ACM integration with ALB is automatic. Cross-account access via RAM.

**Why the others are wrong:**
- Public ACM certificates are only for public domains validated via DNS/email/HTTP. Cannot use for internal microservices. Amazon's CA won't issue for private domains.
- Exporting private key from Private CA violates security best practice and isn't necessary. ACM handles certificate lifecycle without key export.
- Self-managed CA on EC2 requires operational overhead (patching, HA, backup), manual rotation processes, security risk of exposed CA private key. ACM Private CA is managed service eliminating these issues.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A security audit reveals that an S3 bucket containing financial records has sensitive data classification inconsistencies. The bucket has 5TB of PDFs and Excel files with potential credit card numbers and taxpayer IDs. The security team needs to identify which objects contain sensitive data within 48 hours. What should be done?

A) Create Macie sensitive data discovery job with managed data identifiers for financial information, specify S3 bucket as scope
B) Enable S3 Intelligent-Tiering with Archive Access tier to automatically classify objects based on access patterns
C) Create Lambda function to download each object, scan content with regex patterns for PII, tag objects accordingly
D) Enable GuardDuty S3 Protection to monitor bucket for sensitive data exfiltration attempts

<details>
<summary>Answer</summary>

**Correct: A**

Macie is purpose-built for sensitive data discovery in S3. Managed data identifiers detect credit card numbers (PCI DSS compliance), taxpayer IDs, and other financial PII. Discovery job scans specified buckets/prefixes. Provides findings with severity ratings, object locations, sensitive data types found. Completes within hours for 5TB.

**Why the others are wrong:**
- Intelligent-Tiering optimizes storage costs based on access patterns, doesn't classify or scan data content. No PII detection capability.
- Lambda approach requires downloading all 5TB (expensive egress, slow, timeout issues). No built-in PII detection patterns. Reinventing Macie functionality. Scanning 5TB with Lambda would take days and cost significantly more.
- GuardDuty detects threats like unusual API calls or credential compromise. S3 Protection focuses on access patterns and exfiltration, not content scanning for PII. Doesn't identify what data exists in objects.

</details>

---

**Q5.** An organization uses custom employee ID format (3 letters followed by 5 digits, e.g., ABC12345) in documents stored in S3. Macie must detect these employee IDs in addition to standard PII. The pattern appears in both structured (CSV) and unstructured (PDF, DOCX) files. Which solution meets this requirement?

A) Create custom data identifier with regular expression [A-Z]{3}[0-9]{5}, keywords 'employee', 'staff', 'EMP', proximity 50 characters
B) Configure S3 Object Lambda to scan objects in real-time and tag those matching employee ID pattern
C) Enable CloudWatch Logs Insights to parse S3 access logs and detect employee ID patterns in object keys
D) Create EventBridge rule to trigger Lambda function that validates employee ID format in S3 object metadata

<details>
<summary>Answer</summary>

**Correct: A**

Macie custom data identifiers support regex patterns for proprietary PII formats. Keywords and proximity increase detection accuracy (pattern must appear near keyword). Works with both structured and unstructured content. Maximum proximity is 300 characters. Once created, use in discovery jobs alongside managed identifiers.

**Why the others are wrong:**
- S3 Object Lambda transforms objects during retrieval, doesn't perform content scanning or create findings. Not integrated with Macie. Would need to build entire scanning solution.
- Access logs show who accessed what objects, not object content. Employee IDs are inside files, not in object keys or logs. CloudWatch Logs Insights queries logs, doesn't scan S3 object content.
- Object metadata doesn't contain file content. EventBridge triggers on API events, not content patterns. Would need to download and scan every object - inefficient and doesn't integrate with Macie findings.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A company must encrypt all S3 objects with keys that can be rotated on-demand and where AWS cannot decrypt data. The solution must support automatic server-side encryption for all new objects and allow immediate key rotation without re-encrypting existing objects. Which encryption method meets these requirements?

A) SSE-KMS with customer managed CMK, enable automatic key rotation annually
B) SSE-KMS with customer managed CMK, create new CMK version for rotation, update bucket default encryption
C) SSE-C with customer provided AES-256 keys stored in on-premises HSM
D) Client-side encryption using AWS Encryption SDK with customer managed CMK in KMS

<details>
<summary>Answer</summary>

**Correct: B**

SSE-KMS with customer managed CMK meets AWS cannot decrypt (you control key policy). Creating new CMK and updating bucket default encryption provides on-demand rotation. Existing objects stay encrypted with old CMK (still accessible via key policy), new objects use new CMK. No re-encryption needed. AWS doesn't have access without your key policy permission.

**Why the others are wrong:**
- Automatic rotation is annual only, not on-demand. AWS KMS automatic rotation is every 365 days and cannot be triggered manually. Also, rotation rotates key material but uses same CMK ID - doesn't allow immediate complete key change.
- SSE-C requires client to provide encryption key with every PUT/GET request. No automatic encryption for new objects. Key management entirely on customer (no KMS). Operationally complex, not automatic.
- Client-side encryption meets 'AWS cannot decrypt' but doesn't support automatic server-side encryption. Application must encrypt before upload. Not transparent to users/applications. Doesn't meet 'automatic for all new objects' requirement.

</details>

---

**Q7.** A healthcare application stores patient records in S3 with SSE-KMS encryption using a customer managed CMK. Compliance requires that AWS administrators cannot access encrypted data. The security team discovers that IAM users with s3:GetObject permission can read encrypted objects. What is the MOST restrictive solution?

A) Remove s3:GetObject permission from all IAM users and require use of pre-signed URLs generated by application
B) Modify KMS key policy to explicitly deny kms:Decrypt for all principals except specific application IAM role
C) Enable S3 Block Public Access and remove all bucket policies granting read access
D) Change encryption to SSE-C so only application with customer-provided keys can decrypt

<details>
<summary>Answer</summary>

**Correct: B**

With SSE-KMS, accessing object requires both S3 permission (s3:GetObject) AND KMS permission (kms:Decrypt). KMS key policy explicitly denying kms:Decrypt for all principals except application role prevents AWS administrators from decrypting even with s3:GetObject. Deny statements override allow statements. Most restrictive approach.

**Why the others are wrong:**
- Pre-signed URLs don't solve the core issue - they still require underlying permissions. If user has permission to generate pre-signed URL, they can access object. Doesn't prevent AWS administrator access to KMS key.
- Block Public Access prevents anonymous access. Bucket policies are separate from IAM user permissions. Doesn't address KMS decryption permissions. AWS administrators could still access if they have KMS permissions.
- SSE-C requires providing key with every request - operationally complex. Doesn't use KMS for centralized key management and audit logging. Doesn't meet requirement more effectively than KMS key policy restriction.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** An application uploads files containing customer personal information to S3, where they are processed by a Lambda function and stored in DynamoDB. The Lambda function runs in a private subnet without internet access and uses VPC endpoints for S3 and DynamoDB. To ensure encryption in transit for all data flows in this architecture, what is the MOST complete solution?

A) Enable S3 bucket policy requiring aws:SecureTransport condition AND attach VPC endpoint policies denying non-TLS requests to both S3 and DynamoDB endpoints
B) Configure the application to upload to S3 using client-side encryption with AWS Encryption SDK AND enable DynamoDB encryption at rest using AWS managed keys
C) Configure the Lambda function to use HTTPS endpoints when calling S3 and DynamoDB APIs AND enable default encryption on the S3 bucket using SSE-S3
D) Attach an IAM policy to the Lambda execution role requiring SSL connections AND configure S3 Transfer Acceleration with TLS 1.2 minimum

<details>
<summary>Answer</summary>

**Correct: A**

This solution provides comprehensive encryption in transit enforcement. The S3 bucket policy with aws:SecureTransport condition blocks any non-HTTPS uploads from the application. VPC endpoint policies on both S3 and DynamoDB endpoints ensure that all API calls from the Lambda function (which routes through these endpoints) must use TLS. Together, these controls enforce encryption in transit for the upload flow (application to S3) and the processing flows (Lambda to S3 and DynamoDB). VPC endpoints inherently use HTTPS, but the endpoint policies provide an additional enforcement layer and defense-in-depth.

**Why the others are wrong:**
- This addresses encryption at rest (client-side encryption and DynamoDB encryption at rest), not encryption in transit. Client-side encryption protects data before transmission but doesn't enforce TLS for the actual network transfer. The question specifically asks for encryption in transit enforcement.
- While configuring Lambda to use HTTPS endpoints is good practice, VPC endpoints already use HTTPS by default, making this partially redundant. More importantly, this option does nothing to enforce encryption in transit for the application's upload to S3. S3 bucket encryption (SSE-S3) is encryption at rest, not in transit.
- IAM policies cannot enforce SSL/TLS at the transport layer - they control authorization, not encryption protocols. S3 Transfer Acceleration is unnecessary for a VPC endpoint architecture and doesn't address encryption in transit for Lambda-to-DynamoDB communication. This solution doesn't comprehensively enforce encryption in transit across all data flows.

</details>

---

