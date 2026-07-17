# Lean 4 Environment

## Overview

This setup uses **Lean 4 + Mathlib 4** as a formal proof assistant, accessed
through an MCP (Model Context Protocol) server. The Lean toolchain lives in
`/opt/lean-home` — a Docker named volume that stays on the native Linux
filesystem.

## Why a Separate Volume?

**CRITICAL on WSL:** Lean 4 cannot compile on NTFS-mounted Windows drives
(`/mnt/c/...`). File watching (`inotify`) fails and `lake build` produces
cryptic errors. The `lean-home` named volume lives on the WSL ext4 filesystem,
solving this.

On native Linux, this is still fine — named volumes are just convenient.

## Installation

### 1. Install elan (Lean version manager)

```bash
# On the Docker HOST (WSL or Linux):
curl https://raw.githubusercontent.com/leanprover/elan/master/elan-init.sh -sSf | sh
```

This installs to `~/.elan/bin/`. Make sure `/opt/lean-home` is populated:

```bash
export ELAN_HOME=/opt/lean-home
elan toolchain install leanprover/lean4:v4.31.0-rc1
elan default leanprover/lean4:v4.31.0-rc1
```

### 2. Create a Lean project

```bash
cd /opt/lean-home
lake new lean-projects/your-project
cd lean-projects/your-project
```

### 3. Download Mathlib cache

Mathlib is huge. Without the precompiled `.olean` cache, the first build
takes hours:

```bash
lake exe cache get
```

### 4. Build

```bash
lake build
```

## MCP Server Configuration

The Lean LSP MCP server is defined in `config.yaml`:

```yaml
mcp_servers:
  lean_lsp:
    command: uvx
    args:
    - lean-lsp-mcp
    env:
      LEAN_PROJECT_PATH: /opt/lean-home/lean-projects/your-project
      PATH: /opt/lean-home/bin:/opt/lean-home/toolchains/leanprover--lean4---v4.31.0-rc1/bin:/usr/local/bin:/usr/bin:/bin
      ELAN_HOME: /opt/lean-home
```

This gives Hermes access to Lean 4 LSP tools:
- `lean_goal` — inspect proof state
- `lean_build` — compile the project
- `lean_completions` — autocomplete
- `lean_diagnostic_messages` — errors and warnings
- `lean_finder` — semantic search in Mathlib
- And 20+ more tools

## Verify

```bash
# Inside the container:
docker compose exec hermes bash
export ELAN_HOME=/opt/lean-home
export PATH="/opt/lean-home/bin:$PATH"
elan --version
lake --version
cd /opt/lean-home/lean-projects/your-project && lake build
```

## Notes

- Version: Lean 4 + Mathlib v4.31.0-rc1 (latest tested working version as of 2026-07)
- The `lean-lsp-mcp` tool is installed via `uv tool install lean-lsp-mcp`
- Container needs access to `/opt/lean-home` (via the `lean-home` volume)
