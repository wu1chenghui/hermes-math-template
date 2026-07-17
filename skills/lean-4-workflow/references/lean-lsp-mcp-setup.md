# lean-lsp-mcp Setup for Hermes Agent

## Overview

[lean-lsp-mcp](https://github.com/oOo0oOo/lean-lsp-mcp) (399⭐, 0.26.2) exposes Lean LSP
capabilities as MCP tools — sub-second feedback for goal inspection, tactic testing,
mathlib search, and diagnostics. Works alongside `lake env lean` and `lake build`.

## What You Get (26 tools)

| Tool | Purpose | Replaces |
|------|---------|----------|
| `lean_goal(file, line)` | Exact goal state at a line | Full `lake env lean` compile |
| `lean_diagnostic_messages(file)` | Per-file error/warning check | Waiting for build |
| `lean_multi_attempt(file, line, snippets)` | Test multiple tactics in parallel | Manual retry |
| `lean_local_search("keyword")` | Fast local + mathlib search | Guessing theorem names |
| `lean_leanfinder("goal")` | Semantic, goal-aware search | |
| `lean_leansearch("natural language")` | Semantic search | |
| `lean_loogle("type pattern")` | Type-pattern search | |
| `lean_hammer_premise(file, line, col)` | Premise suggestions for simp/aesop/grind | |
| `lean_code_actions(file, line)` | Apply "Try this" suggestions | |
| `lean_hover_info(file, line, col)` | Understand types | |

## Installation Steps

### Step 1: Install mcp SDK (Hermes dependency)

```bash
source /opt/data/.venv/bin/activate
uv pip install mcp
```

Hermes needs the `mcp` Python package to support the MCP protocol.

### Step 2: Install lean-lsp-mcp

```bash
uv tool install lean-lsp-mcp
```

This makes `uvx lean-lsp-mcp` available globally.

### Step 3: Configure in Hermes config.yaml

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  lean_lsp:
    command: "uvx"
    args:
      - lean-lsp-mcp
    env:
      LEAN_PROJECT_PATH: "/opt/lean-home/lean-projects/e"
      PATH: "/opt/lean-home/bin:/opt/lean-home/toolchains/leanprover--lean4---v4.31.0-rc1/bin:/usr/local/bin:/usr/bin:/bin"
      ELAN_HOME: "/opt/lean-home"
```

**First try (usually works)**: Set `env.PATH` and `env.ELAN_HOME` in config.yaml
via `hermes config set` (confirmed working with Hermes 0.16.0 / DeepSeek). Then
`/reload-mcp` or restart Hermes.

**Fallback when `env.PATH` doesn't work**: Replace the entire block with `/bin/sh -c`
to set PATH before anything starts:

```yaml
mcp_servers:
  lean_lsp:
    command: /bin/sh
    args:
      - -c
      - >-
        PATH="/opt/lean-home/bin:/opt/lean-home/toolchains/leanprover--lean4---v4.31.0-rc1/bin:/usr/local/bin:/usr/bin:/bin";
        export ELAN_HOME="/opt/lean-home";
        export LEAN_PROJECT_PATH="/opt/lean-home/lean-projects/e";
        exec /usr/local/bin/uvx lean-lsp-mcp
```

**⚠️ Config changes**: After modifying `mcp_servers` config, run `/reload-mcp`.
If the server was started with the old environment, a full Hermes restart may
be needed for env changes to take effect.

**Note**: The `hermes config set` CLI does NOT handle array/list values correctly.
`hermes config set mcp_servers.lean_lsp.args "['lean-lsp-mcp']"` stores args as a string,
not a YAML list. `hermes config set mcp_servers.lean_lsp.args.0 "lean-lsp-mcp"` stores
it as a dict key (`0`), not a list element. **Always edit the YAML file directly**
or use a Python script to write the config. After modifying the file, the MCP
server must be **restarted** — config changes take effect on next Hermes launch,
not live.

### Step 4: Restart Hermes

MCP servers are loaded at Hermes startup. After changing config.yaml, restart Hermes
to activate the MCP tools. The tools are named `mcp_{server_name}_{tool_name}`,
e.g. `mcp_lean_lsp_lean_goal`.

## Verification

After restart, a message like this confirms the server connected:

> MCP servers have been reloaded. Added servers: lean_lsp.
> 26 MCP tool(s) now available.

## MCP-First Protocol for Daily Lean Work

When MCP tools are available, prefer this workflow over raw compilation:

**Planning (understand the goal):**
1. `lean_goal(file, line)` — see exact proof state
2. Up to 3 search tools (time-boxed ~30s): `lean_local_search` first,
   then `lean_leanfinder`/`lean_leansearch`/`lean_hammer_premise`
3. Record top candidate lemmas

**Work (fill the proof):**
1. Refresh `lean_goal(file, line)`
2. Generate 2-3 candidate proof snippets
3. `lean_multi_attempt(file, line, snippets=[...])` — test in parallel
4. `lean_diagnostic_messages(file)` — verify
5. If "Try this" suggestion → `lean_code_actions(file, line)`

**Stuck detection:** Same sorry fails 2-3 times with no new approach?
Re-analyze the goal, search more lemmas, make a new plan.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|------|
| `mcp` SDK not available | Not installed in venv | `uv pip install mcp` |
| `uvx lean-lsp-mcp` not found | Tool not installed | `uv tool install lean-lsp-mcp` |
| MCP config not taking effect | Config in wrong file | Use `hermes config path` to find correct file |
| `hermes config set` stores list as string | CLI limitation | Edit YAML directly |
| MCP tools not showing after restart | Config syntax error | Check YAML with Python yaml.safe_load |
| `lake: No such file or directory` | PATH missing — `env.PATH` in Hermes config doesn't propagate | Use `/bin/sh -c` wrapper to set PATH inline (see Step 3 fallback) |
| Config changes not taking effect after `/reload-mcp` | `/reload-mcp` doesn't re-read config.yaml | Exit and restart Hermes entirely |
| `hermes mcp test lean_lsp` **times out** (30s+) | Mathlib build incomplete | MCP server tries to initialize Lean LSP on startup but LSP needs `.olean` files. Wait for `lake build` to finish, then re-test. |
| `hermes mcp list` **times out** or hangs | Same root cause | MCP server startup blocks while waiting for LSP initialization. Cancel with Ctrl-C; the server will respond once the full mathlib build completes. |
| `mcp_lean_lsp_*` tools return **empty or error responses** mid-session | Missing .olean for the specific module | Run `lake env lean <file>` first to compile on demand, then retry the MCP tool. |

**Key insight**: MCP server readiness is a function of mathlib build completeness.
- No `.olean` files at all: `hermes mcp test` and `hermes mcp list` hang
- Partial `.olean` files: MCP tools work for compiled modules, return empty for uncompiled ones
- Full build: all 26 tools available, sub-second response

## MCP Tools After `lake clean`

If `lake clean` was run (destroying all `.olean` files), MCP tools will be
unusable until `lake build` completes. The `hermes mcp test` command timeouts
are expected during this window. Check build progress with `ps aux | grep lean`.
