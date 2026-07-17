# Missing .olean Modules — Recovery

## Quick check

```bash
cd /opt/lean-home/lean-projects/e
source ~/.elan/env

# Check if a specific module is compiled
find .lake/packages/mathlib/.lake/build/lib \
  -path "*/Target/Module/Path.olean" -not -name "*.hash" -not -name "*.private" \
  2>/dev/null

# Count total mathlib .olean files (expect ~8400+)
find .lake/packages/mathlib/.lake/build/lib -name "*.olean" \
  -not -name "*.hash" -not -name "*.private" -not -name "*.server" | wc -l
```

## Recovery paths

### 1. Module exists but `lake env lean` says it doesn't

The build cache may be stale. Try:

```bash
lake build   # incremental — resumes where it left off, skips done modules
```

### 2. `lake clean` was run (full cache destroyed)

This is the most expensive case — all ~8400 mathlib .olean files are gone.
Run a full rebuild in background:

```bash
cd /opt/lean-home/lean-projects/e
source ~/.elan/env
lake build &
```

This will rebuild everything. Expected time: 2-5 hours with 16 parallel processes
on ext4. Use `notify_on_complete=true` when running via Hermes terminal tool.

### 3. Only some modules are missing (partial build)

If the initial build failed near the end (e.g. proofwidgets widget facet), most
modules are compiled but some are missing. Re-running `lake build` skips the
compiled ones and only builds the missing ones. This is fast (~minutes).

### 4. Module wasn't in the original build scope

`lake build` only compiles modules transitively reachable from the project's
entry points. If you add `import Mathlib.Unrelated.Module` to a file that
wasn't compiled before, its .olean won't exist. Solution: `lake build` will
pick it up and compile it as a dependency.

## Known module restructurings (observed in practice)

The following paths are **not missing but restructured** — they exist but under
different names or within parent modules. Verified against the official
[mathlib4 GitHub source](https://github.com/leanprover-community/mathlib4):

| Expected path (old/wrong) | Actual (verified) |
|---|---|
| `Mathlib/Data/Nat/Parity.olean` | Functions in `Nat.Basic`; `Nat.even_or_odd` works |
| `Mathlib/Data/Nat/Dvd/Basic.olean` | `Dvd` is language primitive; no `Nat.Dvd` module exists |
| `Mathlib/Analysis/SpecialFunctions/Trigonometric.olean` | Now a **directory**. Use `Trigonometric/Basic.olean` or `Trigonometric/Complex.olean` |
| `Mathlib/Data/Nat/Prime/Basic.olean` | **Exists** after full `import Mathlib` build (part of 8496 jobs) |
| `Mathlib/LinearAlgebra/Basic.olean` | Use `Algebra/Module/Basic.olean` |
| `Mathlib/CategoryTheory/Basic.olean` | Use `CategoryTheory/Core.olean` |
| `Mathlib/Algebra/GroupPower/Basic.olean` | Power operations split across algebra submodules |

**Diagnostic command for a specific module:**

```bash
# Check if a specific .olean exists
find /opt/lean-home/lean-projects/e/.lake/packages/mathlib/.lake/build/lib \
  -name "Trigonometric.olean" -not -name "*.hash" -not -name "*.private" 2>/dev/null

# List all top-level Mathlib subdirectories with .olean files
find /opt/lean-home/lean-projects/e/.lake/packages/mathlib/.lake/build/lib/lean/Mathlib \
  -maxdepth 1 -type d | sort
```

**Workaround:** Avoid importing modules whose .olean hasn't been compiled.
Use mathlib's own aliases (e.g., `Nat.even_or_odd` lives in `Nat` itself,
not `Nat.Parity`; `dvd_mul` is part of `Nat.Prime.Basic` directly).

## How to verify the build is safe to use

```bash
# Compile your file — if this passes, the build is valid
lake env lean E/Practice.lean
```

## Prevention: avoid `lake clean`

Before running `lake clean`, ask: **do I need to clear the project build
or the full dependency cache?**

| Need | Command | Time cost |
|------|---------|-----------|
| Clear project .olean only | `rm -rf .lake/build/lib/ .lake/build/ir/` | ~5 seconds |
| Clear project only (via lake) | `lake clean <project-name>` (e.g. `lake clean E`) | ~5 seconds |
| Clear everything (project + deps) | `lake clean` | 40-60 min rebuild (8496 jobs) |
| Clear one specific package | `rm -rf .lake/packages/mathlib/.lake/build/` | hours for mathlib |

**Always prefer `lake clean <project-name>` over bare `lake clean`** unless you
want to rebuild all 8496 mathlib modules.
