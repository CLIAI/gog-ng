## temporary files
If you need to clone other repositories to introspect, or make temporary files, consider containing them under `/tmp/claude/gog-ng/`(linux) or similar place depending on system you are using.

## Local copy
git repos may have local copy under `/mnt/ro/github/${org}/${repo}`, if available try to use that.

## References

For what to look where, keep handy index, what should be searched were and how in `REFERENCES-INDEX.md`, and keep adding there if you find some interesting kind of resources what and where, why and when useful, for future handy index.

* https://github.com/gwwtests/google-api-datamodel-gogcli-mapping
* https://github.com/steipete/gogcli

## Project management

* @ai-docs/REPO_MANAGEMENT.md

## Key documentation

* `README.md` - User guide, quick start, command reference
* `CURRENT_WORK.md` - Current development status
* `FUTURE_WORK.md` - Roadmap with prioritized features (P000-P999)
* `docs/DESIGN_DECISIONS.md` - Architecture rationale (DD-001 through DD-011)
* `docs/DESIGN_NOTES.md` - Critical assumptions about Gmail API
* `docs/WISDOM.md` - Lessons learned, gotchas
* `CONTRIBUTING.md` - Development guidelines

## Tool overview

`gog-ng` is an extensionless executable (uv shebang) wrapping the `gog` CLI.
It provides cached Gmail access with criteria-based search.
Read-only by default. YAML files are authoritative, Markdown is derived.

## Claude skills available

Specialized workflows in `.claude/skills/`:

* `/sync-verify [criteria]` - Daily sync + verification
* `/debug-sync [criteria|thread-id]` - Debug sync failures
* `/new-criteria [name]` - Create new search criteria
* `/query-cache [term|SQL]` - Search cached emails
* `/gog-ng-dev` - Development reference
