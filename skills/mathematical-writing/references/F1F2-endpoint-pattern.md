# Endpoint Reduction — F1/F2 Pattern (Lean-Verified)

> Extracted from the Lean formalization of the N_n 1/2-derivation classification.
> When writing the endpoint reduction section of a derivation-classification paper,
> use this structure instead of ad-hoc case analysis or chain-bridge recurrences.

## The Pattern

For an adjacent source E_{k,k+1} whose image lies in the ideal I = ⟨E_{1,n-1},E_{1,n},E_{2,n}⟩,
apply the half-Leibniz condition to **two commuting pairs**:

| Constraint | Commuting pair | Target | Effect |
|-----------|---------------|--------|--------|
| **F2** | (E_{k,k+1}, E_{12}) | (1,n) | Interior c_k=0; boundary gives Feq |
| **F1** | (E_{k,k+1}, E_{n-1,n}) | (1,n) | Interior a_k=0; boundary gives Feq |

## Boundary handling (the critical detail)

The commuting-pair argument only works when the pair genuinely commutes
**and** the bracket-right term vanishes at the target. At the boundaries:

- **F2 at k=n-1**: The pair still commutes, but [E_{n-1,n}, E_{1,n-1}] = -E_{1,n} ≠ 0,
  so the bracket-right term does NOT vanish. It contributes -a_1, giving a_1 + c_{n-1} = 0 (Feq).
- **F1 at k=1**: Symmetric; contributes c_{n-1}, giving the same Feq.

**Correct bounds**: c_k=0 for 3≤k≤n-2 (NOT k≥3); a_k=0 for 2≤k≤n-3 (NOT k≤n-3).

## Common mistakes

1. Claiming "[E_{k,k+1}, I] = 0 for all k≥3" — FALSE at k=n-1 because
   E_{1,n-1}∈I shares index n-1 with E_{n-1,n}.
2. Claiming a chain-bridge recurrence couples adjacent c_k values — the chain bridge
   at target (2,n) gives zero for interior k (both bracket terms vanish).
3. Off-by-one at boundaries: "k≥3" should be "3≤k≤n-2"; "k≤n-3" should be "2≤k≤n-3".

## Why b_k are unconstrained

$b_k = \operatorname{coeff}(k,k+1; 1,n)$ — the coefficient of $E_{1,n}$.
Since $E_{1,n}$ is central in $N(n,F)$, its brackets with ANY element are zero:
$[b_k E_{1,n}, \cdot] = [\cdot, b_k E_{1,n}] = 0$. Neither F1 (pair with $E_{n-1,n}$)
nor F2 (pair with $E_{12}$) can produce a term involving $b_k$ at target $(1,n)$,
because both bracket-left and bracket-right terms vanish identically.
In Lean, this is structural: `coeff k (k+1) 1 n` does not appear in either
`F1_coeff_relation` or `F2_coeff_relation` (Spanning.lean lines 264-312).

Paper prose: "The $b_k$ are not constrained by either commuting pair, because
$E_{1,n}$ is central and its brackets with $E_{12}$ and $E_{n-1,n}$ are zero."

## Survivors (Lean-verified)

- b-channel: all b_k free (n-1 parameters)
- a-channel: a_1, a_{n-2}, a_{n-1}
- c-channel: c_1, c_2, c_{n-1}
- Feq: a_1 + c_{n-1} = 0 (the ONLY cross-endpoint coupling)
- Total endpoint parameters: 6 - 1 = 5
- Total φ_0 parameters: (n-1) + 5 = n+4

## Lean source files

- `E/Classification/Spanning.lean`: `interior_a_zero`, `interior_c_zero`, `coupling_a1_cn1`, `F1_coeff_relation`, `F2_coeff_relation`
- `E/Classification/DimensionTheorem.lean`: generator family A_k (b-channel) + B,C,D,E,F (5 boundary cocycles)
