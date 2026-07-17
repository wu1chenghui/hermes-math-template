# Paper ↔ Lean Proof-Level Consistency Audit

> **When to run**: After the paper's proof sketch is stable and all Lean proofs are closed.
> **Goal**: Verify that every proof step in the paper corresponds to a real Lean proof
> chain — at the PROOF-LINE level, not just the theorem-name level.

## Why Name-Level Matching Is Not Enough

A naive audit compares theorem statements:

> Paper Lemma 8 ↔ Lean `boundary_rigidity`

But this can be WRONG. Two theorems may have different statement shapes while
proving different intermediate results. The paper's F_eq (a₁ + c_{n-1} = 0) was
initially misidentified as corresponding to Lean's `boundary_rigidity`
(coeff(i,i+1;i,n)=0), when the actual Lean counterpart was `F1_coeff_relation`
(p=1 branch).

**The real question is not "what is the Lean theorem called?" but "does Lean
prove what the paper claims?"**

## Method

### Step 1: List every paper claim that invokes an external result

Go through the paper and extract every sentence of the form:

- "By Lemma X..."
- "From Theorem Y..."
- "The half-Leibniz condition gives..."
- "Expanding the bracket formula..."

Each of these is a **proof obligation** — it claims that a specific mathematical
fact follows from a specific source.

### Step 2: For each proof obligation, find the Lean proof chain

Do NOT just look up the theorem name in Lean's file outline. Instead:

1. Read the ACTUAL Lean proof body (use `lean_hover_info` or `read_file`)
2. Trace which lemmas it invokes internally
3. Verify that the conclusion matches what the paper claims at that point

The output is a proof-chain table:

```
Paper step | Lean lemma(s) used | Lean internal steps | Match?
```

### Step 3: Flag mismatches

Three types of mismatches:

- **DISCREPANCY**: The paper claims X but Lean proves Y (different statement)
- **EMBEDDED**: The paper has a standalone lemma but Lean embeds it in a larger proof
  (presentation choice, not an error)
- **STRONGER**: The paper claims less than what Lean actually proves (not an error,
  but an opportunity to strengthen the paper)

### Step 4: Check for "paper-only" claims

Are there proof steps in the paper that have NO Lean counterpart? These might be:

- Genuinely obvious (index arithmetic, algebra in a field)
- Hidden assumptions (the paper assumes something Lean needed to prove)
- Gaps (the paper claims something Lean doesn't prove)

## The `boundary_rigidity` / F_eq Case Study

This session discovered that a name-level match was wrong:

| Paper | Initially matched to | Actually matches |
|-------|---------------------|-----------------|
| F_eq bridge (a₁+c_{n-1}=0) | `boundary_rigidity` | `F1_coeff_relation` (p=1 branch) + `F2_coeff_relation` (p=n-1 branch) |

The Lean `boundary_rigidity` proves coeff(i,i+1;i,n)=0 for interior i — a
DIFFERENT result about a different target coordinate. Only by reading the
actual proof body of `F1_coeff_relation` could we trace the exact chain:

```
commuting_adj_F1       → (p,p+1) and (n-1,n) commute
bracketSource_commuting_eq_zero → bracketSource = 0
Phi=0 → bracketIdentity = 0
bracketIdentity_at_1n_canonical → expands to 4 indicator terms
Simplify → a_p + (if p=1 then c_{n-1} else 0) = 0
  p=1 → a₁ + c_{n-1} = 0  ← PAPER'S F_EQ
  p>1 → a_p = 0           ← CHANNEL RESTRICTION
```

## Pitfalls

- **Don't trust theorem names**: Two theorems can have the same name in different
  modules, or different names for the same result. Always read the statement.
- **Don't stop at the `:=`**: The Lean proof body may derive the paper's claim as
  an intermediate step or as a special case. The paper's "Lemma X" may be a
  corollary of a more general Lean theorem.
- **Lean may be MORE general**: `F1_coeff_relation` proves both F_eq AND channel
  restrictions. The paper separates them into two subsections for narrative
  reasons. This is fine — just document it.
- **"Equivalent after..." is dangerous**: Don't claim two statements are
  "equivalent after applying Lemma Y" unless you've verified the derivation
  explicitly. Report unverified equivalences as unverified.
