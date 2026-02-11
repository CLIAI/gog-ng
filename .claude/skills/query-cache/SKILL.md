---
name: query-cache
description: Query cached emails using SQL or yq. Use for searching, filtering, and analyzing cached email data.
argument-hint: "[search-term or SQL query]"
allowed-tools: Bash, Read, Grep, Glob
---

# Query Email Cache

Search and analyze cached email data using multiple methods.

## Determine query method from `$ARGUMENTS`

### If argument looks like SQL (contains SELECT, FROM, WHERE, etc.)

Run directly:

```bash
gog-ng query "$ARGUMENTS"
```

### If argument is a search term or keyword

Use multiple approaches:

**1. Grep through Markdown files (fastest for text search):**

```bash
grep -ril "$ARGUMENTS" cache/*/*/default/thread-*.md 2>/dev/null | head -20
```

**2. Search with yq through YAML (structured search):**

```bash
for f in cache/*/*/default/thread-*.yaml; do
  matches=$(yq ".messages[] | select(.subject // \"\" | test(\"$ARGUMENTS\"; \"i\")) | .subject" "$f" 2>/dev/null)
  if [ -n "$matches" ]; then
    echo "=== $f ==="
    echo "$matches"
  fi
done
```

**3. SQL query via SQLite (if index exists):**

```bash
gog-ng rebuild-db 2>/dev/null
gog-ng query "SELECT thread_id, subject, from_addr, date FROM messages WHERE subject LIKE '%$ARGUMENTS%' OR body LIKE '%$ARGUMENTS%' ORDER BY date DESC LIMIT 20"
```

### If no argument provided

Show useful example queries:

```
Available query patterns:
  /query-cache "company name"           - Search all fields
  /query-cache "SELECT COUNT(*) ..."    - Direct SQL
  /query-cache from:recruiter@co.com    - Search by sender
  /query-cache subject:interview        - Search by subject
```

Also rebuild the database if stale:

```bash
gog-ng rebuild-db
gog-ng query "SELECT COUNT(*) as total_messages FROM messages"
gog-ng query "SELECT group_name, COUNT(*) as threads FROM threads GROUP BY group_name"
```

## Output

Present results in a readable table format. Include thread IDs so the user can drill into specific threads.
