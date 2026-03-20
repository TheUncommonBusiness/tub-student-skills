# TUB Student Skills

Official skill marketplace for [The Uncommon Business](https://theuncommonbusiness.com) students.

Skills supercharge your Claude experience — they teach Claude how to do specific tasks the way TUB teaches them.

## How to Install (Cowork)

Your admin can add this marketplace to your Cowork organization:

1. Go to **Organization Settings > Plugins**
2. Click **Add marketplace**
3. Enter: `TheUncommonBusiness/tub-student-skills`
4. Click **Sync**

Once synced, all team members can browse and install skills from the plugin list.

## How to Install (Claude Code)

```bash
# Install a specific plugin
claude plugin install TheUncommonBusiness/tub-student-skills --plugin idea-validator
```

Or install manually:

```bash
gh repo clone TheUncommonBusiness/tub-student-skills
cp -r tub-student-skills/plugins/idea-validator/skills/idea-validator ~/.claude/skills/
```

## Available Skills

| Skill | Version | Description |
|---|---|---|
| **idea-validator** | 1.0.2 | Pressure-test a digital product idea before building. Scores across 5 dimensions — BUILD, RESHAPE, or SHELF. |
| **tub-skill-creator** | 1.1.0 | Build custom Claude skills for your business. Full lifecycle: architecture, writing, testing, and deployment. |

## Repository Structure

```
tub-student-skills/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest
├── plugins/
│   ├── idea-validator/
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json       # Plugin metadata
│   │   └── skills/
│   │       └── idea-validator/
│   │           └── SKILL.md
│   └── tub-skill-creator/
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── skills/
│           └── tub-skill-creator/
│               ├── SKILL.md
│               ├── references/
│               ├── agents/
│               ├── scripts/
│               └── ...
└── README.md
```

## Want to Contribute?

Built something great? We'd love to add it to the library.

1. Fork this repo
2. Create your plugin in `plugins/your-skill-name/` following the structure above
3. Include a `.claude-plugin/plugin.json` and a `skills/your-skill-name/SKILL.md`
4. Open a pull request

The TUB team reviews all submissions.

---

Built with Claude Code by The Uncommon Business
