# Skill: Skill Builder

## What it does

A meta-skill that guides teams through the full skill lifecycle: intent capture, drafting, test case generation, iterative refinement, and publishing to the shared library.

Aligned with the Claude Skill Creator pattern but kept platform-agnostic. Key additions over a vanilla skill builder: step-gated information gathering, connector scoping enforcement, a context weight estimate (🟢/🟡/🔴) with automatic lighter-version proposals for heavy skills, and team sharing conventions (naming, versioning, ownership).

**Skill file:** [skill_builder.md](./skill_builder.md)
**Org conventions:** [resources/references/org_conventions.md](./resources/references/org_conventions.md) — fill this in before using the skill builder with your team

---

## When to use it

Trigger this skill when someone:

- Asks to create, write, or design a skill
- Describes a repetitive task they want to automate
- Says "I want a skill for..." or "how do I automate..."
- Asks how to structure a skill for team sharing

---

## Spec

```yaml
name: skill_builder
description: >
  Guides users through structured information gathering and produces a
  complete skill package: description, name, context weight estimate,
  and trigger validation tests. Enforces scope constraints on all connectors
  to prevent context bloat.
trigger: >
  When someone asks to create, write, or design a skill; describes a
  repetitive task to automate; or says "I want a skill for..." or
  "how do I automate...".
inputs:
  - name: task_description
    type: string
    description: The repetitive task the user wants to automate.
  - name: trigger_phrase
    type: string
    description: A sample phrase the user would type to invoke the skill.
  - name: connectors
    type: list[string]
    description: External systems the skill needs to reach.
  - name: output_format
    type: string
    description: The expected output (summary, ticket, message, page, etc.)
  - name: audience
    type: enum[self, team, organization]
    description: Who will use the skill.
output:
  returns: >
    Four deliverables: skill description, 2 name options with recommendation,
    context weight estimate (🟢/🟡/🔴) with lighter alternative if 🔴,
    and 5 trigger validation prompts.
  does_not_return: >
    Partially specified skills, skills with unscoped connectors,
    or skills assessed as 🔴 without a proposed lighter version.
on_failure: >
  If the user cannot answer a scoping question (e.g. which Confluence space),
  do not proceed. Ask them to consult their admin or temporarily remove
  the connector from the skill.
constraints:
  - Never produce a connector reference without a scope filter.
  - Always produce the context weight estimate before finalizing the description.
  - If audience is "organization", prefix the skill name with "shared-".
  - Always include the v1 refinement reminder in the output.
```

---

## Design Notes

**Why information gathering is step-gated**

Asking all questions at once leads to vague or incomplete answers. One question at a time produces the specific scoping needed to write accurate connector constraints and a reliable trigger condition.

**Why context weight is a first-class deliverable**

Most skill quality problems manifest as context bloat after several turns. By making the weight estimate visible at creation time — and by requiring a lighter alternative for 🔴 skills — teams catch the problem before it reaches users.

**Why the lighter-version proposal is automatic**

If the builder only warns about heavy skills without offering a fix, teams either ignore the warning or abandon the skill entirely. An automatic lighter proposal keeps momentum while enforcing good practice.

**Why v1 naming is enforced**

Versioning in the name signals to all users that the skill will change. It also prevents premature stabilization — teams are more willing to iterate a `v1` than something named without a version suffix.

**Adapting to your platform**

The connector list in Step 1 should be updated to match the tools your organization actually has. Keep the list bounded — offering too many options leads to over-scoped skills.
