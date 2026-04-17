# Skill: Skill Auditor

## Contents

- [What it does](#what-it-does)
- [When to use it](#when-to-use-it)
- [Spec](#spec)
- [Scoring Model](#scoring-model)
- [Design Notes](#design-notes)

---

## What it does

Audits a team's skills across two lenses: **context bloating risk** (connector
scope, volume, audience amplification) and **best practices** (trigger quality,
skill body hygiene, naming, ownership). Each skill receives a context bloat
score and a best practices checklist, then the report surfaces Quick Wins
ranked across both dimensions.

Every finding includes a concrete BEFORE/AFTER fix ready to paste. Designed
to be run periodically as skills accumulate, and before any skill is promoted
to org-wide sharing.

**Skill file:** [skill_auditor.md](./skill_auditor.md)

---

## When to use it

- Periodic team skill health checks
- Before promoting a skill from team → org-wide
- When users report that "it gets confused after a few turns"
- After onboarding a new connector or data source
- When the skill library grows beyond ~10 skills and nobody knows which ones are expensive

---

## Spec

```yaml
name: skill_auditor
version: v1
description: >
  Audits team skills for context bloating risks and produces a prioritized
  remediation report with concrete before/after fixes. Activate when someone
  asks to audit skills, flag bloated skills, review context costs, or pastes
  skill descriptions for review.
audience: organization
connectors: []
context_weight: light
trigger: >
  "audit our skills" / "which skills are bloating context" /
  "review team skills" / "run the anti-context-bloating check" /
  "flag skills that need fixing" / "which skills cost the most"
inputs:
  - name: skills
    type: list[skill_description]
    description: >
      Skill descriptions, names + connector lists, or workflow descriptions.
      Provided by the user in conversation.
output:
  returns: >
    Structured audit report: summary table, per-skill breakdown with score
    and BEFORE/AFTER fix, terminology leak check for shared skills,
    quick wins list, and structural recommendations.
  does_not_return: >
    Vague warnings without fixes, or passes for connectors with no scope filter.
on_failure: >
  If no skills are provided, ask the user to paste descriptions or list
  skill names and connectors. Do not proceed without at least one skill.
constraints:
  - Never pass a connector with no scope filter.
  - Always produce a BEFORE/AFTER fix for every finding.
  - Always end with the Quick Wins section.
  - Flag org-wide CRITICAL skills first.
```

---

## Scoring Model

| Dimension | Max points | Key signal |
|---|---|---|
| Connector scope risk | 3 per connector | Missing project/space, result limit, or time window |
| Connector volume risk | 3 | 3+ connectors or chained skills |
| Audience amplification | 3 | Org-wide multiplies the cost of every flaw |

A skill scored 9+ is CRITICAL regardless of how well it is otherwise written.
Audience amplification is intentional: a flawed personal skill is a minor
problem; the same skill shared org-wide is a systemic one.

---

## Design Notes

**Why BEFORE/AFTER is mandatory**

Audit reports that only name problems get filed and forgotten. Forcing a
BEFORE/AFTER fix for every finding means the output is immediately actionable.
The user can copy the corrected version without doing additional work.

**Why the Quick Wins section is always last**

Teams rarely implement every recommendation from an audit. The Quick Wins
section gives them a clear, prioritized starting point. Placing it last
ensures they've seen the full picture before acting on shortcuts.

**Why audience amplification is a scoring dimension**

Context bloat from a personal skill is a personal problem. The same flaw in
an org-wide skill multiplies across every user and every conversation. Scoring
audience separately ensures org-wide skills are held to a higher standard even
when their connector scope looks clean.

**Why the terminology leak check exists**

Skills written by one team often embed internal jargon, project names, or
assumed context that breaks when the skill is shared with other teams. This
check surfaces those assumptions before they cause confusion at scale.
