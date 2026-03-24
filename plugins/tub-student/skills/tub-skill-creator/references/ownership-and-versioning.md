# Ownership and Versioning Guide

How skill ownership works, how to collaborate on skills you did not build, how versioning keeps everything clean, and how to deprecate skills gracefully.

---

## Ownership

### Who owns a skill?
The creator is the default owner. Ownership is recorded in the project README under the "Owner" field. If there is no README (quick-build skills without a project folder), the `author` field in the skill registry serves the same purpose.

### Transferring ownership
Ownership can transfer. When it does, update the README "Owner" field and the registry `author` field. Add a changelog entry noting the transfer.

### Ownership does not mean exclusivity
Owning a skill does not mean no one else can touch it. Skills are shared tools. Others can and should improve them. Ownership just means there is one person who is the go-to for questions, major decisions, and final say on direction.

---

## Editing Someone Else's Skill

### Before you touch it
Check the README (or registry) to see who owns the skill. Knowing who built it helps you understand the intent behind design choices that might not be obvious.

### Patch-level changes (typos, small tweaks)
Go ahead. Fix the typo, clarify the wording, correct the formatting. Bump the patch version, add a changelog entry with your name, and deploy. You do not need to loop in the owner for small fixes.

Examples of patch-level changes:
- Fixing a typo in the SKILL.md
- Clarifying a confusing instruction
- Updating a broken reference path
- Fixing a formatting issue

### Minor and major changes
These deserve a heads-up. Tell the user who owns the skill and suggest they loop the owner in before making significant changes.

If the user wants to proceed anyway, make the edit with clear attribution in the changelog. The owner will see what changed and who changed it next time they look.

Examples of minor changes (bump minor version):
- Adding a new capability or output option
- Adding a new reference file
- Restructuring a section for clarity
- Expanding the skill to handle a new use case

Examples of major changes (bump major version):
- Changing the core purpose or behavior
- Restructuring the entire SKILL.md
- Removing capabilities that users depend on
- Changing the invocation pattern

### Changelog Attribution

Every changelog entry includes who made the change. This is not optional. It is how the team tracks who did what.

```
| Version | Date       | Changes                                        |
|---------|------------|------------------------------------------------|
| 1.0.0   | 2026-03-05 | Initial release. (Carter)                      |
| 1.1.0   | 2026-03-12 | Added email sequence framework. (Callan)       |
| 1.1.1   | 2026-03-15 | Fixed typo in voice guide. (Jordan)            |
| 2.0.0   | 2026-03-20 | Rebuilt around new ICP framework. (Carter)      |
```

---

## Version Numbering

We use semantic versioning: `MAJOR.MINOR.PATCH`

### MAJOR (1.0.0 to 2.0.0)
The skill's core purpose or structure changed significantly. Users who relied on the old behavior might need to adjust.

Triggers:
- Core purpose changed
- Output structure fundamentally different
- Capabilities removed
- Breaking changes to how the skill is invoked

### MINOR (1.0.0 to 1.1.0)
New capability or meaningful improvement. Same core behavior, just better or broader.

Triggers:
- New feature or output option added
- New reference file added
- Significant improvement to quality or coverage
- New use case supported

### PATCH (1.0.0 to 1.0.1)
Small fix. Nothing about the skill's behavior changed in a meaningful way.

Triggers:
- Typo fix
- Wording clarification
- Formatting correction
- Minor instruction tweak

### Starting versions
- **Production-ready skills:** Start at `1.0.0`
- **Work-in-progress or untested drafts:** Start at `0.1.0` to signal "not ready yet"

---

## When to Bump

Every time anything inside the `skill/` folder changes, the version bumps. Every time. No exceptions.

### The bump checklist
1. Make the edit to the skill
2. Bump the version in the SKILL.md frontmatter (`version:` field)
3. Bump the version line in the SKILL.md body (the `> **v1.0.1**` line after the frontmatter)
4. Add a changelog entry in the README (if using a project folder)
5. Update `skill-registry.json` if the skill is registered
6. Re-deploy to the runtime location

### Auto-detection of skill edits

When Claude edits any file inside `.claude/skills/`, it automatically handles versioning:

1. **Detect the edit.** Any create or modify operation on files inside `.claude/skills/<skill-name>/` triggers this flow.
2. **Auto-bump the version.** Increment the patch version automatically (e.g., `1.0.0` to `1.0.1`). If the user explicitly says the change is a new feature, bump minor instead (`1.0.0` to `1.1.0`). Never ask the user to bump the version themselves.
3. **Prompt to deploy.** Ask: "I updated [skill-name] to v[new-version]. Want me to push this to the marketplace?" Options: "Yes, push it" or "Not yet."
4. **If yes:** Look up the skill's audience, run the Marketplace Push Procedure for the correct repos, and update the registry.
5. **If not yet:** Confirm the version was bumped locally. The skill-manager audit will flag it as out of sync later.

This applies to all edits: description changes, reference file updates, typo fixes, new sections, everything. If you touched it, it gets versioned.

**Exception:** If the edit is part of an initial deployment (the skill is being created for the first time), skip auto-detection. The deployment flow handles versioning already.

---

## Tracking Deployed vs. Current Versions

Skills can exist in multiple places: the Skill Lab project folder, the runtime location (`.claude/skills/`), and marketplace repos. These versions can drift.

### The rule
The Skill Lab version is always the latest. The runtime copy is whatever was last deployed. They should match, but sometimes they do not.

### What to do
- Always tell the user the current version when working on a skill
- After any edit, check if the deployed version matches
- Show the version delta: "The Skill Lab version is v1.2.0 but the deployed version is v1.1.0. Want me to deploy the update?"
- The `/skill-manager audit` command catches version mismatches across all locations

---

## Deprecation

### When to deprecate
- The skill was replaced by a better version
- It was built for an initiative that no longer exists
- Its functionality was absorbed into another skill
- The team agrees it is not useful anymore

### Deprecation process

**Step 1: Confirm with the owner.**
Do not deprecate someone else's skill without their knowledge. If the owner is unavailable, get confirmation from the user requesting the deprecation.

**Step 2: Update the README.**
Change the Status field from "Active" to "Deprecated" and add a note explaining why:
```
**Status:** Deprecated — replaced by content-engine v2.0.0 (2026-03-20)
```

**Step 3: Prefix the folder name.**
Add `_deprecated-` to the beginning of the folder name. This is the one exception to the no-rename rule. Example: `hook-engine/` becomes `_deprecated-hook-engine/`.

**Step 4: Add a final changelog entry.**
```
| 1.3.1 | 2026-03-20 | DEPRECATED — replaced by content-engine. (Carter) |
```

**Step 5: Do NOT delete anything.**
Deprecated skills stay in place. Someone might need to reference them later. The prefix keeps them visually separated and out of active listings.

### Deprecated skills in browse results
- Exclude deprecated skills from the main browse list when showing available skills
- Show them if a user specifically asks ("show me deprecated skills" or "what happened to hook-engine?")
- The `_deprecated-` prefix makes them easy to filter programmatically

### Undeprecating
If a deprecated skill needs to come back:
1. Remove the `_deprecated-` prefix
2. Update the README status back to "Active"
3. Add a changelog entry noting the reactivation
4. Re-deploy if it was removed from runtime locations
