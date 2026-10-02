# Chapter 3 Lab: Infrastructure Security: Network Defenses & AI Guardrails

**Estimated time:** 35 minutes

## Exercise

Deploy Network Firewall with Suricata rules blocking C2 domains, configure WAF WebACL with rate-based rule on ALB, attach Bedrock Guardrail to prevent prompt injection and PII leakage in generative AI chatbot

## Key takeaway

VPC gateway endpoints keep S3/DynamoDB traffic private without NAT/IGW; Bedrock Guardrails denied topics prevent prompt injection (OWASP LLM01) while PII filters prevent data leakage (OWASP LLM06).
