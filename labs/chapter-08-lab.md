# Chapter 8 Lab: Data Protection: Encryption at Rest, in Transit, and Data Classification

**Estimated time:** 24 minutes

## Exercise

Create Secrets Manager secret for RDS password with automatic rotation every 30 days, configure Macie discovery job to scan S3 buckets for PII, create custom data identifier for employee ID format, export findings to Security Hub

## Key takeaway

Parameter Store SecureString is cost-effective for non-rotating encrypted configuration data; Amazon Macie discovery jobs run on schedule (not real-time) and scale to organization-wide S3 bucket scanning for sensitive data.
