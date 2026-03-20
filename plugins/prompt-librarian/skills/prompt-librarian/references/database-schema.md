# Master Prompt Library — Database Schema

**Database URL:** `https://www.notion.so/97a29d66c4bc4563899c3da6d7585954`
**Data Source ID:** `collection://0ad146f1-e0cf-4592-8f64-98ecf5d4cc92`

## Fields

### Name (title) — REQUIRED
The prompt name. Should be clear and descriptive (e.g., "The Idea Validator", "Brand Voice Extractor").

### Description (text) — REQUIRED
1-2 sentence summary of what the prompt does and what output it produces. Include the framework used if applicable (e.g., "Uses RCRQ framework.").

### Access (multi_select)
Who can see this prompt. Options:
- `Public`
- `Boot Camp`
- `SuperHuman Work`
- `A2A Fall 2025`
- `Uncommon Leaders`
- `Internal`
- `SuperHuman Plus`
- `Foundations of AI`
- `A2AFall-2025`
- `A2A Spring 2026`

### Prompt Category (multi_select)
What business function this prompt serves. Options:
- `Content`
- `Sales`
- `Hiring / HR`
- `Operations`
- `Finance`
- `Research`
- `Marketing`
- `Mindset`
- `Life / Personal`
- `Leadership`

### Skill Level (select)
Complexity tier. Options:
- `Level #1` — Beginner-friendly, copy-paste-and-go
- `Level #2` — Intermediate, requires some customization
- `Level #3` — Advanced, requires business context and iteration

### Tags (multi_select)
Topic tags. Options:
- `Business`
- `Life`
- `Land Investors`
- `Content`
- `Operations`
- `Email`
- `Pitching`
- `Consulting`
- `AI Foundations`
- `Personal`

### Prompt Pack (multi_select)
Optional grouping into themed packs. Options:
- `Mindset & Confidence in Business Growth`
- `Client Retention & Scaling Recurring Revenue`
- `Time Management & Scaling Without Burnout`
- `Offer Positioning & Pricing`
- `Content Marketing & Visibility`
- `Sales & Closing Deals`
- `Lead Generation & Client Acquisition`

### Featured (checkbox)
Set to `__NO__` by default. Only mark `__YES__` if user explicitly requests it.

### Has Video (checkbox)
Whether a walkthrough video exists. Default `__NO__`.

### Video Link (url)
URL to walkthrough video if Has Video is checked.

### TUB Connect Preview (checkbox)
Whether the prompt is in preview/staging on TUB Connect. Default `__NO__`.

### TUB Connect Published (checkbox)
Whether the prompt is published to TUB Connect. Default `__NO__`.

### AI Summary (text)
Auto-generated summary. Optional — can be filled by Claude if useful.

### Created By / Assigned To (person)
Notion person field. Cannot be set programmatically — will be auto-assigned based on session context.
