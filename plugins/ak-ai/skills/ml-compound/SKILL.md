---
name: ml-compound
description: Knowledge capture after any ML experiment, session, or discovery. Call at end of any ml:autonomous task or manually after any significant finding. Writes to docs/solutions/ and MemPalace.
---

# ml:compound — Knowledge Capture

Run this at the end of any ML session or experiment. Takes what just happened and turns it into searchable, reusable knowledge.

## Step 1 — Extract What Happened

Reflect on the current session or task. Identify:

1. **What was the goal?** (one sentence)
2. **What was tried?** (list approaches, models, configs)
3. **What worked?** (results, metrics, key findings)
4. **What didn't work?** (failures, dead ends — these are valuable too)
5. **What's the reusable insight?** (what would you tell yourself before starting?)
6. **Tags**: pick from `[eval, finetune, dataset, model-exploration, diffusion, llm, pipeline, debugging]`

If you don't have clear answers to all of these, that's fine — capture what you do know.

---

## Step 2 — Write Solution File

Create a file at:
```
${AK_ML_PROJECT}/docs/solutions/YYYY-MM-DD-<slug>.md
```

Use today's date. Slug should be 3-5 words describing the topic (e.g., `2026-05-13-qwen-json-finetune.md`).

Template:
```markdown
---
date: YYYY-MM-DD
tags: [tag1, tag2]
models: [model-name-if-applicable]
outcome: success | partial | failure
---

# <Topic Title>

## Goal
One sentence.

## What Was Tried
- Approach 1: ...
- Approach 2: ...

## Results
Key metrics or qualitative outcomes.

## Key Insight
The one thing worth remembering. What you'd do differently or the same next time.

## Reusable Pattern (if any)
Code snippet or config that worked and can be reused.
```

---

## Step 3 — Write to MemPalace

Call `mempalace_diary_write` with a summary of the session. If MemPalace MCP is not available, skip this step — the solution file is the primary record.

Format the diary entry as:
```
[YYYY-MM-DD] ML session: <goal>. Tried: <brief list>. Outcome: <result>. Insight: <key takeaway>.
```

---

## Step 4 — Check for New Reusable Skill

Ask yourself: **Is there a pattern here that I'd want as a slash command for future sessions?**

Signals that a new skill is worth creating:
- You solved a recurring problem in a non-obvious way
- You found a config or prompt pattern that generalises
- The task took multiple tries and the solution wasn't obvious

If yes: write a new skill file to `~/.claude/skills/ml-<name>.md` following the skill format (frontmatter with name/description, then clear step-by-step instructions). Keep it short — 1-2 pages max.

If no: skip.

---

## Step 5 — Report

Output a one-paragraph summary of what was compounded:
- Solution file path created
- Key insight captured
- Whether a new skill was generated
