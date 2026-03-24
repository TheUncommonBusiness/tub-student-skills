# Deployment Guide

How to deploy skills at TUB: audience selection, platform routing, marketplace push, YAML sanitization, post-deployment checklist, and platform-specific handling.

---

## Audience Selection

Before deploying any skill, ask who should have access. Use AskUserQuestion with these options:

- **"Just for me"** — Saves to `User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/`. Works in Claude Code only. No GitHub. No registry update. No Slack notification.
- **"The team"** — Deploys to `.claude/skills/` on the shared drive and pushes to the `tub-skills` GitHub repo. Available on both Claude Code and Cowork automatically.
- **"The team and students"** — All of the above, plus pushes to the `tub-student` plugin and the `tub-student-skills` public mirror repo.

### Team follow-up: Mandatory vs. Optional

If the user picks "The team" or "The team and students," ask a follow-up:

- **"Mandatory"** — Everyone gets this skill automatically. Goes into the `tub-mandatory` plugin. Set `audience: "mandatory"` in frontmatter.
- **"Optional"** — Users choose to install it. Goes into the `tub-optional` plugin. Set `audience: "optional"` in frontmatter.

---

## Audience-to-Plugin Mapping

| Audience value | Plugin in tub-skills repo | Also push to tub-student-skills? |
|---|---|---|
| `mandatory` | `tub-mandatory` | No |
| `optional` (or `internal`) | `tub-optional` | No |
| `student` | `tub-student` | Yes |
| `all` | `tub-mandatory` or `tub-optional` (ask user) + `tub-student` | Yes |

---

## Post-Deployment Checklist

This is mandatory. Never skip any step. Never defer steps to "later." If a step fails, stop and resolve it before moving on.

### Step 1: SKILL.md Validation
Before deploying, verify the SKILL.md has ALL required frontmatter fields:
- Top-level: `name`, `version`, `description`
- `metadata:` block: `audience`, `cowork_compatible`, `requires_cli`, `requires_env`

If any field is missing, add it before deploying. Do not deploy skills with incomplete frontmatter.

### Step 2: Deploy to Drive
Copy the skill folder to `.claude/skills/<skill-name>/`. Only runtime files: SKILL.md, references/, scripts/, assets/. Never copy source-material/ or evals/.

### Step 3: Update the Registry
Update `.claude/skill-registry.json` immediately:
- Set `version` to match SKILL.md
- Set `deployed: true`
- Set `repo` array to list all target repos based on audience
- Set `repo_synced: false` (will be set to `true` after repo push succeeds)
- Set `updated` to today's date
- Set `plugin` to the correct plugin name

### Step 4: Push to Marketplace Repo(s)
Run the Marketplace Push Procedure (below) for all target repos based on audience. After each successful push, update the registry entry's `repo_synced` to `true`.

### Step 5: Notion Cataloging
Ask the user if they want to catalog in the TUB Skills Lab Notion database. If yes, invoke `/skill-manager catalog`.

### Step 6: Sync Audit
Run `/skill-manager audit` to verify all locations match. This step is mandatory. Do not skip it.

---

## Marketplace Push Procedure

For each target repo, run these steps in order:

### 1. Check GitHub auth
Run `gh auth status`. If not authenticated, skip this repo entirely. Set `repo_synced: false` in the registry and warn: "Not pushed to [repo] — someone with GitHub access can sync it later."

### 2. Clone the repo
Shallow clone to `/tmp/<repo-name>/`:
```bash
git clone --depth 1 https://github.com/TheUncommonBusiness/<repo-name>.git /tmp/<repo-name>/
```

### 3. Copy the skill folder
Copy into the correct plugin's skills directory:
```
plugins/<plugin-name>/skills/<skill-name>/
```
Include: SKILL.md, references/, scripts/, assets/, and any other runtime files.
Do NOT include: source-material/, evals/, workspace/, or other build artifacts.
If the skill already exists there, replace it.

### 4. Sanitize the SKILL.md copy
Apply Marketplace YAML Sanitization (see below) to the copy in the repo. Never modify the original on Google Drive.

### 5. Commit
```bash
git add -A && git commit -m "deploy: <skill-name> v<version>"
```

### 6. Push
```bash
git push origin main
```

### 7. Verify
```bash
gh api "repos/TheUncommonBusiness/<repo-name>/commits?per_page=1"
```
Confirm the latest commit matches what you just pushed.

### 8. Handle failures
If push fails (e.g., conflict), try:
```bash
git pull --rebase origin main && git push origin main
```
If it still fails, warn the user and set `repo_synced: false` in the registry.

---

## Marketplace YAML Sanitization

Cowork's YAML parser is stricter than Claude Code's. Always sanitize the repo copy of the SKILL.md. Never modify the original on Google Drive or in `.claude/skills/`.

### What to sanitize

**1. Strip the `metadata:` nested block entirely.**
Cowork does not handle nested YAML objects. Remove the whole thing:
```yaml
# REMOVE this entire block from the repo copy:
metadata:
  audience: "internal"
  cowork_compatible: true
  requires_cli: []
  requires_env: []
```

**2. Remove `argument-hint:` if present.**
Not a supported field in Cowork's parser.

**3. Replace apostrophes and curly quotes in the `description:` value.**
Change `you're` to `you are`, `don't` to `do not`, etc. Cowork's parser can misinterpret these inside quoted strings.

