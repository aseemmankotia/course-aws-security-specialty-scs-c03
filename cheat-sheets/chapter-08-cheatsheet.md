# Data Protection: Encryption at Rest, in Transit, and Data Classification

## Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Parameter Store SecureString |
|---------|----------------|-----------------------------|
| **Cost** | $0.40/secret/month + $0.05/10K calls | Standard: Free, Advanced: $0.05/month |
| **Automatic Rotation** | Yes (Lambda-based, built-in for RDS/Redshift/DocumentDB) | No - manual rotation only |
| **Use Case** | Database credentials, API keys needing auto-rotation | Config data, manual rotation acceptable |
| **Rotation Strategies** | Single-user (brief downtime), Multi-user (zero downtime) | N/A |
| **Encryption** | KMS encryption automatic | KMS encryption with SecureString |
| **Cross-account** | Resource policy + KMS policy | Via RAM or cross-account IAM |

**Exam Gotcha**: Secrets Manager multi-user rotation creates NEW user before disabling old → both AWSCURRENT and AWSPREVIOUS work during rotation = zero downtime.

## ACM vs ACM Private CA

| Feature | ACM (Public Certificates) | ACM Private CA |
|---------|--------------------------|----------------|
| **Cost** | Free certificates | $400/month for CA + $0.75/cert if issued via API (free if via ACM) |
| **Use Case** | Public-facing websites, CloudFront, ALB/NLB | Internal services, microservices, private domains, VPN |
| **Validation** | DNS, Email, HTTP challenge | None - you control CA |
| **Auto-Renewal** | Yes (ACM managed) | Yes if issued through ACM, No if API-issued |
| **Domain Types** | Public domains only | Any domain (internal/private) |
| **Cross-Account** | Certificate must be in same account as ALB/CloudFront | Share CA via RAM, issue certs per account |

**Exam Gotcha**: Private CA costs $400/month but unlimited certificate issuance via ACM. For 50+ internal services, cheaper than alternatives.

## S3 Encryption Methods

| Method | Key Management | Performance | Use Case |
|--------|---------------|-------------|----------|
| **SSE-S3** | AWS managed, automatic | Fastest (no KMS calls) | Default encryption, no audit requirements |
| **SSE-KMS** | Customer managed CMK | KMS API quota applies | Need audit logs, key rotation, access control |
| **SSE-C** | Customer provides key per request | Customer manages keys | Regulatory requirement to control keys outside AWS |
| **Client-Side** | Customer encrypts before upload | Application complexity | Need to encrypt before AWS, end-to-end control |

**Required Permissions for SSE-KMS**: 
- Read object: `s3:GetObject` + `kms:Decrypt`
- Write object: `s3:PutObject` + `kms:Encrypt` + `kms:GenerateDataKey`

**Encryption in Transit**:
- Enforce TLS: Bucket policy with `"Condition": {"Bool": {"aws:SecureTransport": "false"}}` → Deny
- All AWS SDK calls use HTTPS by default (S3, DynamoDB, etc.)
- VPC Endpoints = private + encrypted (TLS by default)

## Amazon Macie

### Managed Data Identifiers (Built-in)
- Credit card numbers (Visa, Mastercard, Amex, Discover)
- AWS credentials (access keys, secret keys)
- Private keys (RSA, SSH, PGP)
- U.S. Social Security Numbers
- Passport numbers (multiple countries)
- Driver's license numbers
- Bank account numbers
- HIPAA/PCI DSS regulated data types

### Custom Data Identifiers
- **Regex Pattern**: Define proprietary format (e.g., `[A-Z]{3}[0-9]{5}` for employee IDs)
- **Keywords**: Optional terms that increase confidence (e.g., "employee", "staff")
- **Proximity**: Max 300 characters between regex match and keyword
- **Ignore Words**: Terms that indicate false positive

**Exam Pattern**: Custom identifier = regex + keywords + proximity value ≤ 300

### Sensitive Data Discovery Jobs
- **Scope**: Specific buckets/prefixes or entire account
- **Schedule**: One-time, daily, weekly, monthly
- **Sampling**: 5%, 20%, 50%, 100% of objects
- **File Types**: Supports unstructured (PDF, DOCX, images with OCR) and structured (CSV, JSON, Parquet)
- **Output**: Findings with severity, object location, data types detected

**Exam Gotcha**: Discovery job is for finding sensitive data locations. Not real-time monitoring (that's GuardDuty S3 Protection).

## KMS Key Rotation

| Rotation Type | Frequency | CMK ID Change | Use Case |
|--------------|-----------|---------------|----------|
| **Automatic** | Every 365 days (fixed) | No - same CMK ID, new material | Compliance requirement for annual rotation |
| **Manual** | On-demand | Yes - create new CMK | Immediate rotation needed, key compromise |

**Automatic Rotation Details**:
- Rotates key material, not CMK ID
- Old material retained for decrypting existing data
- Cannot trigger manually or change frequency
- Free (no additional cost)

**Manual Rotation Process**:
1. Create new CMK
2. Update application/bucket to use new CMK
3. Old CMK retained for decrypting old data
4. Optionally disable old CMK after re-encryption

## Key Exam Numbers

- Secrets Manager: $0.40/secret/month + $0.05/10,000 API calls
- ACM Private CA: $400/month for CA
- Parameter Store Advanced: $0.05/parameter/month (Standard = free)
- KMS automatic rotation: 365 days (cannot change)
- Macie custom identifier proximity: max 300 characters
- SSE-KMS request quota: 5,500/sec (shared with other KMS operations)

## Common Exam Scenarios

1. **Database credential rotation with zero downtime** → Secrets Manager multi-user rotation
2. **Internal microservices need TLS certs** → ACM Private CA ($400/month unlimited certs)
3. **Config values change monthly, need encryption** → Parameter Store SecureString (cheaper than Secrets Manager)
4. **Detect proprietary PII format in S3** → Macie custom data identifier with regex
5. **Prevent AWS admins from reading encrypted S3 data** → KMS key policy explicit deny on kms:Decrypt
6. **Enforce HTTPS uploads to S3** → Bucket policy aws:SecureTransport condition
7. **Rotate KMS key immediately** → Create new CMK (automatic rotation is 365 days only)
