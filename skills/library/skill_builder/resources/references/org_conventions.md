# Organization Conventions Reference

This file is loaded by the skill builder when drafting or reviewing a skill.
Fill in each section for your organization. Fields marked [REQUIRED] must be
completed before the skill builder can enforce your conventions accurately.
Fields marked [OPTIONAL] improve output quality but are not blocking.

---

## Naming Convention [REQUIRED]

Pattern: `[team]-[function]-[main-tool]-v[n]`

**Registered team prefixes:**

| Prefix | Team | Owner |
|---|---|---|
| `gtm` | Go-to-Market | @name-or-handle |
| `eng` | Engineering | @name-or-handle |
| `product` | Product | @name-or-handle |
| `shared` | Cross-team / org-wide | @name-or-handle |
| *(add rows)* | | |

**Rules:**
- Always use kebab-case
- Always include a version suffix (`-v1`, `-v2`, …)
- Org-wide skills must use the `shared-` prefix
- Function token should be a verb or noun (e.g. `brief`, `review`, `recap`, `spec`)

---

## Teams and Ownership [REQUIRED]

List the teams that create or consume skills, and the person responsible for
approving org-wide skills.

| Team | Skill owner | Escalation contact |
|---|---|---|
| Go-to-Market | | |
| Engineering | | |
| Product | | |
| *(add rows)* | | |

**Skill library maintainer (org-wide approvals):** *(name or handle)*

---

## Available Connectors [REQUIRED]

List every connector available in your environment. For each one, define the
default scope constraints the skill builder should enforce.

| Connector | Default scope | Max results | Default time window |
|---|---|---|---|
| *(e.g. Jira)* | *(e.g. team project only)* | *(e.g. 10)* | *(e.g. last 30 days)* |
| *(e.g. Confluence)* | *(e.g. team space only)* | *(e.g. 5 pages)* | *(e.g. last 60 days)* |
| *(e.g. Slack)* | *(e.g. designated channel only)* | *(e.g. 20 messages)* | *(e.g. last 7 days)* |
| *(e.g. CRM)* | *(e.g. assigned accounts only)* | *(e.g. 5 records)* | *(e.g. current quarter)* |

**Hard rules:**
- No connector may be used without a project/space/channel scope
- No connector may be used without a result limit
- Skills rated 🔴 HEAVY require explicit approval before org-wide publishing

---

## Approved Filters [OPTIONAL]

List the standard filter values teams should use. This helps the skill builder
suggest consistent filters rather than letting each team invent their own.

**Jira:**
- Approved projects: *(list)*
- Approved labels: *(list)*
- Approved issue types: *(list)*

**Confluence:**
- Approved spaces: *(list)*
- Page types in scope: *(e.g. decisions, specs, runbooks)*

**Slack:**
- Approved channels for skill use: *(list)*

**CRM / other:**
- *(add as needed)*

---

## Context Weight Thresholds [OPTIONAL]

Override the default thresholds if your environment has different cost
sensitivity.

| Flag | Default score range | Your override |
|---|---|---|
| 🟢 HEALTHY | 3–4 | |
| 🟡 WATCH | 5–6 | |
| 🟠 AT RISK | 7–8 | |
| 🔴 CRITICAL | 9+ | |

---

## Audience Tiers [OPTIONAL]

Define what each audience level means in your organization.

| Tier | Meaning | Approval required? |
|---|---|---|
| `self` | Creator only | No |
| `team` | One named team | Team owner |
| `organization` | All teams / all users | Library maintainer |

---

## Terminology Glossary [OPTIONAL]

List internal terms that should be avoided in skill bodies to prevent
terminology leaks when skills are shared across teams.

| Internal term | Safe alternative |
|---|---|
| *(e.g. "the pod")* | *(e.g. "the team")* |
| *(e.g. project codenames)* | *(e.g. product name)* |
| *(add rows)* | |

---

## Publishing Checklist [OPTIONAL]

Steps required before a skill is added to the shared library in your org.

- [ ] *(e.g. Reviewed by team owner)*
- [ ] *(e.g. Tested against 5 real prompts)*
- [ ] *(e.g. Context weight assessed and documented)*
- [ ] *(e.g. Approved by library maintainer for org-wide skills)*
