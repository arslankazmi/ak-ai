# ak-ai — Claude Code notes

Maintenance rules for this plugin repo. Read before making any change.

## What this repo is

A public marketplace of Claude Code skills. All changes land via pull request — branch protection blocks direct pushes to `main` and requires review.

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

- **New skill added** → add a row with `ak-ai:<name>` and a clean description
- **Skill removed** → remove its row
- **Skill description changes** → update the description column to match the SKILL.md frontmatter

Stale READMEs lie to users.

## Skill structure

Each skill is `plugins/ak-ai/skills/<name>/SKILL.md` with frontmatter:

```yaml
---
name: <name>
description: <one-line description>
---
```

## No `/Users/` paths

Skills here must be portable. No hardcoded user paths — use `${AK_ML_PROJECT}` or environment variables.

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
