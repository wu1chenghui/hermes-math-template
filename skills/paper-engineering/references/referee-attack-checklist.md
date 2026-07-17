# Referee Attack Audit — Seven-Round Checklist

> Run this AFTER the rigorous audit. Performed as HOSTILE REFEREE.
> Goal: find every possible attack vector and close it.
> Mindset: you WANT to reject this paper. Find the weakness.

---

## Round 1: Hidden Assumptions

### Attack
For every sentence containing "thus", "hence", "therefore", "clearly",
"it follows", "by induction", "by confluence", "without loss of generality",
"immediately", "directly", "routine", "obviously":

**Demand**: "Show me the missing step. It is not obvious to me."

### Specific targets
- "The full verification is a straightforward computation" → Prove it or cite it.
- "These are routine case analyses" → Show one. Cite the rest.
- "It follows that..." → It follows from WHAT exactly? Quote the proposition.
- "w decreasing by at least 1 per step" → Justify the bound.

---

## Round 2: Quantifier Audit

### Attack
For every theorem statement: does the proof actually prove ∀ or only ∃?

### Method
| Statement | Claimed | Proved | Verdict |
|---|---|---|---|
| "For any split i<k<j" | ∀ | Uses generic i,k,j | ✅ |
| "For any i with i+3≤n" | ∀ | Generic i | ✅ |

### Common errors
- Theorem says ∀ but proof only constructs one example
- Induction hypothesis stated as ∀ but applied only to a specific case
- "There exists" in proof but theorem claims "for all"

---

## Round 3: Boundary Audit

### Attack
Check every index bound. Every "n≥5". Every width=1,2,3,4 case.

### Specific checks
- Does Lemma 5.2 at i=1 REALLY need 1+4≤n? (n≥5 required)
- Does the base case of induction handle w=2 correctly?
- Does the sub-split in Lemma 5.1 actually have i+1 < i+3? (always true)
- Are there any off-by-one errors in width bounds?
- Does "p ≤ n-1" guarantee the needed index condition?

### Common failures
- Induction base case not checked for smallest possible input
- Width-1 boundary not separately handled
- n≥5 used where n≥6 would be needed (discovered too late)

---

## Round 4: Induction Legitimacy

### Attack
For every induction: does the recursive call STRICTLY decrease the measure?
Does the induction hypothesis actually APPLY to the recursive call?

### Method
For each induction, verify:
1. Base case: does it cover ALL minimal elements?
2. Recursive call: measure(child) < measure(parent)?
3. Applicability: does the child satisfy the theorem's hypotheses?

### Common failures
- Induction on (w, d) but child has w=1 (below w≥2 domain)
- Induction hypothesis requires w≥2 but child has w=1
- Target width decreases but source width stays same (wrong measure)
- "By induction hypothesis" applied to a case the IH doesn't cover

---

## Round 5: Cancellation Audit

### Attack
For every algebraic manipulation: is it valid in Field(char≠2)?

### Specific checks
- Every division by 2: preceded by "char F ≠ 2, so 2 is invertible"
- Every cancellation (a+b=a+c ⇒ b=c): valid in any field
- Every field_simp/ring operation: valid in Field F
- "Multiplying by 2" when 2=0 in the field? (excluded by char≠2)
- Subtraction used where addition+negation is meant

---

## Round 6: Dependency Minimality

### Attack
Does each theorem use ONLY the dependencies it claims?

### Method
For each theorem, list its claimed dependencies. Read the proof.
Does it silently use a result from a different section?

### Goal
Remove unnecessary dependencies. A theorem that claims to need
only §2-3 but actually uses §5 has a hidden dependency.

---

## Round 7: Hostile Referee Full Pass

### Attack
Read every sentence. Ask of each one:

- "Why is this true?"
- "What if this is false?"
- "Is there a counterexample?"
- "What hidden case is not covered?"
- "Does this actually follow from the previous sentence?"

### Specific patterns to attack
- "This is the central algebraic fact" → Is it really? Prove it's central.
- "Without loss of generality" → Justify the reduction.
- "One checks that..." → Check it. Show the check.
- "The verification is routine" → Show it or cite it explicitly.
- Parenthetical "(Equivalently: ...)" → Are the two statements actually equivalent?

### Go/No-Go criterion
The paper passes when: even a hostile referee, reading line by line,
cannot find a single step that requires the reader to fill in missing logic.
