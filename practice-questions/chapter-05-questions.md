## Chapter 5: IAM Advanced: Federation, ABAC, and Cross-Account Access — Practice Questions

**Q1.** A company uses IAM Identity Center to manage access across 50 AWS accounts in their organization. They need developers to access EC2 instances in development accounts but only S3 buckets in production accounts. The security team wants centralized policy management without creating IAM roles in each account. Which solution BEST meets these requirements?

A) Create IAM roles in each account with trust relationships to IAM Identity Center, then assign users to roles via the Identity Center console
B) Create two permission sets in IAM Identity Center with appropriate policies, assign the development permission set to the development account group and production permission set to the production account group
C) Use AWS Organizations SCPs to restrict developer access patterns, then create individual IAM users in each account
D) Configure SAML federation directly between the identity provider and each AWS account with custom assertion mapping

<details>
<summary>Answer</summary>

**Correct: B**

IAM Identity Center permission sets are deployed automatically to assigned accounts as IAM roles. This provides centralized management - you define the permission set once and assign it to specific principals for specific accounts. The Identity Center service handles role creation and trust policy configuration automatically.

**Why the others are wrong:**
- This defeats the purpose of IAM Identity Center - you'd be manually creating roles when Identity Center automates this via permission sets. The centralized management requirement is not met.
- SCPs are guardrails that restrict maximum permissions but don't grant access. Individual IAM users violate the centralized management requirement and don't scale to 50 accounts.
- Direct SAML federation to each account requires configuring 50 separate SAML applications and trust relationships. IAM Identity Center already handles SAML federation centrally and propagates access automatically.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An application running in GitHub Actions needs to deploy to AWS without storing long-term credentials. The security team requires that only workflows from the main branch of the specific repository can assume the AWS role. Which configuration correctly implements this?

A) Create an IAM user for GitHub Actions with access keys, restrict the user with an IP address condition matching GitHub's webhook IPs
B) Create an OIDC identity provider for token.actions.githubusercontent.com, create an IAM role with trust policy condition: "StringEquals": {"token.actions.githubusercontent.com:sub": "repo:orgname/reponame:ref:refs/heads/main"}
C) Configure SAML federation between GitHub and AWS, create an IAM role that trusts GitHub's SAML assertions with attribute-based filtering
D) Use IAM Roles Anywhere with X.509 certificates issued to GitHub Actions runners

<details>
<summary>Answer</summary>

**Correct: B**

GitHub Actions uses OIDC, not SAML. The OIDC provider for GitHub Actions is token.actions.githubusercontent.com. The subject (sub) claim format repo:orgname/reponame:ref:refs/heads/main ensures only workflows from the specified repository and branch can assume the role. This is the standard pattern for GitHub Actions federation.

**Why the others are wrong:**
- Storing access keys violates the requirement for no long-term credentials. GitHub Actions runs on shared runners with dynamic IPs, making IP-based restrictions ineffective and unreliable.
- GitHub Actions does not support SAML federation - it uses OIDC. This is a common confusion point but GitHub specifically implements OIDC, not SAML.
- IAM Roles Anywhere is designed for on-premises servers and workloads that can use X.509 certificates. GitHub Actions uses OIDC tokens, not certificates. This is the wrong federation mechanism.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A company has 500 on-premises Linux servers that need to call AWS APIs to upload logs to S3 and send metrics to CloudWatch. The security policy prohibits storing AWS credentials on the servers. Which solution provides the MOST secure authentication?

A) Install the AWS CLI on each server and configure it with IAM user access keys stored in encrypted files
B) Use IAM Roles Anywhere with a private certificate authority, issue X.509 certificates to each server, create an IAM role that trusts the CA
C) Set up a VPN to AWS and use EC2 instance profiles attached to a bastion host that the servers proxy through
D) Configure OIDC federation with the company's identity provider and have servers request OIDC tokens before making AWS API calls

<details>
<summary>Answer</summary>

**Correct: B**

IAM Roles Anywhere is specifically designed for this use case - providing on-premises servers with temporary AWS credentials using X.509 certificates. The server presents its certificate to the IAM Roles Anywhere endpoint, which validates it against the trusted CA and issues temporary credentials via STS. No long-term credentials are stored.

**Why the others are wrong:**
- Access keys are long-term credentials that violate the security policy. Even encrypted, they require secure distribution, rotation, and storage - exactly what the requirement prohibits.
- This adds significant complexity, requires all API calls to route through a bastion host (single point of failure), and the bastion itself would need credentials. It doesn't scale to 500 servers making independent API calls.
- On-premises Linux servers don't have built-in OIDC token generation capabilities. OIDC federation is designed for web applications and services with OIDC providers, not individual servers.

</details>

---

