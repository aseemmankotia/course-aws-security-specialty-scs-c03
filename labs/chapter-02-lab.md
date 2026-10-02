# Chapter 2 Lab: Incident Response: Automated Playbooks & Forensics

**Estimated time:** 30 minutes

## Exercise

Build Lambda function triggered by GuardDuty UnauthorizedAccess:EC2/SSHBruteForce finding that isolates instance by replacing security group, creates forensic EBS snapshot, and posts to SNS for analyst review

## Key takeaway

Deleting compromised credentials is insufficient - you must also attach explicit Deny policies to terminate active sessions; forensic evidence capture prioritizes memory and snapshots over stopping instances.
