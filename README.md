# ak-ai

Open-source Claude Code skills and AI tools by [Arslan Kazmi](https://github.com/arslankazmi).

A curated subset of the private `ak` plugin — skills promoted via PR after passing portability and quality checks.

---

## Install

Add the marketplace and install the plugin:

```bash
claude plugin marketplace add arslankazmi/ak-ai
claude plugin install ak-ai@ak-ai
```

Then in Claude Code, run `/reload-plugins` (or restart) and the skills appear under the `ak-ai:` namespace.

Set the `AK_ML_PROJECT` environment variable to wherever you want sandbox outputs to live:

```bash
export AK_ML_PROJECT=~/your-ml-project
```

---

## Skills

| Skill | Description |
|-------|-------------|
| `ak-ai:ml-autonomous` | Autonomous ML task supervisor — orchestrates parallel sub-agents (supervisor on Opus, sub-agents on Sonnet), verifies the result runs + tests pass + UI loads |
| `ak-ai:ml-compound` | Knowledge capture after any ML experiment or session — writes structured solution files to `docs/solutions/`, optionally to MemPalace |
| `ak-ai:ml-curate-dataset` | Build, filter, and version a training or finetuning dataset for both text (LLM) and image caption (diffusion) pipelines, with Claude-as-judge quality filtering |

Invoke any of them as `/ak-ai:<skill-name>` in a Claude Code session.

---

## Maintenance

Skills land here via the `ak:promote` workflow from the private [`arslankazmi/ak`](https://github.com/arslankazmi/ak) repo — every addition arrives as a pull request and must pass review before merging.

See [`CLAUDE.md`](./CLAUDE.md) for repo conventions.

---

## License

MIT
