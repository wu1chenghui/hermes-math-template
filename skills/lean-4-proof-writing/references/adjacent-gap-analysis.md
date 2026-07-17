# Adjacent Gap Analysis (Phase VA) — Updated 2026-06-08

## What's proven

| Component | Method | Status |
|-----------|--------|--------|
| **Nonadjacent diagonalization** (j-i ≥ 2) | Strong induction on target length via `source_exists` | ✅ Proven in `Diagonalization.lean` |
| **RecursiveSystem C_{ij} = c** (all i<j) | Classification via periodicity + collapse | ✅ Proven in `Recursion.lean` |
| **Bridge** HalfDerivation → RecursiveSystem | `diagCoeff` + `recursion_holds` | ✅ Proven in `Bridge.lean` |
| **Type C interior** (u < i < i+1 < v < n) | `cond_zero(i,i+1,v,v+1,u,v+1)` Term 1 alone survives | ✅ `typeC_interior_zero` in `Adjacent.lean` |
| **Corner coefficient (1,n)** | Distinct from Type C — outside `cond_zero` coverage | 🆕 Identified, needs bracket-level argument |

## What's in progress

| Component | Method | Status |
|-----------|--------|--------|
| **Type_A(i) for 1 ≤ i ≤ n-2** | Right bracket, descending induction | 🟡 Has boundary sorry + compile error on `Nat.decreasingInduction` syntax |
| **Type_B(i) for 2 ≤ i ≤ n-1** | Left bracket, ascending induction | 📋 Not yet implemented (entire `sorry`) |
| **Type C boundary (v=n)** | Needs separate analysis — `(1,n)` is a corner coefficient, not standard Type C | 📋 Open |

## Type C Interior — FIXED ✅

Replaced the broken `typeC_zero` lemma with `typeC_interior_zero`. Key insight: `cond_zero(i,i+1,v,v+1,u,v+1)` isolates `D.f i(i+1) u v` in Term 1:

```
cond_zero(i,i+1,v,v+1,...,u,v+1):
  Term 1: if u < v ∧ (v+1) = (v+1) → D.f i(i+1) u v    ← TARGET, fires
  Term 2: if u = v ∧ (v+1) < (v+1) → 0                   ← u ≠ v, no
  Term 3: if u = i ∧ (i+1) < (v+1) → 0                   ← u ≠ i, no
  Term 4: if u < i ∧ (v+1) = (i+1) → 0                   ← v ≠ i, no
```

### The `ite_eq_right_iff.mpr` pattern for `(if P then A else 0) = 0`

`simp [h]` where `h : ¬P` does NOT rewrite `(if P then ...)`. Use:

```lean4
  have h_term4 : (if u < i ∧ (v+1 : ℕ) = (i+1 : ℕ) then D.f v (v+1) u i else 0) = 0 := by
    apply ite_eq_right_iff.mpr
    intro h
    rcases h with ⟨_, h_eq⟩
    have hv_ne_ip1 : v + 1 ≠ i + 1 := by omega
    exact absurd h_eq hv_ne_ip1
```

## Corner coefficient `D.f i(i+1) 1 n` — OUTSIDE cond_zero coverage

### cond_zero term classification technique

A systematic method to check whether a coefficient `D.f i j u v` can be isolated by any `cond_zero` equation:

**Template:** All 4 positions where `D.f i j` appears in cond_zero (with source pair `(i,j)` and target `(u',v')`):

| Term | Pattern | Condition | Coefficient |
|------|---------|-----------|-------------|
| **T1** | `(if u' < c ∧ v' = d then f i j u' c else 0)` | `u' < c`, `v' = d` | `f i j u' c` |
| **T2** | `(if u' = c ∧ d < v' then f i j d v' else 0)` | `u' = c`, `d < v'` | `f i j d v'` |
| T3 | `(if u' = i ∧ j < v' then f c d j v' else 0)` | `u' = i`, `j < v'` | `f c d j v'` |
| T4 | `(if u' < i ∧ v' = j then f c d u' i else 0)` | `u' < i`, `v' = j` | `f c d u' i` |

**For `D.f i(i+1) 1 n` with source `(i,i+1)`:**

- **T1:** Need `u' < c ∧ v' = d` with `u' = 1`, `c = n`. But `c < d` requires `n < d` with `d ≤ n` → impossible. Or `u' = 1, c = 1` with `u' < c` requires `1 < 1` → impossible.
- **T2:** Need `u' = c ∧ d < v'` with `d = 1`, `v' = n`. But `c < d` requires `c < 1` → impossible. Or `u' = d = n` with `c < n` and `d < v' = n`, requires `n < n` → impossible.
- **T3 (as cross-term f(i,i+1) via (c,d) = (i,i+1)):** Need `u' = ? ∧ (i+1) < v'` with target `(i+1, v') = (1,n)`. Requires `i+1 = 1` → impossible.
- **T4 (as cross-term):** Need `u' < ? ∧ v' = (i+1)` with target `(u', i+1) = (1,n)`. Requires `i+1 = n` → impossible since `i+1 < n`.

