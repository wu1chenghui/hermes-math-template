# Computing evAdjFun coordinates of basis generators

**Context**: In the `kerΦ` / AdjData framework (RepresentationBridge.lean), `evAdjFun` maps a coefficient table `V F = ℕ⁴ → F` to `AdjData F n = (Fin (n-1)→F) × (Fin (n-1)→F) × (Fin (n-1)→F)`, capturing the three I-channels a=(1,n-1), b=(1,n), c=(2,n) at each adjacent source.

To prove `kerΦ = span{A_i, B, C, D, E, F}`, compute `evAdjFun` of each generator's coefficient function (`coeff_A`, `coeff_B`, …, `coeff_F`). This gives each generator's AdjData coordinates, which P3 (Spanning) then uses via `simp`.

## Pattern template (CURRENT — see `simp-dsimp-evadjfun.md` for rationale)

```lean4
lemma evAdjFun_coeff_G (hn : 3 ≤ n) : evAdjFun (coeff_G (F := F) (n := n)) =
    (a_fun, b_fun, c_fun) := by
  apply Prod.ext
  · ext p; dsimp [evAdjFun, coeff_G]; split_ifs <;> first | rfl | exfalso; omega
  · apply Prod.ext
    · ext p; dsimp [evAdjFun, coeff_G]; split_ifs <;> first | rfl | exfalso; omega
    · ext p; dsimp [evAdjFun, coeff_G]; split_ifs <;> first | rfl | exfalso; omega
```

Use `dsimp` (not `simp`) to unfold — `simp` runs `split_ifs` internally,
producing unpredictable residue goals. See `simp-dsimp-evadjfun.md` for the
full failure analysis.

> **Deprecated** (2026-06-23): the old pattern using `by_cases h : p.val = k; rw [h]; simp` was superseded by the uniform `dsimp` + `split_ifs` + `first | rfl | exfalso; omega` pattern. The old pattern required per-lemma tuning and left 12 unsolved arithmetic goals. The new pattern closed all 15 components across 5 lemmas in uniform style.

## Common pitfalls

- **`subst` fails on projections**: `subst h` with `h : p.val = k` fails because `p.val` is a projection. Use `rw [h]` instead.
- **`split_ifs` with conjunction guards**: `if (A ∧ B) then …` often causes `split_ifs` to produce unexpected case counts. Prefer `by_cases h : key_condition` + `omega` for the secondary condition.
- **`omega` can't handle `∧`**: When the guard is a conjunction, `omega` cannot use `h : A ∧ B`. Use `obtain ⟨ha, hb⟩ := h; omega`.
- **`omega` and `n-1 ≠ n`**: `omega` doesn't handle `¬(n-1 = n)` (non-linear). Pass it via `show n-1 ≠ n from by omega` as a `simp` lemma.
- **`Prod.ext` vs `refine ⟨…, …⟩`**: `Prod.ext` creates TWO independent subgoals. Do NOT wrap it with `refine ⟨?_, ?_⟩` (which expects a single `∧` goal). Use `apply Prod.ext` then fill two blocks.
- **`ext` on `(A × B) × C`**: AdjData is left-associated: `(A × B) × C`. `ext <;> try ext <;> ext p` may fail. Use explicit nested `apply Prod.ext` + `ext p`.

## Per-generator signatures

| Generator | a-channel | b-channel | c-channel |
|-----------|-----------|-----------|-----------|
| `A_i` (1≤i≤n-1) | 0 | `if p.val+1 = i then 1 else 0` | 0 |
| `B` | 0 | 0 | `if p.val = 0 then 1 else 0` |
| `C` | 0 | 0 | `if p.val = 1 then 1 else 0` |
| `F` | `if p.val = 0 then -1 else 0` | 0 | `if p.val = n-2 then 1 else 0` |
| `D` | `if p.val = n-3 then 2 else 0` | 0 | 0 |
| `E` | `if p.val = n-2 then 1 else 0` | 0 | 0 |

### P3b construction: handle coefficients for `evAdj(S) = target_AdjData`

To construct S so that `evAdj(S) = (a, b, c)` where `(a,b,c)` is a
constraint-satisfying AdjData (from P1a/P3a), use:

```
S = Σ_p b_p · A_{p+1}
  + c₀ · B
  + c₁ · C
  + (a_{n-3}/2) · D
  + a_{n-2} · E            ← ★ a_{n-2} alone (NOT a_{n-2}+a₀)
  + (-a₀) · F
```

**Why `a_{n-2}` for E's coefficient (NOT `a_{n-2}+a₀`).**
F's evAdjFun result (from P2) is:
- a-channel: `-1` at position 0; `0` elsewhere (including n-2)
- c-channel: `+1` at position n-2; `0` elsewhere

So F contributes `(-a₀)·(-1) = a₀` to a-channel at 0, and `(-a₀)·(+1) = -a₀`
to c-channel at n-2. F does NOT touch a-channel at n-2.

The a-channel at n-2 comes SOLELY from E: `a_{n-2}·(+1) = a_{n-2}`.
The c-channel at n-2 is `-a₀`, which matches `c_{n-2}` via coupling
`a₀ + c_{n-2} = 0` (from P1a), used in P3c.

This was corrected 2026-06-24; the earlier claim that F contributes +1
to a-channel at n-2 was a design error caught during Lean implementation.
