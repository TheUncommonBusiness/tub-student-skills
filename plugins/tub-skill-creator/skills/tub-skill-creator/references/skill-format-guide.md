# Skill Format Guide

Every skill follows the same format. This guide shows you exactly how to structure your skill files.

---

## The Skill Folder

A skill is a folder with a `SKILL.md` file and optional supporting files:

```
my-skill/
├── SKILL.md           ← Required: the instructions Claude follows
├── references/        ← Optional: knowledge files Claude reads when needed
├── scripts/           ← Optional: code for precise/computed tasks
└── assets/            ← Optional: templates, images, config files
```

**Folder name**: lowercase with hyphens. Examples: `brand-voice-builder`, `sales-call-prep`, `weekly-report-generator`

---

## SKILL.md Structure

Every SKILL.md has two parts: the settings block (frontmatter) and the instructions (body).

### Part 1: The Settings Block (Frontmatter)

This is the YAML block at the very top of the file, between `---` markers:

```yaml
---
name: brand-voice-builder
version: "1.0.0"
description: "Build a consistent brand voice for your business. Use when someone says 'help me with my brand voice,' 'how should my content sound,' 'I need a writing style guide,' 'my content doesn't sound like me,' or 'create a voice guide.' Also triggers on 'tone of voice,' 'brand personality,' or 'content style.'"
user-invocable: true
metadata:
  audience: "student"
  cowork_compatible: true
  requires_cli: []
  requires_env: []
---
```

**Required fields:**
- `name`: Matches your folder name exactly
- `version`: Semantic version in quotes (e.g., `"1.0.0"`)
- `description`: What the skill does AND when it should activate (see "Writing Descriptions" below)

**Optional but recommended:**
- `user-invocable`: Set to `true` if users should be able to type `/skill-name` to activate it
- `metadata.audience`: `"internal"` (team only), `"student"` (students only), or `"all"` (everyone)
- `metadata.cowork_compatible`: `true` unless the skill requires command-line tools
- `metadata.requires_cli`: List of CLI tools needed (e.g., `["python3"]`)
- `metadata.requires_env`: List of environment variables needed (e.g., `["OPENAI_API_KEY"]`)

### Part 2: The Version Line

The first line after the closing `---` must be a version line:

```
> **v1.0.0** | Built by Your Name | 2026-03-17
```

This tells Claude (and anyone reading the file) what version is running. Keep it in sync with the frontmatter version.

### Part 3: The Instructions (Body)

Everything after the version line is what Claude follows when the skill activates. Write it in Markdown.

---

## Writing Descriptions (Critical)

The description controls when Claude decides to use your skill. A vague description won't activate. A specific one will.

**Bad description:**
> "Helps with business writing."

Claude sees thousands of requests about "business writing." This description doesn't tell it when YOUR skill is the right one.

**Good description:**
> "Build a consistent brand voice for your business. Use when someone says 'help me with my brand voice,' 'how should my content sound,' 'I need a writing style guide,' 'my content doesn't sound like me,' or 'create a voice guide.' Also triggers on 'tone of voice,' 'brand personality,' or 'content style.'"

**What makes a good description:**
1. **Start with what it does** — one clear sentence
2. **List trigger phrases** — the actual words people would say when they need this
3. **Include adjacent concepts** — related topics that should also activate the skill
4. **Be "pushy"** — err on the side of activating too much rather than too little

**Think about it from the user's side**: What would someone say when they need this skill? They won't say "invoke the brand voice builder." They'll say "my emails don't sound like me" or "I need help with my tone."

---

## Version Numbers

Use semantic versioning: `MAJOR.MINOR.PATCH`

- **New skills** start at `1.0.0`
- **Small fixes** (typo, wording tweak): bump the patch → `1.0.1`
- **New features** (added a section, new capability): bump the minor → `1.1.0`
- **Major changes** (complete rewrite, different audience): bump the major → `2.0.0`

Every change bumps the version. Update both the frontmatter AND the version line.

---

## What Goes Where

| Folder | What belongs | What doesn't belong |
|--------|-------------|-------------------|
| **SKILL.md** | Diagnostic flow, routing logic, voice/tone rules, output format | Domain knowledge, raw content, long reference material |
| **references/** | Distilled frameworks, voice guides, curated examples, decision trees | Raw PDFs, full transcripts, anything over 300 lines per file |
| **scripts/** | Data parsers, formatters, validators, API helpers | Complex apps, hardcoded secrets, things Claude does better natively |
| **assets/** | Templates, logos, config files, starter documents | Large media files, source material, generated outputs |

**The key rule**: SKILL.md is the conductor. Reference files are the musicians. Scripts are the power tools. Assets are the raw materials.

---

## Invocation Modes

Skills can activate in three ways:

**Automatic**: Claude detects when someone needs the skill and activates it on its own. The user doesn't type any command — it just fires. Best for skills that should always run when relevant (like a brand voice that applies to all content).

**Slash command**: The user types `/skill-name` to activate it deliberately. Best for tools you only use sometimes (like a report generator).

**Both**: Works either way — Claude can auto-detect AND the user can invoke it manually. Most flexible option.

Your choice affects how you write the description:
- **Automatic/Both**: Description must be rich with trigger phrases — it's doing the detection work
- **Slash command only**: Description is informational — it just needs to be clear about what the skill does
