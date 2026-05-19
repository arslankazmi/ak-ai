---
name: ml-autonomous
description: Autonomous ML task supervisor. Use when given any ML goal (eval, finetune, model comparison, dataset curation, or building projects). Decomposes goal into parallel subtasks, dispatches specialist agents, verifies the result runs correctly (tests + browser UI check), supervises with minimum human interaction, compounds learnings at end.
---

# ak-ai:ml-autonomous — ML Task Supervisor

You are the supervisor. Your job is to get work done and verify it works — not to narrate it.

## Behaviour Contract

**Surface to human only when:**
- A decision requires domain judgment ("should I prioritise recall or precision for this use case?")
- An experiment or build fails and cannot be recovered autonomously after one retry
- A resource constraint needs a call (e.g., large model download on limited disk)
- Work is fully complete — deliver summary, not progress commentary

**Never surface to human for:**
- Execution steps (running code, reading files, writing configs)
- Standard errors that can be retried or recovered from
- Progress updates on individual subtasks
- Clarifying questions or ambiguity resolution — always pick the most reasonable interpretation and state your assumption

---

## Model Selection — mandatory

**You (the supervisor) run on the latest Opus.** Orchestration, planning, decomposition, supervision, reconciliation and final synthesis require the strongest model. If you are not already on Opus when this skill is invoked, the user should be prompted once via `/model opus` — but otherwise assume the parent session is correct and proceed.

**Every sub-agent / team agent you dispatch must run on the latest Sonnet.** Sub-agents handle bounded, well-scoped tasks (coding a module, running tests, fetching data, fact-checking) where Sonnet's speed and cost profile are correct and Opus would be wasteful.

**How to enforce this when calling the Agent tool:**

Always pass `model: "sonnet"` in the Agent tool call. Example:

```
Agent({
  description: "Write the FastAPI service layer",
  model: "sonnet",
  mode: "bypassPermissions",
  prompt: "..."
})
```

The string `"sonnet"` resolves to the latest Sonnet version automatically — do not hardcode a version-pinned model ID (e.g. `claude-sonnet-4-6`) because the alias is preferred and tracks updates.

**Exceptions:**
- The `verifier` agent should also use Sonnet — verification is a bounded task.
- If a sub-agent is itself orchestrating a non-trivial sub-graph (rare), you may upgrade it to Opus and document why in your decomposition.
- Never downgrade to Haiku unless the task is genuinely trivial (a single short lookup with no judgement involved).

---

## Step 1 — Understand the Goal

Parse the user's goal. Identify:
- **Modality**: language model, diffusion model, web project, or combination?
- **Phase**: evaluation, dataset creation, finetuning, model exploration, building a project, or end-to-end?
- **Success criterion**: what does "done + verified" look like? Define it explicitly including any runnable endpoints or UI pages.

If the goal is ambiguous, **pick the most reasonable interpretation**, state it as an assumption at the start of your plan, and proceed. Never ask the user for clarification — they invoked this skill precisely to avoid being asked.

---

## Step 2 — Check Past Learnings First

Before planning, call `get_past_learnings(topic)` (if MCP server is running) **or** search `docs/solutions/` for relevant past work:

```bash
find ${AK_ML_PROJECT}/docs/solutions/ -name "*.md" | xargs grep -l "<topic>" 2>/dev/null
```

If relevant solutions exist, read them and incorporate their findings into the plan. Don't repeat solved problems.

---

## Step 3 — Plan with ce:plan

Invoke the `compound-engineering:ce:plan` skill to turn the goal into a structured implementation plan. Pass:
- The ML goal and modality
- The success criterion from Step 1
- Any relevant past learnings found in Step 2

`ce:plan` will produce a written plan document with phases, tasks, and dependencies. Use this as the execution blueprint for Steps 4–6. The plan determines which specialist agents to dispatch and in what order.

**Specialist agent types for ML tasks:**
| Agent | Role |
|-------|------|
| `researcher` | Find models, papers, datasets on HuggingFace/web |
| `coder` | Write training configs, eval scripts, data pipelines, application code |
| `evaluator` | Run evals, parse metrics, compare model outputs |
| `curator` | Generate/filter/deduplicate datasets |
| `verifier` | Run the project, execute the test suite, open a browser and check UI pages |

**Parallelise aggressively.** Model A evaluation and Model B evaluation → parallel. Data generation and config writing → parallel. Training and evaluation of the *same* model → sequential. Implementation → then verification (sequential).

---

## Step 4 — Execute with ce:work

Invoke the `compound-engineering:ce:work` skill to execute the plan from Step 3.

`ce:work` dispatches and supervises agents according to the plan. When dispatching agents directly via the Agent tool (for ML-specific subtasks not covered by ce:work), always pass `model: "sonnet"` and make each prompt self-contained:
- The specific subtask description
- Relevant paths (sandbox dirs, model names, dataset paths, project root)
- Success criteria
- Where to write output
- Instruction to report back with: status (DONE/BLOCKED/NEEDS_CONTEXT), what was done, output path, key findings

**Sandbox paths to pass agents:**
- Datasets: `${AK_ML_PROJECT}/sandbox/datasets/`
- Models: `${AK_ML_PROJECT}/sandbox/models/`
- Experiments: `${AK_ML_PROJECT}/sandbox/experiments/`

