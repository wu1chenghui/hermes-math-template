# Induction Contamination Check — When a Counterexample Does (and Doesn't) Break an Induction

## The Problem

You find a counterexample to a theorem's full statement. The theorem uses induction.
A natural fear: "does the counterexample propagate through the induction, collapsing
the entire proof?"

This reference provides a method to check systematically.

## Method: Trace All Recursive Call Targets

For each recursive call in the induction step, compute:
1. The child source (what source does the IH get called on?)
2. The child target (what target does the IH try to prove?)
3. Can the child target be the counterexample?

If NO child target can be the counterexample, the induction logic is NOT contaminated.
The counterexample only affects the top-level full statement, not the induction's validity.

## Example: P_1 and coeffOf_nonadjacent (June 2026)

**Counterexample**: P_1(E_{12}) = E_{1n}. So c_{12}^{1,n} = 1 ≠ 0.

**Theorem statement**: ∀(i,j,u,v), (u,v)≠(i,j) → c_{ij}^{uv} = 0.  ← FALSE (fails at (1,2)→(1,n))

**Question**: Does this break the induction for j-i ≥ 2?

**Induction structure** (coeffOf_nonadjacent, h_ge2 branch):
- Split at k = i+1
- Four recursive branches: IA, IB, IIA, IIB

**Trace**:
| Branch | Child Source | Child Target | = (1,n)? |
|--------|-------------|--------------|----------|
| IA     | (i,i+1)     | (u, i+1)     | ❌ i+1 < n |
| IB     | (i,i+1)     | (j, v)       | ❌ j > 1   |
| IIA    | (i+1,j)     | (i+1, v)     | ❌ i+1 ≥ 2 |
| IIB    | (i+1,j)     | (u, i)       | ❌ i < n   |

**Conclusion**: No recursive call ever targets (1,n). The induction for nonadjacent
sources is NOT contaminated. Only the theorem's full-claim statement needs correction.

## When It IS Contaminated

If ANY recursive branch can produce the counterexample's target, the induction proof
genuinely needs restructuring. Example: if IA's child target could be (1,n), then
the IH call to prove c_{i,i+1}^{(1,n)} = 0 would need the full-strength statement
(which is false).

## Verification in Lean

Use `lean_verify` to check for `sorryAx` in the transitive axiom closure:
```
mcp_lean_lsp_lean_verify(file_path="...", theorem_name="...")
```
If `sorryAx` appears, the theorem transitively depends on a `sorry`.
