# Verified Import Paths (Lean 4.31.0-rc1 / mathlib master)

Verified against the actual mathlib4 GitHub source and local `.olean` build.
Last updated: 2026-06-07

## ✅ Verified Working Paths

These import paths are confirmed to exist and compile correctly:

### Core
```
Mathlib.Data.Nat.Prime.Basic
Mathlib.Data.Nat.GCD.Basic
Mathlib.Data.Real.Basic
Mathlib.Data.Complex.Basic
Mathlib.Data.Int.Basic
Mathlib.Data.Nat.Basic
Mathlib.Data.Set.Basic
Mathlib.Data.Finset.Basic
Mathlib.Tactic
Mathlib.Algebra.Module.Basic
```

### Analysis
```
Mathlib.Analysis.Complex.Trigonometric
Mathlib.Analysis.SpecialFunctions.Trigonometric.Basic
Mathlib.Analysis.SpecialFunctions.Trigonometric.Complex
Mathlib.Analysis.SpecialFunctions.Exp
```

### Algebra
```
Mathlib.Algebra.BigOperators.Finprod
Mathlib.Algebra.BigOperators.Intervals
Mathlib.Algebra.BigOperators.Pi
Mathlib.Algebra.Module.Basic
Mathlib.Data.Nat.GCD.Basic
```

### Topology
```
Mathlib.Topology.Instances.Discrete
Mathlib.Topology.Instances.Complex
Mathlib.Topology.Instances.Int
Mathlib.Topology.Instances.Nat
Mathlib.Topology.Instances.Rat
```

### Category Theory
```
Mathlib.CategoryTheory.Core
Mathlib.CategoryTheory.Action
Mathlib.CategoryTheory.Limits.Shapes.Terminal
```

### Measure Theory
```
Mathlib.MeasureTheory.MeasurableSpace
Mathlib.MeasureTheory.OuterMeasure
```

## ❌ Dead Paths (do NOT exist)

These paths are frequently referenced in older tutorials but **do not exist**
in current mathlib. Confirmed via mathlib4 GitHub source and local build.

| Dead path | What happened | What to use instead |
|-----------|--------------|--------------------|
| `Data.Nat.Dvd.Basic` | dvd is a language primitive in Init | No import needed; `Nat.dvd` is always available |
| `Data.Nat.Parity` | Removed as separate file; functions in `Nat.Basic`/`Std` | Import `Data.Nat.Basic` or `Data.Nat.Prime.Basic` |
| `Algebra.GroupPower.Basic` | Split across `Algebra/*/Power` modules | Import specific `Algebra.Group.Basic` or `Algebra.Ring.Power` |
| `LinearAlgebra.Basic` | Split into submodules | Use `Algebra.Module.Basic` |
| `CategoryTheory.Basic` | Split into submodules | Use specific submodule (e.g., `CategoryTheory.Core`) |
| `Topology.Instances.Real` | Never existed | Use `Topology.Instances.Discrete` or a specific instance |
| `Algebra.Order.Basic` | Split into `Algebra.Order.*` submodules | Use specific submodule (e.g., `Algebra.Order.Monoid`) |
| `Algebra.BigOperators.Basic` | Split into `Algebra.BigOperators.*` submodules | Use `Algebra.BigOperators.Finprod` or `Intervals` |
| `Analysis.SpecialFunctions.Trigonometric` | Was a single file, became a **directory** | Use `...Trigonometric.Basic` or `...Trigonometric.Complex` |

## 🔍 How to verify a path yourself

```bash
# Quick source check
ls /opt/lean-home/lean-projects/e/.lake/packages/mathlib/Mathlib/X/Y/Z.lean

# Quick compiled check
ls /opt/lean-home/lean-projects/e/.lake/packages/mathlib/.lake/build/lib/lean/Mathlib/X/Y/Z.olean

# Lake compilation check
echo 'import Mathlib.X.Y.Z' > /tmp/test.lean
cd /opt/lean-home/lean-projects/e && source ~/.elan/env
lake env lean /tmp/test.lean 2>&1 | grep error
```

## 📝 Notes

- **`Trigonometric` was split from a single file to a directory**: Old tutorials
  reference `Analysis.SpecialFunctions.Trigonometric` (a single `.lean` file).
  In current mathlib, it's a **directory** containing `Basic.lean`, `Complex.lean`,
  `Deriv.lean`, etc. Always include a submodule name after `Trigonometric.`.

- **Partial imports are preferred over `import Mathlib`**: Even after full build,
  `import Mathlib` loads ~60s of submodules. Always use specific paths.
