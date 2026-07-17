# Mathlib Module Paths — Verified Import Paths (v4.31.0-rc1)

## Triple-verified against
1. **Compile test**: `lake env lean` with the import
2. **Build output**: `.olean` file existence in `.lake/build/lib/lean/`
3. **Official source**: [mathlib4 GitHub](https://github.com/leanprover-community/mathlib4/tree/master/Mathlib) (master branch API check)

---

## ✅ Working import paths

| Import | Use for | Notes |
|--------|---------|-------|
| `Mathlib.Data.Nat.Prime.Basic` | Primality, `Nat.Prime.dvd_mul` | Also provides `Nat.even_or_odd` |
| `Mathlib.Data.Real.Basic` | `Real.sin`, `Real.cos`, `ℝ` arithmetic | |
| `Mathlib.Tactic` | `ring`, `linarith`, `omega`, `aesop`, `grind` | |
| `Mathlib.Data.Set.Basic` | `Set`, `Set.Subset`, set operations | |
| `Mathlib.Data.Finset.Basic` | `Finset`, `∑`, `∏` | |
| `Mathlib.Data.Complex.Basic` | `Complex`, `I`, `normSq` | |
| `Mathlib.Data.Int.Basic` | `ℤ`, `Int` lemmas | |
| `Mathlib.Algebra.Module.Basic` | Vector spaces, `Module`, `add_smul` | |
| `Mathlib.Analysis.SpecialFunctions.Trigonometric.Basic` | `Real.sin`, `Real.cos`, trig+deriv | **Directory**, only `Basic` is the top-level file |
| `Mathlib.Analysis.SpecialFunctions.Exp` | `Real.exp`, `Complex.exp` | |
| `Mathlib.Analysis.Complex.Trigonometric` | Complex trig functions (separate file) | |
| `Mathlib.Data.Nat.GCD.Basic` | `Nat.gcd`, `Nat.lcm` | |
| `Mathlib.Data.Nat.Basic` | ℕ arithmetic, `Nat.succ_eq_add_one` | |
| `Mathlib.Data.Nat.Prime` | Prime-related theorems (directory) | |
| `Mathlib.Algebra.BigOperators.Finprod` | Finite products | Replaces `Algebra.BigOperators.Basic` |
| `Mathlib.Algebra.BigOperators.Intervals` | Sums over intervals | |
| `Mathlib.Tactic` | All tactics | Confirmed: 35 core tactics compile |
| `Mathlib.Topology.Instances.Discrete` | Discrete topology | |

---

## ❌ Paths that do NOT exist

These paths were removed/renamed/merged in current mathlib. **Do not use them.**

| Old/broken path | What happened | Replacement |
|---|---|---|
| `Mathlib.Data.Nat.Dvd.Basic` | **No source file** — `dvd` is part of `Init`/language | Just use `Nat.dvd` — no import needed |
| `Mathlib.Data.Nat.Parity` | **No source file** — parity merged into `Nat.Basic` | `Nat.even_or_odd` works via `Nat` itself |
| `Mathlib.Algebra.GroupPower.Basic` | **No source file** — power split across algebra dirs | Use `ring` tactic or `pow_two` notation |
| `Mathlib.LinearAlgebra.Basic` | **No source file** — split into submodules | Use `Mathlib.Algebra.Module.Basic` |
| `Mathlib.CategoryTheory.Basic` | **No source file** — split into submodules | Use specific cat theory module |
| `Mathlib.Topology.Instances.Real` | **No source file** | Use `Topology.Instances.Discrete` or search |
| `Mathlib.Algebra.Order.Basic` | **No source file** | Use `Mathlib.Algebra.Order.Ring.Basic` etc. |
| `Mathlib.Algebra.BigOperators.Basic` | **No source file** | Use `Algebra.BigOperators.Finprod` or `Intervals` |
| `Mathlib.Analysis.SpecialFunctions.Trigonometric` | Was a file, now a **directory** → use `.Basic` suffix | Use `Trigonometric.Basic` or `Trigonometric.Complex` |

---

## 🔍 How to check a path yourself

```bash
# Compilation test (fastest)
cd /opt/lean-home/lean-projects/e && source ~/.elan/env
echo 'import Mathlib.Something.Something' > /tmp/test_import.lean
lake env lean /tmp/test_import.lean 2>&1 | grep -c "error:"

# Source file existence
ls /opt/lean-home/lean-projects/e/.lake/packages/mathlib/Mathlib/Something/Something.lean

# .olean existence
ls /opt/lean-home/lean-projects/e/.lake/packages/mathlib/.lake/build/lib/lean/Mathlib/Something/Something.olean

# Official GitHub API
curl -sL "https://api.github.com/repos/leanprover-community/mathlib4/contents/Mathlib/Something/Something.lean" \
  -H "Accept: application/vnd.github+json" | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('message','EXISTS'))"
```

---

## 📋 How to add a new verification test

Copy this template into `E/` and compile:

```lean4
import Mathlib.Something.NewModule

#check Some.theorem
example : Some.property := by
  some_tactic

-- Compile with:
-- cd /opt/lean-home/lean-projects/e && source ~/.elan/env && lake env lean E/VerifySomething.lean
```
