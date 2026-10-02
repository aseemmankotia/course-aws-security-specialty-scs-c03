# Chapter 3: Infrastructure Security Cheat Sheet

## Security Groups vs NACLs

| Feature | Security Groups | NACLs |
|---------|----------------|-------|
| **State** | Stateful (return traffic auto-allowed) | Stateless (must explicitly allow both directions) |
| **Rule evaluation** | All allow rules evaluated, most permissive wins | Rules evaluated in number order, first match wins |
| **Default** | Deny all inbound, allow all outbound | Allow all inbound and outbound |
| **Rule capacity** | 60 inbound + 60 outbound rules per group | 20 rules per NACL (can request increase to 40) |
| **Scope** | Instance-level (ENI) | Subnet-level |
| **Exam trap** | Can't deny - only allow. Multiple allows = cumulative | Must allow ephemeral ports (1024-65535) for return traffic |

## AWS Network Firewall

| Component | Purpose | Exam Key Point |
|-----------|---------|----------------|
| **Firewall endpoint** | Traffic inspection point in subnet | Max 100 Gbps per endpoint; deploy in dedicated 'inspection subnet' |
| **Stateful rule groups** | Domain filtering, IPS, protocol inspection | Strict ordering = first match wins; Default = most restrictive wins |
| **Stateless rule groups** | 5-tuple filtering (src/dst IP:port, protocol) | Processed before stateful; use for coarse filtering |
| **Suricata IPS** | Compatible with Suricata rules | Import open-source rules (ET Open, etc.) |
| **Rule evaluation order** | Stateless → Stateful (strict priority order) → Default action | |

**Route table pattern:**
Public subnet → IGW route via firewall endpoint → Firewall → IGW
Private subnet → NAT/Internet route via firewall endpoint → Firewall → NAT/IGW

## AWS Shield

| Feature | Standard | Advanced |
|---------|----------|----------|
| **Cost** | Free | $3,000/month + data transfer |
| **Protection** | L3/L4 network DDoS | L3/L4 + application layer (L7) |
| **Supported resources** | All AWS resources | CloudFront, Route 53, ALB, NLB, EIP |
| **DDoS Response Team** | No | 24/7 DRT support |
| **Cost protection** | No | DDoS-related scaling charges refunded |
| **Health-based detection** | No | Automatic mitigation based on app health |
| **Exam scenario** | Default protection | Application layer attack, need DRT, cost guarantee |

## AWS WAF

**Rule types:**
- **Regular rules**: Match conditions (IP sets, geo, headers, body)
- **Rate-based rules**: Block IPs exceeding request threshold (5-minute fixed window)
- **Managed rule groups**: AWS (free) or Marketplace (paid)

**Actions:** ALLOW, BLOCK, COUNT, CAPTCHA

**Managed rule groups (AWS):**
- Core Rule Set (OWASP Top 10)
- Known Bad Inputs (CVE patterns)
- Bot Control (bot detection/mitigation)
- IP Reputation List (malicious IPs)

**Rate limit calculation:**
Legitimate traffic: X requests/hour ÷ 12 = Y requests per 5 minutes → set threshold above Y with headroom

## VPC Endpoints & PrivateLink

| Type | Use Case | Exam Scenario |
|------|----------|---------------|
| **Gateway endpoint** | S3, DynamoDB | Free; route table entry; private subnet access to S3/DDB |
| **Interface endpoint** | Most AWS services | Powered by PrivateLink; costs per hour + data; ENI in subnet |
| **PrivateLink (custom)** | Expose your service to other accounts | Provider creates endpoint SERVICE with NLB; consumer creates interface endpoint |

**Exam trap:** PrivateLink works only AWS-to-AWS. For third-party SaaS not on AWS, use Direct Connect or VPN.

## Amazon Bedrock Guardrails (NEW C03)

| Feature | Purpose | Configuration |
|---------|---------|---------------|
| **Content filters** | Block harmful content | Categories: HATE, INSULTS, SEXUAL, VIOLENCE, MISCONDUCT, PROMPT_ATTACK. Thresholds: NONE, LOW, MEDIUM, HIGH |
| **Denied topics** | Block entire topics semantically | Define topics (e.g., 'competitor products'); uses semantic matching, not keywords |
| **Word filters** | Block specific words/phrases | Exact match or regex; use for precise term blocking (profanity, internal codes) |
| **Sensitive info filters** | PII redaction | Entity types: NAME, SSN, EMAIL, PHONE, etc. Actions: BLOCK (reject) or ANONYMIZE (replace) |
| **Contextual grounding** | Prevent hallucinations | Verify responses against source documents; thresholds for factual accuracy |

**OWASP LLM Top 10 (exam-relevant):**
1. **Prompt Injection** → Mitigate with denied topics (semantic detection)
2. **Insecure Output Handling** → Mitigate with content filters + sensitive info filters
3. **Training Data Poisoning** → Not configurable in Bedrock (AWS managed)
6. **Sensitive Information Disclosure** → Mitigate with PII filters (BLOCK or ANONYMIZE)

**Guardrail evaluation:**
- Applied to prompts (user input) and/or model responses (output)
- Configure separately: input guardrails vs output guardrails

## Exam Gotchas

1. **Security group chaining doesn't cascade rules**: If SG-A allows SG-B and SG-B allows 0.0.0.0/0, only instances with SG-B get internet access. SG-A instances don't inherit the 0.0.0.0/0 rule.

2. **Network Firewall strict ordering**: Priority 1 PASS rule for *.example.com beats Priority 2 DROP rule for admin.example.com. First match wins.

3. **WAF rate-based rules**: 5-minute window is fixed. Can't use 1-minute or 1-hour windows.

4. **Bedrock denied topics vs word filters**: Denied topics = semantic (flexible, catches variations). Word filters = exact match (precise but bypassable).

5. **Shield Advanced cost**: Question may give attack scenario costing $50K in scaling charges. Shield Advanced ($3K/month) includes cost protection = ROI justification.

6. **NACLs and ephemeral ports**: Stateless = must allow 1024-65535 outbound for return traffic from internet requests. Security groups auto-allow return traffic.

## CLI Examples

```bash
# Create rate-based WAF rule (100 req per 5 min)
aws wafv2 create-web-acl --scope REGIONAL \
  --rules '[{"Name":"RateLimit","Priority":1,"Statement":{"RateBasedStatement":{"Limit":100,"AggregateKeyType":"IP"}},"Action":{"Block":{}}}]'

# Create Bedrock Guardrail with PII filters
aws bedrock create-guardrail --name MyGuardrail \
  --sensitive-information-policy-config '{"piiEntitiesConfig":[{"type":"EMAIL","action":"ANONYMIZE"},{"type":"SSN","action":"BLOCK"}]}'
```
