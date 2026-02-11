---
name: gog-ng-dev
description: Development conventions and checklist for gog-ng changes. Loaded automatically when modifying code to ensure consistent quality.
user-invocable: true
allowed-tools: Read, Grep, Glob, Bash
---

# gog-ng Development Reference

## Before making changes

1. Read `CURRENT_WORK.md` for current status and phase
2. Check `docs/DESIGN_NOTES.md` for critical assumptions
3. Read `docs/WISDOM.md` for known gotchas

## Critical constraints

* **READ-ONLY by default** - Never add commands that send, delete, or modify emails
* **YAML is authoritative** - Markdown is always derivable from YAML
* **Composite keys** - Always use `(client, id)` for uniqueness
* **Thread mutability** - Messages can appear anytime; never assume cache completeness
* **Content-aware updates** - Only write files if content meaningfully changed

## After making changes

### Version update (3 places)

1. `gog-ng` - `@click.version_option(version="X.Y.Z")`
2. `README.md` - Version History table
3. `CURRENT_WORK.md` - Status header

### Testing checklist

```bash
gog-ng --version
gog-ng sync --criteria example-project -vv
gog-ng list
grep -E "\[.*\]\(http" cache/*/*/default/thread-*.md 2>/dev/null | head -5
gog-ng rebuild-db
gog-ng query "SELECT COUNT(*) FROM messages"
```

### Documentation update checklist

| File | When to update |
|------|---------------|
| `CURRENT_WORK.md` | Always - mark tasks complete, add new |
| `README.md` | New commands, flags, user-facing changes |
| `FUTURE_WORK.md` | Complete a phase, add planned work |
| `docs/DESIGN_NOTES.md` | Architecture or assumption changes |
| `docs/WISDOM.md` | New gotchas or lessons learned |

## Common gotchas

* `ruamel.yaml` returns `CommentedSeq`/`CommentedMap`, not plain Python types - convert before passing to `python-frontmatter`
* Gmail uses base64url (not base64) - use `base64.urlsafe_b64decode()` with padding fix
* Always use `shlex.quote()` for Gmail search queries passed to shell
* Strip `<style>`, `<script>`, `<head>`, `<table>`, `<img>` from HTML before markdownify
* `gog` CLI requires `--client` flag always

## Version numbering

* **Major**: Breaking changes to cache format or CLI interface
* **Minor**: New features, commands, significant enhancements
* **Patch**: Bug fixes, documentation, minor improvements
