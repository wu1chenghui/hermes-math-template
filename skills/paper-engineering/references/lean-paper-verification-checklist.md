# Lean-to-Paper Verification Checklist

Checklist developed during the HalfDer(N_n) paper audit (2026-07-09).
Use when verifying a paper proof against a Lean formalization.

## 1. Constraint Source Audit

For each constraint claimed in the paper, locate its Lean origin:

- [ ] Which Lean theorem provides this constraint?
- [ ] What commuting/non-commuting pair does it use?
- [ ] What target coordinate is projected to?
- [ ] Does the paper use the same equation type (Ψ=0 vs full half-Leibniz)?

## 2. Boundary Index Audit

For every index range claim ("for k≥3", "for k≤n-3"):

- [ ] Check k=n-1 separately (right boundary often gives coupling, not zero)
- [ ] Check k=1 separately (left boundary often gives coupling, not zero)
- [ ] Compare with Lean's `interior_a_zero` / `interior_c_zero` bounds

## 3. Commuting vs Non-Commuting Pairs

For every Ψ=0 equation in the paper:

- [ ] Verify [x,y]=0 via bracket formula computation
- [ ] If [x,y]≠0, the equation MUST be 2φ[x,y] = Ψ(x,y,φ)
- [ ] Check that the bracket-source term is correctly included

Common failure: Case 2(v) of image restriction where Ψ(E_{i+1,u},E_{i,i+1})=0
is used but [E_{i+1,u},E_{i,i+1}]=-E_{i,u}≠0.

## 4. Chain Bridge vs Direct Φ Equations

- [ ] Does the paper claim a chain-bridge recurrence couples adjacent variables?
- [ ] Verify by expanding the chain bridge at the claimed target
- [ ] If both bracket terms vanish at that target, the recurrence is fabricated

In the half-derivation paper: chain bridge at (E_{k,k+1},E_{k+1,k+2}) with
target (2,n) gives 0=0 for interior k. No coupling exists.

## 5. Parameter Count Consistency

- [ ] List all surviving variables from the constraints
- [ ] Count independent relations
- [ ] Verify the count matches Lean's generator family cardinality
- [ ] Check that no variable is claimed both zero and surviving

## 6. Self-Pair Verification

- [ ] If paper has self-pair constraints: verify they are NOT tautologies
- [ ] [T(E),E] + [E,T(E)] = 0 is always a tautology (antisymmetry)
- [ ] Once im(φ)⊆I, self-pair imposes no new constraints

## Concrete Example: HalfDer(N_n) Endpoint Reduction

**Wrong (original paper):**
- c_k = 0 for k≥3 (WRONG: k=n-1 gives Feq, not zero)
- Chain bridge couples c_k to c_{k+1} (WRONG: brackets vanish)
- "c_2=0 (obtained below)" (WRONG: c_2 survives, nothing proves it zero)

**Correct (Lean-verified):**
- F2 (pair with E_{12}): c_k=0 for 3≤k≤n-2; k=n-1 gives a_1+c_{n-1}=0
- F1 (pair with E_{n-1,n}): a_k=0 for 2≤k≤n-3; k=1 gives a_1+c_{n-1}=0
- No chain bridge coupling needed
- Survivors: a_1, a_{n-2}, a_{n-1}, c_1, c_2, c_{n-1} (6→5 by Feq)