**Q4.** A SaaS vendor needs access to customer AWS accounts to provide managed services. The security team is concerned about the vendor using confused deputy attacks to access other customers' resources. Which trust policy configuration BEST mitigates this risk?

A) {"Effect":"Allow","Principal":{"AWS":"arn:aws:iam::vendor-account:root"},"Action":"sts:AssumeRole","Condition":{"StringEquals":{"sts:ExternalId":"unique-customer-id-12345"}}}
B) {"Effect":"Allow","Principal":{"AWS":"arn:aws:iam::vendor-account:user/service"},"Action":"sts:AssumeRole","Condition":{"IpAddress":{"aws:SourceIp":"vendor-ip-range"}}}
C) {"Effect":"Allow","Principal":{"AWS":"arn:aws:iam::vendor-account:root"},"Action":"sts:AssumeRole","Condition":{"StringLike":{"aws:userid":"vendor-*"}}}
D) {"Effect":"Allow","Principal":{"Service":"vendor-service.amazonaws.com"},"Action":"sts:AssumeRole"}

<details>
<summary>Answer</summary>

**Correct: A**

External ID is the AWS-recommended solution for preventing confused deputy attacks in third-party cross-account access scenarios. Each customer provides a unique External ID to the vendor. When the vendor assumes the role, they must provide the correct External ID. This prevents the vendor from being tricked into using one customer's credentials to access another customer's resources.

**Why the others are wrong:**
- IP-based conditions are ineffective for cloud services that use dynamic infrastructure. The vendor could still be confused into accessing wrong customer resources if they have both External IDs. This doesn't prevent confused deputy attacks.
- Using aws:userid doesn't prevent confused deputy attacks. The vendor's IAM principals would match this condition regardless of which customer account they're trying to access. This provides no customer-specific verification.
- This trust policy trusts an AWS service principal, not a vendor's AWS account. vendor-service.amazonaws.com is not a valid service principal format, and this doesn't address cross-account third-party access.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A company uses SAML 2.0 federation with Azure AD for AWS access. Users report they can authenticate but receive "not authorized to perform sts:AssumeRoleWithSAML" errors. The IAM role trust policy is: {"Effect":"Allow","Principal":{"Federated":"arn:aws:iam::123456789012:saml-provider/AzureAD"},"Action":"sts:AssumeRoleWithSAML","Condition":{"StringEquals":{"SAML:aud":"https://signin.aws.amazon.com/saml"}}}. What is the MOST likely cause?

A) The Azure AD SAML assertion is not including the correct role ARN in the https://aws.amazon.com/SAML/Attributes/Role attribute
B) The SAML:aud condition should be "https://aws.amazon.com/saml" instead of "https://signin.aws.amazon.com/saml"
C) The trust policy should use "sts:AssumeRole" action instead of "sts:AssumeRoleWithSAML"
D) The IAM role's permission policy does not allow the actions the users are trying to perform

<details>
<summary>Answer</summary>

**Correct: A**

For SAML federation, Azure AD must include the AWS role ARN and SAML provider ARN in the SAML assertion attribute https://aws.amazon.com/SAML/Attributes/Role in the format: arn:aws:iam::account:role/rolename,arn:aws:iam::account:saml-provider/providername. If this attribute is missing or malformed, users authenticate successfully but cannot assume the role. The error message specifically indicates the AssumeRoleWithSAML call is failing.

**Why the others are wrong:**
- The SAML:aud value https://signin.aws.amazon.com/saml is correct for AWS SAML federation. This is the standard audience value AWS expects. Changing this would break authentication entirely.
- The trust policy must use sts:AssumeRoleWithSAML for SAML federation. Using sts:AssumeRole would only work for direct cross-account access, not federated access. This is a critical distinction.
- The error occurs during role assumption (sts:AssumeRoleWithSAML), before permission policies are evaluated. If the role's permission policy was the issue, users would assume the role successfully but get permission denied errors when performing actions.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** A company wants to implement ABAC to allow developers to manage only EC2 instances they created, without creating separate policies for each developer. All developers are in the Developer IAM Identity Center permission set. Which solution implements this correctly?

A) Add session tags to the permission set with CostCenter and Department attributes, create IAM policy: {"Effect":"Allow","Action":"ec2:*","Resource":"*","Condition":{"StringEquals":{"ec2:ResourceTag/Owner":"${aws:username}"}}}
B) Configure the permission set to pass the user's email as a session tag, create IAM policy: {"Effect":"Allow","Action":"ec2:*","Resource":"*","Condition":{"StringEquals":{"ec2:ResourceTag/Owner":"${aws:PrincipalTag/email}"}}}
C) Use resource-based policies on EC2 instances that grant access only to the creating user's ARN
D) Create dynamic IAM policies using AWS Lambda that updates policies when EC2 instances are created with aws:userid tags

