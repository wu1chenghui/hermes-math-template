# Polish Checklist — Final-Pass Tells

Run AFTER the standard audits (negative space, authenticity, error patterns)
and the proof completeness audit. These are subtle tells that make a paper read
like a development artifact rather than a published math paper. They survive the
standard audits because none of them are "wrong" — they're just not quite right.

Source: mathematician-reviewer cold read of a derivation-classification paper
that had passed all standard skill audits.

## Tells to Remove

| # | Pattern | Why It's a Tell | Fix |
|---|---------|-----------------|-----|
| 1 | Internal code names as labels (`F1`, `F2`, `P1a`) | Exposes the implementation structure. A mathematician labels by WHAT the thing is, not its position in a dependency graph. | Use descriptive headers: `\textit{Pair with $E_{12}$.}` not `\textbf{F2 constraint}` |
| 2 | `one checks` / `One checks` | Tells the reader what to do rather than stating a fact. A mathematician just states the result with its justification. | `$[E_{12},E_{v,v+1}]=0$ because the index sets are disjoint` |
| 3 | Non-standard case labels (`Regime`) | "Regime" is unusual in math papers. Use "Case" or embed the condition in prose. | `If $u=1$, then ...` |
| 4 | Same fact repeated 4+ times | Suggests the author doesn't trust the reader's memory. A mathematician states a fact once and cites it. | Extract to an Observation block early in the section, then cite "by the Observation" |
| 5 | `The aim of this paper is to classify...` immediately before the Theorem | Redundant filler. The Theorem IS the aim. | Delete the sentence; go directly from literature review to Theorem statement |
| 6 | `belong to` in formal membership contexts | Use the symbol `∈` for membership and `⊆` for subset. "Belong to" is English prose; math papers use symbols for formal statements. | `im(ω₃) ⊆ I` not "images belong to I" |
| 7 | Lemma with a one-line proof citing an equation from 30 lines earlier | Makes the Lemma feel like an afterthought. Either expand to a self-contained 3-line proof, or remove the Lemma and cite the equation directly. | Give a self-contained bracket expansion proof |
| 8 | `"see the Lean formalization"` or `"verified in Lean"` as proof | A math paper must stand on its own. The Lean proof guides structure but the paper must present the argument independently. | Delete the reference; write the actual computation or state "solving this system forces..." |
| 10 | Tutorial-style bracket expansions | `only the term $E_{1v}$ of $\\varphi(E_{12})$ can produce the target $(1,v+1)$, via $[E_{1v},E_{v,v+1}]=E_{1,v+1}$, contributing $\\pi_{1v}$`. Survives ALL standard audits (no forbidden words, clean English, correct math) — but reads like LECTURE NOTES, not a journal paper. A mathematician fills in bracket expansions themselves; explaining each step is the tell of an author who doesn't trust the reader. | `The first bracket contributes $\\pi_{1v}(\\varphi(E_{12}))$ (from $E_{1v}$).` Or better: state the simplified equation directly and let the reader identify the terms. Pattern: delete `\"only... can produce... via... contributing...\"`, delete `\"all other terms have column≠v or row≠1\"`, delete δ-symbol walkthroughs. Common in image-restriction / endpoint-reduction passages of derivation-classification papers. |

## Positive Patterns to Adopt

| Pattern | Example |
|---------|---------|
| Observation block for repeated facts | `\smallskip\noindent\textit{Observation.} No adjacent basis element $E_{k,k+1}$ is a commutator in $N_n$: ...` |
| Direct statements with justification | `$[E_{n-1,n},E_{1,n-1}]=-E_{1,n}$ by~\eqref{eq:bracket}, so ...` |
| Descriptive section headers | `\textit{Pair with $E_{12}$.}` instead of `\textbf{F2 constraint}` |
| Self-contained lemma proofs | Even a short lemma gets a 3-4 line proof showing the key bracket expansion |
| Prose-embedded case distinctions | `If $u=1$, then ... If $u=2$, the same pattern applies. If $u\ge3$, set $x=...$` instead of labeled "Regime" blocks |

## Application Order

Run this checklist LAST, after all mathematical fixes and standard audits pass.
These are the final 5% that separate a correct paper from a polished one.
