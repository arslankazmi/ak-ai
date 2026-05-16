# ak-ai — Claude Code notes

Maintenance rules for this public plugin repo. Read before making any change.

## What this repo is

The public, open-source counterpart to the private [`arslankazmi/ak`](https://github.com/arslankazmi/ak) plugin. Skills land here exclusively via the `ak:promote` workflow — never by direct push to `main`. Branch protection enforces PR-based merges.

## Structure

This is a Claude Code **marketplace** — not a drop-in skills folder. The structure must remain:

```
ak-ai/
├── .claude-plugin/
│   └── marketplace.json     ← lists the ak-ai plugin
├── plugins/
│   └── ak-ai/
│       ├── .claude-plugin/
│       │   └── plugin.json  ← plugin manifest, version
│       └── skills/
│           └── <skill>/SKILL.md
├── README.md
├── CLAUDE.md
└── LICENSE
```

If you move skill files out of `plugins/ak-ai/skills/`, `claude plugin install` will break.

## ⚠️ Always update `README.md` on every change

The skill table in `README.md` is the user-facing index. Any change to the skill set **must** update it in the same commit:

- **New skill promoted** → add a row with `ak-ai:<name>` and a clean description
- **Skill removed** → remove its row
- **Skill description changes** → update the description column to match the SKILL.md frontmatter

Stale READMEs lie to users.

## Skill structure

Each skill is `plugins/ak-ai/skills/<name>/SKILL.md` with frontmatter:

```yaml
---
name: <name>
description: <one-line description — NO 🌐 prefix, no public: flag here>
---
```

Public copies are clean — the `public: true` flag and `🌐` prefix are private-repo concerns. The `ak:promote` skill strips them on the way in.

## No `/Users/` paths

Skills here must be portable. The `ak:promote` script greps for `/Users/` before opening a PR and blocks the promotion if any leak. If you're editing a skill directly here (rare), follow the same rule.

## Versioning

Plugin version is in `plugins/ak-ai/.claude-plugin/plugin.json`. Bump it when shipping breaking changes; a minor bump per release is fine.

## Reload after install

After `claude plugin install ak-ai@ak-ai`, users should run `/reload-plugins` in any active Claude Code session.

## Checklist before merging any PR

- [ ] Branch protection rules satisfied (PR + 1 review)
- [ ] No `/Users/` paths in any new or modified SKILL.md
- [ ] README skill table reflects the change
- [ ] `plugin.json` version bumped if appropriate
- [ ] All skill files live under `plugins/ak-ai/skills/`, not at repo root
