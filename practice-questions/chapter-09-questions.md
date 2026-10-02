## Chapter 9: Security Foundations: Governance, Compliance, and Multi-Account Architecture — Practice Questions

**Q1.** A financial services company is implementing a multi-account AWS environment and needs to prevent any account from disabling CloudTrail logging or deleting S3 buckets containing audit logs. The solution must apply to all current and future accounts across the organization. Which approach BEST meets these requirements?

A) Create an IAM policy denying cloudtrail:StopLogging and s3:DeleteBucket actions and attach it to all IAM roles in each account
B) Implement a Service Control Policy (SCP) with Deny statements for cloudtrail:StopLogging and s3:DeleteBucket actions, attached to the root of the organization
C) Use AWS Config rules to detect CloudTrail停用 and S3 bucket deletion, then use Systems Manager Automation to remediate violations
D) Enable AWS Control Tower Detective guardrails to monitor CloudTrail status and S3 bucket configurations across all accounts

<details>
<summary>Answer</summary>

**Correct: B**

SCPs provide preventive controls that apply organization-wide to all accounts and principals, including the root user. They are inherited down the OU hierarchy and cannot be overridden at the account level, making them ideal for hard boundaries on critical security controls.

**Why the others are wrong:**
- IAM policies can be modified or deleted by users with sufficient permissions within each account, and don't prevent root user actions. This approach also doesn't automatically apply to new accounts.
- Config rules are detective controls that identify violations after they occur, not preventive controls. They can trigger remediation, but there's a window where the violation exists, unacceptable for critical audit requirements.
- Control Tower detective guardrails monitor and detect violations but don't prevent them. They are detective controls, not preventive. The requirement explicitly needs prevention.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A company uses AWS Organizations with the following OU structure: Root > Security OU > Logging Account, Root > Workloads OU > Production OU > App Accounts. An SCP attached to the Workloads OU denies ec2:RunInstances for instance types larger than m5.xlarge. An SCP attached to the Production OU allows ec2:RunInstances for all instance types. What is the effective permission for launching a c5.4xlarge instance in an App Account?

A) Allowed, because the Production OU SCP explicitly allows all instance types and is more specific than the Workloads OU SCP
B) Denied, because SCPs use intersection logic and the Workloads OU Deny applies to all child OUs including Production
C) Allowed, because child OU SCPs override parent OU SCPs in the inheritance hierarchy
D) Denied, unless the App Account has an explicit Allow SCP attached directly to it

<details>
<summary>Answer</summary>

**Correct: B**

SCPs work through intersection - all applicable SCPs must allow an action for it to be permitted. Deny statements in any parent OU apply to all child OUs and accounts. A Deny always wins over an Allow. The Workloads OU Deny restricts instance sizes for Production OU and all its accounts.

**Why the others are wrong:**
- SCPs don't work with specificity rules - they use intersection. A Deny in any parent OU cannot be overridden by an Allow in a child OU. There is no concept of 'more specific' SCP.
- Child OU SCPs do not override parent SCPs. All SCPs in the hierarchy from root to account are evaluated together using intersection logic with Deny always winning.
- SCPs at all levels (root, parent OUs, child OUs, and account level) are intersected together. An explicit Allow at the account level cannot override a Deny from a parent OU.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A security team needs to ensure all EC2 instances across 200 AWS accounts are encrypted with customer-managed KMS keys and have approved security groups. Instances must be automatically flagged within 15 minutes of non-compliance. Which solution provides the MOST operationally efficient approach?

A) Deploy AWS Config with encrypted-volumes and approved-security-groups managed rules in each account, aggregate results to a central account using Config Aggregator
B) Create a conformance pack with encrypted-volumes and vpc-sg-open-only-to-authorized-ports rules, deploy via CloudFormation StackSets to all accounts
C) Use Control Tower Detective guardrails for EBS encryption and security group monitoring across all enrolled accounts in the landing zone
D) Deploy a Config conformance pack from the organization management account using AWS Config organization rules feature to all member accounts

<details>
<summary>Answer</summary>

**Correct: D**

AWS Config organization rules (including conformance packs) can be deployed centrally from the management account to all member accounts in one operation. This is the most operationally efficient - single deployment point, automatic application to new accounts, and centralized management. Config evaluates rules typically within 10-15 minutes.

