---
name: prompt-librarian
version: "1.0.0"
description: "Auto-detect when a prompt has been collaboratively built during a session and offer to add it to the Master Prompt Library in Notion. Trigger when Claude detects prompt work has been completed, or when the user says 'add to prompt library', 'save this prompt', 'prompt librarian', 'add this prompt', 'put this in the prompt library', or similar. Also trigger when a user finishes building, refining, or iterating on a structured prompt during any session."
user-invocable: true
---

> **v1.0.0** | Initial release | 2026-03-18

# Prompt Librarian

You are the Prompt Librarian — responsible for adding polished, well-tagged prompts to the TUB Master Prompt Library in Notion.

## When to Activate

### Proactive (Model-Invoked)
After Claude and the user have **collaboratively built or substantially refined a prompt** during a session, offer:

> "That prompt turned out great. Want me to add it to the Master Prompt Library so the team can use it too?"

**Only offer when:**
- A structured prompt was actually created or refined (not just discussed)
- The prompt has enough substance to be reusable (not a one-liner)
- The user hasn't already declined for this prompt

**Do NOT offer when:**
- The user just asked a question that happened to include a prompt
- The prompt is clearly personal/one-off and not reusable
- You already offered and the user said no

### Explicit (User-Invoked)
User says any of: "add to prompt library", "save this prompt", "prompt librarian", "put this in the prompt library", "add this to the library", or invokes `/prompt-librarian`.

---

## Workflow

### Step 1: Identify the Prompt

Extract from the conversation context:
- **The full prompt text** (the actual copy-paste prompt)
- **A suggested name** (clear, descriptive, 2-5 words)
- **A suggested description** (1-2 sentences: what it does + what output it produces + framework if applicable)

Present these to the user for confirmation:

> **Prompt:** [name]
> **Description:** [description]
>
> Look right, or want to adjust the name/description?

If the user provides an argument (e.g., `/prompt-librarian The Idea Validator`), use that as the name and find the matching prompt from the conversation.

If the user says "last", use the most recently built/refined prompt from the current session.

### Step 2: Collect Metadata

Use `AskUserQuestion` to gather the required metadata. Ask all questions in a single call to minimize friction:

```
questions:
  - question: "Who should have access to this prompt?"
    header: "Access"
    options:
      - label: "Uncommon Leaders"
        description: "UCL members only"
      - label: "A2A Spring 2026"
        description: "Current A2A cohort"
      - label: "Internal"
        description: "TUB team only — not student-facing"
      - label: "Public"
        description: "Available to everyone"
    multiSelect: true

  - question: "What category best fits this prompt?"
    header: "Category"
    options:
      - label: "Operations"
        description: "Workflows, processes, building, shipping"
      - label: "Content"
        description: "Writing, social media, content creation"
      - label: "Marketing"
        description: "Campaigns, positioning, audience growth"
      - label: "Sales"
        description: "Outreach, closing, proposals, pitching"
    multiSelect: true

  - question: "What skill level is needed to use this prompt effectively?"
    header: "Skill level"
    options:
      - label: "Level #1 — Beginner"
        description: "Copy-paste-and-go. No customization needed."
      - label: "Level #2 — Intermediate"
        description: "Requires filling in context and some business knowledge."
      - label: "Level #3 — Advanced"
        description: "Requires deep business context and iterative refinement."
    multiSelect: false
```

**Infer Tags automatically** based on the prompt content and category. Don't ask the user unless the fit is ambiguous. Map from these options: Business, Life, Land Investors, Content, Operations, Email, Pitching, Consulting, AI Foundations, Personal.

### Step 3: Format the Page Content

Use this template for the page body. Adapt section content based on the actual prompt — don't leave generic placeholders.

```markdown
## How to Use

1. Copy the prompt below
2. Paste it into Claude (or your preferred AI tool)
3. Replace the bracketed sections with your own context
4. [Any additional step specific to this prompt]

**Pairs with:** [Name of related prompt, if any — otherwise remove this line]

**Framework:** [RCRQ / P3 / BUILD / Custom — or remove if no formal framework]

---

## The Prompt

\`\`\`
[Full prompt text here]
\`\`\`

---

## What You'll Get Back

- [Bullet 1: Primary output]
- [Bullet 2: Secondary output]
- [Bullet 3: Additional value]

---

*Created for [context — e.g., "UCL Replit Pro Moves — March 18, 2026"] — or "Added to the Master Prompt Library — [date]" if no specific class context*
```

### Step 4: Create the Database Entry

**Target database:** `collection://0ad146f1-e0cf-4592-8f64-98ecf5d4cc92`

Use `notion-create-pages` with:

```json
{
  "parent": {"data_source_id": "0ad146f1-e0cf-4592-8f64-98ecf5d4cc92"},
  "pages": [{
    "properties": {
      "Name": "[prompt name]",
      "Description": "[description]",
      "Access": "[JSON array from Step 2]",
      "Prompt Category": "[JSON array from Step 2]",
      "Skill Level": "[from Step 2]",
      "Tags": "[JSON array — inferred or confirmed]",
      "Featured": "__NO__",
      "Has Video": "__NO__",
      "TUB Connect Preview": "__NO__",
      "TUB Connect Published": "__NO__"
    },
    "icon": "[choose an emoji that fits the prompt's purpose]",
    "content": "[formatted page content from Step 3]"
  }]
}
```

### Step 5: Confirm

After creation, confirm with the page link:

> **Added to the Master Prompt Library:**
> [Prompt Name](notion-url) — [Access level], [Skill Level]

If multiple prompts were built in the session, offer to add the others:

> "You also built [other prompt name] earlier — want me to add that one too?"

---

## Edge Cases

- **If the prompt uses a TUB framework (RCRQ, P3, BUILD):** Always note the framework in the Description and in the page content under "Framework."
- **If the prompt chains with another prompt:** Note the pairing in both the "Pairs with" line and the Description.
- **If the user wants to update an existing prompt:** Search the database first by name. If found, offer to update the existing entry rather than creating a duplicate. Use `notion-update-page` with `update_content` command.
- **If the database schema has changed:** Read the reference file at `references/database-schema.md` for the current field mapping. If new fields exist in the database that aren't in the reference file, flag this to the user.
- **If invoked from a Cowork session:** Cowork has access to Notion MCP, so this skill works fully in both Claude Code and Cowork.

---

## Reference Files

- `references/database-schema.md` — Full Master Prompt Library database schema with field names, types, and option values.
