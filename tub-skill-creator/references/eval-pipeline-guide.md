# Eval Pipeline Guide

This guide covers the full testing and optimization mechanics. Read this when running formal evaluations (Full or Light tier) or description optimization.

---

## Overview: The Testing Loop

The core loop is simple — repeat until satisfied:

1. Run the skill on test prompts
2. Look at the results (in the eval viewer)
3. Get feedback from the user
4. Improve the skill
5. Run again

What makes this rigorous is the infrastructure around it: parallel runs, grading, benchmarks, and a visual reviewer.

---

## Test Cases and Assertions

### Creating test cases

Save test cases to `evals/evals.json`. Each test case has:
- **id**: A unique number
- **prompt**: The realistic user message to test with
- **expected_output**: Human-readable description of success
- **files**: Optional input files the test needs
- **expectations**: Verifiable statements to check (added after first run)

```json
{
  "skill_name": "brand-voice-builder",
  "evals": [
    {
      "id": 1,
      "prompt": "I run a real estate coaching business and my content sounds like everyone else's. Help me find my voice.",
      "expected_output": "A voice guide with specific vocabulary, tone rules, and examples",
      "files": [],
      "expectations": []
    }
  ]
}
```

Start with prompts only — don't write expectations yet. Run the skill first, see what happens, then define what "good" looks like.

### Writing good assertions

