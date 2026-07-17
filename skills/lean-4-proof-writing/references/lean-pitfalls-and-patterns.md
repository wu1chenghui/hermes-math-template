# Lean 4 Pitfalls and Patterns (Session-Learned)

## Pitfall: `lean_file_outline` times out on large files (≥1000 lines)

The MCP LSP tool `lean_file_outline` (a.k.a. `mcp_lean_lsp_lean_file_outline`) has a 120s
hard timeout. Files with 1000+ lines trigger this timeout, returning no results at all.
Centering.lean (1285 lines) is a known case in the 1/2-derivation project.

**Workaround for large-file structure exploration:**

1. **`read_file` with `limit=60-100`** — get imports, module docstring, and namespace
2. **`search_files` with regex** — find declarations:
   ```
   search_files(pattern="^(theorem|lemma|def) ", path="E/Classification/Centering.lean",
                output_mode="content", context=0)
   ```
3. **Reserve `lean_file_outline` for files <500 lines** — where it's the fastest option

Do NOT retry `lean_file_outline` on the same large file — it will not get faster.

## Pitfall: `push_neg` is deprecated → use `push Not`

`push_neg` was deprecated in leanprover/lean4:v4.31.0-rc1.
Use `push Not` instead. The `push` tactic with the `Not` configuration
achieves the same effect (pushing negations past conjunctions/disjunctions).

```lean4
-- ❌ Deprecated
  push_neg at h

-- ✅ Correct
  push Not at h
```

If you need `push_neg` in a project, add this macro:
```lean4
open Lean.Parser.Tactic in
macro "push_neg" cfg:optConfig loc:(location)? : tactic =>
  `(tactic| push $cfg:optConfig Not $[$loc]?)
```

## Pitfall: `_` placeholders in `if` conditions prevent `simp` type inference

When using `simp` on `if COND then _ else 0`, Lean can't infer the
type of `_`, causing "don't know how to synthesize placeholder" errors.

```lean4
-- ❌ Fails: can't infer type of _
have hT : (if u = i ∧ i+1 < v then _ else 0) = 0 := by simp

-- ✅ Works: explicit type
have hT : (if u = i ∧ i+1 < v then (0 : F) else 0) = 0 := by simp [hu_ne_i]
```

## Pattern: Simplifying 8 if-conditions with `split_ifs` + `push Not` + `omega`

When a goal or hypothesis contains many `if` expressions with `∧` conditions
over concrete ℕ comparisons, use this repeatable pattern:

```lean4
macro "simpl8" : tactic =>
  `(tactic|
    repeat' (split_ifs at h_compat <;>
      (try (rcases ‹_› with ⟨h1,h2⟩; omega); try (push Not at *; omega)))
    simp at h_compat)
```

This handles both branches of each `split_ifs`:
- **True branch**: `rcases` decomposes the `∧` hypothesis, then `omega` closes arithmetic contradictions
- **False branch**: `push Not` converts `¬(A ∧ B)` into `¬A ∨ ¬B`, then `omega` handles the disjunction

After all `split_ifs`, a final `simp` cleans up `0` terms.

**Why `simp` alone fails**: `simp` can't decompose `¬(A ∧ B)` — it treats it as a single proposition, not as component-level facts that `omega` can use.

**When to use**: Whenever an equation has 3+ if-conditions over concrete ℕ index comparisons, and you've already substituted the known index values via `rw`.

## Pitfall: `Fin n` bound proof is `.2`, not `.property`

`Fin n` is defined as `{ val : ℕ // val < n }`, a subtype. The bound proof
is the second projection `.2`, NOT `.property`:

```lean4
-- ❌ Invalid field 'property'
have : p.val < n-1 := p.property

-- ✅ Correct
have : p.val < n-1 := p.2
```
