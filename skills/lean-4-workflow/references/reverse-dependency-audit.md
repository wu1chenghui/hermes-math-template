# Reverse Dependency Audit — "Does Any Theorem Already Prove This?"

## When to use

When a coefficient/lemma resists all local proof attempts AND you've already done
occurrence audit (which equations mention it), switch to REVERSE dependency audit:

> **Search the entire project for any theorem whose CONCLUSION already states what
> you need — not in the proof body, not as an intermediate step, but as the final
> statement.**

This is distinct from:
- **Occurrence audit** — "which half-Leibniz equations involve this coefficient?"
- **DAG module audit** — "who imports this module / who calls this lemma?"
- **Standard dependency audit** — "what does lemma X depend on?"

## The method

```
For target statement P (e.g., "coeff(i,i+1; i,n) = 0"):
  1. Search the ENTIRE project (not just frontier file) for theorem/lemma
     signatures matching P's shape.
  2. Use regex patterns that match the CONCLUSION, not proof-body occurrences.
     E.g., 'lemma.*coeffOf.*\d+.*n\b' finds signatures, not internal uses.
  3. Check each match: does it conclude exactly P, or a specific instance of P?
  4. Classify: CLOSED (exact match), PARTIAL (instance for specific i/n),
     IRRELEVANT (different target), or EMPTY (nothing found).
```

## Interpretation

| Result | Meaning | Action |
|--------|---------|--------|
| EMPTY | No theorem in the project concludes P | P is a genuine gap; must be proven or the interface redesigned |
| PARTIAL | P holds for specific i/n but not generally | Check if those instances chain together; if not, gap remains |
| CLOSED (but not wired) | Someone already proved P; DAG is disconnected | Wire the existing theorem into the frontier — L2136 pattern |

## The L2136 pattern

Named after a case where `coeffOf D i (i+1) 2 (i+1) = 0` was proven in
`internal_det_step` but `imageContainment_skeleton` never called it. The math
existed; the DAG hadn't been connected. Reverse dependency audit catches this
because the theorem's CONCLUSION matches, even though its callers don't.

## v=n case study (2026-06-28)

Applied to `coeff(i,i+1; i,n) = 0` for centered D' (i ≥ 3):

- **EMPTY** — no theorem in the entire project concludes this.
- ImageContainment.lean:2072-2081 marks it `sorry` with a comment claiming
  "bracketSource via partner(i+1,n) couples GOAL to diagonal coeff(i,n;i,n)".
  Manual verification shows this mechanism produces 0=0 (all diagonal terms
  vanish for centered D'), yielding NO constraint on GOAL.
- **Confirmed gap** — the paper's mechanism (as recorded in the Lean comment)
  is insufficient. A new mathematical argument is needed.

**Follow-up (2026-06-28 g):** Experimental nullspace verification (n=5,6,7) confirmed
the statement IS true (dim C = n+4). The missing constraint is a global syzygy —
a linear combination of many half-Leibniz equations that is non-trivial only
when taken together, invisible term-by-term.