Assertions are checks that can be objectively verified:
- **Good**: "The output includes at least 5 example sentences in the brand voice"
- **Good**: "The voice guide specifies words to avoid"
- **Bad**: "The output is good" (subjective, can't verify)

For subjective skills (writing, creative work), skip formal assertions. Use the eval viewer for human review instead. Don't force quantitative checks on qualitative output.

---

## Running Evaluations

### Spawning parallel runs

For **Full eval**, launch these in the SAME turn (not sequentially):
- **With-skill run**: Claude has access to the skill being tested
- **Without-skill run (baseline)**: Same prompt, no skill access

Both runs execute as subagent tasks. Launching them together means they run in parallel — cutting test time in half.

**Directory structure:**
```
skill-workspace/
└── iteration-1/
    ├── eval-1-brand-voice-builder/
    │   ├── with_skill/outputs/     ← skill's output files
    │   └── without_skill/outputs/  ← baseline output files
    ├── eval-2-email-sequence/
    │   ├── with_skill/outputs/
    │   └── without_skill/outputs/
    └── benchmark.json
```

For **Light eval**, skip the without-skill baseline. Just run with-skill on 2 test cases.

### Drafting assertions while runs execute

While the subagent runs are working in the background, draft assertions for each test case. This is productive use of wait time.

Add assertions to `eval_metadata.json` in each eval directory. Use descriptive names that explain what's being checked.

### Capturing timing data

**Critical**: When a subagent task completes, the notification includes `total_tokens` and `duration_ms`. Save these to `timing.json` immediately — they cannot be recovered after the fact.

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

---

## Grading

After runs complete, spawn a grader subagent that reads `agents/grader.md`.

The grader:
1. Reads the full execution transcript
2. Examines output files (not just what the transcript claims)
3. Grades each assertion: PASS or FAIL with specific evidence
4. Extracts and verifies claims beyond the formal assertions
5. Critiques the evals themselves (flags weak assertions)

**Critical field names** (the eval viewer depends on exact names):
- `text`: The expectation string
- `passed`: Boolean (true/false)
- `evidence`: Quote or description supporting the verdict

NOT `name`, NOT `met`, NOT `details`. Exact field names matter.

Output goes to `grading.json` in each eval directory.

---

## Benchmarking

After grading, run benchmark aggregation:

```
python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <name>
```

This produces `benchmark.json` with:
- **Per-configuration stats**: Mean, stddev, min, max for pass_rate, time, and tokens
- **Delta**: How much better (or worse) the skill performed vs. baseline
- **Analyst notes**: Patterns the aggregate numbers might hide

### What the analyst looks for

- **Non-discriminating assertions**: Passes 100% in both configs — not proving skill value
- **Always-failing assertions**: May be broken or beyond capability
- **High-variance results**: Flaky or non-deterministic — investigate before trusting
- **Time/token tradeoffs**: Skill takes longer but produces better results? Worth knowing

---

## The Eval Viewer

Generate the interactive HTML reviewer:

```
nohup python eval-viewer/generate_review.py <workspace>/iteration-N > /dev/null 2>&1 &
```

For iteration 2+, include previous results for comparison:
```
nohup python eval-viewer/generate_review.py <workspace>/iteration-N --previous-workspace <workspace>/iteration-(N-1) > /dev/null 2>&1 &
```

**For Cowork/headless environments** (no browser):
```
python eval-viewer/generate_review.py <workspace>/iteration-N --static output.html
```

### What the user sees

**Outputs tab** (one test case at a time):
- The prompt that was tested
- The skill's output
- Previous iteration's output (collapsed, for comparison)
- Formal grades with pass/fail and evidence
- Feedback textarea for notes

**Benchmark tab**:
- Pass rates for with-skill vs without-skill
- Timing and token usage
- Analyst observations

Navigate with arrow keys or prev/next buttons.

### CRITICAL: Generate the viewer BEFORE evaluating outputs yourself

Always show the user actual results in the viewer before forming opinions. The viewer is how the user sees what their skill actually produced.

### Reading feedback

After the user reviews in the viewer, they save feedback. Read `feedback.json`:
- Array of `{run_id, feedback, timestamp}`
- Empty feedback = user thought it was fine
- Focus improvements on test cases with specific complaints

---

## The Improvement Loop

### Four principles for improving skills

1. **Generalize from feedback** — Don't overfit to test cases. The skill must work across thousands of invocations. If you're stuck on a specific test case, try a different approach.

2. **Keep the prompt lean** — Read the execution transcripts (not just final outputs). Look for unproductive steps. If Claude spent time doing something that didn't help, remove the instruction that caused it.

3. **Explain the why** — If you find yourself writing "ALWAYS" or "NEVER" in all caps, that's a signal to reframe. Explain the reasoning instead. Claude responds better to context than commands.

4. **Look for repeated work** — If every test case independently wrote the same helper script, bundle it in `scripts/`. That pattern means future invocations would waste time reinventing the wheel.

### Running iterations

After improving the skill:
1. Rerun all test cases into `iteration-N+1/`
2. Use the original baseline OR the previous version as comparison
3. Launch the viewer with `--previous-workspace` for side-by-side
4. Stop when: user is happy, all feedback is empty, or no meaningful progress

---

## Blind Comparison (Advanced, Optional)

For high-stakes skills, run an independent blind comparison:

1. Give two outputs (A and B) to the comparator agent (`agents/comparator.md`)
2. The comparator doesn't know which output came from which version
3. It generates a rubric, scores both, and picks a winner
4. Then the analyzer (`agents/analyzer.md`) explains why the winner won

This removes bias from evaluation. Most skills don't need this — human review is usually sufficient. Use it when you need rigorous proof that one version is better.

---

## Description Optimization

For skills that activate automatically (not just slash commands), optimize the trigger description.

### Step 1: Generate eval queries

Create 20 realistic test queries:
- **8-10 should-trigger**: Different phrasings of requests this skill should handle. Include formal, casual, detailed, abbreviated versions. Include edge cases and adjacent domains.
- **8-10 should-not-trigger**: Near-misses that overlap in keywords but need a different skill. These are more valuable than obviously irrelevant queries.

**Queries must be realistic** — detailed, messy, with context:
- **Bad**: "Help me with content" (too abstract)
- **Good**: "ok so my boss wants me to write a weekly email newsletter but everything I write sounds like a corporate press release, how do I make it sound more human?"

### Step 2: Review with user

Use `assets/eval_review.html` — an interactive HTML template where the user can edit queries, toggle should-trigger flags, and add/remove entries. They export the final set as `eval_set.json`.

### Step 3: Run the optimization loop

```
python -m scripts.run_loop --eval-set <path> --skill-path <path> --model <model> --max-iterations 5 --verbose
```

What it does:
- Splits 60% train / 40% held-out test (stratified by should_trigger)
- Runs each query 3 times for reliable trigger rates
- Uses Claude with extended thinking to propose improved descriptions
- Re-evaluates each iteration on both train and test
- Selects the best description by **test score** (not train) to avoid overfitting
- Opens an HTML report showing results per iteration

### Step 4: Apply the result

Take the `best_description` from the output and update the SKILL.md frontmatter. Show the user before/after and report the accuracy improvement.

---

## Platform Adaptations

### Claude.ai (no subagents)

- Run skill tests yourself (sequentially, not parallel)
- Skip baseline comparisons (not meaningful without independent execution)
- Present results inline in conversation (no browser viewer)
- Skip quantitative benchmarking
- Description optimization requires CLI — skip in Claude.ai

### Cowork (no browser display)

- Subagents work fine — run parallel tests normally
- Use `--static` flag for viewer (generates standalone HTML file)
- Viewer's "Submit All Reviews" button should say "Done Reviewing" — feedback is collected conversationally
- Description optimization works via `claude -p` subprocess

---

## JSON Schema Reference

For exact field names and structures for all data files (evals.json, grading.json, timing.json, benchmark.json, comparison.json, analysis.json), see `references/schemas.md`. The eval viewer depends on exact field names — using `config` instead of `configuration` or putting `pass_rate` at the wrong nesting level will cause empty/zero values.
