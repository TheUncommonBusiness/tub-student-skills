# TUB Student Skills

Official skill library for [The Uncommon Business](https://theuncommonbusiness.com) students.

Skills supercharge your Claude Code experience — they teach Claude how to do specific tasks the way TUB teaches them.

## How to install a skill

1. Browse the `skills/` folder and find one you like
2. Open Claude Code and say:

```
Install the [skill-name] skill from TheUncommonBusiness/tub-student-skills
```

Claude will handle the rest.

**Or install manually:**

```bash
# Clone just the skill you want
gh repo clone TheUncommonBusiness/tub-student-skills
cp -r tub-student-skills/skills/[skill-name] ~/.claude/skills/[skill-name]
```

## Available skills

| Skill | Description |
|---|---|
| *Coming soon* | New skills are added regularly — star this repo to get notified |

## Want to contribute?

Built something great in Claude Code? We'd love to add it to the library.

1. Fork this repo
2. Add your skill to `skills/your-skill-name/` with a `SKILL.md`
3. Open a pull request with a description of what it does

The TUB team reviews all submissions.

## Skill format

Every skill needs a `SKILL.md` file with this structure:

```markdown
---
name: my-skill-name
description: "What this skill does in one sentence"
---

# Instructions for Claude

[Your skill instructions here]
```

See any existing skill for a full example.

---

Built with Claude Code by The Uncommon Business
