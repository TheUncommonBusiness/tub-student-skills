---
name: skills-browser
version: "1.0.0"
description: "Browse all available skills in the TUB ecosystem. Use when someone says /skills, show me all skills, what skills do we have, what skills are available, list skills, browse skills, what can you do, or help me find a skill. Shows a clean table of every skill with name, description, and how to invoke it."
user-invocable: true
---
> **v1.0.0** | Built by Carter Jensen | 2026-03-20

# Skills Browser

When invoked, display a clean, scannable table of all available skills. Read each loaded skill's name and description from the current session context — do not hardcode a list.

## How to Display

Show the skills in a formatted table grouped by category:

```
Available Skills — [N] total

STRATEGY & BUSINESS
  /9ff              — Strategic advisor from the 9FF mastermind (beliefs, sales, tax, offers)
  /alex-hormozi     — Alex Hormozi as a virtual business partner (offers, pricing, growth)
  /idea-validator   — Pressure-test a product idea (BUILD / RESHAPE / SHELF verdict)
  /lead-magnet      — Design lead magnets using Hormozi's $100M Leads frameworks

CONTENT & WRITING
  /callan-voice     — Write in Callan Faulkner's authentic voice (all channels)
  /hook-engine      — Generate scroll-stopping hooks from 150 real posts
  /ig-intel         — Instagram competitor intelligence and content analysis

BUILDING & OPS
  /skill-creator    — Build custom Claude skills (full lifecycle)
  /skills           — Browse all available skills (you are here)
  /skill-manager    — Manage, audit, catalog, and sync skills across platforms
  /sop-creator      — Turn transcripts or drafts into polished SOPs
  /notion-designer  — Apply TUB Notion design standards to pages

RESEARCH & NEWS
  /last30           — Research any topic from the last 30 days (Reddit, X, YouTube, etc.)
  /ai-news          — Compile latest AI news into a Notion briefing

TOOLS
  /nano-banana      — Generate and edit images (Gemini CLI)
  /framework-builder — Extract teachable frameworks from any source
  /prompt-librarian — Save built prompts to the Master Prompt Library
```

## Important Behavior

1. **This is a REFERENCE display, not a hardcoded list.** The table above is an example of the format. When actually displaying, read the skills that are currently loaded in the session and show those. New skills added to the ecosystem should appear automatically.

2. **After showing the table**, ask: "Want to know more about any of these? Just say the name or type the slash command."

3. **If a user asks about a specific skill**, read that skill's SKILL.md and give a 2-3 sentence summary of what it does, when it triggers, and how to use it.

4. **Group skills by function**, not alphabetically. Use the categories above as a starting point, but adjust if new skills don't fit.

5. **Show the slash command** as the primary identifier — it's how users invoke skills deliberately.

6. **Works on all platforms** — Claude Code, Cowork, and Claude.ai. No CLI tools needed.
