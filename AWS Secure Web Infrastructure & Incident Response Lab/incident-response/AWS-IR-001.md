---

## 2. Incident Report File: `incident-response/AWS-IR-001.md`

```markdown
# Incident Response Report: AWS-IR-001

**Incident ID:** AWS-IR-001  
**Severity:** Medium  
**Status:** Resolved / Remediated  
**Target Resource:** `WEB-SG` (Security Group)  
**Region:** `eu-west-1`  

---

### 1. Incident Summary
An unauthorized security group rule modification was detected on security group `WEB-SG`. Ingress rule permitting unrestricted public access (`0.0.0.0/0`) on port TCP 8080 was injected, breaking the baseline least-privilege network architecture.

---

### 2. Detection & Forensic Analysis
The incident was audited by querying centralized management events recorded in AWS CloudTrail.

#### Query Executed:
```bash
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AuthorizeSecurityGroupIngress \
  --region eu-west-1 \
  --max-results 5
