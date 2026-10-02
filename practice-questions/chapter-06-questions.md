## Chapter 6: IAM Troubleshooting: Denies, Access Analyzer, and Least Privilege — Practice Questions

**Q1.** A security engineer receives an AccessDenied error when attempting to terminate an EC2 instance in the production account. CloudTrail shows errorCode: AccessDenied and errorMessage: 'User is not authorized to perform: ec2:TerminateInstances'. The user has an IAM policy granting ec2:*, an SCP at the OU level denying ec2:TerminateInstances on production instances, and a permissions boundary allowing all EC2 actions. Which is the MOST likely cause?

A) The permissions boundary is blocking the action because it doesn't explicitly allow ec2:TerminateInstances
B) The SCP deny at the OU level is overriding the IAM policy allow
C) The IAM policy needs to explicitly list ec2:TerminateInstances instead of using a wildcard
D) The resource-based policy on the EC2 instance is denying the action

<details>
<summary>Answer</summary>

**Correct: B**

SCPs are evaluated before identity-based policies and permissions boundaries. An explicit Deny in an SCP cannot be overridden by any Allow statement in identity-based policies or permissions boundaries. The SCP deny takes precedence.

**Why the others are wrong:**
- Permissions boundaries work by intersection with identity policies - they limit maximum permissions but don't add denies. The boundary allows ec2:*, so it's not the blocker.
- ec2:* already includes ec2:TerminateInstances. Wildcards expand to include all actions in that namespace.
- EC2 instances don't have resource-based policies. Only certain services like S3, SQS, KMS support resource-based policies.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An organization uses IAM Access Analyzer and discovers a finding showing 'External Access' to an S3 bucket from account 111122223333. The security team confirms this account is a trusted partner. What is the BEST approach to handle this finding?

A) Delete the bucket policy granting access to account 111122223333
B) Create an archive rule in Access Analyzer to suppress findings for principal 111122223333
C) Add account 111122223333 to the Access Analyzer trusted zone of trust
D) Disable Access Analyzer for this bucket to prevent false positives

<details>
<summary>Answer</summary>

**Correct: B**

Archive rules in IAM Access Analyzer allow you to automatically archive findings that match specific criteria (like trusted external accounts). This maintains visibility while reducing noise. The finding remains accessible but is automatically marked as reviewed.

**Why the others are wrong:**
- Deleting the policy would break legitimate business access to the trusted partner - this is intentional external access that needs to continue.
- Access Analyzer zone of trust is defined at analyzer creation time as your organization or account. You cannot add individual external accounts to the zone of trust after creation. Archive rules are the correct mechanism for trusted external principals.
- You cannot selectively disable Access Analyzer per resource. Archive rules are the proper way to handle expected findings without disabling the security control.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A company needs to generate least-privilege IAM policies for 50 developers based on their actual AWS usage over the past 90 days. Which approach provides the MOST accurate least-privilege policies?

A) Use IAM Access Advisor to review last accessed services and manually create policies
B) Enable CloudTrail and use Access Analyzer policy generation based on CloudTrail events
C) Use AWS Config to track resource changes and generate policies from configuration history
D) Review AWS Cost Explorer to identify used services and create policies accordingly

<details>
<summary>Answer</summary>

**Correct: B**

IAM Access Analyzer policy generation analyzes CloudTrail events over a specified time period (up to 90 days) and automatically generates policies based on actual API calls made. This produces the most accurate least-privilege policies based on real usage patterns, including specific actions and resources.

**Why the others are wrong:**
- Access Advisor only shows service-level access (last accessed time for services), not specific actions or resources. It provides less granular data than CloudTrail-based policy generation.
- AWS Config tracks resource configuration changes, not IAM actions or API calls. It cannot determine what API calls users made or generate IAM policies.
- Cost Explorer shows billing data for services used, but doesn't capture specific API actions, resource ARNs, or conditions needed for proper IAM policies.

</details>

---

**Q4.** A developer receives this error when trying to create an RDS database: 'errorCode: InvalidPermission.NotFound'. CloudTrail shows the API call was rds:CreateDBInstance. The user has an IAM policy with 'Effect: Allow, Action: rds:*, Resource: *'. What is the MOST likely cause?

A) The RDS service is not available in the selected region
B) The user lacks the required iam:PassRole permission to pass the DB instance role
C) An SCP is blocking RDS database creation in this account
D) The VPC security group specified doesn't exist

<details>
<summary>Answer</summary>

**Correct: B**

InvalidPermission.NotFound typically indicates missing iam:PassRole permission. When creating RDS instances, users must pass an IAM role to the instance. Even with rds:* permissions, you need explicit iam:PassRole for the specific role being assigned to the DB instance.

**Why the others are wrong:**
- Regional unavailability would return a different error (InvalidParameterValue or UnsupportedOperation), not InvalidPermission.NotFound which is an authorization error.
- An SCP deny would show 'AccessDenied' as the errorCode in CloudTrail, not InvalidPermission.NotFound. Different error codes indicate different policy evaluation failures.
- Missing security group would return InvalidParameterValue or InvalidGroup.NotFound, not InvalidPermission.NotFound. The error code specifically indicates a permissions issue, not a resource validation issue.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** An organization wants to prevent all member accounts from creating Internet Gateways while still allowing VPC creation. Which SCP statement accomplishes this?

