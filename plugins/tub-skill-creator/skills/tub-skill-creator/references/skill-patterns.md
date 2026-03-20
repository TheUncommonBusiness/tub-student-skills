# Skill Patterns: Before & After

Real examples showing what separates great skills from okay ones. Use these as models when designing your own.

---

## Pattern 0: The Well-Distilled Simple Skill

### The most common (and most effective) pattern

```
skill-name/
├── SKILL.md (80-120 lines)
│   ├── Role + tone definition
│   ├── Diagnostic question flow
│   ├── "Read references/knowledge.md" (always loaded)
│   ├── Output format
│   └── Quality checks
└── references/
    └── knowledge.md (150-250 lines)
        ├── Core frameworks (extracted from source material)
        ├── Key heuristics and decision rules
        ├── Voice patterns (if applicable)
        └── Worked examples
```

This pattern works for ~80% of skills people actually build. The entire skill loads ~300-370 lines total. Claude has full context with room to think.

**When to use this**: You have source material (transcripts, books, frameworks) and want Claude to apply that knowledge in conversations. The knowledge is one domain, not multiple independent domains.

**Why it works**: Research shows that LLM performance degrades with every additional document loaded into context — even when the model can perfectly retrieve the information. One dense, well-organized file gives Claude everything it needs without the routing overhead of deciding which file to load when.

**When to upgrade to Pattern 1**: You find that loading ALL the knowledge on every request is wasteful — the user asks about pricing and Claude has 200 lines of unrelated content marketing frameworks in context. That's the signal to split.

---

## Pattern 1: Monolithic vs. Modular (Before/After)

### Before: The Monolith (520 lines, one file)

```
SKILL.md contains:
- 15 business frameworks (Value Equation, CLOSER, pricing models...)
- Voice guide (signature phrases, vocabulary, tone rules...)
- 30 tactical playbooks (pricing, closing, hooks, ads, retention...)
- Strategic thinking patterns (how to evaluate a business...)
- Diagnostic question flow
- Output format rules

Result: Claude loads ALL of this on EVERY request. User asks a simple
pricing question → Claude has 500+ lines of unrelated frameworks
competing for its attention. Output is generic because everything
is fighting for space.
```

### After: The Router + References (152-line SKILL.md + 4 reference files)

```
SKILL.md (152 lines) contains:
- Role definition: "You're a business partner, not an answer engine"
- Diagnostic flow: Ask → Understand → Give take → Ask if they want deeper
- Routing logic:
    When user needs voice/tone → read references/voice-guide.md
    When applying frameworks → read references/frameworks.md
    When doing strategy → read references/strategic-thinking.md
    When giving tactical advice → read references/playbooks.md
- Response rules: Short first response + questions, NOT 200-word monologue

references/voice-guide.md (50 lines) — loaded on every response
references/frameworks.md (200 lines) — loaded only for offer/framework questions
references/strategic-thinking.md (150 lines) — loaded only for business evaluation
references/playbooks.md (200 lines) — loaded only for tactical advice

Result: User asks a pricing question → Claude loads SKILL.md (152 lines)
+ voice-guide (50 lines) + playbooks (200 lines) = 402 lines total.
The other 350 lines of irrelevant content stay out of the way.
Output is specific because Claude only sees what's relevant.
```

**The difference**: Same knowledge, 10x better output. The architecture made the knowledge accessible instead of overwhelming.

---

## Pattern 2: Diagnostic-First Flow

### How the Hook Engine Does It

The hook engine doesn't start writing hooks immediately. It extracts 8 "raw ingredients" first:

1. **Audience** — Who are we talking to?
2. **Pain** — What's the problem they feel?
3. **Desire** — What do they want instead?
4. **Stakes** — What happens if they don't act?
5. **Proof** — What evidence do we have?
6. **Objection** — What's stopping them?
7. **Mechanism** — What's the solution?
8. **Surprise** — What's counterintuitive?

It needs a minimum of 4 before it will write a single hook. This hard gate prevents generic output.

**Why this works**: Without the ingredients, you get hooks like "5 Tips to Grow Your Business" (generic). With them, you get hooks like "My client Kim cut her workload from 30 hours to 10 — here's the system that did it" (specific, proof-backed, outcome-driven).

### How the Lead Magnet Skill Does It

Five sequential steps — you can't skip ahead:

1. **Diagnose** — What do you sell? Who buys? How do you get customers? What have you tried?
2. **Pick Type** — Reveal Problem, Trial, or One Step (based on diagnosis)
3. **Pick Mechanism** — Software, Information, Services, or Physical (based on capacity)
4. **Build Blueprint** — Concept, bridge, naming, CTA, distribution plan
5. **Quality Check** — Run 4 pass/fail tests before shipping

Each step loads only the reference file it needs. Step 2 loads `frameworks.md`. Step 4 loads `naming.md`. Step 5 loads `quality-checks.md`.

**The pattern**: Ask sharp questions → narrow down options → build the specific thing → verify quality. Never skip straight to building.

