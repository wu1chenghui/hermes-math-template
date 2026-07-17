# Clean Rebuild with LSP Management

## Problem

After `lake clean` deletes all `.olean` files, the running LSP server
(`lake setup-file`) detects missing dependencies and spawns `lean` worker
processes to rebuild mathlib in the background. These workers consume
significant CPU and memory (10-15 workers, ~1GB each), competing with
any manual `lake build` you start.

If you kill `lake clean && lake build` mid-flight, the orphaned workers
(now reparented to PID 1) keep running, and the LSP may spawn more.

## Correct Procedure

**Step 1: Kill the LSP server**

```
# Find and kill the lake setup-file process (the LSP)
ps aux | grep 'lake setup-file' | grep -v grep | awk '{print $2}' | xargs kill
```

**Step 2: Kill all orphaned mathlib workers**

Workers are `lean` processes compiling mathlib modules:

```
# Identify by args containing '.lake/packages/mathlib'
ps aux | grep '[/]lean.*\.lake/packages/mathlib' | grep -v grep | awk '{print $2}' | xargs -r kill
```

Wait 3 seconds, verify none remain. If respawning, a parent process is still alive — redo step 1.

Note: `pkill -f '/lean.*mathlib'` can match itself and receive its own signal. Prefer the `ps | grep | awk | xargs kill` pattern to avoid this.

**Step 3: Run `lake clean` (foreground)**

```
cd <project> && source ~/.elan/env && lake clean
```

**Step 4: Run `lake build` (background with notification)**

```
cd <project> && source ~/.elan/env && lake build
```

Expected: 30-60 minutes for mathlib ~2900 modules + project files.
Use `terminal(background=true, notify_on_complete=true, timeout=3600)`.

**Step 5: Restart LSP**

The MCP bridge (`lean-lsp-mcp`) may auto-restart the LSP when needed.
If not, restart manually or restart the MCP server.

## Verification

After build completes:
- `find .lake/build -name '*.olean' | wc -l` should show thousands of files
- `ps aux | grep 'lake setup-file'` should show the LSP running
- LSP MCP tools (`lean_goal`, `lean_diagnostic_messages`, etc.) should work
