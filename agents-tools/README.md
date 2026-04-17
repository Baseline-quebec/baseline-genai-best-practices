# Agents & Tools

Agentic AI — systems that plan, use tools, and take sequences of actions — is where GenAI gets powerful and where things can go wrong fast.

## Contents

- [Agentic Design Principles](#agentic-design-principles)
- [Patterns](./patterns/) — Proven agentic architectures
- [Examples](./examples/) — Annotated real-world implementations
- [Tool Design Guidelines](#tool-design-guidelines)

---

## Agentic Design Principles

### 1. Minimal footprint

Agents should request only the permissions they need, store only what they need, and prefer reversible actions over irreversible ones.

> Before giving an agent a capability, ask: "What's the worst case if this goes wrong?"

### 2. Prefer reversible actions

```
❌ delete_record(id)            → irreversible
✅ archive_record(id)           → recoverable
✅ delete_record(id, dry_run=True)  → confirm before acting
```

### 3. Confirm before consequential actions

For actions with significant side effects — sending emails, modifying data, making purchases — require explicit confirmation.

### 4. Fail loudly, not silently

When an agent encounters an unexpected state, stop and surface the issue. Never guess and continue.

### 5. Log every action

Every tool call and its result should be logged. You need this for debugging, auditing, and improving the system.

---

## Tool Design Guidelines

**Be specific about inputs and outputs**
```python
# Too vague
def search(query: str) -> str: ...

# Better
def search_knowledge_base(
    query: str,
    max_results: int = 5,
    min_relevance_score: float = 0.7
) -> list[SearchResult]: ...
```

**Return structured, parseable outputs** — models handle JSON better than free-form text.

**Include error states in the schema**
```python
class ToolResult(BaseModel):
    success: bool
    data: dict | None
    error: str | None
```

**Make tools idempotent** — calling the same tool twice should not cause duplicate side effects.

**Rate-limit and timeout every tool** — never let an agent make unlimited calls.

---

## Red Flags

- Agent has write access to production systems without human review
- No logging of tool calls
- Irreversible tools with no confirmation step
- No maximum iteration limit
- Agent can modify its own instructions
