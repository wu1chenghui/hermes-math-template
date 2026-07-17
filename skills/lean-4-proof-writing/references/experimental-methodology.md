# Lean 4 Experimental Methodology

Pattern developed during the 1/2-derivation classification project (N_n, dim=n).
Used for systematic enumeration and pattern discovery BEFORE theorem proving.

## When to use

You need to explore coefficient patterns, enumerate projections, or verify
combinatorial hypotheses across multiple lengths. The goal is data, not proofs.

## Template

```lean
import Mathlib.Tactic
-- Do NOT import project modules (E.*) unless absolutely necessary.
-- Each import requires .olean, creating slow builds and dependency headaches.

/-! Doc comment -/

section Framework
-- Define data structures and analysis functions
structure MyTerm where ...
def analyze (i N u v : ℕ) : ...
end Framework

section Runner
def runAll (i N : ℕ) : IO Unit := do
  for u in [1 : i+N+2] do
    for v in [u+1 : i+N+2] do
      -- compute and print
#eval! runAll 5 4
#eval! runAll 5 6
end Runner
```

## Key rules

1. **Use `#eval!` not `#eval`.** The `!` variant tolerates `sorry` axioms in
   imported modules (mathlib often has them). Without `!`, evaluation aborts.

2. **Import only `Mathlib.Tactic`** for `omega`. Avoid importing project modules
   (`E.*`) — they require compiled `.olean` files, which may not exist if the
   file isn't in `E.lean`'s import chain.

3. **Run with `lake env lean --run <file>`**, not `lake build`. This compiles
   and executes in one step, without needing the file in the build graph.

4. **Use `omega` for all index arithmetic.** It's the most reliable tactic for
   `Nat` inequalities in experimental contexts.

5. **Prefer `Id.run do` with mutable lists** over functional folds for
   readability in enumeration code.

## Pitfalls discovered

### `Module` is a reserved name in mathlib

```lean
-- ❌ Name collision with CategoryTheory
inductive Module where ...

-- ✅ Use a different name
inductive FMod where ...
```

### `linarith` fails on `Field F`

```lean
-- ❌
h : a + b = 0
⊢ a = -b
linarith  -- fails: linarith needs LinearOrderedCommRing

-- ✅
calc a = -b := by linarith  -- still fails
-- Use:
have := eq_neg_of_add_eq_zero_left h
-- Or: linarith works on ℕ, ℤ, ℚ, ℝ, NOT arbitrary Field
```

### `simp` with `if u=i` conditions

```lean
-- ❌ simp doesn't use hu_lt_i : u < i to derive u ≠ i
have hu_lt_i : u < i := by omega
simp [hu_lt_i] at h  -- leaves unsolved goal: u = i → ...

-- ✅ Explicitly provide u ≠ i
have hu_ne_i : u ≠ i := by omega
simp [hu_ne_i] at h
```

### `subst` clears variables

```lean
-- ❌ subst removes c and v from context entirely
subst huc_eq; subst hvj
-- Now c and v are gone, but later references to them fail

-- ✅ Use rw to keep variables accessible
rw [huc_eq, hvj]
```

### String formatting in Lean 4

```lean
-- ❌ No String.repeat
" ".repeat n           -- does not exist
-- ✅ Use List.replicate + String.join
String.join (List.replicate n " ")

-- ❌ No padding in s! interpolation
s!"{name,-15}"          -- syntax error
-- ✅ Manual padding
def padRight (s : String) (n : Nat) : String := ...
```

### `ForIn` on `List` in `IO`

```lean
-- ✅ Works directly
for (x, y) in myList do
  IO.println s!"({x},{y})"
```

### `List.qsort` may not exist

```lean
-- ❌
myList.qsort (λ a b => ...)  -- may not be available
-- ✅ Sort manually or iterate unsorted
for x in myList do ...
```
