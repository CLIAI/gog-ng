---
name: debug-sync
description: Debug sync issues with gog-ng. Use when sync fails, threads are missing, or content looks wrong. Structured debugging workflow with maximum verbosity.
argument-hint: "[criteria-name or thread-id]"
allowed-tools: Bash, Read, Grep, Glob
---

# Debug Sync Issues

Structured debugging workflow for gog-ng sync problems.

## Steps

### 1. Identify the problem scope

Determine if `$ARGUMENTS` is a criteria name or a thread ID (thread IDs are hex strings like `19c34ebefab106e9`).

### 2. If criteria name: Run sync with maximum verbosity

```bash
gog-ng sync --criteria $ARGUMENTS -vvvv 2>&1 | head -200
```

Look for:

* API errors or timeouts
* Missing threads or messages
* Parse failures
* Shell escaping issues in search queries

### 3. If thread ID: Compare cached vs fresh data

```bash
# Find cached file
find cache/ -name "thread-$ARGUMENTS.yaml" 2>/dev/null

# Fetch fresh from API (determine client from cached path)
gog --client default --json gmail thread get $ARGUMENTS --full | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Messages: {len(d.get(\"messages\",[]))}'); [print(f'  {m[\"id\"]}: {m.get(\"snippet\",\"\")[:80]}') for m in d.get('messages',[])]"

# Compare message counts
yq '.messages | length' cache/*/*/default/thread-$ARGUMENTS.yaml 2>/dev/null
```

### 4. Check for common issues

**Thread mutability** - new messages in existing threads:

```bash
# Count messages in YAML vs API
yq '.messages[].id' cache/*/*/default/thread-$ARGUMENTS.yaml 2>/dev/null
```

**Body extraction** - missing or empty bodies:

```bash
yq '.messages[].body' cache/*/*/default/thread-$ARGUMENTS.yaml 2>/dev/null | head -20
```

**HTML conversion** - broken markdown:

```bash
yq '.messages[].body_markdown' cache/*/*/default/thread-$ARGUMENTS.yaml 2>/dev/null | head -20
```

### 5. Check docs/WISDOM.md

Read `docs/WISDOM.md` for known gotchas that match the symptoms.

### 6. Report findings

Summarize:

* Root cause (or best hypothesis)
* Affected threads/messages
* Suggested fix or workaround
