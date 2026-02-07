# Superpowers - Fork Maintenance

## Fork Relationship

- **Origin (push)**: `cameronsjo/superpowers`
- **Upstream (pull)**: `obra/superpowers`

## Syncing Upstream

```bash
git fetch upstream
git merge upstream/main
```

Resolve conflicts in favor of local customizations when they're in files you've intentionally modified. For everything else, prefer upstream.

## Where Customizations Live

- `skills/` — skill content modifications
- `README.md` — re-owned header and installation instructions
- `CLAUDE.md` — this file (fork-only)

## What Not to Touch When Merging

- Upstream's `.claude-plugin/plugin.json` version bumps — accept those, they track upstream releases
- Upstream's new skills — accept and review, don't auto-reject

## Local Development

Skills are markdown files in `skills/*/SKILL.md`. Test by restarting Claude Code after edits.