---

## Pattern 3: Conditional Reference Loading

### The Business Partner Skill (4 reference files, loaded by context)

```markdown
## Your Toolbox (in references/)

- **voice-guide.md** — ALWAYS read this before responding. It defines how
  you talk: short declarations, specific numbers, zero corporate jargon.

- **frameworks.md** — Read when applying specific frameworks (Value Equation,
  Grand Slam Offer, CLOSER). Don't read for general conversation.

- **strategic-thinking.md** — Read when evaluating a business opportunity or
  finding constraints. Skip for tactical advice.

- **playbooks.md** — Read when giving tactical advice on pricing, closing,
  hooks, ads, or retention. Skip for strategic discussions.
```

**Key detail**: Voice loads ALWAYS (50 lines — small enough to justify). Everything else loads conditionally. A tactical conversation loads ~400 lines total, not ~600.

### The Lead Magnet Skill (6 reference files, loaded by step)

```markdown
## When to Load Each Reference

- User needs to choose type or mechanism → read references/frameworks.md
- User needs real examples → read references/examples-and-models.md
- User needs naming help → read references/naming-and-hooks.md
- User needs format/delivery guidance → read references/delivery-and-formats.md
- User needs conversion benchmarks → read references/math-and-benchmarks.md
- User is reviewing quality → read references/quality-checks.md

DO NOT load all 6 at once. Each step of the flow loads its own reference.
```

**The pattern**: Map each reference file to a specific phase of the conversation. Write the loading rules explicitly in the SKILL.md.

---

## Pattern 4: Hard Quality Gates

### Hook Engine: Architecture Diversity Check

Before presenting a batch of hooks, the skill runs a self-check:

```
## Architecture Coverage
Every batch MUST include all 5 top architectures:
✅ Hypothetical Reset — Hook #3
✅ Bold Declaration — Hook #7
✅ Proof-Led Authority — Hook #1
✅ Pattern Interrupt — Hook #5
✅ Story-Driven Curiosity — Hook #9

If any architecture is missing, generate more hooks until all 5 are covered.
Do NOT present the batch until this check passes.
```

This prevents lazy output where all 10 hooks use the same structure.

### Lead Magnet Skill: "Complete But Narrow" Test

Before finalizing a lead magnet concept, it runs a binary test:

> **Could someone take this lead magnet, use it, and walk away having solved that ONE problem?**
>
> - If YES → it's complete. Ship it.
> - If NO → it's a teaser. Fix it.
>
> "Complete" doesn't mean comprehensive. It means it fully solves one micro-problem.
> "Narrow" doesn't mean shallow. It means focused on ONE specific thing.

**The pattern**: Build a pass/fail check INTO the skill's instructions. The skill verifies its own output before presenting it. This catches problems before the user even sees them.

---

## Pattern 5: Voice as a Separate Reference File

### When voice matters, extract it

The Business Partner skill has a dedicated `voice-guide.md`:

```markdown
# Voice Guide

## Tone
- Short, declarative opening statements followed by longer explanations
- Signature bridges: "The thing is...", "Reality is...", "Here's the deal..."
- Casual but precise — contractions good, vague bad

## Vocabulary
- NEVER use: "synergy," "leverage," "perhaps," "possibly," "it depends"
- ALWAYS use: specific numbers, direct statements, concrete examples

## Sentence Rhythm
- Lead with the point, then explain
- Short sentences for emphasis. Longer ones for reasoning.
- Break complex ideas into "Step 1... Step 2... Step 3..." format
```

**Why separate?** Voice instructions mixed into the SKILL.md get lost in the noise. A standalone file is:
- Reusable across skills (same voice, different skill)
- Easy to update (change the voice without touching the skill logic)
- Loaded every time (small enough to justify always-on)

**When to extract voice**: Any skill that has a specific persona, brand voice, or communication style. If you find yourself writing "sound like..." or "talk like..." in the SKILL.md, that's your signal.

---

## Common Anti-Patterns

### 1. "The Knowledge Dump"
Everything in one SKILL.md. 500+ lines. No reference files. Claude loads all of it on every request.
**Fix**: Split into SKILL.md (routing) + reference files (knowledge). Load conditionally.

### 2. "The Answer Machine"
Skill immediately produces output without asking any questions. Generic results every time.
**Fix**: Add a diagnostic flow. Ask 3-5 sharp questions before producing anything.

### 3. "The Raw Dump"
Full transcripts, entire book chapters, or unedited presentations in the `references/` folder.
**Fix**: Distill. 300 pages of transcripts should become 100 lines of key frameworks and examples.

### 4. "The Missing Format"
Skill produces output but doesn't define what it should look like structurally.
**Fix**: Specify the output format with sections, structure, and an example.

### 5. "The Scope Creep"
Skill tries to do 5 things. Does all of them poorly.
**Fix**: Pick the ONE thing it does best. Build that. Add more later if needed.