**4. CRITICAL: The `description:` field MUST be a single-line double-quoted string.**
Do NOT use YAML folded scalars (`>`), block scalars (`|`), or multiline values. Cowork silently skips the entire skill if the description uses these formats. No error message, the skill just never appears.

Keep the repo description concise (under ~200 characters). The full description stays in the Google Drive version only.

**5. Keep only these top-level frontmatter fields:**
`name`, `version`, `description`, `user-invocable`

### Example: Before and after

**Google Drive version (stays unchanged):**
```yaml
---
name: my-skill
version: "1.0.0"
description: >
  Does something when you're working on X. This is a long
  description that spans multiple lines.
user-invocable: true
argument-hint: "Describe your situation"
metadata:
  audience: "internal"
  cowork_compatible: true
  requires_cli: []
  requires_env: []
---
```

**Marketplace repo version (sanitized):**
```yaml
---
name: my-skill
version: "1.0.0"
description: "Does something when you are working on X. Concise single-line description."
user-invocable: true
---
```

Only the SKILL.md frontmatter needs sanitization. Reference files and other supporting files are fine as-is.

---

## Student-Facing Protections

Four hard rules that apply to every student-audience skill. No exceptions.

### 1. No shared org API keys
Student skills always require each student to provide their own keys. The "Share one key with the org" option is never offered for student skills.

### 2. Key scan before deployment
Automatically scan SKILL.md and any bundled files for hardcoded keys. Look for patterns: `sk-`, `AIza-`, `gsk_`, and long alphanumeric strings. If found, halt deployment and tell the user to remove them.

### 3. Mandatory Setup section
Any skill with `requires_env` or `requires_cli` must have a `## Setup` section that includes:
- What the student needs
- Where to get it (direct links)
- Step-by-step instructions written for non-technical users
- A verification command to test the setup
- Cost warnings if applicable

If the author does not provide this, generate it automatically.

### 4. Registry enforces individual keys
All student-audience skills must have `"key_sharing": "individual"` in the registry. No exceptions.

---

## API Key Handling (Team Skills Only)

When deploying a team skill that requires API keys, ask:
- **"Share one key with the org"** — Store in `.claude/.env` at the shared Launch Points root. Warn the user about visibility and billing implications.
- **"Each user provides their own"** — Do not store any key. Make sure the SKILL.md has a clear setup section with env var names, where to get keys, and export commands.

This step only applies to team skills. Personal skills skip it (the author manages their own keys). Student skills always use individual keys (see protections above).

---

## Cowork Session Limitations

Cowork cannot write to `.claude/skills/` directly and cannot push to GitHub. Here is how each audience option works from Cowork:

### "Just for me" — works normally
Cowork CAN write to the user's deliverables folder. Save the skill to `User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/<skill-name>/`.

Then walk the user through manual Cowork installation:
1. Go to Browse plugins (top-right of Cowork sidebar)
2. Click the Personal tab
3. Click Local uploads, then the + button
4. Upload the SKILL.md file from deliverables
5. Cowork will prompt to add the skill — click Add

### "The team" or "The team and students" — Slack handoff
Save the skill to the user's deliverables folder, then send a deployment request to Slack #AgentUpdates tagging @Carter Jensen:

> **Skill Deployment Request**
> **Skill:** [skill-name] v[version]
> **Built by:** [user name]
> **Audience:** [The team / The team and students]
> **Location:** `User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/[skill-name]/`
> **Description:** [one-line description from SKILL.md]
>
> This skill was built in Cowork and needs a Claude Code user to deploy it via Rule 8.

### Skill updates from Cowork
Same Slack handoff pattern, but with an update request message:

> **Skill Update Request**
> **Skill:** [skill-name] (currently v[current] to proposed v[bumped])
> **Updated by:** [user name]
> **What changed:** [1-2 sentence summary]
> **Location:** `User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/[skill-name]/`

---

## GitHub Repos

Three repos in play:

| Repo | Visibility | Purpose |
|------|-----------|---------|
| `TheUncommonBusiness/tub-skills` | Private | Source of truth for Cowork sync. Contains 3 plugins: tub-mandatory, tub-optional, tub-student. Both team and student Cowork orgs sync from here. |
| `TheUncommonBusiness/tub-student-skills` | Public | Mirror of just the tub-student plugin. For students on Claude Code who cannot access the private repo. |
| `TheUncommonBusiness/tub-student-skills-org` | -- | DEPRECATED. Student Cowork org now syncs from tub-skills directly. |

---

## Registry Entry Format

Every deployed skill gets an entry in `.claude/skill-registry.json` with these fields: `name`, `version`, `source`, `author`, `description`, `audience`, `cowork_compatible`, `requires_cli`, `requires_env`, `key_sharing`, `installed`, `updated`, `deployed`, `plugin`, `repo` (array of target repos), `repo_synced` (whether all repos match).

Key fields to watch: `deployed` (on shared drive?), `plugin` (which Cowork plugin), `repo` (which marketplace repos), `repo_synced` (all repos match deployed version?).

---

## Skill Archives

Before deploying updates, archive the previous version to `Skills Repository [CCW-LP]/skill-archives/` with a date suffix (e.g., `hook-engine-2026-03-15/`).

To roll back, say "roll back [skill name]." Claude restores from `skill-archives/` (or Google Drive version history), redeploys to `.claude/skills/`, and pushes to marketplace repos if needed. Student-facing skills can also revert from git history.
