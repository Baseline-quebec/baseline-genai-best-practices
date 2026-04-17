# Skill Library

Each skill lives in its own subfolder. The folder name is the skill name in snake_case.

```
library/
├── search_knowledge_base/
│   └── README.md
├── summarize_thread/
│   └── README.md
└── ...
```

Each skill `.md` file must start with YAML frontmatter:

```yaml
---
name: skill_name
version: v1
description: One-line description.
audience: self | team | organization
connectors: [list of connectors, or empty]
context_weight: light | moderate | heavy
trigger: "Sample trigger phrase(s)"
author: team-or-person
---
```

Each `README.md` should cover:

- **What it does** — one-paragraph description
- **When to use it** — trigger conditions
- **Skill Prompt** — the full prompt text to paste into your assistant platform
- **Spec** — full YAML following the [skill specification format](../README.md#skill-specification-format)
- **Design notes** — non-obvious decisions, tradeoffs, known limitations

## Index

| Skill | Description |
|---|---|
| [skill_builder](./skill_builder/) | Meta-skill that guides teams through creating token-efficient, well-scoped skills |
| [skill_auditor](./skill_auditor/) | Audits team skills for context bloating and best practices issues; produces a scored, actionable remediation report |
