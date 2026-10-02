## Chapter 7: Data Protection: KMS, Encryption, and Key Management — Practice Questions

**Q1.** A financial services company needs to encrypt data in DynamoDB tables across three AWS regions (us-east-1, eu-west-1, ap-southeast-1). The application must decrypt data in any region without cross-region API calls. Compliance requires automatic key rotation and centralized key management. Which solution BEST meets these requirements?

A) Create a customer managed CMK in each region and use AWS Organizations to manage policies centrally
B) Create a multi-region customer managed CMK with primary key in us-east-1 and replica keys in other regions, enable automatic rotation
C) Use AWS managed keys (aws/dynamodb) in each region and enable DynamoDB Global Tables for replication
D) Create a customer managed CMK in us-east-1 and grant cross-region access through key policy

<details>
<summary>Answer</summary>

**Correct: B**

Multi-region KMS keys provide the same key ID and key material across regions, enabling local decryption without cross-region calls. Automatic rotation is supported and rotates all replicas simultaneously. This is the only solution that meets all requirements.

**Why the others are wrong:**
- Separate CMKs in each region have different key IDs and materials, requiring application logic to track which key encrypted which data. Not centralized key management.
- AWS managed keys cannot be multi-region keys and don't support cross-region encryption scenarios. You cannot control rotation schedules or key policies for AWS managed keys.
- KMS keys cannot grant cross-region access. KMS API calls are region-specific. This would require cross-region API calls which violates the requirement.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A security engineer needs to allow a Lambda function in Account A to use a KMS CMK in Account B for 7 days to decrypt configuration data during a migration. The solution must not require modifying the key policy and should automatically expire. What is the MOST appropriate approach?

A) Add the Lambda execution role to the key policy's Principal statement with a time-based condition
B) Create a KMS grant for the Lambda execution role with a 7-day retirement date using CreateGrant API
C) Use AWS STS to assume a role in Account B that has permissions to use the KMS key
D) Enable automatic key rotation and share the key alias with Account A

<details>
<summary>Answer</summary>

**Correct: B**

KMS grants are designed for temporary, programmatic access without modifying key policies. The CreateGrant API accepts RetireGrant parameter or grants can be created with constraints. Grants automatically expire and can be revoked programmatically, perfect for time-limited delegation.

**Why the others are wrong:**
- This requires modifying the key policy which violates the requirement. Time-based conditions in IAM are complex and require explicit deny statements after expiry.
- While STS assume role works for cross-account access, it still requires the key policy to allow the assumed role. Doesn't meet 'no key policy modification' requirement. Sessions expire but role permissions don't automatically expire in 7 days.
- Key rotation is unrelated to cross-account access. Key aliases cannot be shared across accounts. This doesn't address the access requirement.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A healthcare company must meet FIPS 140-2 Level 3 compliance for cryptographic operations on patient data. The company needs dedicated hardware cryptographic modules with single-tenant access. Which solution meets this requirement?

A) Use KMS customer managed keys with imported key material from on-premises HSM
B) Deploy AWS CloudHSM cluster with at least 2 HSMs across multiple AZs
C) Use KMS customer managed keys with AWS_CLOUDHSM key material origin
D) Enable FIPS mode on KMS customer managed keys using key policy conditions

<details>
<summary>Answer</summary>

**Correct: B**

CloudHSM provides FIPS 140-2 Level 3 validated HSMs with single-tenant, dedicated hardware access. Minimum 2 HSMs across AZs required for HA. KMS is Level 2 (multi-tenant). CloudHSM is the only AWS service that meets Level 3 compliance.

**Why the others are wrong:**
- Imported key material still uses KMS infrastructure which is FIPS 140-2 Level 2, not Level 3. The HSM validation level is determined by where the cryptographic operations occur, not where key material originates.
- AWS_CLOUDHSM origin allows KMS to use CloudHSM for key storage but crypto operations still happen in KMS (Level 2). This is for custom key store, not direct CloudHSM usage meeting Level 3.
- KMS does not have a 'FIPS mode' toggle. All KMS operations are FIPS 140-2 Level 2. You cannot upgrade KMS to Level 3 via configuration.

</details>

---

**Q4.** An application encrypts objects in S3 using SSE-KMS with a customer managed CMK. The security team requires that only objects tagged with 'Environment=Production' can be encrypted with this key. How should this be implemented?

A) Add a condition to the KMS key policy: "Condition": {"StringEquals": {"aws:RequestTag/Environment": "Production"}}
B) Add a condition to the KMS key policy: "Condition": {"StringEquals": {"s3:x-amz-server-side-encryption-context": "Environment=Production"}}
C) Add a condition to the KMS key policy: "Condition": {"StringEquals": {"kms:EncryptionContext:x-amz-server-side-encryption-context": "Environment=Production"}}
D) Add a condition to the S3 bucket policy requiring the encryption context to match the object tag

<details>
<summary>Answer</summary>

**Correct: B**

When using SSE-KMS, S3 passes object tags as encryption context in the 's3:x-amz-server-side-encryption-context' condition key. The KMS key policy can check this to enforce ABAC based on S3 object tags. This is the correct syntax for encryption context validation.

**Why the others are wrong:**
- aws:RequestTag checks tags being applied during the API call (like tagging the KMS key itself), not the encryption context or S3 object tags. Wrong context for this scenario.
- The encryption context condition key in KMS policy is 'kms:EncryptionContext:key', not 's3:x-amz-server-side-encryption-context'. This syntax is invalid for encryption context checking.
- S3 bucket policy cannot directly enforce KMS key policy conditions. While you can require SSE-KMS, the encryption context validation must happen in the KMS key policy. The enforcement point is KMS, not S3.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A company created a customer managed CMK 18 months ago with automatic key rotation enabled. The CMK has been used to encrypt 500GB of data in S3. After rotation, what actions are required to decrypt the previously encrypted data?

