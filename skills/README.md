# Skills

Skills are reusable, composable units of capability you give to an AI agent or assistant. Think of them as named procedures the model can learn to invoke, somewhere between a tool call and a full agent.

## Contents

- [What Is a Skill?](#what-is-a-skill)
- [Skill Anatomy](#skill-anatomy)
- [Skill Design Principles](#skill-design-principles)
- [Skill Specification Format](#skill-specification-format)
- [Connectors](#connectors)
- [Context Management](#context-management)
- [Sharing Skills in Teams](#sharing-skills-in-teams)
- [Library](./library/): Individual skill definitions ready to use or adapt
- [Anti-Patterns](#anti-patterns)

---

## What Is a Skill?

A **skill** bundles together:

1. A clear **trigger**: when should the model reach for this?
2. A defined **capability**: what does it do?
3. **Constraints**: what should it never do?
4. Optional **examples**: illustrative input/output pairs.

Skills differ from raw tools: a tool is a function signature; a skill is the intent, behavior, and guardrails that surround it.

---

## Skill Anatomy

Every skill lives in its own folder:

```
skill-name/
├── skill-name.md     ← required: YAML frontmatter + prompt body
├── README.md         ← required: context, spec, design notes
└── resources/        ← optional
    ├── scripts/      ← reusable code the skill can invoke
    ├── references/   ← documentation loaded into context as needed
    └── assets/       ← templates, examples, output files
```

### Progressive Disclosure

Skills load in three levels, from smallest to largest:

| Level | What loads | When |
|---|---|---|
| **Frontmatter** | `name` + `description` only | Always, every request |
| **Skill body** | Full prompt instructions | When the skill is active |
| **Bundled resources** | Scripts, references, assets | Only when explicitly needed |

This matters for cost and latency. The frontmatter is the only part that's
always in context, so it must be tight and complete enough to drive triggering.
The body is in context during active use; keep it under 500 lines. Resources
are unlimited in size but loaded on demand; scripts can run without being read
into context at all.

**If the skill body approaches 500 lines**, move supporting content to
`resources/references/` and reference it clearly from the main file with
guidance on when to load it.

### Writing the Description Field

The description drives triggering. It is the most important field in the
frontmatter. Write it to be specific and slightly forward-leaning:

```
❌ "Helps with GTM briefs."

✅ "Creates GTM briefs from CRM deal data. Use this skill whenever
    someone mentions a new deal, asks for a brief, or says 'prepare
    the launch doc' even if they don't use the word brief."
```

Under-triggering is a more common failure than over-triggering. Name the exact
phrases and situations that should activate the skill.

---

## Skill Design Principles

### 1. One skill, one responsibility

A skill should do exactly one thing well. If you find yourself writing "and also..." in the description, split it.

### 2. Name for the action, not the implementation

```
❌ run_search_api_v2
✅ search_knowledge_base
```

Names should read like verbs a human would say.

### 3. Write the trigger condition explicitly

The model needs to know *when* to use the skill, not just *what* it does.

```yaml
trigger: >
  When the user asks to look up a customer record,
  mentions an account ID, or says "find the account".
```

Without a clear trigger, the model either over-uses or ignores the skill.

### 4. Scope the output

Specify what the skill returns, and what it does *not* return. Unbounded output leads to bloated context and unpredictable behavior.

```yaml
output:
  returns: "Matching customer records (max 5), structured as JSON"
  does_not_return: "Raw database rows, PII fields, internal IDs"
```

### 5. Define failure behavior

What should the model do when the skill fails, returns nothing, or is unavailable?

```yaml
on_failure: "Inform the user the record was not found. Do not guess or hallucinate an answer."
```

### 6. Keep skills stateless when possible

Skills that depend on prior skill calls are fragile. Design each skill to operate on its own inputs, not on accumulated agent state.

---

## Skill Specification Format

Every skill file in the [library](./library/) must begin with YAML frontmatter, followed by the full prompt body:

```yaml
---
name: skill_name
version: v1
description: One-line description of what the skill does.
audience: self | team | organization
connectors: [list of connectors, or empty]
context_weight: light | moderate | heavy
trigger: "Sample trigger phrase(s)"
author: team-or-person
---
```

| Field | Purpose |
|---|---|
| `name` | Snake_case identifier, matches the folder and file name |
| `version` | Start at `v1`; increment when behavior changes meaningfully |
| `description` | One sentence, used in indexes and search |
| `audience` | Determines naming prefix: `organization` → `shared-` prefix required |
| `connectors` | Explicit list keeps context weight visible at a glance |
| `context_weight` | `light` / `moderate` / `heavy`, see [Context Management](#context-management) |
| `trigger` | Representative phrase(s) that should activate the skill |
| `author` | Team or person responsible for maintaining the skill |

Below the frontmatter, write the prompt body the model will receive.

A lightweight YAML spec for the skill's behavior (separate from the frontmatter) can also be included in the companion `README.md`:

```yaml
name: search_knowledge_base
description: Search internal documentation for answers to user questions.
trigger: >
  When the user asks a factual question that may be answered by internal docs,
  or says "look it up", "check the docs", or "is there a guide for".
inputs:
  - name: query
    type: string
    description: The user's question, paraphrased as a search query.
  - name: max_results
    type: integer
    default: 5
output:
  returns: "Up to max_results document excerpts with titles and relevance scores."
  does_not_return: "Full document text, file paths, or internal metadata."
on_failure: "Tell the user no results were found. Suggest they rephrase or contact support."
constraints:
  - Do not call this skill for questions the model can answer from training knowledge alone.
  - Do not expose document IDs or internal URLs to the user.
examples:
  - input: "How do I reset a user's password?"
    query: "password reset user instructions"
  - input: "What's our refund policy?"
    query: "refund policy"
```

---

## Connectors

A **connector** is the integration layer between a skill and an external system: an API, database, SaaS platform, or internal service. Where a skill defines *what* and *when*, a connector defines *how to reach the outside world*.

### Connectors vs. Skills vs. Tools

| Concept | What it is | Who defines it |
|---|---|---|
| Tool | A callable function signature | Developer |
| Connector | Auth + transport + retry logic for an external system | Developer / platform |
| Skill | Intent, trigger, constraints, and failure behavior around one capability | AI system designer |

A skill often depends on one or more connectors. The connector handles the plumbing; the skill handles the meaning.

### Connector Design Principles

**Isolate credentials from skill logic**

Connectors own authentication. Skills never receive raw API keys, tokens, or secrets; they receive a connector handle.

```
❌ skill receives: { api_key: "sk-...", endpoint: "https://..." }
✅ skill receives: connector_id = "crm_salesforce"
```

**Standardize error surfaces**

Every connector should normalize errors into a consistent shape before returning them to the skill layer.

```python
class ConnectorError(Exception):
    code: str        # "auth_failed" | "rate_limited" | "not_found" | "unavailable"
    retryable: bool
    message: str
```

Skills can then handle `rate_limited` and `unavailable` uniformly, regardless of which system they connect to.

**Enforce rate limits at the connector level**

Rate limiting belongs in the connector, not scattered across individual skills. A single connector manages its own quota so multiple skills sharing it don't exceed it collectively.

**Log at the connector boundary**

Log every outbound request and inbound response at the connector level: method, status, latency, and a redacted payload. This is your audit trail and your first debugging surface.

**Make connectors independently testable**

A connector should be usable and testable without any skill or agent context. If you can't call `connector.search("test")` in isolation, the boundary is wrong.

### Connector Specification Format

```yaml
name: crm_salesforce
description: Read-only access to Salesforce contact and opportunity records.
auth:
  method: oauth2
  scopes: [read_contacts, read_opportunities]
  credentials_source: vault/salesforce-prod
base_url: https://your-org.salesforce.com/services/data/v57.0
rate_limit:
  requests_per_minute: 100
  burst: 20
timeout_ms: 5000
retry:
  max_attempts: 3
  backoff: exponential
  retryable_codes: [429, 503]
error_mapping:
  401: auth_failed
  403: permission_denied
  404: not_found
  429: rate_limited
  5xx: unavailable
skills_using_this_connector:
  - search_crm_contacts
  - get_opportunity_details
```

### Red Flags

- A skill that constructs HTTP requests directly instead of going through a connector
- Credentials embedded in skill definitions or prompt templates
- No timeout on connector calls; one slow upstream will stall the entire agent
- Connectors that return raw upstream responses without normalizing errors
- Multiple skills independently implementing retry logic for the same system

---

## Context Management

Every token in the context window is a cost: in latency, in money, and in model attention. Skills and connectors are primary sources of context bloat when poorly designed. Managing context deliberately is not an optimization; it is a correctness concern.

### Why Context Bloat Happens

- Connector responses return full upstream payloads instead of projected fields
- Skills accumulate intermediate results that are never pruned
- Tool call history is retained verbatim across many turns
- System prompts grow unbounded as new instructions are appended
- Failed attempts and retries leave behind noise before the final answer

### Principles

**Return the minimum necessary**

Every skill output should answer one question: *what does the model need next?* Strip everything else before it enters context.

```
❌ Return full Salesforce contact object (47 fields)
✅ Return { name, email, account_name, last_activity_date }
```

Define a projection per skill, not per connector. The same connector may serve skills with very different information needs.

**Summarize, don't accumulate**

When a skill produces a long intermediate result that will be referenced later, summarize it before storing it in context. Preserve the summary, not the raw payload.

```
After search_knowledge_base returns 5 excerpts:
❌ Keep all 5 full excerpts in context for later reference
✅ Summarize: "Docs confirm password reset requires admin role. See article KB-4421."
```

**Prune completed steps**

Once a skill's output has been acted on, it no longer needs to live in full in the context. Either collapse it to a one-line fact or drop it entirely.

**Cap list results at the skill layer**

Never let a skill return an unbounded list. Set a hard `max_results` default and document it. If the user needs more, they should paginate explicitly, not receive 200 items at once.

```yaml
inputs:
  - name: max_results
    type: integer
    default: 5
    max: 20
```

**Keep system prompts surgically small**

Each skill added to a system prompt costs tokens on every single request. Write trigger conditions precisely so skills stay out of context when not relevant. Prefer loading skills dynamically over listing all of them upfront.

**Separate reasoning context from retrieval context**

Retrieved content (documents, records, search results) should be clearly delimited and explicitly bounded. Mixing retrieval results into the reasoning thread makes both harder to manage.

```
<retrieved_context max_tokens="2000">
  ...connector output here...
</retrieved_context>
```

### Context Budget Pattern

For complex multi-skill agents, assign an explicit token budget per zone:

| Zone | Purpose | Budget guidance |
|---|---|---|
| System prompt | Skills, persona, constraints | As small as possible; every token repeats |
| Retrieved context | Connector outputs, docs, records | Bounded per turn; summarize across turns |
| Conversation history | Prior user/assistant turns | Sliding window or summarized beyond N turns |
| Reasoning scratchpad | Chain-of-thought, plans | Collapse before final response |

The exact numbers depend on your model and use case, but the discipline of *having* a budget per zone prevents unbounded growth.

### Red Flags

- Skill output size is not documented or bounded
- The agent's context grows monotonically; nothing is ever pruned or summarized
- Connector responses are passed directly to the model without projection
- System prompt length increases every sprint as new instructions are added
- "It gets confused after a few turns": almost always a context accumulation problem

---

## Sharing Skills in Teams

A skill that works for one person is a shortcut. A skill that works for a team is infrastructure. The gap between the two is almost entirely about discipline around naming, ownership, and lifecycle.

### Naming Convention

Consistent names make skills discoverable and signal scope at a glance:

```
[team]-[function]-[main-tool]-v[n]

gtm-brief-hubspot-v1
eng-review-jira-v1
shared-weekly-recap-slack-v1
```

- Use `shared-` as a prefix for any skill used across multiple teams
- Always include a version suffix; it signals that the skill is expected to evolve
- Use snake_case for file and folder names; kebab-case for display names

### Ownership

Every shared skill must have a named owner: a person or team responsible for keeping it working and up to date.

```yaml
author: gtm-team
```

Without a named owner, shared skills quietly rot. No one updates the trigger when the process changes. No one tightens the connector scope when it starts bloating context.

**Rule:** if a skill has no owner, it should not be in the shared library.

### Versioning and Evolution

- Start every skill at `v1`. It is a signal, not a judgment; all skills start as drafts.
- Treat the first 5 real-world uses as a calibration period. Expect to revise.
- When behavior changes meaningfully, increment the version in the frontmatter and file name. Do not silently overwrite `v1`.
- Keep the previous version in the library until all users have migrated. Deprecate explicitly with a note in the README.

```
skill_builder/
├── skill_builder_v1.md     ← deprecated, kept for reference
├── skill_builder_v2.md     ← current
└── README.md
```

### Review Before Publishing

Before adding a skill to the shared library, it should pass a lightweight review:

- [ ] Frontmatter is complete (name, version, audience, connectors, context_weight, trigger, author)
- [ ] Every connector has a scope filter
- [ ] Context weight is assessed and documented
- [ ] Trigger condition is specific enough to avoid false positives
- [ ] Failure behavior is defined
- [ ] At least 3 trigger validation prompts have been tested

Use the [skill_builder](./library/skill_builder/) skill to produce all of the above automatically.

### Avoiding Skill Proliferation

Shared libraries drift toward bloat. A few rules to slow it down:

- **Prefer adapting an existing skill** over creating a new one. If a skill covers 80% of your use case, extend it rather than fork it.
- **One skill per distinct user intent.** If two skills share a trigger condition, they should be merged.
- **Retire skills that are no longer used.** A skill that hasn't been triggered in 90 days is a candidate for removal. Archive, don't delete, so the pattern is recoverable.

### Communication

When a shared skill changes, the author is responsible for notifying users:

- Changelog entry in the skill's README
- Announcement in the team channel for `🔴 HEAVY` skills or breaking changes
- Silent update acceptable only for `🟢 LIGHT` skills with no connector changes

---

## Anti-Patterns

**Skill sprawl**: dozens of single-use skills with overlapping triggers confuse the model. Consolidate skills that serve the same user intent.

**Missing trigger**: a skill with no trigger condition will be used inconsistently or not at all.

**Leaking internals**: skills that return raw database rows, internal IDs, or system paths expose implementation details to the user and to the model's context.

**Stateful skills**: skills that write to a shared scratchpad or rely on previous skill outputs create hidden dependencies. Prefer passing data explicitly through inputs.

**No failure path**: if the skill can fail (network error, no results, access denied), the model needs explicit instructions for what to do. Without them, it will improvise, poorly.

**Overlapping with tools**: a skill that just wraps a single tool call with no additional logic or guardrails adds friction without value. Either use the tool directly or add meaningful constraints.
