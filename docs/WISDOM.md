# gog-ng - Development Wisdom & Lessons Learned

A collection of insights, gotchas, and best practices accumulated during development.

## Philosophy

### Build Incrementally, Document Continuously

This project was built in phases, each one tested and committed before moving to the next.

* Keeps the codebase always working
* Makes debugging easier (smaller change sets)
* Creates useful git history for understanding decisions
* Allows pausing and resuming work cleanly

### YAML as Source of Truth

The dual-file approach (YAML + Markdown) may seem redundant, but it's intentional:

* **YAML**: Machine-readable, exact structure, queryable with `yq`
* **Markdown**: Human-readable, LLM-friendly, includes computed fields

If you need to regenerate all Markdown files, you can do so from YAML. The reverse is not true.

### Content-Aware Updates

Don't just overwrite files - check if content meaningfully changed first:

* Keeps git history clean (no noise from timestamp-only changes)
* Makes `git diff` useful for reviewing actual changes
* Reduces filesystem writes

## Technical Gotchas

### 1. Gmail Thread Mutability (CRITICAL)

**Assumption that was WRONG**: "If a thread exists in cache, all its messages are cached."

**Reality**: Messages can appear in threads at any time, even older ones.

**Solution**: Always fetch thread metadata, compare message IDs, only fetch content for NEW messages.

### 2. ruamel.yaml Types vs Standard Python

`ruamel.yaml` returns special types (`CommentedSeq`, `CommentedMap`) that don't always work with other libraries.

**Solution**: Convert to plain Python types before passing to frontmatter:

```python
def to_plain_python(obj):
    if hasattr(obj, 'items'):
        return {k: to_plain_python(v) for k, v in obj.items()}
    elif hasattr(obj, '__iter__') and not isinstance(obj, (str, bytes)):
        return [to_plain_python(i) for i in obj]
    return obj
```

### 3. HTML Email Cleanup

Email HTML is messy. We strip: `<style>`, `<script>`, `<head>`, layout tables, `<img>` tags, HTML comments.

**Goal**: Readable content, not faithful HTML rendering.

### 4. Gmail Search Query Escaping

Complex Gmail queries with spaces need proper shell escaping.

**Solution**: Use `shlex.quote()` and `shell=True`.

### 5. Base64url vs Base64

Gmail API uses base64url encoding (not standard base64): `-` and `_` instead of `+` and `/`, may omit padding.

**Solution**: Use `base64.urlsafe_b64decode()` with padding fix.

## Quotes Worth Remembering

> "Gmail threads are NOT immutable. Messages can appear at any time."

> "YAML is authoritative. Markdown is derived."

> "Only update if content meaningfully changed."

> "Composite keys: always use (client, id) for uniqueness."

> "READ-ONLY by default."