A) {"Effect":"Deny","Action":"ec2:CreateInternetGateway","Resource":"*"}
B) {"Effect":"Deny","Action":"ec2:*","Resource":"arn:aws:ec2:*:*:internet-gateway/*"}
C) {"Effect":"Allow","Action":"ec2:*","Resource":"*","Condition":{"StringNotEquals":{"ec2:ResourceType":"internet-gateway"}}}
D) {"Effect":"Deny","NotAction":"ec2:CreateInternetGateway","Resource":"*"}

<details>
<summary>Answer</summary>

**Correct: A**

A simple explicit Deny on ec2:CreateInternetGateway prevents Internet Gateway creation across all accounts in the OU/organization while allowing all other EC2 actions like VPC creation. SCPs work with explicit denies - this is the most direct and effective approach.

**Why the others are wrong:**
- Resource-level restrictions in SCPs don't prevent creation of new resources. The Internet Gateway doesn't exist yet, so you can't deny actions on its ARN. Action-based deny is required for creation prevention.
- SCPs don't support Allow statements - they only restrict permissions through Deny. An SCP with Effect Allow has no impact. SCPs create guardrails by denying specific actions.
- NotAction with Deny would block everything EXCEPT CreateInternetGateway, which is the opposite of what's needed. This would prevent VPC creation and all other EC2 operations.

</details>

---

**Q6.** A security team uses IAM Policy Simulator to test a new policy before deployment. The simulator shows 'allowed' for s3:PutObject, but when deployed, the user receives AccessDenied. What is the MOST likely explanation?

A) IAM Policy Simulator doesn't evaluate Service Control Policies (SCPs)
B) The S3 bucket has a bucket policy with an explicit Deny that the simulator cannot evaluate
C) IAM Policy Simulator has a lag time of up to 24 hours for policy changes
D) The simulator doesn't account for MFA requirements in Condition blocks

<details>
<summary>Answer</summary>

**Correct: B**

IAM Policy Simulator evaluates only IAM policies (identity-based and resource-based policies attached to the IAM entity). It does NOT evaluate resource-based policies on the target resource (like S3 bucket policies). A bucket policy with explicit Deny would cause AccessDenied in production but wouldn't be visible in the simulator.

**Why the others are wrong:**
- While IAM Policy Simulator doesn't evaluate SCPs in standard mode, you can optionally include SCPs and permissions boundaries in the simulation. More importantly, the most common missed element is bucket policies, which cannot be simulated at all.
- IAM Policy Simulator evaluates policies in real-time with no delay. Policy changes are reflected immediately in the simulator.
- The simulator DOES evaluate Condition blocks including MFA requirements. You can specify context keys like aws:MultiFactorAuthPresent in the simulation.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A company needs to ensure that no S3 bucket in any account can be made public, even if an administrator explicitly allows it. The solution must work across all existing and future accounts in the organization. Which approach provides the STRONGEST protection?

A) Create an SCP denying s3:PutBucketPolicy and s3:PutBucketAcl on all buckets
B) Enable S3 Block Public Access at the organization level using AWS Organizations
C) Create a Resource Control Policy (RCP) enforcing private-only S3 buckets
D) Use AWS Config rules to detect and auto-remediate public S3 buckets

<details>
<summary>Answer</summary>

**Correct: B**

S3 Block Public Access at the organization level (managed through AWS Organizations) provides the strongest control. It prevents public access regardless of bucket policies, ACLs, or access points. This setting cannot be overridden at the account or bucket level when set at the organization level, providing an absolute guardrail.

**Why the others are wrong:**
- Denying s3:PutBucketPolicy and s3:PutBucketAcl prevents modifying policies but doesn't address existing public configurations and doesn't prevent public access through other mechanisms like access points. It also breaks legitimate use cases requiring policy updates.
- Resource Control Policies (RCPs) were introduced in 2025 for resource-level access control but S3 Block Public Access is still the AWS-recommended and strongest control specifically for preventing public S3 access. RCPs are broader and more complex for this specific use case.
- AWS Config rules are detective controls that identify violations after they occur. They don't prevent the configuration from being set. Even with auto-remediation, there's a window where the bucket could be public.

</details>

---

**Q8.** A security engineer needs to create a least-privilege policy that allows all EC2 actions EXCEPT RunInstances with instance types larger than t3.medium. Which policy structure is MOST appropriate?

A) Use Effect: Allow with Action: ec2:* and a Condition block with StringNotEquals on ec2:InstanceType
B) Use Effect: Deny with Action: ec2:RunInstances and Condition: StringNotEquals for t3.small, t3.micro, t3.medium
C) Use two statements: Allow ec2:* with NotAction on RunInstances, and Allow RunInstances with Condition limiting instance types
D) Use Effect: Allow with NotAction: ec2:RunInstances on large instance types

<details>
<summary>Answer</summary>

**Correct: C**

The correct approach uses two statements: (1) Allow all EC2 actions except RunInstances, (2) Allow RunInstances only with condition limiting instance types to t3.medium or smaller. This provides precise least-privilege control - full access to EC2 management but restricted instance launch capabilities.

**Why the others are wrong:**
- StringNotEquals would allow everything EXCEPT the listed types, meaning it would allow larger instances (t3.large, t3.xlarge, etc.) and deny the small ones - opposite of the requirement.
- Effect: Deny with StringNotEquals creates a double negative that allows large instances. Also, listing every allowed instance type is not maintainable. The Condition would need StringEquals with allowed types or StringNotLike with a pattern for large instances.
- NotAction cannot be used with resource type restrictions. NotAction applies to actions, not resources or conditions. This syntax is invalid and wouldn't achieve the desired restriction.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

