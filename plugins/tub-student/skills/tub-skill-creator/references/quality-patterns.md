# Quality Patterns

Patterns for building quality checks, output formats, and self-assessment into skills. Read this when designing quality gates for a new skill.

---

## Quality Gates: Build Verification Into the Skill

A quality gate is a check the skill runs on its own output before presenting it to the user. Think of it as a pre-flight checklist — the pilot runs it every time, not just when something feels wrong.

### Why quality gates matter

Without them, the skill's output quality depends entirely on whether Claude "felt like" being thorough today. With them, there's a structural guarantee: the output either passes the checks or gets reworked.

### The "Complete But Narrow" Test

The simplest and most powerful quality gate. Before finalizing output, ask:

> **Could someone take this output, use it, and accomplish the ONE thing it's supposed to do?**
>
> - **YES** → Ship it.
> - **NO** → Fix it.

Common failure modes:
- **Teaser, not complete**: Gives a taste but user can't actually use it without more work
- **Comprehensive, not narrow**: Covers everything but solves nothing specifically
- **Looks good, missing substance**: Well-formatted but content is generic/obvious

### Building Quality Gates Into Your Skill

Add a self-check section near the end of your SKILL.md:

```markdown
## Before Presenting Output

Run these checks silently. If any fail, fix the output before showing it.

1. [ ] Does the output fully solve the ONE problem it was asked to solve?
2. [ ] Is every recommendation specific to THIS user's situation (not generic)?
3. [ ] Are all required sections present and filled with real content?
4. [ ] Does the tone match the voice guide?
5. [ ] Could the user take this and use it TODAY without further customization?
```

For skills with domain-specific quality criteria, put the checks in a reference file:

```
When reviewing output quality → read references/quality-checks.md
```

---

## Output Format Specification

Defining what the output looks like is as important as defining what it says. Without a format spec, Claude invents a different structure every time.

### Why format matters

Compare these two outputs from the same skill:

**Without format spec:**
> Here are some ideas for your lead magnet. You could try a free guide about real estate investing, or maybe a checklist for first-time buyers. Another option is a video series...

**With format spec:**
> ## Lead Magnet Blueprint
> **Type**: Reveal Problem (shows prospects what's broken)
> **Mechanism**: Information (PDF guide)
> **Name**: "The 5 Hidden Costs That Kill First-Time Buyers"
> **Bridge**: Solves "what to watch for" → reveals "how to negotiate" → connects to coaching program
> **CTA**: "Download the free guide" → email capture → 3-email nurture → discovery call
> **Distribution**: Instagram bio link + cold email signature + website popup

The second output is usable. The first is a conversation.

### How to specify output format

In your SKILL.md, include a template:

```markdown
## Output Format

Every [deliverable type] must include these sections:

### [Section 1 Name]
[What goes here — 1-2 sentences describing the content]

### [Section 2 Name]
[What goes here]

### [Section 3 Name]
[What goes here]

**Example:**
[Include one worked example showing what good output looks like]
```

### Format patterns by skill type

**Workflow skills** (SOP builder, delegation tool):
- Numbered steps with clear actions
- Each step: What to do, who does it, when it's done
- Acceptance criteria for the whole workflow

**Voice/persona skills** (brand voice, business partner):
- Response format varies by context (not rigid sections)
- Quality is in the voice, not the structure
- Include "what good looks like" examples instead of templates

**Knowledge expert skills** (pricing advisor, strategy tool):
- Recommendation + rationale + data
- Framework application showing the work
- Next steps / action items

**Content generator skills** (hook writer, email builder):
- Labeled output items (hook #1, email #2)
- Scoring or quality annotation per item
- Variety check (different approaches, not 10 versions of the same thing)

---

## Voice/Tone Extraction

When a skill has a specific voice — a persona, a brand, or a communication style — extract it into its own reference file.

### When to extract

Extract voice when:
- The skill should sound like a specific person or brand
- Voice instructions would take more than 10-15 lines in the SKILL.md
- The same voice might be used across multiple skills
- You find yourself writing "sound like..." or "talk like..." in the instructions

Don't extract when:
- The skill just needs to be "clear and professional" (that's Claude's default)
- Voice instructions are 5 lines or fewer (just put them in the SKILL.md)

### What goes in a voice reference file

```markdown
# Voice Guide: [Name/Brand]

## Core Tone
[2-3 sentences describing the overall feel]

## Signature Patterns
- [Specific phrase or opening pattern]: "[Example]"
- [Another pattern]: "[Example]"

## Vocabulary Rules
- ALWAYS use: [specific words/phrases that define this voice]
- NEVER use: [words that break the voice]

## Sentence Rhythm
[How sentences flow — short punchy? Long flowing? Mix?]

## Examples
[3-5 examples of text in this voice — the most important section]
```

### Loading voice conditionally

Small voice files (under 50 lines) can load on every response — the overhead is worth the consistency.

Large voice files (50+ lines) should load conditionally, like other references. But voice is special — if the skill has a specific persona, load voice FIRST, before any other reference.

---

## Self-Assessment Checklists

For skills that produce complex deliverables, include a checklist the skill runs through before finalizing.

### Example: Proposal Builder Checklist

```markdown
## Pre-Delivery Check

Before presenting the proposal:

1. **Client specificity**: Does the proposal reference the client's specific
   situation, industry, and goals? (Not generic templates with names swapped)

2. **Scope clarity**: Is every deliverable described clearly enough that the
   client knows exactly what they're getting? (No vague "support" or "consulting")

3. **Timeline realism**: Are dates specific (not "TBD") and achievable?

4. **Pricing transparency**: Is the total clear? Are payment terms stated?
   No hidden costs or ambiguous "starting at" language?

5. **Next step**: Is there ONE clear action the client takes next?
   (Not "let me know your thoughts" — that's not a next step)

If any check fails, fix it before presenting.
```

### The key principle

Quality gates should be **specific to what the skill produces**. Generic checklists ("is it good? is it clear?") don't catch real problems. Specific checks ("does every hook use a different architecture? does the proposal reference the client's industry?") do.

Build the checklist based on what you've seen go wrong. Every time a skill produces output that's "almost right but missing something," that missing thing should become a checklist item.
