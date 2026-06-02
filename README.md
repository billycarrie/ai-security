# RAG Security Scanner
 
**Detecting PII and Prompt Injection in AI Retrieval Pipelines**
 
Author: Billy Carrie | Framework: OWASP LLM Top 10 | Updated June 2026
 
---
 
Companies are increasing their use of Retrieval-Augmented Generation (RAG) to power AI assistants by pulling documents from internal knowledge bases and inserting them directly into model prompts. The problem is that retrieved documents are untrusted input. They can contain Personally Identifiable Information like SSNs, salaries, and email addresses, or they can contain hidden instructions designed to hijack the model's behavior. To protect critical company data from being leaked or stolen, I built a pre-prompt scanner that sits between the retrieval layer and the model, detecting and neutralizing both threats before they reach Anthropic's AI context window. The same principle that drives input validation in traditional AppSec applies here as well, never pass raw external content to a model without inspection.
 
---

## Artifacts
 
### Environment Setup
The virtual environment was created using pipenv with Python 3.11, isolating all project dependencies from the rest of the system. All three packages - anthropic, python-dotenv, and rich - installed successfully and are confirmed via pipenv graph.
 
![Terminal Virtual Environment Creation](docs/Terminal%20Virtual%20Environment%20Creation.png)
 
---
 
### Knowledge Base - Document Store
Five simulated enterprise documents were created in `documents.py` to represent a realistic internal knowledge base. Two contain PII, one contains a prompt injection attack embedded inside a normal-looking project update, and two are clean.
 
![Document Store](docs/Document_Store.png)
 
---
 
### PII Detection Patterns
The scanner uses pre-compiled regex patterns to detect six categories of sensitive data: SSN, Email, Phone, EIN, Salary, and Credit Card numbers. Pre-compiling with `re.compile()` improves performance when scanning large document sets.
 
![PII Patterns](docs/PII%20Patterns.png)
 
---
 
### Injection Detection Patterns
A separate pattern list targets phrases commonly used in prompt injection attacks - phrases like "ignore all previous instructions", "you are now in admin mode", and "resume normal operation". Any document matching these patterns is blocked entirely before reaching the model.
 
![Detection Injection Patterns](docs/Detection_Injection%20Patterns.png)
 
---
 
### Scan Document Function
The `scan_document()` function runs both checks on every retrieved document. Injection detection runs first - if a match is found, the entire document content is replaced with a blocked notice and nothing from that document reaches the model. PII redaction runs second, replacing sensitive values inline with labeled placeholders.
 
![Scan Document Object](docs/Scan_Document%20Object.png)
 
---
 
### Scanner Results - Vulnerable vs. Secured Pipeline
The terminal output shows both pipelines running against the same query. The vulnerable pipeline sends all five raw documents to the model with no filtering. The secured pipeline blocks doc_005, redacts PII from doc_002 and doc_004, and sends only sanitized content to the model. The difference in model responses shows exactly what the scanner controls.
 
![Scanner Results](docs/Scanner%20Results.png)
 
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

## OWASP LLM Top 10 Controls
 
| OWASP Risk | Description | Control in This Project |
|---|---|---|
| LLM01 - Prompt Injection | Attacker embeds instructions inside retrieved content to hijack model behavior | Injection pattern detection in `scanner.py` blocks the entire document before it reaches the model context |
| LLM06 - Sensitive Information Disclosure | Model outputs PII sourced from retrieved documents | Regex-based PII detection redacts SSNs, emails, salaries, phone numbers, and EINs before the prompt is built |

---

## **Author**

**Billy Carrie** — IAM Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/billycarrie/)
