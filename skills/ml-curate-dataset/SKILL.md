---
name: ml-curate-dataset
description: Build, filter, and version a training or finetuning dataset. Handles both text (LLM) and image caption (diffusion) pipelines. Use when you need training data for a model.
---

# ml:curate-dataset — Dataset Curation Pipeline

Produces a clean, versioned, ready-to-train dataset. Works for both language model finetuning (JSONL prompt/completion pairs) and diffusion model training (image captions with style tags).

## Step 1 — Clarify the Dataset Goal

Identify:
- **Task**: what capability should the model learn? (e.g., "JSON extraction from messy text", "photorealistic portrait generation in renaissance style")
- **Modality**: "text" for LLMs, "image_captions" for diffusion
- **Size**: how many examples? Default 500 for text, 200 for image captions
- **Source**: generate synthetic data, filter existing data, or both?

If both generating and filtering: generate first, then filter.

---

## Step 2A — Generate Synthetic Data (if needed)

If MCP server is running, use `generate_dataset` tool:
```
generate_dataset(
  task_desc="<task description>",
  n=<count>,
  modality="text" | "image_captions",
  output_name="<descriptive-name>"
)
```

If MCP server is not available, use the Claude API directly:
- For text: generate prompt/completion pairs as JSONL, 10 per API call, write to `sandbox/datasets/<name>/data.jsonl`
- For image captions: generate caption + style tag pairs, write to same format

**Quality bar:** Every generated example should be specific, varied, and realistic. Avoid repetitive or generic examples.

---

## Step 2B — Filter Existing Data (if you have a dataset)

If filtering an existing JSONL file, use `filter_dataset` tool:
```
filter_dataset(path="sandbox/datasets/<name>/data.jsonl", quality_threshold=0.7)
```

Manual filtering criteria to check:
- Remove examples shorter than 20 tokens (too thin)
- Remove near-duplicates (check for >80% token overlap)
- Check label/format consistency

---

## Step 3 — Inspect the Dataset

Always inspect before declaring done:
```bash
head -5 sandbox/datasets/<name>/data_filtered.jsonl | python3 -c "import sys,json; [print(json.dumps(json.loads(l), indent=2)) for l in sys.stdin]"
```

Check:
- Format is correct (has expected keys)
- Examples are diverse (not repetitive)
- No obviously malformed entries

---

## Step 4 — Create Dataset Card

Write `sandbox/datasets/<name>/README.md`:
```markdown
# <Dataset Name>

**Task**: <what this trains>
**Modality**: text | image_captions
**Size**: N examples (raw: X, filtered: Y, removal rate: Z%)
**Created**: YYYY-MM-DD
**Quality threshold**: 0.7

## Format
Each line is JSON with keys: `prompt`, `completion`

## Generation
<Brief description of how it was created>
```

---

## Step 5 — Report

Output:
- Dataset path
- Final count
- Quality removal rate
- 2-3 example rows
- Recommended next step (e.g., which finetuning backend to use)

Then call `ml:compound` to save the dataset creation approach as a learning.
