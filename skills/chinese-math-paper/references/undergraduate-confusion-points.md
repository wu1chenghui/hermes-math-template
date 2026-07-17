# Undergraduate Confusion Points (Math Paper Pedagogy)

Eight points where undergraduates predictably stumble when reading a
formal Lie algebra proof. Address these proactively in lecture notes.

## 1. Dynkin diagrams for nilpotent algebras

**Confusion**: Dynkin diagrams classify semi-simple Lie algebras. Why use
them for nilpotent N_n?

**Answer**: N_n is the positive root space of sl_n. The "simple roots"
E_{k,k+1} are just the algebraic generators. The Dynkin diagram language is
borrowed for visual intuition — lines between nodes mean "these two bracket
non-trivially."

## 2. "Cocycle" terminology

**Confusion**: The word "cocycle" appears with no definition.

**Answer**: Borrowed from Lie algebra cohomology. Maps satisfying
D[x,y] = [Dx,y] + [x,Dy] are called 1-cocycles. Half-derivations are close
relatives. "Boundary cocycle" = sparse solution supported only near Dynkin
endpoints.

## 3. SEq / rank-nullity conversion

**Confusion**: How does dim HalfDer = dim Hom - dim SEq? The jump feels
like magic.

**Answer**: Think of T as a d²-dimensional column vector. Each φ ∈ SEq is a
row of the coefficient matrix. SEq = row space. dim HalfDer = variables -
independent equations = d² - dim SEq. Walk through in 4 explicit steps.

## 4. Image restriction → I

**Confusion**: Why does self-pairing force the image into exactly
span{E_{1,n-1}, E_{1,n}, E_{2,n}}?

**Answer**: Geometric intuition — these three elements are at the "extreme
upper-right corner" of the matrix. Row indices minimal (1 or 2), column
indices maximal (n-1 or n) → they almost commute with all E_{k,k+1}.
Everything else produces non-cancelling brackets. The rigorous proof
expands T(E_{k,k+1}) in the basis and checks each E_{pq} individually.

## 5. Bracket computation mechanics

**Confusion**: How to compute [E_{ab}, E_{cd}] quickly?

**Answer**: The simplified formula for strictly upper triangular matrices:
[E_{ab}, E_{cd}] = δ_{bc}E_{ad}. Domino rule: non-zero iff first's column
equals second's row. Almost never need the second term δ_{da}E_{cb}.

## 6. "Diagonal" means self-coefficient

**Confusion**: N_n has zero main diagonal. What "diagonal"?

**Answer**: The "diagonal" in diagonal propagation means π_{ij}(T(E_{ij})) —
the coefficient of E_{ij} itself in T(E_{ij})'s expansion. Example:
T(E_{12}) = 3E_{12} + 5E_{13} → self-coefficient = 3. The remarkable fact:
all n-1 adjacent sources have the SAME self-coefficient.

## 7. Width reduction — not circular

**Confusion**: "Iterating gives width 1" sounds like infinite regress.

**Answer**: "Breaking sticks" analogy — source (i,j) is a stick of length
w=j-i. Each chain bridge split reduces width by at least 1. After w-1
splits, all pieces have width 1. Termination is guaranteed.

## 8. Linear independence proof

**Confusion**: "Technical details omitted" — how to actually prove
n+5 maps are independent?

**Answer**: "Test point" method. Assume linear combination = zero map.
Evaluate at specific E_{ij}, project to specific π_{uv}. Each test isolates
one coefficient. Build a table: (test point, projection, which e_α survives,
conclusion c_α=0). Work through all n+5 systematically.
