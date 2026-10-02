# IAM Advanced: Federation, ABAC, Cross-Account Access

## IAM Identity Center (formerly AWS SSO) vs IAM Roles

| Aspect | IAM Identity Center | Direct IAM Roles |
|--------|-------------------|------------------|
| **Use case** | Workforce identity, multi-account access | Application/service access, cross-account |
| **Management** | Centralized permission sets | Individual role in each account |
| **Deployment** | Permission set creates role automatically | Manual role creation |
| **Role name** | AWSReservedSSO_SetName_hash | Custom name |
| **Max session** | 12 hours (permission set config) | 12 hours (role config) |
| **Federation** | Built-in SAML to Identity Center | Configure SAML per account |

## Federation Methods Comparison

| Method | Use Case | API Call | Principal Type | Example |
|--------|----------|----------|----------------|----------|
| **SAML 2.0** | Enterprise SSO (Okta, Azure AD, Google) | AssumeRoleWithSAML | Federated: arn:aws:iam::acct:saml-provider/name | Users authenticate to IdP, get SAML assertion, exchange for temp creds |
| **OIDC** | Web apps, GitHub Actions, mobile | AssumeRoleWithWebIdentity | Federated: arn:aws:iam::acct:oidc-provider/url | GitHub workflow gets OIDC token, exchanges for AWS creds |
| **IAM Roles Anywhere** | On-prem servers, non-AWS compute | AssumeRole via Roles Anywhere | Service: rolesanywhere.amazonaws.com | Server presents X.509 cert, gets temp creds |
| **Cross-account** | AWS account accessing another AWS account | AssumeRole | AWS: arn:aws:iam::acct:root or user/role | Service in Acct A assumes role in Acct B |

## SAML Federation Requirements

**AWS Side:**
1. IAM SAML identity provider (IdP metadata XML uploaded)
2. IAM role trust policy:
```json
{
  "Effect": "Allow",
  "Principal": {"Federated": "arn:aws:iam::123456789012:saml-provider/MyIdP"},
  "Action": "sts:AssumeRoleWithSAML",
  "Condition": {"StringEquals": {"SAML:aud": "https://signin.aws.amazon.com/saml"}}
}
```

**IdP Side:**
- SAML assertion MUST include: `https://aws.amazon.com/SAML/Attributes/Role` attribute
- Format: `arn:aws:iam::account:role/rolename,arn:aws:iam::account:saml-provider/providername`
- Optional: `https://aws.amazon.com/SAML/Attributes/RoleSessionName`

**Common errors:** Missing Role attribute → "not authorized to perform sts:AssumeRoleWithSAML"

## OIDC Federation Trust Policy (GitHub Actions Example)

```json
{
  "Effect": "Allow",
  "Principal": {"Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"},
  "Action": "sts:AssumeRoleWithWebIdentity",
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:aud": "sts.amazonaws.com",
      "token.actions.githubusercontent.com:sub": "repo:myorg/myrepo:ref:refs/heads/main"
    }
  }
}
```

**Sub claim patterns:**
- Specific branch: `repo:org/repo:ref:refs/heads/branchname`
- Any branch: `repo:org/repo:ref:refs/heads/*`
- Pull request: `repo:org/repo:pull_request`

## External ID for Third-Party Access

**Problem:** Confused deputy attack - vendor tricked into accessing wrong customer's resources

**Solution:** Each customer provides unique External ID to vendor

```json
{
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::vendor-account:root"},
  "Action": "sts:AssumeRole",
  "Condition": {"StringEquals": {"sts:ExternalId": "unique-customer-id"}}
}
```

**Exam key:** Third-party cross-account = External ID is best practice

## ABAC (Attribute-Based Access Control)

**Concept:** Use tags on principals and resources to control access dynamically

**Session tags:** Passed during AssumeRole, become `aws:PrincipalTag/tagkey`

**ABAC policy example:**
```json
{
  "Effect": "Allow",
  "Action": "ec2:*",
  "Resource": "*",
  "Condition": {
    "StringEquals": {
      "ec2:ResourceTag/Owner": "${aws:PrincipalTag/email}",
      "ec2:ResourceTag/Department": "${aws:PrincipalTag/Department}"
    }
  }
}
```

**Enforcing tags on resource creation (S3 example):**
```json
{
  "Effect": "Allow",
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::bucket/*",
  "Condition": {
    "StringEquals": {"s3:x-amz-tagging/Project": "${aws:PrincipalTag/Project}"}
  }
}
```

**Tag condition keys:**
- `aws:PrincipalTag/key` - tag on the caller (from session tags or resource tags on user/role)
- `ec2:ResourceTag/key` - tag on EC2 resource
- `s3:x-amz-tagging/key` - tag being applied during PutObject

## STS API Calls Summary

| API | Used For | Max Duration | Returns |
|-----|----------|--------------|----------|
| AssumeRole | Cross-account, service roles | 12 hours | AccessKeyId, SecretAccessKey, SessionToken |
| AssumeRoleWithSAML | SAML federation | 12 hours | Same + Subject, Issuer |
| AssumeRoleWithWebIdentity | OIDC federation | 12 hours | Same + SubjectFromWebIdentityToken |
| GetSessionToken | MFA for IAM user | 36 hours | Same credentials |
| GetFederationToken | Custom federation broker | 36 hours | Same credentials |

## Session Duration Limits

- **IAM role default:** 1 hour
- **IAM role maximum:** 12 hours (set MaxSessionDuration on role)
- **Federation default:** 1 hour  
- **Federation maximum:** 12 hours (if role allows)
- **GetSessionToken/GetFederationToken:** 36 hours max
- **IAM Identity Center permission set:** 1-12 hours

## IAM Roles Anywhere

**Components:**
1. Trust anchor: AWS Private CA or uploaded CA certificate
2. Profile: Maps to IAM role
3. X.509 certificate on server (issued by trusted CA)
4. Credential helper tool on server

**Trust policy:**
```json
{
  "Effect": "Allow",
  "Principal": {"Service": "rolesanywhere.amazonaws.com"},
  "Action": ["sts:AssumeRole", "sts:TagSession", "sts:SetSourceIdentity"],
  "Condition": {
    "StringEquals": {"rolesanywhere:CertificateField/Subject": "CN=server.example.com"}
  }
}
```

**Exam trap:** Roles Anywhere is for on-premises with X.509 certs. Don't confuse with OIDC (GitHub Actions) or SAML (enterprise SSO).
