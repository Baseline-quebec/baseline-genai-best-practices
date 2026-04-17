# Governance & Ethics

Building AI responsibly isn't a blocker — it's a competitive advantage. Organizations that get governance right ship faster, with fewer incidents, and with more stakeholder trust.

## Contents

- [Risk Assessment Framework](./frameworks/risk-assessment.md)
- [Pre-Launch Checklist](./checklists/pre-launch.md)
- [Data Governance](./checklists/data-governance.md)
- [Key Principles](#key-principles)

---

## Key Principles

### 1. Know what your model can and can't do

Every model has failure modes. Know them before your users discover them.

- Hallucination rates vary by domain — test in your specific context
- Models are not up-to-date — build in date awareness or retrieval
- Performance drops on out-of-distribution inputs — test edge cases

### 2. Keep humans in the loop for high-stakes decisions

High-stakes = human review required:
- Hiring or performance evaluation
- Medical or legal advice
- Credit or financial decisions
- Content moderation at scale

### 3. Be transparent with users

Users have a right to know they're interacting with AI. Disclose it.

### 4. Log everything you'd want in an incident post-mortem

Minimum logging: input prompt, model response, model version, timestamp, user action taken.

### 5. Define "unacceptable output" before you ship

It's much harder to define this after an incident. Answer before launch:
- What outputs would embarrass us publicly?
- What outputs could harm a user?
- What outputs could expose us to legal liability?

---

## Useful External Frameworks

| Framework | Best for |
|---|---|
| [EU AI Act](https://artificialintelligenceact.eu/) | Regulatory compliance (EU market) |
| [NIST AI RMF](https://www.nist.gov/system/files/documents/2023/01/26/AI%20RMF%201.0.pdf) | Risk management at org level |
| [Montreal AI Ethics Institute](https://montrealethics.ai/) | Practical ethics guidance |
