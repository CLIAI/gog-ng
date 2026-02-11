# gog-ng - Current Work

**For new assistants**: This file tracks current development status. Start here to understand what's done and what's next.

## Status: v0.1.0 - Initial Release (Ported from gogwrapper)

**Last updated**: 2026-02-11

**What works:**

* Full Gmail sync with body extraction (text, HTML, markdown)
* YAML (authoritative) + Markdown (derived) cache format
* Criteria-based search with global exclusions
* Verbose logging for debugging (`-v` to `-vvvv`)
* SQLite index for queries
* Multi-client support

**What's next**: See FUTURE_WORK.md for prioritized roadmap

## Origin

This tool was ported and generalized from `gogwrapper.py` (v0.5.2) in a private repository. Key changes from the source:

* Renamed from `gogwrapper.py` to `gog-ng` (extensionless executable)
* Environment variables: `GOGWRAPPER_*` -> `GOG_NG_*`
* "lead" concept renamed to "group" (more generic)
* Timezone made configurable (env var `GOG_NG_TIMEZONE`, default UTC)
* Default client changed from domain-specific to `"default"`
* All domain-specific language removed from code and docs
* DB path: `/tmp/gog-ng/index.sqlite3`
* Version reset to 0.1.0

## Completed

* [x] Port core CLI framework from gogwrapper
* [x] Generalize all domain-specific language
* [x] Create example criteria templates
* [x] Port and generalize documentation
* [x] Set up FUTURE_WORK.md with roadmap
* [x] Create .gitignore
