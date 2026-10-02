# Chapter 7 Lab: Data Protection: KMS, Encryption, and Key Management

**Estimated time:** 25 minutes

## Exercise

Create KMS customer managed key with key policy allowing specific IAM role, create grant for Lambda function to use key temporarily, configure automatic rotation, create multi-Region key replica in second region

## Key takeaway

KMS grants provide temporary, programmatic key access without modifying key policy - ideal for dynamic principals; automatic key rotation rotates backing material annually while preserving the same key ID/ARN.
