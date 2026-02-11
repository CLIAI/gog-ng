# References Index

Handy index of what to search where and why.

## Core Dependencies

| Resource | What | When useful |
|----------|------|-------------|
| [steipete/gogcli](https://github.com/steipete/gogcli) | The `gog` CLI tool we wrap | Understanding available commands, flags, output formats |
| [gwwtests/google-api-datamodel-gogcli-mapping](https://github.com/gwwtests/google-api-datamodel-gogcli-mapping) | Google API data model mapping | Understanding Gmail API structure, ID uniqueness, field semantics |

## Google API Documentation

| Resource | What | When useful |
|----------|------|-------------|
| Gmail API Threads | Thread structure, message lists | Understanding thread mutability, message ordering |
| Gmail API Messages | Message format, MIME parts | Body extraction, attachment handling |
| Gmail API Search | Query syntax | Building criteria queries (`from:`, `subject:`, `newer_than:`) |

## Python Libraries

| Library | What | When useful |
|---------|------|-------------|
| [ruamel.yaml](https://yaml.readthedocs.io/) | YAML library with round-trip support | Deterministic YAML output, preserving formatting |
| [python-frontmatter](https://python-frontmatter.readthedocs.io/) | Parse/write YAML frontmatter in Markdown | Thread Markdown generation |
| [sqlite-utils](https://sqlite-utils.datasette.io/) | SQLite helper library | Database indexing, queries |
| [click](https://click.palletsprojects.com/) | CLI framework | Command definitions, options, arguments |
| [markdownify](https://github.com/matthewwithanm/python-markdownify) | HTML to Markdown conversion | Email body conversion |

## Design References

| Resource | What | When useful |
|----------|------|-------------|
| RFC 2822 | Internet Message Format | Message-ID uniqueness, cross-account correlation |
| RFC 5322 | Internet Message Format (update) | Header field specifications |
| base64url encoding | Gmail API body encoding | Decoding email bodies (differs from standard base64) |
