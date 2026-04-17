# Prompt Engineering

Writing good prompts is a craft. This section covers the patterns and principles that consistently produce better outputs across models and use cases.

## Contents

- [Core Principles](#core-principles)
- [Patterns](./patterns/) — Reusable prompt structures
- [Examples](./examples/) — Annotated real-world examples
- [Anti-Patterns](#anti-patterns)

---

## Core Principles

### 1. Be explicit about the output format

Models don't read minds. If you want a JSON object, a bulleted list, or a 3-paragraph essay — say so.

```
❌ "Summarize this document."
✅ "Summarize this document in 3 bullet points. Each bullet should be one sentence."
```

### 2. Give context, not just instructions

A model with context outperforms a model without it, even with the same instruction.

```
❌ "Write a follow-up email."
✅ "Write a follow-up email to a potential client who attended our AI workshop last Tuesday
    but hasn't responded to our initial outreach. Tone: friendly but professional.
    Goal: book a 30-minute discovery call. Keep it under 150 words."
```

### 3. Use examples (few-shot)

One good example is worth 10 lines of instructions.

```
Classify the sentiment of customer feedback.

Examples:
- "The onboarding was painful" → negative
- "Works as expected" → neutral
- "This saved us hours every week" → positive

Now classify: "Setup took longer than I'd like but the results are solid."
```

### 4. Assign a role — but keep it grounded

```
❌ "You are an all-knowing oracle with infinite wisdom..."
✅ "You are a senior technical writer helping a software team document their API."
```

### 5. Ask for reasoning before the answer (chain-of-thought)

For complex tasks, asking the model to reason first reduces errors.

```
"Before giving your final recommendation, think through the trade-offs of each option."
```

### 6. Iterate, don't overengineer

Start simple. Add complexity only when simple prompts fail.

---

## Anti-Patterns

| Anti-pattern | Why it fails | Fix |
|---|---|---|
| Vague instructions | Model fills gaps with assumptions | Be explicit about format, length, tone |
| Over-relying on "be concise" | "Concise" means different things | Give a word/sentence count |
| No negative constraints | Model does what you didn't want | Add "Do not include..." |
| Ignoring temperature | Default settings aren't always right | Set low for factual, higher for creative |
| Single-shot complex tasks | Too much in one step fails | Break into chained prompts |
