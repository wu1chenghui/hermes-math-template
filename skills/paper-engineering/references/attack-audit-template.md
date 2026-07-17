# Referee Attack Audit — Methodology & Checklist

> Use when the paper draft is complete and you need to find every possible attack point.

## Seven Rounds

### Round 1: Hidden Assumptions
Search for: "thus", "hence", "therefore", "clearly", "it follows",
"by induction", "without loss", "immediately", "directly", "routine".

For each: can a hostile referee demand to see the missing step?
If yes → write the step explicitly.

### Round 2: Quantifiers
For every theorem statement:
- ∀ in the statement → verify the proof uses generic elements, not specific choices.
- ∃ in the proof → verify the chosen element satisfies all conditions.

Classic attack: "the proof claims ∀k but only checks k=i+1."

### Round 3: Boundary
Check every index bound against the standing assumptions (n≥5, char≠2).
Check every width case: w=1 (adjacent), w=2, w=3, w=4.
Check split validity at boundaries: i<k<j when i,j are at extremes.

### Round 4: Induction Legitimacy
For every induction:
- Base case: does it exist and is it proved?
- Recursive call: does the measure strictly decrease?
- For lexicographic induction: does at least one component strictly decrease?

### Round 5: Cancellation
For every algebraic manipulation:
- Is every division by 2 justified by char≠2?
- Is every subtraction in a field valid?
- Is every "multiply both sides by 2" using valid field multiplication?

### Round 6: Dependency Minimality
For each theorem: does it use only the dependencies it claims?
Remove each claimed dependency — does the proof still work? If yes, the proof
may have an unstated dependency or an over-stated dependency.

### Round 7: Full Hostile Referee
Read every sentence as a hostile referee. For each sentence ask:
- "Why?"
- "Is there another case?"
- "Is there a hidden assumption?"
- "Does this follow from what was proved earlier?"

## Red Flags (Must Fix)

- A lemma used in the inductive step when it's only needed for the base case.
- ONLY/PAIR lemmas (for commuting sources) applied to non-commuting pairs.
- "The bracket condition shows..." without specifying which bracket.
- "Rearranging yields..." without showing the algebra.
- Induction measure doesn't strictly decrease (e.g., p+2≤n fails at boundary).

## Yellow Flags (Should Fix)

- "Straightforward computation" without a reference to where.
- "Routine case analysis" that a referee might actually want to see.
- Compressed algebra: "multiplying by 2 and simplifying" instead of explicit steps.
