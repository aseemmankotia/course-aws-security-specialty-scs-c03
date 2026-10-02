# Chapter 5 Lab: IAM Advanced: Federation, ABAC, and Cross-Account Access

**Estimated time:** 30 minutes

## Exercise

Configure IAM Identity Center permission set for developers with ABAC tags, create SAML federation trust with Okta, build cross-account role in Account B trusting Account A with external ID, test STS AssumeRole with session tags

## Key takeaway

IAM Identity Center permission sets are the preferred multi-account access method over individual IAM roles; ABAC conditions filter allowed resources but don't grant base permissions - identity policy must still allow the action.