**Why the others are wrong:**
- This requires deploying Config rules individually to each account, then setting up aggregation. More operational overhead than organization-wide deployment. Requires 200 separate deployments and ongoing management.
- StackSets require manual management of stack instances across accounts and don't provide the centralized compliance dashboard that organization Config rules provide. Also requires deployment orchestration.
- Control Tower guardrails are limited to the specific controls AWS provides. The requirement for 'approved security groups' likely needs custom rules that Control Tower detective guardrails don't support out of box.

</details>

---

**Q4.** A development team pushes Python packages to CodeArtifact. The security team requires that all packages must originate from PyPI through CodeArtifact's upstream repository, and developers must not be able to publish arbitrary packages directly to CodeArtifact. Which configuration enforces this requirement?

A) Configure CodeArtifact repository with PyPI as external connection, grant developers codeartifact:ReadFromRepository but deny codeartifact:PublishPackageVersion
B) Enable CodeArtifact package origin controls set to ALLOW_UPSTREAM, configure PyPI external connection, remove direct publish permissions from developer roles
C) Create two CodeArtifact repositories: one connected to PyPI (upstream), one for team packages (downstream), grant developers read-only access to both
D) Use repository policies to require package signatures from PyPI, configure CodeArtifact to reject unsigned packages from any other source

<details>
<summary>Answer</summary>

**Correct: B**

Package origin controls with ALLOW_UPSTREAM setting ensures packages can only enter the repository through configured upstream sources (PyPI external connection). Combined with removing PublishPackageVersion permissions, this prevents developers from publishing arbitrary packages while allowing consumption of vetted PyPI packages.

**Why the others are wrong:**
- Denying PublishPackageVersion is part of the solution, but without package origin controls, this doesn't enforce that packages MUST come from PyPI. A user with publish permissions could still add packages from other sources.
- This creates operational complexity but doesn't enforce the origin requirement. Developers could still have publish access to the downstream repository, allowing arbitrary package publication.
- PyPI packages are not signed with a signature format that CodeArtifact can verify in repository policies. Package origin controls, not signature verification, is the mechanism to enforce upstream sources for public repositories.

</details>

---

**Q5.** A company requires that all Lambda functions be signed by authorized developers before deployment, and container images in ECR must have signatures verified before use in production. The solution must integrate with existing CI/CD pipelines and provide cryptographic proof of code origin. What is the MOST appropriate solution?

A) Use AWS Signer to create code signing configurations for Lambda functions with attached signing profiles, and use AWS Signer signing jobs for container images with ECR signature verification policies
B) Enable ECR image scanning with EventBridge rules to prevent unsigned deployments, and use AWS Certificate Manager to issue certificates for Lambda code signing
C) Implement AWS KMS to encrypt Lambda deployment packages and ECR images at rest, then configure IAM policies to restrict decryption to authorized developers only
D) Use AWS Certificate Manager Private CA to issue code signing certificates for both Lambda and ECR, then configure resource-based policies to enforce signature verification

<details>
<summary>Answer</summary>

**Correct: A**

AWS Signer is the AWS-native service specifically designed for code signing. For Lambda, you create signing configurations and attach signing profiles to functions to enforce signature verification. For container images, AWS Signer supports signing jobs that create signatures, and ECR can be configured with signature verification policies to ensure only signed images are deployed. This provides cryptographic proof of code origin and integrates seamlessly with CI/CD pipelines through APIs and CLI commands.

**Why the others are wrong:**
- ECR image scanning detects vulnerabilities (CVEs) but does not provide code signing or cryptographic proof of origin. AWS Certificate Manager is for SSL/TLS certificates and identity certificates, not code signing certificates. This conflates vulnerability scanning with signature verification.
- AWS KMS provides encryption at rest, not code signing or signature verification. Encrypting deployment packages and images proves confidentiality, not authenticity or origin. IAM policies on KMS keys cannot replace cryptographic signatures that prove who created the code.
- AWS Certificate Manager Private CA can issue certificates but is not integrated with Lambda code signing configurations or ECR signature verification. AWS Signer is purpose-built for code signing workflows and provides the native integration needed. ACM Private CA would require custom implementation of the entire signing and verification workflow.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** An organization uses ECR to store container images for Kubernetes workloads. The security team needs to identify container images with OS-level vulnerabilities (glibc, openssl) and language package vulnerabilities (npm, pip packages) within minutes of image push. Which ECR feature provides the MOST comprehensive vulnerability detection for this requirement?

A) ECR basic scanning with Clair engine integration for OS package CVE detection
B) ECR enhanced scanning powered by Amazon Inspector for OS and language package vulnerability detection
C) ECR lifecycle policies to automatically delete images older than 30 days combined with manual vulnerability assessments
D) Integration with third-party container scanning tools through ECR API and EventBridge events

