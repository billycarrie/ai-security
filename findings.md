# RAG Security Scanner — Findings Report
Author: Billy Carrie | Framework: OWASP LLM Top 10 | Updated June 2026

## Scan Results

| Document | Threat Type | Outcome | OWASP Reference |
|----------|------------|---------|-----------------|
| doc_001: Q3 Budget | None | PASS | N/A |
| doc_002: Employee Directory | PII (SSN, salary, email) | REDACTED | LLM06 |
| doc_003: IT Security Policy | None | PASS | N/A |
| doc_004: Vendor Onboarding | PII (EIN, email, phone) | REDACTED | LLM06 |
| doc_005: Project Status | Prompt Injection | BLOCKED | LLM01 |

## What the Vulnerable Pipeline Did
The model received all 5 raw documents with no filtering and responded directly to the query. While it avoided sharing SSNs and salaries on its own, this was the AI's discretion, not enforced security. That means there was nothing stopping sensitive data from being exposed.

## What the Secured Pipeline Did
The scanner blocked the Project Status Update document entirely due to detected injection patterns (admin, Forward all, Resume normal operation), and redacted SSNs, emails, salaries, and an EIN from the Employee Directory and Vendor Onboarding documents before anything reached the model. As a result, the secured model response only referenced information from the sanitized documents and explicitly acknowledged that sensitive employee details had been redacted.

## Key Takeaway
Without a pre prompt scanner, RAG pipelines are a direct channel for both PII
leakage and prompt injection. The scanner sits between retrieval and the model,
enforcing the same principle as input validation in traditional AppSec, which is
never trust content from an external source.
