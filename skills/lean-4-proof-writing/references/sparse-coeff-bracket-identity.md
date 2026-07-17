# Sparse-coefficient bracket-identity proofs (taming `split_ifs` explosion)

Use when proving a half-Leibniz / Leibniz / cocycle identity stated at the
**coefficient level** as an equality of guarded `if`-sums, where the coefficient
function `coeff_X si sj tu tv` is a sparse nest of `if (si=.. ∧ sj=.. ∧ tu=.. ∧ tv=..) then v else …`.
Canonical example: `E/Classification/BasisCocycles.lean` (`halfDeriv_A..F`),
proving `2 * bracketSource coeff … = bracketIdentity coeff …`.

## The explosion and its real cause

Naive proof `simp [ha, hb, …]; split_ifs <;> …` times out (32768+ subgoals,
fails even at 1M heartbeats). The cause is NOT split_ifs per se — it is that
**general `simp` UNFOLDS the 4-conjunction guard** `si=.. ∧ sj=.. ∧ tu=.. ∧ tv=..`
into 4 *independent* `if`s. With 6 bracket sub-terms that becomes ~15–18
independent conditions → 2^18 branches.

## Winning template — SINGLE-`if` coefficient functions (e.g. A, B, E)

```lean
intro a b c d u v hi hij hjn hk hkl hln hu huv hvn
simp only [bracketSource, bracketIdentity, bracketLeft, bracketRight, coeff_X]
split_ifs <;> first | rfl | (exfalso; omega) | ring
```

Why it works:
- `simp only [.., coeff_X]` expands the bracket *definitions* and the coeff
  *name*, but KEEPS each 4-conjunction as ONE `Decidable` proposition. So
  `split_ifs` sees ~12 conditions (2^12 ≈ 4096), the manageable order.
- **Closer order matters**: put `(exfalso; omega)` BEFORE `ring`. Most branches
  are unreachable index contradictions closed by `omega`; if `ring` runs first
  on them it leaks non-fatal "Try ring_nf" noise. `omega` automatically uses
  conjunction hypotheses (and their negations) from the context.

## Double-`if` coefficient functions (e.g. C, D, F)

A coeff with two active sources (`if g1 then v1 else if g2 then v2 else 0`)
doubles the inner conditions → bare template still hits 2^18. Fix: kill the
**two bracket sub-terms that are identically zero** with local `have`s, `rw`
them away, leaving 4 surviving terms (2^12, fine).

```lean
have h2 : (2:F) ≠ 0 := CharNeTwo.char_ne_two   -- only if coeff uses 2⁻¹
simp only [bracketSource, bracketIdentity, bracketLeft, bracketRight, coeff_X]
have hZ1 : (<exact if-expr of zero term 1>) = 0 := by
  split_ifs <;> first | rfl | (exfalso; omega)   -- 3-condition split, instant
have hZ2 : (<exact if-expr of zero term 2>) = 0 := by
  split_ifs <;> first | rfl | (exfalso; omega)
rw [hZ1, hZ2]
split_ifs <;> first | rfl | (exfalso; omega) | (field_simp; ring)
```

- The `field_simp; ring` closer handles `2·2⁻¹ = 1` style goals (needs `h2` in
  context). Drop the trailing `| ring` — if you see "ring tactic does nothing /
  this tactic is never executed", that branch is dead; remove it.
- **Which two terms are zero differs per type, and encodes the algebra:**
  - bracketSource has 2 terms (guards `b=c`, `a=d`); bracketLeft/Right have 2
    each (`u<c∧v=d`, `u=c∧d<v`; `u=a∧b<v`, `u<a∧v=b`).
  - A term is identically zero when its inner guard forces an index equality
    that contradicts the validity hyps (`1≤a<b≤n`, `1≤c<d≤n`, `1≤u<v≤n`),
    e.g. inner needs `d=1` but `c<d` and `c≥1`; or inner needs `c=n` but
    `c<d≤n`; or needs `b=1` but `a<b` and `a≥1`.
  - If the *whole* `bracketSource` vanishes, the two sources commute — that is
    the "commuting sources" / `[x,y]=0` case (BasisCocycles type F).

## Getting the EXACT post-`simp only` goal text (so `rw` matches)

Hand-written `have`/`rw` targets must match the goal syntactically. Don't guess:
1. Temporarily set the proof body to just `simp only [bracketSource,
   bracketIdentity, bracketLeft, bracketRight, coeff_X]` (no closer, no sorry).
2. Compile → "unsolved goals" error prints the full goal verbatim.
3. Copy the two zero-term `if`-expressions exactly (mind `2⁻¹`, `n - 1`, `-1`)
   into the `have` statements. Annotate the value as `(2:F)` / `(-1:F)` etc.

## Feedback loop

`lake env lean E/<path>.lean` is the reliable single-file check (~15–70s incl.
`lake env` git-check warmup; cap heartbeats per-def with
`set_option maxHeartbeats 400000 in` while iterating so a bad tactic fails fast
instead of grinding to 1M). The lean-lsp MCP server is fast on small files but
may cold-start-timeout (120s) on files with long dependency chains — warm it
once with a `lake env lean` build, and fall back to single-file compile if MCP
still times out. New-architecture files not imported by the main entry point
are NOT built by `lake build`; verify them with `lake env lean` directly.
