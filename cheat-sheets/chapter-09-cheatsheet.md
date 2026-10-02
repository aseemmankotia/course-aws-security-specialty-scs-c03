# Chapter 9 Cheat Sheet: Security Governance & Multi-Account Architecture

## AWS Organizations & SCPs

### SCP Evaluation Rules
- **Inheritance**: SCPs applied at root → OU → account (all levels intersect)
- **Deny wins**: Explicit Deny cannot be overridden by Allow in child OUs
- **Limits**: 5 SCPs per entity (account/OU), including inherited
- **Default**: FullAWSAccess SCP attached to root (allows everything unless denied)
- **Exam trap**: Allow in child OU does NOT override Deny in parent OU

### Common OU Structures
| OU Type | Purpose | Typical SCPs |
|---------|---------|-------------|
| Security | Log archive, audit accounts | Deny deletion of logs, require MFA |
| Infrastructure | Shared services, networking | Deny region restrictions, allow cross-account access |
| Workloads | Production, staging, dev | Environment-specific restrictions |
| Sandbox | Isolated experimentation | Strict service/region limits |

## AWS Config

### Rule Types
- **Managed rules**: AWS-provided (90+ rules), config change or periodic
- **Custom rules**: Lambda-based, your logic, up to 100 per region
- **Organization rules**: Deployed from management account to all members

### Evaluation Frequency
- **Change-triggered**: 10-15 minutes after resource change
- **Periodic**: 1hr, 3hr, 6hr, 12hr, or 24hr intervals
- **Exam key**: Config = detective control (not preventive, has lag time)

### Conformance Packs
- **Pre-built**: CIS AWS Foundations, PCI-DSS, NIST 800-53, HIPAA
- **Deployment**: Organization-wide via management account or per-account
- **Format**: YAML templates with Config rules + remediation actions
- **Use case**: Framework compliance across multiple accounts

### Config Aggregator
- **Purpose**: Centralized compliance view across accounts/regions
- **Scope**: Organization-wide or specific account list
- **Exam tip**: Organization aggregator auto-includes new accounts

## AWS Control Tower

### Components
- **Landing zone**: Multi-account environment with guardrails
- **Account Factory**: Automated account provisioning with baselines
- **Guardrails**: Preventive (SCPs) + detective (Config rules)
- **Mandatory guardrails**: CloudTrail enabled, Config enabled (cannot disable)

### Guardrail Types
| Type | Mechanism | Example |
|------|-----------|--------|
| Preventive (mandatory) | SCP | Disallow changes to CloudTrail |
| Preventive (optional) | SCP | Deny root account actions |
| Detective (mandatory) | Config | Detect CloudTrail disabled |
| Detective (optional) | Config | Detect public S3 buckets |

## Software Supply Chain Security

### CodeArtifact
- **Package origin controls**: ALLOW_UPSTREAM (only from configured sources) vs ALLOW_DIRECT_PUBLISH
- **External connections**: PyPI, npm, Maven Central, NuGet (one per repository)
- **Exam scenario**: Prevent developers publishing arbitrary packages → use ALLOW_UPSTREAM + remove PublishPackageVersion permission

### AWS Signer
- **Lambda signing**: AWSLambda-SHA384-ECDSA platform
- **Container signing**: Notation format, COSE signatures
- **Enforcement**: Attach signing configuration to Lambda function → only signed code runs
- **Exam key**: Provides cryptographic proof of code origin and integrity

### ECR Image Scanning
| Feature | Basic Scanning | Enhanced Scanning |
|---------|---------------|------------------|
| Engine | Clair | Amazon Inspector |
| OS packages | ✓ | ✓ |
| Language packages | ✗ | ✓ (npm, pip, jar, go) |
| Continuous rescan | ✗ | ✓ (new CVEs) |
| Timing | On push | On push + continuous |
| **Exam tip** | OS only | OS + application deps |

### ECR Image Signatures
- **Signing**: AWS Signer creates signatures for container images
- **Verification**: Admission controllers check signatures before deployment
- **Format**: Notation/COSE standard signatures
- **Exam scenario**: Require signed images in prod → Signer + verification policy

## SBOMs (Software Bill of Materials)
- **Purpose**: Track dependencies, licenses, supply chain risk
- **Formats**: SPDX, CycloneDX (industry standards)
- **AWS native**: No built-in SBOM generation in ECR/Inspector as of 2026
- **Recommended**: Integrate Syft/sbom-tool in CI/CD, store in S3, tag images with SBOM locations
- **Exam tip**: SBOM = dependency manifest for supply chain visibility

## Exam Power Tips

1. **SCP inheritance**: All parent SCPs apply to child OUs/accounts, Deny always wins
2. **Config timing**: Detective control with 10-15 min lag (not real-time)
3. **Preventive vs Detective**:
   - Preventive: SCPs, IAM policies, signature verification → BLOCKS action
   - Detective: Config rules, GuardDuty, Security Hub → DETECTS after action
4. **CodeArtifact origin controls**: ALLOW_UPSTREAM = only vetted upstream sources
5. **Enhanced scanning**: Choose when question mentions language packages (npm, pip)
6. **Control Tower**: Account Factory = automated provisioning, mandatory guardrails = cannot disable
7. **Conformance packs**: Deploy organization-wide for framework compliance at scale
8. **Signer**: Use for Lambda functions AND container images (different formats)
