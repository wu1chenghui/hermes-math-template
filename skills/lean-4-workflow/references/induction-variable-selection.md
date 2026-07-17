# Induction Variable Selection — Pattern from 1/2-Derivation Project

> **When the base case is harder than the inductive step, the induction variable is wrong.**

## The Pattern

In the 1/2-derivation classification project, the paper's Theorem 6.4 (off-diagonal
vanishing) was initially proved by **source-width induction** (induction on w(I)=j-i).
This forced a difficult base case: w=2 produces width-1 children, and proving those
children's coefficients vanish required a complex adjacent-source lemma (Lemma 6.3)
with an 8-way case analysis.

Lean's proof of `coeffOf_nonadjacent` uses **target-width induction** (induction on
d=v-u). The induction hypothesis covers children of any source width, eliminating
the boundary problem entirely. The proof shrinks from ~15 lines to ~8 lines.

## Diagnostic

If you find yourself writing extensive case analyses for induction base cases,
stop and re-examine the induction variable. The right variable makes base cases
trivial; the wrong variable makes them the hardest part of the proof.

## Checklist

1. **Read Lean's `induction'` line.** What does the Lean proof actually induct on?
   It may differ from the most obvious variable.
2. **Verify strict decrease.** Each recursive call must strictly decrease the measure.
3. **Check IH coverage.** The induction hypothesis must cover ALL sub-cases
   encountered in the inductive step, including boundary configurations.
4. **If the IH doesn't cover:**
   - Option A: Change the induction variable.
   - Option B: Broaden the theorem statement (e.g., from "w≥2" to "all sources").
   - Option C: Split into a generic induction + a boundary lemma (with explicit
     dependency documentation to avoid circularity).

## Example from this project

| | Source-width induction (wrong) | Target-width induction (right) |
|---|---|---|
| Variable | w(I)=j-i | d=v-u |
| Base case | w=2 → children w=1 → hard | None (strong induction) |
| IH covers width-1? | No (theorem stated for w≥2) | Yes (IH for all source widths) |
| Proof length | ~15 lines | ~8 lines |
| Lean alignment | No (`coeffOf_nonadjacent` uses d=v-u) | Yes |