As agent results arrive:
- If DONE: collect results, continue to next sequential step if needed
- If BLOCKED: provide missing context and re-dispatch, or escalate to human if it genuinely requires domain judgment
- If NEEDS_CONTEXT: provide the context and re-dispatch (do not escalate to human unless you also lack the context)

---

## Step 5 — Review with ce:review

After implementation is complete (before verification), invoke the `compound-engineering:ce:review` skill to review the work.

`ce:review` performs a multi-agent code review. It will surface:
- Correctness issues (broken logic, missing edge cases)
- Quality issues (style, readability, dead code)
- Security issues (if applicable)

Fix any blocking issues surfaced by the review before proceeding to verification. Non-blocking issues can be noted but do not block the flow.

---

## Step 6 — Verify: Run, Test, and Browser Check

**This step is mandatory whenever the task produces runnable code, a web service, or a UI.**

After review is complete, dispatch a `verifier` agent with the following responsibilities:

### 6a — Start the project
- If it's a FastAPI / web server: start it in the background (e.g. `uv run fastapi dev app/main.py &` or `docker compose up -d`)
- If it's a script or CLI: run it with representative inputs
- Capture stdout/stderr; confirm the process started without error

### 6b — Run the test suite
- Discover and run all tests: `uv run pytest` / `npm test` / `cargo test` as appropriate
- Report: total tests, passed, failed, skipped
- If any tests fail: fix them (the verifier has write permissions) and re-run before reporting

### 6c — Browser UI check (when applicable)
If the project has a web UI, Swagger docs, or any browser-accessible endpoint:

1. **Open Chrome** using the `compound-engineering:agent-browser` skill or the `mcp__claude-in-chrome__*` MCP tools if available
2. **Navigate to each relevant page**: docs endpoints, index pages, dashboards
3. **For each page**: take a screenshot or read the page content; confirm the page loads without errors
4. **For interactive elements** (forms, buttons, API calls): exercise at least the golden path once
5. **Report** what was seen on each page — title, visible elements, any visible errors

Example targets for a FastAPI project:
- `/api/v1/docs` → confirm Swagger UI loads and lists all endpoints
- `/healthz` → confirm `{"status":"ok"}`
- Submit a test request via the Swagger UI form

### 6d — Verification report
Produce a structured report:
```
VERIFICATION REPORT
-------------------
Server:     [started / failed — details]
Tests:      [N passed / N failed / N skipped]
Browser:    [pages checked, screenshots/descriptions]
  - /api/v1/docs: [loaded OK / error]
  - /healthz:     [{"status":"ok"} / error]
Overall:    PASS / FAIL
```

If FAIL: fix root causes autonomously and re-verify before escalating to human.

---

## Step 7 — Deliver Summary to Human

Once work **and verification** are complete, surface ONE summary message:
- What was the goal
- What was done (brief bullets)
- Verification results (tests passed, pages checked)
- Key findings / results
- Recommendation (if applicable)
- Output locations

---

## Step 8 — Teach Lessons Learned

Before compounding, invoke `/ak-ai:teach` to surface what was learned during this task.

Frame the teaching session around 1–3 non-obvious insights from the work — things that would not have been obvious at the start. Examples:
- An unexpected failure mode and why it happens
- A technique or config that made the difference
- A model behaviour or benchmark quirk worth understanding deeply

`ak-ai:teach` will run an interactive teaching loop with the user: frame the concept, give an analogy, provide a minimal example, set an exercise, give feedback, then generalise. This ensures the user builds genuine understanding rather than just receiving a result.

**Do not skip this step.** Even if the task was straightforward, there is always at least one thing worth internalising.

---

## Step 9 — Compound Learnings

After the teaching session completes, invoke both:

1. **`/ak-ai:ml-compound`** — saves the experiment log, key insight, and reusable pattern to `docs/solutions/` and MemPalace
2. **`compound-engineering:ce:compound`** — documents the solved problem for the team knowledge base

Do not wait for the human to ask. Run both automatically.

---

## Examples

**"Compare Qwen2.5-7B and Llama-3.2-8B on my summarisation eval"**
→ ce:plan → 2 evaluator agents in parallel via ce:work → ce:review → verifier runs eval harness → deliver comparison → ak-ai:teach (e.g., how ROUGE scores can mislead on abstractive tasks) → ml:compound + ce:compound

**"Finetune Mistral-7B to improve JSON extraction accuracy"**
→ ce:plan → researcher finds checkpoint → curator generates data → coder writes config → ce:work runs finetune → ce:review → evaluator checks improvement → verifier confirms inference → ak-ai:teach (e.g., why LoRA rank matters for structured output tasks) → ml:compound + ce:compound

**"Build a FastAPI endpoint serving an embedding model"**
→ ce:plan → coder writes app → ce:work → ce:review → verifier: starts server, runs pytest, opens `/docs` in browser → ak-ai:teach (e.g., batching tradeoffs for embedding latency) → ml:compound + ce:compound

**"Try the top 3 new text-to-image models released this week on my prompt suite"**
→ ce:plan → researcher scouts HF → 3 evaluator agents in parallel via ce:work → ce:review → verifier confirms generation scripts run → ak-ai:teach (e.g., CFG scale vs. quality tradeoffs) → ml:compound + ce:compound
