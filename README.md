# ak-ai-public

Open-source Claude Code skills and AI tools, curated from the private `ak-skills` plugin.

## Install

These skills can be used directly by dropping them into your Claude Code skills directory, or installed as a plugin (see Claude Code plugin docs).

To use directly:
```bash
mkdir -p ~/.claude/skills/
cp -r skills/* ~/.claude/skills/
```

Set the `AK_ML_PROJECT` env var to a directory where the skills can write outputs (datasets, experiments, knowledge base):
```bash
export AK_ML_PROJECT=~/your-ml-project
```

## Skills

- **ml-autonomous** — autonomous ML task supervisor that decomposes a goal into parallel subtasks, dispatches specialist agents, supervises with minimum human interaction, and compounds learnings at the end.
- **ml-compound** — knowledge capture after any ML experiment or session. Writes structured solution files to `docs/solutions/` and optionally to MemPalace.
- **ml-curate-dataset** — build, filter, and version a training or finetuning dataset. Handles both text (LLM) and image caption (diffusion) pipelines with Claude-as-judge quality filtering.

## License

MIT
