# KMS, Encryption & Key Management Cheat Sheet

## Key Material Origins
| Origin | Auto Rotation | Use Case | Limitation |
|--------|---------------|----------|------------|
| AWS_KMS | ✅ Yearly | Standard use | AWS controls material |
| EXTERNAL | ❌ Manual only | BYOK compliance | You manage backup/expiry |
| AWS_CLOUDHSM | ❌ Manual only | Custom key store | CloudHSM cluster required |

## KMS Key Types
| Type | Description | Rotation | Cost |
|------|-------------|----------|------|
| AWS managed (aws/service) | AWS creates, service-linked | ✅ Every 3 years | Free |
| Customer managed | You create, full control | ✅ Yearly (opt-in) | $1/month + API calls |
| Multi-region | Same key in multiple regions | ✅ Primary rotates all | $1/month per region |

## Key Policy vs IAM Policy vs Grant
| Method | Modification | Temporary | Use Case |
|--------|--------------|-----------|----------|
| Key Policy | Policy update | ❌ Permanent | Cross-account, service access |
| IAM Policy | IAM update | ❌ Permanent | User/role permissions |
| Grant | CreateGrant API | ✅ Expires/retires | Temp delegation, no policy change |

**Evaluation:** Both key policy AND IAM policy must allow = dual authorization

## Multi-Region Keys
- **Primary key:** can create replicas, rotate material
- **Replica keys:** read-only, same key ID and material
- **Same key ID across regions** = no cross-region API calls
- **Rotation:** primary rotation = all replicas rotate
- **Use case:** global apps, DR, cross-region data replication
- **Cannot convert:** existing single-region → multi-region

## CloudHSM vs KMS
| Feature | KMS | CloudHSM |
|---------|-----|----------|
| FIPS 140-2 | Level 2 | Level 3 |
| Tenancy | Multi-tenant | Single-tenant |
| Control | AWS managed | You manage cluster |
| Min HA | N/A | 2 HSMs, 2 AZs |
| Use case | Most encryption | Custom crypto, Level 3 compliance |

## ABAC with KMS Keys
**Tag KMS key:** `Environment=Production`

**Key policy condition:**
```json
"Condition": {
  "StringEquals": {
    "aws:ResourceTag/Environment": "Production"
  }
}
```

**S3 SSE-KMS with object tags:**
```json
"Condition": {
  "StringEquals": {
    "s3:x-amz-server-side-encryption-context": "Environment=Production"
  }
}
```

## Encryption Context
- **Purpose:** Additional auth data for encrypt/decrypt
- **Format:** Key-value pairs (not encrypted, logged in CloudTrail)
- **Policy check:** `kms:EncryptionContext:key-name`
- **S3 integration:** Object tags → encryption context
- **Best practice:** Use for ABAC and audit trails

## Critical Limits
| Limit | Value | Impact |
|-------|-------|--------|
| Grants per key | 50,000 | Retire old grants |
| Encrypt/decrypt TPS | 5,500-100,000/sec | Use bucket/table keys |
| CMK max size | 4 KB | For data keys, not data |
| Key policy max size | 32 KB | Keep policies concise |
| Imported key expiration | You define | Can be no expiration |

## Exam Traps
1. **AWS managed key policy:** Cannot modify - need customer managed for custom access
2. **Automatic rotation:** Only AWS_KMS origin + symmetric keys
3. **Multi-region conversion:** Cannot convert existing key - must create new
4. **Grants max:** 50k limit - retire old ones, don't create infinite grants
5. **FIPS Level 3:** Only CloudHSM, not KMS custom key store
6. **Dual authorization:** Key policy AND IAM policy must both allow
7. **Cross-region KMS calls:** Don't work - use multi-region keys instead
8. **Key policy root principal:** Enables IAM policies, doesn't grant direct access

## Common Patterns
**Separation of duties:** Customer managed CMK + key policy denying S3 admin decrypt

**Temp 3rd party access:** CreateGrant with retirement date

**Global encryption:** Multi-region primary in home region, replicas in DR regions

**BYOK compliance:** Import external key material, you manage lifecycle

**High-volume S3 encryption:** Enable S3 bucket keys (reduces KMS API calls 99%)
