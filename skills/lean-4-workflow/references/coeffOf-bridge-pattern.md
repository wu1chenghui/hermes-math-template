# coeffOf Bridge Pattern

## When to use

When you need to put a `HalfDerivation`'s coefficient table into `kerΦ` (which
requires `IsValidSupp`), but the raw `.coeff` field may have arbitrary garbage
at invalid indices.  Use `coeffOf D` as the canonical representative instead
of `D.coeff`.

## Why coeffOf

`coeffOf D i j u v` is defined as:
- `D.coeff i j u v` when all indices are valid (`1≤i<j≤n`, `1≤u<v≤n`)
- `0` otherwise

Consequences:
- **IsValidSupp is automatic**: `coeffOf` returns 0 at every invalid index tuple
- **coeffOf equals coeff at valid positions**: via `coeffOf_f` lemma
- **coeffOf is linear**: `coeffOf_add`, `coeffOf_sub`, `coeffOf_smul`, `coeffOf_neg`

## The bridge problem

When using `coeffOf D` for kerΦ membership, the Phi condition requires:
```
Phi(coeffOf D) i j p q u v = 0    (for all valid top-level indices)
```

We already have `halfDerivation_phi_zero D` giving `Phi(D.coeff) ... = 0`.
Need to bridge: `Phi(coeffOf D) = Phi(D.coeff)` at valid indices.

Phi = 2*bracketSource - bracketIdentity, so we need two bridges:
1. `bracketSource(coeffOf D) = bracketSource(D.coeff)`
2. `bracketIdentity(coeffOf D) = bracketIdentity(D.coeff)`

## Primitive-before-composite strategy

**Do NOT prove the bracketIdentity bridge in one 16-branch lemma.**
Instead, break into primitives:

### bracketSource (2 conditions, 4 branches)

Key insight: the branch where BOTH `j=c` AND `i=d` is unreachable under validity
hypotheses (`i<j` and `c<d` give `d<d`).  Use `omega` to close it directly.

```lean
lemma coeffOf_bracketSource_eq (D) (i j c d u v) (hi hij hjn hc hcd hdn hu huv hvn) :
    bracketSource (fun a b e f => coeffOf D a b e f) i j c d u v =
    bracketSource D.coeff i j c d u v := by
  unfold bracketSource
  by_cases hjc : j = c
  · have hid : i < d := by omega
    have h1 : coeffOf D i d u v = D.coeff i d u v := coeffOf_f D i d u v hi hid hdn hu huv hvn
    by_cases hid' : i = d
    · omega  -- j=c and i=d contradicts i<j and c<d
    · simp [hjc, hid', h1]
  · by_cases hid' : i = d
    · have hcj : c < j := by omega
      have h2 : coeffOf D c j u v = D.coeff c j u v := coeffOf_f D c j u v hc hcj hjn hu huv hvn
      simp [hjc, hid', h2]
    · simp [hjc, hid']
```

### bracketLeft (2 conditions, 4 branches)

Conditions: `u < c ∧ v = d`, `u = c ∧ d < v`.
Positions: `(i,j; u,c)`, `(i,j; d,v)`.

```lean
lemma coeffOf_bracketLeft_eq (D) (i j c d u v) (hi hij hjn hc hcd hdn hu huv hvn) :
    bracketLeft (fun a b e f => coeffOf D a b e f) i j c d u v =
    bracketLeft D.coeff i j c d u v := by
  unfold bracketLeft
  by_cases h1 : u < c ∧ v = d
  · have hpos : coeffOf D i j u c = D.coeff i j u c :=
      coeffOf_f D i j u c hi hij hjn hu h1.left (by omega)
    by_cases h2 : u = c ∧ d < v
    · have hpos2 : coeffOf D i j d v = D.coeff i j d v :=
        coeffOf_f D i j d v hi hij hjn (by omega) h2.right hvn
      simpa [h1, h2, hpos, hpos2]
    · simpa [h1, h2, hpos]
  · by_cases h2 : u = c ∧ d < v
    · have hpos2 : coeffOf D i j d v = D.coeff i j d v :=
        coeffOf_f D i j d v hi hij hjn (by omega) h2.right hvn
      simpa [h1, h2, hpos2]
    · simpa [h1, h2]
```

### bracketRight (2 conditions, 4 branches)

Conditions: `u = i ∧ j < v`, `u < i ∧ v = j`.
Positions: `(c,d; j,v)`, `(c,d; u,i)`.  Symmetric to bracketLeft.

### Assembly (1 line each)

```lean
lemma coeffOf_bracketIdentity_eq (D) ... := by
  unfold bracketIdentity
  rw [coeffOf_bracketLeft_eq D ..., coeffOf_bracketRight_eq D ...]

lemma coeffOf_Phi_eq (D) ... := by
  unfold Phi
  rw [coeffOf_bracketSource_eq D ..., coeffOf_bracketIdentity_eq D ...]
```

## Pitfalls

### `obtain` destroys `simpa` targets
`obtain ⟨huc, hvd⟩ := h1` where `h1 : P ∧ Q` removes `h1` from context.
Later `simpa [h1, ...]` fails with "Unknown identifier h1".

**Fix**: Do NOT `obtain`. Use `.left` / `.right` for component access:
```lean
  by_cases h1 : u < c ∧ v = d
  · have hpos := coeffOf_f ... h1.left ...
    simpa [h1, hpos]
```

### `simpa` over `rw` for lambda-wrapped hypotheses
When a hypothesis wraps a lambda expression:
```lean
hne : (fun i j u v => coeffOf D i j u v) i j u v ≠ 0
```
`rw [hcoeff] at hne` fails: "Did not find an occurrence".  Use `simpa`:
```lean
have hne' : D.coeff i j u v ≠ 0 := by simpa [hcoeff] using hne
```

### `simp` over-rewriting with equality hypotheses inside conjunctions
When `h2 : u = c ∧ d < v` and the goal contains `coeffOf ... u ...`,
`simp [h2]` rewrites `u` to `c` in coefficient positions, mismatching
`coeffOf_f` lemmas that reference the original index.
Prefer `simpa [h1, h2, hpos, ...]` — it matches the simplified form
against the target rather than rewriting the target.
