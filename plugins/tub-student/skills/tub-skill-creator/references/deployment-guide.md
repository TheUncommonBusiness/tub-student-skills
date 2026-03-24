# Deployment Guide

How to deploy skills at TUB: audience selection, platform routing, marketplace push, YAML sanitization, and platform-specific handling.

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

After every skill deployment or update:

1. **Validate SKILL.md** — Ensure `name`, `version`, `description` are set. Ensure `metadata:` block has `audience`, `cowork_compatible`, `requires_cli`, `requires_env`.
2. **Deploy to Drive** — Copy to `.claude/skills/<skill-name>/`. Only runtime files: SKILL.md, references/, scripts/, assets/. Never copy source-material/ or evals/.
3. **Update the registry** — Update `.claude/skill-registry.json` with version, audience, plugin, repo array, repo_synced, and today's date.
4. **Push to GitHub** — Run the Marketplace Push Procedure (below) for all target repos. Update `repo_synced` in registry after each successful push.
5. **Optional: Notion catalog** — Ask if the user wants to catalog in the TUB Skills Lab (`/skill-manager catalog`).

---

## GitHub Repos

Both repos use the **official Claude Code plugin marketplace format** — each has a `.claude-plugin/marketplace.json` at root, and each plugin folder has a `.claude-plugin/plugin.json` manifest.

| Repo | Visibility | Purpose |
|------|-----------|---------|
| `TheUncommonBusiness/tub-skills` | Private | Source of truth for Cowork sync. Contains 3 plugins: tub-mandatory, tub-optional, tub-student. |
| `TheUncommonBusiness/tub-student-skills` | Public | Mirror of just the tub-student plugin. For students on Claude Code who cannot access the private repo. |

### Repo structure (tub-skills)

```
tub-skills/
  .claude-plugin/
    marketplace.json           ← lists all 3 plugins
  plugins/
    tub-mandatory/
      .claude-plugin/
        plugin.json            ← plugin manifest (name, version, description)
      skills/
        callan-voice/SKILL.md
        skill-manager/SKILL.md
        ...
    tub-optional/
      .claude-plugin/
        plugin.json
      skills/
        hook-engine/SKILL.md
        9ff-advisor/SKILL.md
        ...
    tub-student/
      .claude-plugin/
        plugin.json
      skills/
        idea-validator/SKILL.md
        tub-skill-creator/SKILL.md
        ...
```

Cowork syncs from these repos automatically via Organization Settings > Plugins. Admins control visibility per plugin using installation preferences: Required, Installed by default, Available for install, or Not available.

---

## Marketplace Push Procedure

For each target repo, run these steps in order:

### 1. Check GitHub auth
Run `/opt/homebrew/bin/gh auth status`. If not authenticated, skip this repo. Set `repo_synced: false` in the registry and warn: "GitHub push skipped — someone with access can sync it later."

### 2. Clone the repo
Shallow clone to `/tmp/<repo-name>/`:
```bash
/opt/homebrew/bin/gh repo clone TheUncommonBusiness/<repo-name> /tmp/<repo-name>/ -- --depth 1
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
Apply YAML sanitization (see below) to the **repo copy only**. Never modify the original on Google Drive.

### 5. Commit and push
```bash
cd /tmp/<repo-name>
git add -A && git commit -m "deploy: <skill-name> v<version>"
git push origin main
```

### 6. Handle failures
If push fails (e.g., conflict), try `git pull --rebase origin main && git push origin main`. If it still fails, warn the user and set `repo_synced: false`.

### 7. Clean up
```bash
rm -rf /tmp/<repo-name>
```

**Important**: Do NOT create new plugin folders. All skills go inside the existing plugin folder for their audience. Do NOT edit marketplace.json for individual skills — it lists plugins, not skills. Only bump the plugin version in plugin.json and the corresponding marketplace.json entry when skills are added or updated.

---

## Marketplace YAML Sanitization

When pushing a skill to any GitHub repo, sanitize the SKILL.md copy in the repo. Never modify the original on Google Drive.

**Rules:**
1. **Strip the entire `metadata:` block** — Cowork's YAML parser cannot handle nested objects
2. **Remove `argument-hint:`** if present
3. **Keep only these top-level fields**: `name`, `version`, `description`, `user-invocable`, `disable-model-invocation`
4. **`description` MUST be a single-line double-quoted string** — never use folded scalars (`>`) or block scalars (`|`). Cowork silently skips skills with multiline descriptions. Replace apostrophes with plain alternatives (`you're` → `you are`)

Only the SKILL.md frontmatter needs sanitization. Reference files, scripts, and other supporting files are copied as-is.

---

## Student-Facing Protections

Four hard rules for every student-audience skill. No exceptions.

### 1. No shared org API keys
Student skills always require each student to provide their own keys. The "Share one key with the org" option is never offered.

### 2. Key scan before deployment
Scan SKILL.md and bundled files for hardcoded keys (`sk-`, `AIza-`, `gsk_`, long alphanumeric strings). Halt deployment if found.

### 3. Mandatory Setup section
Any skill with `requires_env` or `requires_cli` must have a `## Setup` section with: what they need, where to get it (direct links), step-by-step instructions for non-technical users, a verification command, and cost warnings if applicable. Generate it automatically if the author doesn't provide it.

### 4. Registry enforces individual keys
All student-audience skills must have `"key_sharing": "individual"` in the registry.

---

## API Key Handling (Team Skills Only)

When deploying a team skill that requires API keys, ask:
- **"Share one key with the org"** — Store in `.claude/.env` at the shared Launch Points root. Warn about visibility and billing.
- **"Each user provides their own"** — Do not store any key. Ensure the SKILL.md has a clear setup section.

Personal skills skip this (the author manages their own keys). Student skills always use individual keys.

---

## Cowork Session Limitations

Cowork cannot write to `.claude/skills/` or push to GitHub.

### "Just for me" — works normally
Save to `User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/<skill-name>/`. Walk the user through manual Cowork installation (Browse plugins > Personal > Local uploads > upload SKILL.md).

### "The team" or "The team and students" — Slack handoff
Save to the user's deliverables folder, then send a deployment request to Slack #AgentUpdates tagging @Carter Jensen:

> **Skill Deployment Request**
> **Skill:** [skill-name] v[version]
> **Built by:** [user name]
> **Audience:** [The team / The team and students]
> **Location:** `User Deliverables [CCW-LP]/[user]-deliverables/[name]_skills/[skill-name]/`
> **Description:** [one-line description]

### Skill updates from Cowork
Same Slack handoff, but with an update request noting what changed and the proposed version bump.

---

## Registry Entry Format

Every deployed skill gets an entry in `.claude/skill-registry.json`:

Key fields: `name`, `version`, `description`, `audience`, `plugin` (which Cowork plugin), `cowork_compatible`, `requires_cli`, `requires_env`, `deployed` (on shared drive?), `repo` (which marketplace repos), `repo_synced` (all repos match deployed version?), `installed`, `updated`.

---

## Skill Archives

Before deploying updates, archive the previous version to `Skills Repository [CCW-LP]/skill-archives/` with a date suffix. To roll back, say "roll back [skill name]" — Claude restores from archives or Google Drive version history and redeploys.
