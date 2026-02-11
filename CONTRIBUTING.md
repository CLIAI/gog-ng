# Contributing to gog-ng

## Spirit of This Project

This tool is built for **practical email management** - helping users efficiently cache, search, and organize Gmail data. Design decisions favor:

* **Reliability over cleverness** - Simple, debuggable code
* **Human readability** - Folder structures you can navigate, files you can read
* **Machine parseability** - Structured data for automation
* **Incremental improvement** - Small, tested changes over big rewrites
* **Clear documentation** - Future contributors should understand why

## Development Guidelines

### Code Style

* Python 3.11+ with type hints
* Use `click` for CLI interface
* Use `ruamel.yaml` for YAML (not PyYAML) - deterministic output
* PEP 723 for script dependencies (uv shebang)

### Version Numbering

Format: `v{major}.{minor}.{patch}`

* **Major**: Breaking changes to cache format or CLI interface
* **Minor**: New features, commands, or significant enhancements
* **Patch**: Bug fixes, documentation updates, minor improvements

Update version in:

1. `gog-ng` - `@click.version_option(version="X.Y.Z")`
2. `README.md` - Version History table
3. `CURRENT_WORK.md` - Status header

### Critical Design Rules

1. **YAML is authoritative** - Markdown is derived, can be regenerated
2. **Content-aware updates** - Only write if content meaningfully changed (ignore `fetched_at`)
3. **Thread mutability** - Gmail threads can have messages added at any time
4. **Composite keys** - Always use `(client, id)` for uniqueness
5. **READ-ONLY by default** - Never send, delete, or modify emails via `gog`

### Testing Workflow

```bash
# Verify tool works
./gog-ng --version
./gog-ng criteria --list

# Test sync with dry run
./gog-ng sync --criteria example-project --dry-run

# Test sync with verbose output
./gog-ng sync --criteria example-project -vv

# Verify outputs
./gog-ng list
./gog-ng rebuild-db
./gog-ng query "SELECT COUNT(*) FROM messages"
```

### Debugging Tips

```bash
# Verbose sync to see what's happening
./gog-ng sync --criteria example-project -vvvv

# Check gog CLI directly
gog --client default --json gmail thread get THREAD_ID --full | jq .

# Inspect cached YAML
yq '.messages[0]' cache/default/*/default/*.yaml | head -50
```

## Error Handling Philosophy

* **Fail loudly** - Don't silently skip errors
* **Continue when safe** - One failed thread shouldn't stop entire sync
* **Log errors to stderr** - Keep stdout clean for data
* **Provide context** - Include thread ID, client, what was attempted