A) Re-encrypt all S3 objects with the new key material using S3 batch operations
B) Update all applications to use the new key ID generated during rotation
C) No action required - KMS automatically uses the correct backing key for decryption
D) Create a key alias pointing to the new key version and update application configurations

<details>
<summary>Answer</summary>

**Correct: C**

KMS automatic rotation rotates the backing key (cryptographic key material) annually but keeps the same CMK ID. KMS automatically uses the correct backing key version for decryption based on metadata in the ciphertext. No application changes or re-encryption needed. This is a key feature of KMS key rotation.

**Why the others are wrong:**
- Re-encryption is not required. KMS tracks which backing key version encrypted each ciphertext and uses the corresponding version for decryption. Automatic rotation is transparent to encrypted data.
- The CMK ID remains unchanged during automatic rotation. Only the backing key material rotates. Applications continue using the same key ID. This is the main benefit of automatic rotation over manual key replacement.
- Key aliases are independent of rotation and point to CMK IDs, not key versions. Rotation doesn't affect aliases. Updating aliases is unnecessary and wouldn't help since the CMK ID doesn't change.

</details>

---

**Q6.** A security engineer must evaluate the following KMS key policy statement. An IAM user has an attached policy allowing 'kms:Decrypt' on all resources. Will the user be able to decrypt data encrypted with this CMK? Key Policy: {"Sid": "Enable IAM policies", "Effect": "Allow", "Principal": {"AWS": "arn:aws:iam::123456789012:root"}, "Action": "kms:*", "Resource": "*"}

A) No - the key policy must explicitly list the IAM user ARN in the Principal
B) Yes - the root account principal allows IAM policies to grant access to the key
C) No - kms:Decrypt must be explicitly listed in the key policy Action
D) Yes - IAM user policies always override KMS key policies for decrypt operations

<details>
<summary>Answer</summary>

**Correct: B**

The key policy allows 'arn:aws:iam::123456789012:root' which represents the AWS account. This enables IAM policies in that account to grant access to the key. When the root account is in the Principal, IAM policies are evaluated. The user's IAM policy with kms:Decrypt will work. This is the standard pattern for enabling IAM policy evaluation.

**Why the others are wrong:**
- When the account root is in the key policy Principal, individual IAM users don't need explicit listing. The root account principal delegates to IAM. This is how KMS key policies enable IAM-based access control.
- kms:* includes kms:Decrypt. Wildcards work in key policies. The Action statement already covers decrypt operations.
- IAM policies never override key policies. Both must allow the action (logical AND). The key policy must enable IAM policies first (via root principal), then IAM policies can grant access. This is dual authorization.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A company needs to import their own key material into KMS for regulatory compliance. The key material must be generated from an on-premises FIPS-certified HSM. Which limitation applies to this configuration?

A) Imported key material cannot be used with automatic key rotation
B) Imported key material must be reimported every 90 days due to AWS security requirements
C) KMS will automatically backup the imported key material to S3 for disaster recovery
D) Imported key material cannot be used with multi-region keys

<details>
<summary>Answer</summary>

**Correct: A**

CMKs with imported key material do not support automatic key rotation. You must manually rotate by creating a new CMK, importing new material, and updating applications. AWS cannot rotate material it didn't generate. This is a critical limitation of BYOK (Bring Your Own Key).

**Why the others are wrong:**
- There is no mandatory 90-day reimport requirement. You set the expiration when importing or can import without expiration. The material remains until you delete it or it expires per your configuration.
- KMS never stores key material outside the HSMs, even for imported keys. KMS does not backup key material to S3. You are responsible for backing up imported key material outside AWS.
- Multi-region keys DO support imported key material. You import the same material to the primary and all replica keys. This allows BYOK with multi-region functionality.

</details>

---

**Q8.** An application uses AWS managed keys (aws/s3) to encrypt S3 objects. The security team now requires separation of duties where the S3 administrator cannot decrypt objects. What is the MINIMUM change required?

A) Enable S3 bucket key to reduce KMS costs and improve performance
B) Create a customer managed CMK with a key policy that doesn't allow the S3 admin role to decrypt
C) Switch to SSE-C (customer provided keys) and manage keys outside AWS
D) Enable MFA delete on the S3 bucket to prevent unauthorized decryption

<details>
<summary>Answer</summary>

**Correct: B**

AWS managed keys have default policies that allow S3 service and anyone with S3 access to use the key. You cannot modify AWS managed key policies. Customer managed CMK is required to implement custom key policies that deny specific principals (like S3 admin) from decrypt operations while allowing encrypt. This enables separation of duties.

**Why the others are wrong:**
- S3 bucket keys reduce KMS API calls but don't change access control. The underlying aws/s3 key policy still allows anyone with S3 access to decrypt. Doesn't solve the separation of duties requirement.
- SSE-C provides separation but requires managing keys outside AWS entirely, distributing keys securely, and handling key rotation manually. This is far more complex than 'minimum change'. Customer managed CMK is simpler.
- MFA delete protects against object deletion, not decryption. It's unrelated to KMS key access control. S3 admins can still decrypt with MFA delete enabled if they have kms:Decrypt permission.

</details>

---

