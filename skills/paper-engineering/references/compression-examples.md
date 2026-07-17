# Compression Examples — Lean Theorems → Mathematical Statements

> Worked examples from the 1/2-derivation classification project.
> Principle: many Lean theorems = atomic machine steps. Few mathematical
> statements = human insights.

---

## Example 1: Width Properties (10 → 3)

### Lean theorems (10)
```
left_width_lt          k-i < j-i
right_width_lt         j-k < j-i
width_strictly_decreases  both children smaller
width_sum_eq           (k-i)+(j-k) = j-i
children_positive      0 < k-i, 0 < j-k
reduction_terminates   width measure → finite paths
reduction_is_acyclic   no cycles
leaf_iff_adjacent      leaf ⇔ width=1
all_maximal_nodes_adjacent  max path ends at adjacent
nonadjacent_has_child  width≥2 → has split
```

### Why they are separate in Lean
Each is a different arithmetic operation (Nat.sub_lt_sub_right, omega,
Nat.sub_pos_of_lt, etc.). The proof assistant needs each as a lemma.

### Compressed mathematical statements (3)

**Proposition 2.1** (Width Monotonicity). w(I_L), w(I_R) < w(I),
w(I_L)+w(I_R)=w(I), both positive.

**Proposition 2.2** (Finiteness). The reduction graph is a finite DAG.
Every maximal path has length ≤ w(I)−1.

**Proposition 2.3** (Leaf Characterization). Leaf ⇔ adjacent ⇔ w(I)=1.

### Compression logic
Props 2.1-2.3 group theorems by mathematical concept:
- Width arithmetic (5→1)
- Finiteness consequences (2→1)
- Leaf structure (3→1)

---

## Example 2: Diagonal Evaluation (7 → 3)

### Lean theorems (7)
```
eval_diagonal          2c = L+R
eval_diagonal_half     c = ½(L+R)
eval_edge_weight       c = ½L + ½R
eval_diagonal_invariant  L+R independent of split
eval_nonadjacent_expand  expand at k=i+1
eval_leftmost_leaf     leftmost child is leaf
eval_leaf_representation  root = convex combination of leaves
```

### Compressed (3)

**Proposition 3.1** (Diagonal Averaging). c_I = ½c_{I_L} + ½c_{I_R}.

**Theorem 3.2** (Confluence). c_{I_L(k₁)} + c_{I_R(k₁)} = c_{I_L(k₂)} + c_{I_R(k₂)}.

**Corollary 3.3** (Canonical Decomposition). Expand at k=i+1; left child is leaf.

### Compression logic
eval_diagonal + eval_diagonal_half + eval_edge_weight are three forms
of the SAME equation (multiply/divide by 2, distribute). One proposition.

eval_nonadjacent_expand + eval_leftmost_leaf are immediate corollaries
of Prop 3.1 with k=i+1. One corollary.

eval_leaf_representation is a structural remark, not a separate theorem.

---

## Anti-Pattern: What NOT to compress

### skip_two_eq and adjacent_step_eq
These should REMAIN SEPARATE in the paper because:
- skip_two_eq: separates leaves into two parity classes
- adjacent_step_eq: bridges the parity classes

They are genuinely different mathematical ideas, even though both use confluence.
Compressing them into one lemma would obscure the proof structure.

### Rule of thumb
If two Lean theorems embody DISTINCT mathematical insights, keep them separate
in the paper. Only compress when they are different formalizations of the SAME insight.
