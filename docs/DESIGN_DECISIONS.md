# gog-ng - Design Decisions

This document captures key design decisions with alternatives considered and rationale.

---

## Architecture Overview

```
Gmail API
    |
    v
+-------------------------------------+
| cache/{client}/{group}/{topic}/     |
|                                     |
|  thread-{id}.yaml  <-- AUTHORITATIVE|
|  thread-{id}.md    <-- DERIVED      |
+-------------------------------------+
    |
    v (merge operation)
+-------------------------------------+
| cache/{clients}_merge/{group}/{topic}|
+-------------------------------------+
    |
    v (index operation)
+-------------------------------------+
| /tmp/gog-ng/index.sqlite3          |
| (ephemeral, rebuilt from YAML)      |
+-------------------------------------+
```

**Key Principles:**

* **YAML files are authoritative** - all other formats derived
* **Single-client caches are source of truth** - merge views are computed
* **SQLite is ephemeral** - rebuilt from YAML anytime
* **Git tracks everything** - enables diff-based verification

---

## DD-001: Incremental Sync Strategy

**Decision:** Replace entire thread file on sync

### Options Considered

| Option | Description | Pros | Cons |
|--------|-------------|------|------|
| **A. Replace (CHOSEN)** | Re-fetch entire thread, overwrite file | Simple, consistent state, no merge issues | Re-downloads existing messages |
| **B. Append + mark** | Add only new messages, update frontmatter | Bandwidth efficient | Complex merging, potential inconsistencies |
| **C. Configurable** | Default replace, `--append` flag | Flexibility | Implementation complexity, two code paths |

### Rationale

1. **Simplicity**: No merge logic, no partial states, no corruption risk
2. **Git as safety net**: `git diff` reveals exactly what changed between syncs
3. **Gmail API efficiency**: Thread endpoint returns all messages in single call anyway
4. **Deterministic output**: Same thread always produces same file (minus `fetched_at`)

---

## DD-002: Thread File Format (Dual YAML + Markdown)

**Decision:** Dual files per thread - YAML (authoritative) + Markdown (derived, human-readable)

| File | Purpose | Authority |
|------|---------|-----------|
| `thread-{id}.yaml` | Machine-readable, full fidelity | **AUTHORITATIVE** |
| `thread-{id}.md` | Human/LLM readable conversation | Derived (regenerable) |

### Rationale

1. **YAML = source of truth**: All metadata, exact message boundaries, original content
2. **Markdown = presentation**: Optimized for humans and LLMs reading conversations
3. **Programmatic access**: `yq`, `jq` work directly on YAML
4. **No parsing ambiguity**: YAML has clear structure, Markdown is just for viewing
5. **Regenerable**: If Markdown rendering changes, regenerate from YAML

---

## DD-003: Cache Directory Structure

**Decision:** `cache/{client}/{group}/{topic}/thread-{id}.yaml`

### Rationale

1. **Mirrors criteria structure**: criteria define group+topic -> cache organizes by group+topic
2. **Selective sync**: Can sync specific group or topic without touching others
3. **Agent-friendly**: Downstream tools grab entire `{group}/{topic}/` directory
4. **Clear ownership**: Each thread belongs to exactly one group+topic context

---

## DD-004: Criteria File Organization

**Decision:** Hybrid - global exclusions + per-criteria files

### Rationale

1. **Global exclusions**: Newsletters, noreply addresses - apply everywhere
2. **Per-criteria files**: Each use case has specific contacts and subjects
3. **Topic granularity**: Within a group, separate different communication streams
4. **`skip_global_exclusions`**: Override for cases like GitHub notifications (from `notifications@`)

---

## DD-005: Timezone Handling

**Decision:** Configurable timezone via `GOG_NG_TIMEZONE` env var, default UTC

### Rationale

1. **Cross-timezone communication**: Users communicate globally
2. **Configurable**: Different projects may prefer different display timezones
3. **Default UTC**: Unambiguous default, no locale dependency
4. **ISO 8601 in YAML**: Machine-readable dates with offset
5. **Human-friendly in Markdown**: Readable abbreviation (CET/UTC/EST)

---

## DD-006: Conditional File Update (fetched_at optimization)

**Decision:** Only overwrite thread file if content changed (ignoring `fetched_at`)

### Rationale

1. **Clean git history**: Only meaningful changes appear in commits
2. **Efficient commits**: `git status` shows actual updates only
3. **Audit-friendly**: Every git diff represents real changes
4. **Idempotent syncs**: Running sync twice produces no changes if data unchanged

---

## DD-007: ID Uniqueness and Cross-Account Safety

**Decision:** Treat all Google API IDs as account-scoped, use composite keys

### Critical Warning

Google API documentation does NOT specify whether IDs are globally unique or per-account.

### Safety Requirements

1. **Composite keys always**: Internal storage uses `(client, object_id)` tuples
2. **Collision detection**: If same raw ID appears across accounts, verify objects match
3. **Cross-account correlation by content**: Use RFC 2822 `Message-ID` header (globally unique by spec), NOT Gmail API `id` field

| Attribute | RFC 2822 Message-ID | Gmail API id |
|-----------|---------------------|--------------|
| Format | `<unique@domain>` | Opaque string `18d5abc...` |
| Uniqueness | Globally unique (by spec) | Unknown scope |
| Use case | Cross-system correlation | Within-account reference |

---

## DD-008: Merged View Across Accounts

**Decision:** Default to merged pseudo-client view combining multiple accounts

### Pseudo-Client Naming

Format: `{client1}_{client2}_merge`

### Correlation

Use RFC 2822 Message-ID for cross-account message correlation, not Gmail thread IDs.

---

## DD-009: Merge as Derived View

**Decision:** Single-client caches are authoritative; merge view is derived/computed

### Principles

1. **Single-client = source of truth**: Direct API fetch, minimal transformation
2. **Merge = computed view**: Generated from single-client data, never from API directly
3. **Redundancy = reliability**: If merge has bugs, authoritative data intact
4. **Rebuildable**: Regenerate merge from single-client caches anytime

---

## DD-010: Error Handling and Partial Sync

**Decision:** Atomic per-thread writes; continue on individual thread failures

### Behavior

| Scenario | Behavior |
|----------|----------|
| API fails mid-thread-list | Threads fetched so far are written; re-run fetches rest |
| API fails fetching thread | That thread skipped; others continue; warning logged |
| Write fails | Thread not updated; original intact; error logged |

### Why This Works

1. **Per-thread atomic writes**: Each thread file written independently
2. **Git tracks state**: `git status` shows what was updated
3. **Idempotent re-runs**: Running sync again fills gaps naturally

---

## DD-011: SQLite as Secondary Index

**Decision:** Ephemeral SQLite at configurable path (default `/tmp/gog-ng/index.sqlite3`)

### Rationale

1. **Ephemeral by design**: Can be deleted anytime, rebuilt from cache files
2. **Fast queries**: SQL more expressive than file globbing
3. **Source of truth**: YAML files in `cache/` are authoritative
4. **Temporary location**: Signals non-persistent nature