<details>
<summary>Answer</summary>

**Correct: B**

ECR enhanced scanning uses Amazon Inspector's scanning engine to detect vulnerabilities in both OS packages AND language-specific packages (npm, pip, jar, etc.). It provides continuous scanning, automated rescanning when new CVEs are published, and detects vulnerabilities within minutes of image push. This is the most comprehensive native option.

**Why the others are wrong:**
- ECR basic scanning only detects OS-level vulnerabilities and doesn't scan language package dependencies (npm, pip). It cannot identify vulnerabilities in application dependencies, which are critical for modern containerized applications.
- Lifecycle policies address image retention and storage costs but provide no vulnerability detection capabilities. Manual assessments don't meet the 'within minutes' requirement and are not scalable.
- While third-party tools can provide comprehensive scanning, this requires additional integration work, cost, and operational overhead. ECR enhanced scanning provides native integration with results in Security Hub and doesn't require external services.

</details>

---

**Q7.** A regulated healthcare company must prove compliance with CIS AWS Foundations Benchmark and PCI-DSS requirements across 50 AWS accounts. Auditors require evidence that controls are continuously monitored and configuration drift is detected within 1 hour. Which approach provides the MOST efficient compliance evidence collection?

A) Deploy AWS Config conformance packs for CIS and PCI-DSS from the organization management account, export compliance data to S3 for audit reporting
B) Use AWS Audit Manager to continuously collect evidence using CIS and PCI-DSS frameworks, generate audit reports from the centralized evidence locker
C) Implement custom Lambda functions to query AWS Config for each compliance requirement, aggregate results in DynamoDB, generate reports with QuickSight
D) Enable AWS Security Hub CIS and PCI-DSS security standards, use Security Hub findings for compliance reporting

<details>
<summary>Answer</summary>

**Correct: B**

AWS Audit Manager is purpose-built for compliance evidence collection and audit readiness. It automatically collects evidence using prebuilt CIS and PCI-DSS frameworks, maintains an evidence locker, maps evidence to controls, and generates audit-ready reports. This is the most efficient for auditor evidence requirements.

**Why the others are wrong:**
- Config conformance packs detect compliance violations but require manual correlation and export for audit evidence. They don't provide the automated evidence collection and audit report generation that Audit Manager provides.
- This custom solution requires significant development and maintenance effort. It duplicates functionality available in Audit Manager and doesn't provide pre-mapped compliance frameworks or audit-ready evidence formats.
- Security Hub aggregates findings but doesn't provide the automated evidence collection, control mapping, or audit report generation that compliance audits require. Security Hub is for security posture visibility, not compliance evidence management.

</details>

---

**Q8.** A software development organization needs to generate and maintain Software Bill of Materials (SBOMs) for all containerized applications to track dependencies and license compliance. Container images are stored in ECR and must include SBOM data for vulnerability correlation and supply chain risk assessment. What is the recommended approach to integrate SBOM generation into the container build pipeline?

A) Enable ECR enhanced scanning which automatically generates SBOMs in SPDX format for all scanned images and stores them with image metadata
B) Use AWS Signer to sign container images and embed SBOM data in the signature metadata during the signing process
C) Integrate open-source SBOM generation tools (Syft, SBOM-tool) in CI/CD pipeline, store SBOM artifacts in S3, and tag ECR images with SBOM S3 locations
D) Configure Amazon Inspector to generate SBOMs during vulnerability scanning and publish them to Security Hub for centralized tracking

<details>
<summary>Answer</summary>

**Correct: C**

As of 2026, ECR and Inspector do not natively generate or store SBOMs. The recommended approach is to integrate SBOM generation tools into the build pipeline, generate SBOMs in standard formats (SPDX, CycloneDX), store them with image artifacts, and maintain references between images and their SBOMs. This provides supply chain visibility and license tracking.

**Why the others are wrong:**
- ECR enhanced scanning detects vulnerabilities but does not generate or store SBOMs in SPDX or CycloneDX format. Enhanced scanning provides vulnerability findings, not comprehensive dependency manifests.
- AWS Signer provides code signing and signature verification but does not generate SBOMs or provide SBOM embedding capabilities. Signing and SBOM generation are separate security controls with different purposes.
- Amazon Inspector scans for vulnerabilities and identifies packages, but does not generate formal SBOM artifacts in standard formats. Inspector provides vulnerability findings, not supply chain dependency documentation.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

