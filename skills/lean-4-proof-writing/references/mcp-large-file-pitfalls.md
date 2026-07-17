# MCP Tool Pitfalls for Large Lean Files

## lean_file_outline timeout

`lean_file_outline` (and full-mode `lean_diagnostic_messages`) frequently
times out on files over ~1000 lines. The 120s default timeout is insufficient.
Centering.lean (1285 lines, 61KB) is a known offender.

### Symptoms
- `MCP call timed out after 120.0s`
- Works fine on small files (<500 lines), hangs on large ones

### Workaround
1. Use `read_file` with `offset` and `limit` to read the file header (first
   60-100 lines) to understand its structure and imports
2. Use `search_files` to find specific declarations by name within the file
3. For targeted inspection, use `lean_declaration_file`, `lean_hover_info`,
   or `lean_goal` on specific lines instead of scanning the whole file
4. Reserve `lean_file_outline` for files under ~500 lines

### Related pattern
When exploring a large project, prefer `search_files(target='files',
pattern='*.lean')` to discover the file listing, then `read_file` to
inspect individual files. Avoid `lean_file_outline` as a discovery
tool on large files.
