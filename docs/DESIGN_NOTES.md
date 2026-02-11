# gog-ng - Design Notes & Assumptions

## Critical Assumptions About Gmail Threads

### Thread Mutability

**IMPORTANT**: Gmail threads are NOT immutable. Messages can appear at any time:

1. **New messages** - replies added to thread (most common)
2. **Older messages appearing later** - due to:
   - Delayed delivery (network issues, spam filtering delays)
   - Message re-classification (moved from spam/trash)
   - Server-side backfill or sync delays
   - Import/migration operations

**Implication**: We cannot assume "if thread exists in cache, skip entirely". We must:

* Always fetch thread metadata to get current message list
* Compare message IDs with cached messages
* Only fetch full content for NEW messages not in cache
* Re-generate derived files (.md) if any message added

### Message ID Stability

* Gmail `message.id` is stable and unique within an account
* `thread.id` is stable
* `rfc2822_message_id` (Message-ID header) is globally unique across accounts

---

## Design Philosophy

### Why This Architecture?

**Problem**: Email data is messy, APIs are quirky, and requirements evolve.

**Solution**: Layer the design so each layer can be fixed independently:

```
Layer 1: gog CLI        <- We don't control this, just wrap it
Layer 2: YAML cache     <- Authoritative, structured, queryable
Layer 3: Markdown       <- Derived, human/LLM-readable
Layer 4: SQLite index   <- Ephemeral, rebuilt anytime
```

If Layer 3 has bugs, regenerate from Layer 2. If Layer 4 is corrupted, rebuild from Layer 2.

### Human-First, Machine-Friendly

* Folder structure: `cache/{client}/{group}/{topic}/` - navigable in file browser
* File names: `thread-{id}.yaml` - sortable, greppable
* Attachment folders (future): `YYMMDD-HHmm-{subject}-{msg_id}/` - date-sorted

### Fail-Safe Defaults

* READ-ONLY by default
* Content-aware updates (don't overwrite without reason)
* Composite keys (never trust single IDs for cross-account operations)
* Git-tracked cache (can always see what changed)

### Documentation as Code

These design notes ARE the specification. If the code doesn't match, the code is wrong.
Update docs BEFORE implementing - it forces you to think through the design.

---

## API Call Optimization (Future - see FUTURE_WORK.md P110)

### Current Behavior

```
sync:
  for each thread in search results:
    gog gmail thread get {id} --full  # ALWAYS fetches full content
```

### Proposed Behavior

```
sync:
  for each thread in search results:
    if thread in cache:
      gog gmail thread get {id}  # Metadata only (no --full)
      compare message IDs
      for each NEW message:
        gog gmail message get {msg_id} --full  # Only new messages
    else:
      gog gmail thread get {id} --full  # Full fetch for new threads
```