**Conclusion:** `D.f i(i+1) 1 n` cannot appear in **any** cond_zero equation.

### Implications

The `(1,n)` coefficient is NOT a standard Type C boundary. It requires a bracket-level HalfDerivation identity (not NCoeff-level equations) or a propagation argument.

### ⚠️ Central perturbation finding (2026-06-08)

A central perturbation `C_i(E(i,i+1)) = E(1,n)` with all other terms zero IS a
valid 1/2-derivation, verified in Lean. This produces an (n-1)-dimensional family
of 1/2-derivations NOT proportional to the identity.

**Structural explanation:** `Hom(L/[L,L], Z(L))` where L/[L,L] ≅ F^{n-1} and
Z(L) ≅ F·E(1,n). Total dim = 1 (Id) + (n-1) = n.

**Paper gap connection:** The code comments note that Theorem 4.2 has an
adjacent-source gap (no k exists for j=i+1). The code authors assumed this
doesn't affect the main classification (n≥5, D=c·Id) because the RecursiveSystem
only uses nonadjacent diagonals. Our central perturbation shows this assumption
is WRONG — it satisfies all constraints proven in the paper (nonadjacent
off-diagonal=0, all diagonal equations) but is NOT c·Id.

**Status:** The classification theorem `Der_{1/2}(N_n) ≅ F` (n≥5) may need
revision to `≅ F^n`. See `references/central-perturbation-discovery.md` for
full audit details.

This changes the Phase VA plan: instead of proving `Corner(i) = 0`, we may need
to accept the corner coefficient as a genuine degree of freedom and account for
it in the classification. The correct classification would be:
`D(E(i,i+1)) = c·E(i,i+1) + λ_i·E(1,n)` + interior Type C terms (proven zero)
+ Type A/B terms (proven zero by bracket propagation).

## Phase VA restructured 5-step plan

```
Step 1: Type A elimination           ← right bracket, descending induction
Step 2: Type B elimination           ← left bracket, ascending induction
Step 3: Interior Type C              ← cond_zero (v < n only) ✅
Step 4: Corner propagation           ← Corner(i) = Corner(i+1)?
Step 5: Corner = 0                   ← endpoint bracket
```

Terminology: `(1,n)` is now called **CornerCoefficient(i)** = `D.f i(i+1) 1 n`, not "Type C boundary".

## `Nat.decreasingInduction` CORRECT pattern

```lean4
let motive (m : ℕ) (hm : m ≤ n-2) : Prop :=
  1 ≤ m → m+2 ≤ n → ∀ u, 1 ≤ u → u < m → typeA D m u = 0
have h_base : motive (n-2) (by omega) := by
  intro hn2_pos hn2_bound u hu hun2
  ...
have h_step : ∀ k, k < n-2 → motive (k+1) (by omega) → motive k (by omega) := by
  intro k hk_lt h_next
  intro hk_pos hk_bound u hu huk
  ...
have hi_bound : i ≤ n-2 := by omega
have h_result := Nat.decreasingInduction h_step h_base hi_bound
exact h_result hi hin
```

## NCoeff structure reference

```lean4
structure NCoeff where
  f : ℕ → ℕ → ℕ → ℕ → F
  cond : ∀ i j u v, 1 ≤ i → i < j → j ≤ n → 1 ≤ u → u < v → v ≤ n →
    ∀ k, i < k → k < j → ...
  cond_zero : ∀ i j c d, 1 ≤ i → i < j → j ≤ n → 1 ≤ c → c < d → d ≤ n →
    j ≠ c → d ≠ i → ∀ u v, 1 ≤ u → u < v → v ≤ n →
    (if u < c ∧ v = d then f i j u c else 0)
    - (if u = c ∧ d < v then f i j d v else 0)
    + (if u = i ∧ j < v then f c d j v else 0)
    - (if u < i ∧ v = j then f c d u i else 0) = 0
```

## Type defs

```lean4
def typeA (D : NCoeff F n) (i u : ℕ) : F := D.f i (i+1) u (i+1)
def typeB (D : NCoeff F n) (i v : ℕ) : F := D.f i (i+1) i v
def typeC (D : NCoeff F n) (i u v : ℕ) : F := D.f i (i+1) u v
```
