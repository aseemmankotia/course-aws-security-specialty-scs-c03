# Chapter 4 Lab: IAM Policy Evaluation Logic: The Decision Flow

**Estimated time:** 25 minutes

## Exercise

Create IAM user with AdministratorAccess identity policy, attach permissions boundary limiting to EC2 and S3, attempt RDS API call and trace deny through CloudTrail, use IAM Policy Simulator to visualize evaluation

## Key takeaway

IAM evaluation is intersection-based: effective permissions = identity policy ∩ permissions boundary ∩ SCP, with explicit deny in any policy type overriding all allows; cross-account access requires explicit allow in both source identity policy and target resource policy.
