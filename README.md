# RAG Security Scanner
 
**Detecting PII and Prompt Injection in AI Retrieval Pipelines**
 
Author: Billy Carrie | Framework: OWASP LLM Top 10 | Updated June 2026
 
---
 
Companies are increasing their use of Retrieval-Augmented Generation (RAG) to power AI assistants by pulling documents from internal knowledge bases and inserting them directly into model prompts. The problem is that retrieved documents are untrusted input. They can contain Personally Identifiable Information like SSNs, salaries, and email addresses, or they can contain hidden instructions designed to hijack the model's behavior. To protect critical company data from being leaked or stolen, I built a pre-prompt scanner that sits between the retrieval layer and the model, detecting and neutralizing both threats before they reach Anthropic's AI context window. The same principle that drives input validation in traditional AppSec applies here as well, never pass raw external content to a model without inspection.
 
---
 
## Before vs. After: Vulnerable Pipeline vs. Secured Pipeline
 
| Category | Vulnerable Pipeline | Secured Pipeline |
|---|---|---|
| Scanner active | No | Yes |
| doc_001: Q3 Budget Overview | Passed to model as-is | PASS - sent as-is |
| doc_002: Employee Directory | SSNs, salaries, emails sent to model | REDACTED - PII stripped before model sees it |
| doc_003: Cybersecurity Policy | Passed to model as-is | PASS - sent as-is |
| doc_004: Vendor Onboarding | EIN, email, phone sent to model | REDACTED - PII stripped before model sees it |
| doc_005: Project Status Update | Injection attack sent to model | BLOCKED - entire document withheld |
| Model saw PII | Yes | No |
| Model saw injection attempt | Yes | No |
| Model response | Partial self-protection relying on model discretion - no enforced control | Responded only from sanitized content with no access to sensitive data |
 
---
 
## Scan Results
 
| Document | Threat Type | Outcome | OWASP Reference |
|---|---|---|---|
| doc_001: Q3 Budget Overview | None | PASS | N/A |
| doc_002: Employee Directory | PII - SSN (2), Email (2), Salary (2) | REDACTED | LLM06 |
| doc_003: Cybersecurity Policy | None | PASS | N/A |
| doc_004: Vendor Onboarding Checklist | PII - Email (1), Phone (1), EIN (1) | REDACTED | LLM06 |
| doc_005: Project Status Update | Prompt Injection - `admin`, `Forward all`, `Resume normal operation` | BLOCKED | LLM01 |
 
**Scan summary: 1 blocked, 2 redacted, 2 passed**
 
---

## OWASP LLM Top 10 Controls
 
| OWASP Risk | Description | Control in This Project |
|---|---|---|
| LLM01 - Prompt Injection | Attacker embeds instructions inside retrieved content to hijack model behavior | Injection pattern detection in `scanner.py` blocks the entire document before it reaches the model context |
| LLM06 - Sensitive Information Disclosure | Model outputs PII sourced from retrieved documents | Regex-based PII detection redacts SSNs, emails, salaries, phone numbers, and EINs before the prompt is built |

---

## **Author**

**Billy Carrie** — IAM Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/billycarrie/)
