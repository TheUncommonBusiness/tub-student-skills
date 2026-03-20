---
name: tub-skills-browser
version: "1.0.0"
description: "Browse all available TUB student skills. Use when someone says /tub-skills, what skills do I have, what tools are available, show me my skills, list skills, browse skills, what can Claude do, or help me find a skill. Shows a clean table of every student skill with name, description, and how to invoke it."
user-invocable: true
---
> **v1.0.0** | Built by Carter Jensen | 2026-03-20

# TUB Skills Browser (Student Edition)

When invoked, display a clean, scannable list of all available student skills. Read each loaded skill from the current session context — do not hardcode a list.

## How to Display

Show the skills in a friendly, approachable format:

```
Your TUB Skills

These skills are custom tools built by The Uncommon Business to help you
build, grow, and automate your business. Type any slash command to use one.

BUILD & VALIDATE
  /idea-validator    — Pressure-test a product idea before building it.
                       Get a BUILD, RESHAPE, or SHELF verdict with scoring.

  /skill-creator     — Build your own custom Claude skills for your business.
                       Full guided process from idea to finished skill.

BROWSE
  /tub-skills        — See this list again anytime.

More skills are added regularly. Ask "what skills do I have?" to check for new ones.
```

## Important Behavior

1. **This is a REFERENCE display, not a hardcoded list.** The table above is an example format. When actually displaying, read the skills that are currently loaded in the session and show those. New skills added to the student marketplace appear automatically.

2. **Keep it friendly and approachable.** Students may be new to Claude and skills. Use plain language. Briefly explain what each skill does — do not assume they know.

3. **After showing the list**, ask: "Want to try one? Just type the slash command or tell me what you are working on and I will suggest the right skill."

4. **If a student asks about a specific skill**, read that skill and give a 2-3 sentence plain-language summary of what it does and when to use it.

5. **Only show student-audience skills.** Do not display internal team skills even if they happen to be loaded. Filter by audience: show only skills with `audience: "student"` or `audience: "all"`.

6. **Works on all platforms** — Claude Code, Cowork, and Claude.ai. No CLI tools needed.
