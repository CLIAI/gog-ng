---
name: new-criteria
description: Create a new search criteria for gog-ng and test it. Use when adding a new email filter for syncing.
argument-hint: "[criteria-name]"
allowed-tools: Bash, Read, Write, Grep, Glob
---

# Create New Search Criteria

Guide the user through creating and testing a new criteria file.

## Steps

### 1. Understand requirements

Ask the user:

* What emails should this criteria match? (sender, domain, subject, label)
* Which client(s)? (`default`, or other configured clients)
* Group name for cache organization
* Topic name (default: `default`)

### 2. Review existing criteria for reference

```bash
ls criteria/*.yaml
```

Read an existing criteria file as a template (e.g., `criteria/example-project.yaml`).

Also check global exclusions:

```bash
cat criteria/_global.yaml
```

### 3. Create the criteria file

Write `criteria/$ARGUMENTS.yaml` following this schema:

```yaml
group: "group-name"
topic: "default"
description: "Human-readable description of what this matches"
clients:
  - default              # or custom client name
include:
  domains:
    - "@example.com"
  # subjects:
  #   - "keyword"
  # addresses:
  #   - "specific@sender.com"
exclude:
  subjects: []
newer_than: "400d"
max_results: 100
# skip_global_exclusions: false  # set true to bypass _global.yaml
```

### 4. Preview the search query

```bash
gog-ng criteria $ARGUMENTS
```

This shows the constructed Gmail search query without executing.

### 5. Dry run

```bash
gog-ng sync --criteria $ARGUMENTS --dry-run
```

### 6. Test sync

```bash
gog-ng sync --criteria $ARGUMENTS -vv
```

### 7. Verify results

```bash
gog-ng list --group GROUP_NAME
git status cache/
```

### 8. Summary

Report what was created and the initial sync results.
