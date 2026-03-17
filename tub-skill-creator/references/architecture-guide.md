# Architecture-First Skill Design

This guide covers the core philosophy behind building great skills. Read this when designing a new skill's structure — before writing any instructions.

---

## Why Architecture Matters

Without a plan, the default behavior is to cram everything into one file. This produces skills that:

- Are too long (500+ lines) and waste Claude's attention
- Load unnecessary context for every interaction
- Lose focus mid-conversation and give generic output
- Can't be updated without rewriting everything

**The fix**: Design the structure first, then write the instructions.

Think of it like building a house. You don't start nailing boards together — you draw the blueprint. The blueprint shows which rooms exist, how they connect, and what goes in each one. A skill's architecture is the same thing.

---

## The Three Design Decisions

Every skill's architecture comes down to three questions:

### 1. Map the conversation flow

What questions will the skill ask? What paths can the user take?

Draw the flow before writing instructions:
- What does the skill need to know before it can help?
- Are there different paths depending on the user's answer?
- What's the order? (Some questions only make sense after others)

**Example — Lead Magnet Skill:**
```
User says "I need a lead magnet"
  → Ask: What do you sell? Who buys it? How do you get customers now?
  → Pick Type (Reveal Problem / Trial / One Step)
  → Pick Mechanism (Software / Information / Services / Physical)
  → Build the blueprint
  → Run quality checks
```

Each step depends on the previous one. You can't pick the type without understanding the business first.

### 2. Design the reference files

What knowledge does this skill need that Claude doesn't already have?

Split that knowledge into focused files — one topic per file:
- Each file should cover ONE clear subject
- Each file should be under 300 lines (aim for 100-200)
- Total across all files: under 1,000 lines
- If you need more, the skill is probably trying to do too much

**Example — Business Partner Skill:**
```
references/
├── voice-guide.md       ← How to sound like this person (phrases, rhythm, vocabulary)
├── frameworks.md        ← Core business frameworks (Value Equation, CLOSER, etc.)
├── strategic-thinking.md ← How to evaluate businesses and find constraints
└── playbooks.md         ← Tactical moves for specific situations
```

Each file serves a different purpose. Voice loads on every response. Frameworks load when solving offer problems. Playbooks load when giving tactical advice.

### 3. Define the SKILL.md's job

The SKILL.md is a **router and diagnostic engine** — not a knowledge dump.

It should contain:
- The diagnostic question flow (what to ask and when)
- Routing logic (which reference file to read based on what's happening)
- Voice/tone instructions (how to talk to the user)
- Output format specs (what the deliverable looks like)

Domain knowledge lives in `references/`, not in the SKILL.md.

**Think of it this way**: The SKILL.md is the conductor. The reference files are the musicians. The conductor doesn't play every instrument — it tells each musician when to play.

---

## Conditional Loading

This is the key technique that keeps skills fast and focused.

**Bad**: Load all reference files on every request (wastes context, loses focus)
**Bad**: Never load reference files (skill can't access knowledge it needs)
**Good**: Load the right file at the right time

### How to write conditional loading instructions

In your SKILL.md, be explicit about when to read each file:

```
When the user needs help choosing a type → read references/frameworks.md
When the user needs examples → read references/examples.md
When the user needs naming help → read references/naming.md
When reviewing quality → read references/quality-checks.md

DO NOT load all references at once. Only load what the current step needs.
```

This means Claude reads 1-2 files per step instead of 5-6 files upfront. The conversation stays focused and Claude has room to think.

---

## Distillation: Source Material to Reference Files

Raw content (books, transcripts, PDFs, presentations) is the *input* to building a skill — not the skill itself.

### The distillation process

1. **Read the raw content** and identify the 3-5 most important things
2. **Organize by topic** — each topic becomes one reference file
3. **Write lean, focused docs** — only include what the skill needs at runtime
4. **Cut the noise** — tangents, repetition, filler, context that doesn't help

### Why raw content destroys skills

Claude's context window is the text it can see at once. A few megabytes of transcripts can fill most of it, leaving almost no room for the conversation or the work Claude needs to produce.

Beyond size, raw content is noisy. Ten pages of distilled, organized reference material will always outperform hundreds of pages of raw transcripts.

**Think of it like onboarding a new team member.** You wouldn't hand them 600 pages of meeting notes. You'd give them:
- A one-pager on core frameworks
- A style guide for how you talk to clients
- A few great examples of the work

That's exactly what distilled reference files are.

---

## The "Partner Over Answer Engine" Pattern

Great skills don't monologue — they diagnose.

**Bad pattern (answer engine)**:
```
User: "I need help with my sales."
Skill: [dumps 500 words of generic sales advice]
```

**Good pattern (diagnostic partner)**:
```
User: "I need help with my sales."
Skill: "What are you selling, and what's your close rate right now?"
User: "Coaching packages, about 15%."
Skill: "What's your average deal size, and where are most deals dying?"
```

The skill asks sharp, specific questions first. Then it gives targeted advice based on what it learned. The output is better because it's personalized — not generic.

Build this pattern into every skill: **Diagnose → Understand → Route to the right reference → Respond.**

---

## Size Guardrails

These aren't arbitrary limits — they're based on how Claude actually processes information.

| What | Limit | Why |
|------|-------|-----|
| SKILL.md | Under 500 lines | Beyond this, Claude starts losing focus on the instructions |
| Each reference file | Under 300 lines | Keeps each topic digestible; forces you to distill |
| All references combined | Under 1,000 lines | Total context budget for domain knowledge |
| Source material | No limit | Raw inputs stay in source-material/, never deployed |

**When you hit a limit**: Don't just trim words. Ask: "Is this one skill or two?" A skill trying to do too many things is the most common reason for hitting size limits.

---

## When to Keep It Simple

Not every skill needs reference files. If the skill is:
- A formatting shortcut
- A simple template filler
- Straightforward instructions with no domain knowledge

Then just write a clean SKILL.md. No reference files needed. The architecture step still matters — you're just deciding that the answer is "keep it flat."

The goal isn't complexity. The goal is the right structure for what the skill does.
