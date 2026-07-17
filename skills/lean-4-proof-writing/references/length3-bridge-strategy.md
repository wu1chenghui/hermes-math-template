# Length 3 Bridge Strategy — Adjacent-to-Length2 Conversion

## Origin

Developed June 2026 from Lean experiments on the 1/2-derivation classification project.
The `Length3Analysis.lean` experiment enumerated all projections of Length 3 compatibility
(fixed `E(i,i+3)`, split k=i+1 vs k=i+2) and classified the results.

## Key discovery

Length 3 compatibility is NOT the "mother theorem" for AdjacentBridge's 8 lemmas.
Instead, it produces a **larger linear system** where:

- Some patterns match AdjacentBridge (only_B, only_C, only_D)
- Some are **new** — about length-2 sources (i,i+2) and (i+2,i+3)
- PAIR patterns are inter-source **bridges**, not cross-source pairs

## Bridge identities (proven in Length3Bridge.lean)

```lean4
bridge_left :  coeff D i (i+1) u (i+1) = coeff D i (i+2) u (i+2)
bridge_right : coeff D (i+1) (i+3) (i+1) v = coeff D (i+2) (i+3) (i+2) v
```

These connect adjacent-source coefficients (Level 0, no split) to length-2 source
coefficients (Level 1, has split → can use `coeffOf_cond`).

## Accessibility grading (replaces length grading)

```
Level 0: Length 1 source — no split point
Level 1: Length ≥2 source — has split → coeffOf_cond works
```

Bridge identities move coefficients from Level 0 to Level 1.

## New research strategy

```
Old: Length3 compatibility → 8 AdjacentBridge lemmas (DISPROVEN)
New: Bridge Theorem → Length2 vanishing → AdjacentBridge
```

Steps:
1. Generalize bridge_left/bridge_right to a Unified Bridge Theorem
2. Apply `coeffOf_nonadjacent` or `coeffOf_cond` to the length-2 side
3. Conclude adjacent coefficient vanishing

## Proof pattern: explicit projection (replaces `simpl8` macro)

The old `simpl8` macro (`repeat' (split_ifs ...); simp; linarith`) was brittle.
The reliable pattern is:

```lean4
lemma bridge_left ... : coeff D i (i+1) u (i+1) = coeff D i (i+2) u (i+2) := by
  have h_compat := length3_compat D i u v ...
  rw [hv_eq] at h_compat  -- substitute v = i+3
  -- Left side: manually simplify all if-conditions
  have hl : <big if-expression> = coeff D i (i+1) u (i+1) := by
    simp [hu_lt, hu_ne_i, show i+3 ≠ i+1 from by omega, ...]
  have hr : <big if-expression> = coeff D i (i+2) u (i+2) := by
    simp [hu_lt_ip2, hu_ne_i, show i+3 ≠ i+2 from by omega, ...]
  rw [hl, hr] at h_compat
  exact h_compat
```

Key rules:
- Use explicit `have` blocks, not macros
- Pass ALL relevant inequalities to `simp`
- Use `neg_eq_zero.mp` instead of `linarith` on Field F
- Explicit `have hu_ne_i : u ≠ i := by omega` when `simp` leaves `u = i → ...`
