# Chapter 6 Lab: IAM Troubleshooting: Denies, Access Analyzer, and Least Privilege

**Estimated time:** 22 minutes

## Exercise

Enable IAM Access Analyzer in organization, review finding showing S3 bucket shared with external account, generate least-privilege policy from CloudTrail logs, use Policy Simulator to verify permissions before deployment

## Key takeaway

IAM Access Analyzer is regional and must be enabled per region; CloudTrail errorMessage explicitly identifies which policy type caused the deny - read the message carefully to identify SCP vs boundary vs identity policy issues.
