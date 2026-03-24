# Project Structure Guide

How to organize skill development folders, manage source material, and structure supporting files for clean builds and easy updates.

---

## Project Folder Layout

Every skill that involves raw source material (transcripts, PDFs, book notes, slide decks) should use this structure:

```
skill-name/
├── README.md                     ← deployment guide (auto-created)
├── source-material/               ← raw inputs (never deployed)
│   ├── transcript.pdf
│   ├── book-notes.md
│   └── presentation-slides.pdf
└── skill/                         ← deployable output (THIS gets deployed)
    ├── SKILL.md
    ├── references/
    ├── scripts/
    └── assets/
```

**Where project folders live:**
- Internal team skills: `Skills Repository [CCW-LP]/internal-team/<skill-name>/`
- Student-facing skills: `Skills Repository [CCW-LP]/student-facing/<skill-name>/`

**What each piece does:**
- `source-material/` is where users drop raw content for Claude to read during the build. It never gets deployed. It stays in the repo so future updates can reference the original material.
- `skill/` is the actual deployable skill. When it is time to deploy, the contents of this folder get copied to the runtime location. Nothing outside `skill/` ships.
- `README.md` sits at the project root and documents ownership, deployment status, and changelog.

---

## When to Use Project Folders

Not every skill needs a project folder. Here is the decision:

**Use the project folder structure when:**
- The skill has raw source material to distill (books, transcripts, PDFs, slide decks)
- The skill needs multiple reference files built from research
- You want a paper trail of what the skill was built from

**Skip the project folder when:**
- The skill is a quick build from conversation with no raw material
- Everything the skill needs fits in SKILL.md alone or with one small reference file
- The user just wants to get it done fast

When you skip the project folder, build the `skill/` contents directly. No wrapper folder, no README, no source-material directory.

---

## Source Material Drop Workflow

This is the step-by-step for when a user has raw files they want distilled into a skill.

### Step 1: Create the project folder
Set up the folder in the appropriate zone (`internal-team/` or `student-facing/`) with the `source-material/` and `skill/` subdirectories.

### Step 2: Tell the user where to drop files
Give them the exact path. Be specific: "Drop your files into `Skills Repository [CCW-LP]/internal-team/hook-engine/source-material/` and let me know when they are there."

### Step 3: Wait for confirmation
Do not proceed until the user confirms the files are uploaded. Google Drive sync can take a moment.

### Step 4: Inventory what is there
List every file in `source-material/`. Note file types, sizes, and names. Tell the user what you found.

### Step 5: Identify what the skill actually needs
Not everything in source material is equally important. Pick the 3-5 most relevant things for the skill's purpose. Tell the user what you plan to extract and why.

### Step 6: Read and extract
For large files, read in chunks. Pull out the specific knowledge, frameworks, examples, voice patterns, or decision trees that the skill needs at runtime. Discard filler, repetition, and context that does not serve the skill.

### Step 7: Write lean reference files
Create focused reference files in `skill/references/`. Each one should cover a single topic and stay under 300 lines. These are what Claude reads at runtime, so they need to be clean, scannable, and actionable.

### Step 8: Wire the SKILL.md
Update the SKILL.md to load reference files conditionally. Not every run needs every reference. Use clear section headers so Claude knows when to pull which file.

---

## Updating Source Material After Initial Build

When a user adds new source material to an existing skill:

**Do not re-distill everything from scratch.** Start from the existing reference files and figure out what changed.

Three possible outcomes:
1. **New content that covers a new topic** — create a new reference file
2. **Updated content that improves an existing topic** — update the existing reference file
3. **Redundant content that overlaps with what is already there** — no changes needed

After updating, always test the skill. New material can shift behavior in unexpected ways, especially if it introduces conflicting guidance.

---

## Supporting Files Guide

### references/
Distilled knowledge that Claude reads at runtime. This is where frameworks, voice guides, curated examples, domain knowledge, and decision trees live.

Good reference files are:
- Focused on one topic each
- Under 300 lines (aim for 100-200)
- Written for Claude, not for humans (clear, direct, no fluff)
- Actionable, not informational

Bad reference files are:
- Raw dumps of source material
- Kitchen-sink files that cover everything
- Copy-pasted content without curation

### scripts/
Helper code for deterministic tasks. Use scripts when you need precision, not judgment. Examples: data processing, template generators, validation logic, API helpers, format converters.

Scripts run during skill execution. They should be self-contained and well-commented.

### assets/
Static files that the skill uses as inputs. Examples: document templates, starter files, config files, images, sample data.

Assets are things Claude hands to the user or uses as a starting point. They are inputs to the skill, not outputs.

### source-material/
What you read to build the skill. Raw transcripts, book notes, PDFs, slide decks, research docs. Never deployed. Never read at runtime. Stays in the project folder for future reference.

---

## The Mental Model

Think of it this way:

| Folder | Role | When |
|--------|------|------|
| `source-material/` | What you READ to build the skill | Build time only |
| `references/` | What Claude READS at runtime | Every run (conditionally) |
| `scripts/` | What Claude RUNS at runtime | When deterministic work is needed |
| `assets/` | What Claude USES at runtime | When templates or starter files are needed |

---

## Size Guidelines

Keep things lean. Bloated skills are slow to load and hard to maintain.

- **Each reference file:** under 300 lines (aim for 100-200)
- **Total references per skill:** under 1,000 lines across all files
- **SKILL.md itself:** under 500 lines
- **If you need more:** split into multiple skills instead of one mega-skill

---

## README Template

Every project folder gets a README.md at the root. Here is the template:

```markdown
# [Skill Name]

**Version:** [from SKILL.md frontmatter]
**Zone:** [internal-team or student-facing]
**Audience:** [mandatory, optional, student, or all]
**Deployed to:** [runtime path(s)]
**Repo:** [GitHub repo(s) or "none"]
**Owner:** [name and email]
**Created by:** [name, date]
**Last updated by:** [name, date]
**Status:** Active

## What This Skill Does
[1-2 sentence description]

## Folder Structure
[show the tree for this specific project]

## How to Deploy
1. Deploy the `skill/` folder contents using root CLAUDE.md Rule 8
2. source-material/ is NOT deployed
3. Team/student skills go to .claude/skills/
4. Personal skills go to User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/

## How to Update
1. Edit files inside skill/ folder
2. Re-distill if source material changed
3. Bump version in SKILL.md frontmatter + body
4. Add changelog entry below
5. Re-deploy via Rule 8

## Changelog
| Version | Date | Changes |
|---------|------|---------|
| [version] | [date] | Initial release. ([creator name]) |
```

Update the README every time the skill ships a new version. The changelog is the quick-glance history for anyone picking up this skill later.
