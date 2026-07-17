# Large-File `lean_file_outline` Timeout Fallback

## Problem

`lean_file_outline` and `lean_diagnostic_messages` frequently time out on files over
~1000 lines (MCP timeout: 120s). Example: Centering.lean (1285 lines, 61KB) —
`lean_file_outline` timed out, producing no declarations.

## Rule

**Do not use `lean_file_outline` as your primary exploration tool.** It is unreliable
on large files. Use these alternatives instead, in order of preference:

### 1. `search_files` with regex on content (fastest, most reliable)

```bash
# Find a specific theorem by name
search_files(pattern="finrank_halfDer", path="E/Classification/Centering.lean")

# Find all theorem/lemma/def declarations
search_files(pattern="^theorem |^lemma |^def ", path="E/Classification/Centering.lean")

# Find section headers
search_files(pattern="^/-!|^section |^end ", path="E/Classification/Centering.lean")
```

Use `context=5` to see surrounding lines for each hit.

### 2. `read_file` with offset/limit (for structural overview)

```bash
read_file(path="Centering.lean", offset=1, limit=80)    # header: imports + doc
read_file(path="Centering.lean", offset=1200, limit=90)  # tail: main theorem
```

Read the top (imports, module doc) and bottom (main result) first, then
drill into the middle with targeted `search_files` calls.

### 3. `lean_file_outline` — only for small files (<500 lines)

When you do use it, set `max_declarations` to cap output and avoid timeouts
on borderline-large files:

```bash
lean_file_outline(file_path="...", max_declarations=30)
```

## Principle

Search first, then read — never start with `lean_file_outline` on an
unknown file.
