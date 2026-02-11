# gog-ng - Enhanced gog CLI Wrapper

Wraps the [`gog`](https://github.com/steipete/gogcli) CLI to provide local caching, criteria-based search, and structured output for Gmail (with future support for Calendar and other Google services).

## Version History

| Version | Description |
|---------|-------------|
| **v0.1.0** | Initial release - ported and generalized from gogwrapper |

## Documentation Index

| Document | Purpose |
|----------|---------|
| **This file** | User guide, quick start, command reference |
| `CURRENT_WORK.md` | Current development status |
| `FUTURE_WORK.md` | Roadmap, planned features, technical debt |
| `docs/DESIGN_DECISIONS.md` | Architecture rationale (DD-001 through DD-011) |
| `docs/DESIGN_NOTES.md` | Critical assumptions (thread mutability!) |
| `docs/WISDOM.md` | Lessons learned, gotchas, best practices |
| `CONTRIBUTING.md` | Development guidelines |
| `AGENTS.md` | AI assistant instructions |

## Architecture

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
    v (merge operation - future)
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

* **YAML files are authoritative** - machine-readable, full fidelity
* **Markdown files are derived** - human/LLM-readable, regenerable
* **Single-client caches are source of truth** - merge views are computed
* **SQLite is ephemeral** - rebuilt from YAML anytime
* **Git tracks everything** - enables diff-based verification

## Installation

### Prerequisites

* Python 3.11+
* `gog` CLI installed and authenticated (see [steipete/gogcli](https://github.com/steipete/gogcli))
* `uv` installed (for PEP 723 script execution) - `pip install uv` or see [astral-sh/uv](https://github.com/astral-sh/uv)

### Install gog-ng

**Option 1: Symlink to PATH (recommended)**

```bash
# Clone the repository
git clone https://github.com/CLIAI/gog-ng ~/gog-ng

# Create symlink in ~/.local/bin (or /usr/local/bin for system-wide)
ln -s ~/gog-ng/gog-ng ~/.local/bin/gog-ng

# Verify
gog-ng --version
```

**Option 2: Copy to PATH**

```bash
cp ~/gog-ng/gog-ng ~/.local/bin/gog-ng
chmod +x ~/.local/bin/gog-ng
gog-ng --version
```

**Option 3: Run directly**

```bash
git clone https://github.com/CLIAI/gog-ng
cd gog-ng
./gog-ng --version
```

## Quick Start

```bash
# List available criteria
gog-ng criteria --list

# Preview what would be synced (dry run)
gog-ng sync --criteria example-project --dry-run

# Sync threads matching criteria
gog-ng sync --criteria example-project

# Sync with verbose output (debugging)
gog-ng sync --criteria example-project -v     # Cache status
gog-ng sync --criteria example-project -vv    # + message counts
gog-ng sync --criteria example-project -vvv   # + message IDs
gog-ng sync --criteria example-project -vvvv  # + API responses

# List cached threads
gog-ng list --client default

# Rebuild SQLite index for queries
gog-ng rebuild-db

# Query cached data
gog-ng query "SELECT subject, from_addr FROM messages WHERE date > '2025-01'"
```

## Cache Structure

```
cache/
+-- {client}/                          # Per-client cache (e.g., default, work)
|   +-- {group}/                       # Group name (organizational unit)
|       +-- {topic}/                   # Topic (e.g., default, updates)
|           +-- thread-{id}.yaml       # Authoritative data
|           +-- thread-{id}.md         # Human-readable view
+-- {clients}_merge/                   # Merged pseudo-client (future)
    +-- ...
```

## Criteria Files

Define reusable search criteria in `criteria/*.yaml`:

```yaml
# criteria/my-project.yaml
group: "my-project"
topic: "default"
description: "Project communications"

clients:
  - default

include:
  addresses:
    - "teammate@example.com"
  domains:
    - "@example.com"
  subjects:
    - "project update"

exclude:
  addresses:
    - "noreply@example.com"

newer_than: "90d"
max_results: 100
```

Global exclusions in `criteria/_global.yaml` are automatically merged (newsletters, noreply addresses, etc.).

## Commands Reference

### sync

Sync threads matching criteria to local cache.

```bash
./gog-ng sync --criteria NAME [--dry-run] [-v|-vv|-vvv|-vvvv]
```

**Verbosity levels:**

| Flag | Shows |
|------|-------|
| `-v` | Cache status (`[CACHED]/[NEW]`), yaml paths |
| `-vv` | + md path, cached/fetched message counts |
| `-vvv` | + individual message IDs |
| `-vvvv` | + full API responses (truncated to 2000 chars) |

### thread

Fetch a specific thread by ID.

```bash
./gog-ng thread --client CLIENT THREAD_ID --group GROUP [--topic TOPIC] [--force]
```

### list

List cached threads.

```bash
./gog-ng list [--client CLIENT] [--group GROUP] [--topic TOPIC] [--format table|yaml|json]
```

### criteria

Manage search criteria.

```bash
./gog-ng criteria --list           # List all criteria
./gog-ng criteria NAME             # Show criteria + generated query
```

### labels

List Gmail labels for a client.

```bash
./gog-ng labels --client CLIENT
```

### rebuild-db

Rebuild SQLite index from cached YAML files.

```bash
./gog-ng rebuild-db
```

### query

Query SQLite database directly.

```bash
./gog-ng query "SQL" [--format table|json|csv]
```

## Querying with yq/jq

YAML files are designed for easy querying:

```bash
# Get all messages from a specific sender
yq '.messages[] | select(.from == "alice@example.com")' cache/default/*/default/*.yaml

# Extract subjects and dates
yq '[.messages[] | {date: .date, subject: .subject}]' cache/default/*/default/*.yaml

# Find threads with specific participant
yq 'select(.participants[] | contains("alice"))' cache/default/*/default/*.yaml
```

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `GOG_NG_CACHE_DIR` | `./cache` | Cache directory |
| `GOG_NG_DB_PATH` | `/tmp/gog-ng/index.sqlite3` | SQLite path |
| `GOG_NG_CRITERIA_DIR` | `./criteria` | Criteria files |
| `GOG_NG_TIMEZONE` | `UTC` | Display timezone (e.g., `Europe/Warsaw`, `US/Eastern`) |

## Limitations

* **Read-only**: No send/modify operations (by design)
* **Draft exception**: Drafts can be created via `gog` directly
* **Cache staleness**: No automatic invalidation - re-run sync or use `--force`
* **Merge view**: Not yet implemented (see FUTURE_WORK.md P130)