<details>
<summary>Answer</summary>

**Correct: B**

ABAC with session tags uses ${aws:PrincipalTag/tagkey} in policy conditions. IAM Identity Center can pass attributes from the identity source (like email) as session tags. The policy condition then checks if the EC2 resource tag Owner matches the principal's email session tag. This scales without creating per-user policies. The user must tag instances with Owner on creation.

**Why the others are wrong:**
- The variable ${aws:username} is only available for IAM users, not federated sessions from IAM Identity Center. Identity Center users authenticate via federation and don't have an aws:username value. This condition would never match.
- EC2 instances don't support resource-based policies. Only certain services like S3, SNS, and SQS support resource-based policies. This is not applicable to EC2 instances.
- This defeats the purpose of ABAC by creating dynamic, per-user policies. ABAC's benefit is using a single policy with attribute conditions. Lambda-based policy updates add complexity, latency, and potential race conditions.

</details>

---

**Q7.** An application assumes an IAM role via sts:AssumeRole and needs to create S3 objects that can only be accessed by resources tagged with Project=Alpha. The assumed role session needs to enforce this. Which combination is required?

A) The role's permission policy includes a condition requiring aws:ResourceTag/Project equals Alpha for s3:GetObject actions
B) The AssumeRole call includes session tags with Project=Alpha, and the role's permission policy allows s3:PutObject with condition: {"StringEquals":{"s3:x-amz-tagging/Project":"Alpha"}}, and permissions boundary prevents creating objects without this tag
C) Configure S3 bucket policy to deny access unless aws:PrincipalTag/Project equals Alpha, then tag the assumed role with Project=Alpha
D) Use STS session policies passed during AssumeRole to restrict object creation to those tagged with Project=Alpha

<details>
<summary>Answer</summary>

**Correct: B**

This requires session tags passed during AssumeRole, a permission policy condition using s3:x-amz-tagging to enforce tag presence on PutObject, and critically - a permissions boundary to prevent bypassing the tag requirement. The condition alone can be bypassed by not including tags. The permissions boundary ensures tags are mandatory. For GetObject enforcement, you'd use a separate condition on aws:ResourceTag.

**Why the others are wrong:**
- A permission policy condition on the role doesn't enforce tagging requirements on object creation - it only restricts what the role can access. The application could create untagged objects and then not be able to read them, but the creation isn't prevented.
- Bucket policies use aws:PrincipalTag to evaluate the caller's tags, but they can't enforce that created objects have specific tags. Also, you can't tag an IAM role directly - tags on roles don't become aws:PrincipalTag values in requests. Session tags during AssumeRole become principal tags.
- Session policies can only restrict permissions, not add requirements for tags on resources being created. Session policies reduce permissions from the role's permission policy but don't enforce resource tagging. The s3:x-amz-tagging condition in the role policy is needed.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A company has 200 AWS accounts organized under AWS Organizations. They need to grant a third-party security vendor read-only access to all accounts to perform security audits. The vendor should use their own AWS account credentials. Which approach provides the MOST secure and manageable solution?

A) Create a CloudFormation StackSet to deploy an IAM role in all 200 accounts with a trust relationship to the vendor's account and SecurityAudit managed policy, use External ID for additional security
B) Set up IAM Identity Center, create a permission set with SecurityAudit policy, create the vendor as an external user in Identity Center, assign them to all accounts
C) Use AWS Organizations delegated administrator for the vendor's account, granting them cross-account read access via organization policies
D) Create IAM users in each account for the vendor with read-only policies and provide credentials via AWS Secrets Manager

<details>
<summary>Answer</summary>

**Correct: A**

CloudFormation StackSets deploy resources across multiple accounts simultaneously. Deploying an IAM role with trust to the vendor's account (plus External ID for confused deputy protection) is the standard pattern for third-party cross-account access. The vendor uses sts:AssumeRole from their account to access each customer account. This is manageable (one StackSet), secure (External ID, role assumption audited), and follows AWS best practices.

**Why the others are wrong:**
- IAM Identity Center is for employees and internal users, not external third parties with their own AWS accounts. The vendor wants to use their AWS account credentials, not become a user in the customer's Identity Center directory. Identity Center is designed for managing human users within your organization.
- Delegated administrator is for specific AWS services (like Security Hub, GuardDuty) to grant an account management capabilities for that service across the organization. It's not a mechanism for granting a third-party account general read access. This misunderstands delegated administrator's purpose.
- Creating IAM users violates best practices (use roles for cross-account, not users), doesn't scale (200 sets of credentials to manage), and creates long-term credentials instead of temporary ones from AssumeRole. Secrets Manager doesn't solve the fundamental issues.

</details>

---

