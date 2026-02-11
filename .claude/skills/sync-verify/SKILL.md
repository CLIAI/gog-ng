---
name: sync-verify
description: Sync emails and verify results. Use for daily email triage workflow - syncs criteria, lists cached threads, checks git status, and verifies content quality.
argument-hint: "[criteria-name] [--all]"
allowed-tools: Bash, Read, Grep, Glob
---

# Sync and Verify Email Cache

Run the full sync + verification cycle for gog-ng.

## Arguments

* `$ARGUMENTS` can be a criteria name (e.g., `example-project`, `example-notifications`) or `--all` to sync all criteria
* If no argument given, list available criteria and ask which to sync

## Steps

### 1. Pre-sync status

```bash
gog-ng criteria --list
```

Show available criteria to the user.

### 2. Sync

If `$ARGUMENTS` is `--all`, sync each criteria in sequence. Otherwise sync the specified criteria:

```bash
gog-ng sync --criteria $ARGUMENTS -vv
```

Use `-vv` verbosity by default (shows cache status + message counts). If the user wants more detail, increase to `-vvv` or `-vvvv`.

### 3. List cached threads

```bash
gog-ng list
```

### 4. Verify content quality

Check that Markdown files have proper body content with links:

```bash
# Check for body content in recent YAML files
grep -l "body" cache/*/*/default/thread-*.yaml 2>/dev/null | head -5
grep -E "\[.*\]\(http" cache/*/*/default/thread-*.md 2>/dev/null | head -5
```

### 5. Git status

```bash
git status cache/
```

Show a summary of new/modified files. If there are changes, offer to show a brief diff.

### 6. Report

Provide a concise summary:

* Number of threads synced (new vs cached)
* Any errors or warnings
* Git status of cache directory
