# gog-ng - Future Work

## P100: Core Gmail Wrapper (Ported from gogwrapper)

* [x] Core CLI framework (Click-based, uv shebang)
* [x] `sync` command - fetch emails matching criteria to local cache
* [x] `thread` command - fetch specific thread by Gmail ID
* [x] `list` command - list cached threads with filtering
* [x] `criteria` command - manage search criteria
* [x] `labels` command - list Gmail labels
* [x] `rebuild-db` command - rebuild SQLite index from YAML
* [x] `query` command - SQL queries against SQLite index
* [x] Dual YAML (authoritative) + Markdown (derived) cache format
* [x] Content-aware updates (ignore fetched_at in diff)
* [x] Multi-client support per criteria
* [x] HTML-to-Markdown conversion with link preservation
* [x] Verbose logging (-v to -vvvv)
* [x] Global exclusions system (_global.yaml)

## P110: Message-Level Caching Optimization

* [ ] Fetch thread metadata first (without --full)
* [ ] Compare message IDs with cached YAML
* [ ] Only fetch --full for NEW messages not in cache
* [ ] Handle out-of-order message appearance (Gmail threads are mutable)
* [ ] Add --force flag to bypass optimization

## P120: Attachment Download

* [ ] Criteria option: `download_attachments: true`
* [ ] Size limits: `max_attachment_size: 10MB`
* [ ] MIME filtering: `attachment_types: ["pdf", "doc*"]`
* [ ] Storage: `cache/{client}/{group}/{topic}/attachments/{YYMMDD}-{HHmm}-{subject}-{msg_id}/`
* [ ] `_manifest.yaml` with checksums (md5, sha256) for deduplication
* [ ] `_message.md` for self-contained context per attachment folder
* [ ] Skip download if attachment already exists

## P130: Merge Views Across Accounts

* [ ] Pseudo-client "merged" computed from single-client caches
* [ ] RFC 2822 Message-ID correlation across accounts
* [ ] ID collision detection and warnings
* [ ] Cross-account thread linking
* [ ] Provenance tracking in merged files

## P200: Calendar Integration

* [ ] Calendar event caching via `gog calendar` commands
* [ ] Event-to-Email correlation (meeting invites)
* [ ] Unified timeline view
* [ ] iCalUID-based cross-calendar correlation

## P210: Contacts Integration

* [ ] Contact caching via `gog contacts` commands
* [ ] Contact-to-Email correlation
* [ ] Contact group management

## P300: Plugin/Hook System for Downstream Repos

* [ ] Config-driven hook system (pre-sync, post-sync, on-new-thread, etc.)
* [ ] Hook definitions in config file (shell commands, scripts)
* [ ] Template system for downstream repos to customize behavior
* [ ] Example hook: auto-label new threads based on criteria
* [ ] Example hook: notify on new messages matching patterns

## P310: Configuration File System

* [ ] Central config file (`~/.config/gog-ng/config.yaml` or project-local)
* [ ] Timezone configuration (default: UTC, overridable per-project)
* [ ] Default client configuration
* [ ] Custom cache directory per project
* [ ] Criteria directory overrides

## P400: Advanced Querying

* [ ] Full-text search in SQLite (FTS5)
* [ ] Label-based filtering
* [ ] Date range queries with human-friendly syntax
* [ ] Export to JSONL for LLM context

## P410: Incremental Sync

* [ ] historyId tracking per client
* [ ] Incremental updates (only fetch changes since last sync)
* [ ] Cache invalidation on label changes
* [ ] Conflict detection

## P500: Capabilities-Based Permission System

* [ ] `--capabilities=CAPS` CLI flag accepting comma-separated capability list
* [ ] Each capability defined in settings file for user fine-tuning
* [ ] Initial settings file generation (`gog-ng init-capabilities`)
* [ ] Default capability taxonomy:
  * `read-only` - only read operations (list, query, sync to local cache)
  * `writable` - can create drafts, modify local files
  * `writable-user-scoped-impact` - changes affecting only the user's data
  * `writable-shared-group-impact` - changes visible to a group (shared labels, etc.)
  * `writable-global-impact-like-sending-emails` - operations with external impact (send email, etc.)
* [ ] Taxonomy and ontology refinement needed - the above is a starting point
* [ ] Per-command capability requirements (e.g., `sync` needs `read-only`, `draft` needs `writable`)
* [ ] Capability validation before command execution
* [ ] Config file: `~/.config/gog-ng/capabilities.yaml`

## P600: Export Formats

* [ ] `gog-ng export --format mbox` - Standard mbox format
* [ ] `gog-ng export --format eml` - Individual .eml files
* [ ] `gog-ng export --format jsonl` - JSONL for streaming/LLM processing

## P700: Watch Mode

* [ ] `gog-ng watch --criteria NAME --interval 5m`
* [ ] Poll for new messages matching criteria
* [ ] Desktop notification integration (optional)

## P800: Thread Summarization

* [ ] `gog-ng summarize THREAD_ID`
* [ ] Generate LLM-friendly thread summary with key points
* [ ] Configurable summary templates

## P900: Draft Creation Helper

* [ ] `gog-ng draft --reply-to MESSAGE_ID --template templates/reply.md`
* [ ] Template system for common reply patterns
* [ ] Requires `writable` capability (P500)

## Technical Debt

* [ ] Add proper logging (structlog)
* [ ] Add retry logic for transient API failures
* [ ] Add cache size limits and cleanup
* [ ] Add cache integrity verification
* [ ] Add migration support for format changes
* [ ] Test suite

## Won't Do (By Design)

* **Direct email sending** - User must send manually via Gmail UI (unless explicitly enabled via capabilities P500)
* **Modify labels directly** - Read-only by default
* **Delete messages** - Read-only by default
* **OAuth token management** - Delegated to `gog auth`
