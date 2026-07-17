# Qualitative Reading Framework — Q1-Q6 Paragraph Analysis

Use when comparing your paper's prose style against a reference paper.
Not a checklist — a reading method. For each paragraph, answer six questions.

## The Six Questions

| # | Question | What It Reveals |
|---|----------|----------------|
| Q1 | 这一段做了什么？（宣布/计算/总结/过渡） | Paragraph function |
| Q2 | 作者没写什么？（省略了哪些推导/为什么可以省） | Compression strategy |
| Q3 | 读者需要自己补什么？（补多少步） | Information density |
| Q4 | 段与段之间怎么连接？（空白行/连接词/重复关键词） | Rhythm and transitions |
| Q5 | 这一段让我感觉"必然"还是"选择"？（为什么） | Inevitability |
| Q6 | 这一段的情绪是什么？（中性/紧张/释放/疑惑） | Narrative arc |

## Application to Ou's §3 (2007)

### Paragraph 3 (φ(E_{1k}), k=4,...,n)

| Q | Answer |
|---|--------|
| Q1 | Compute — HL expansion → mod-α₁ reasoning → conclusion |
| Q2 | "It is easy to check that [E_{12},φ(E_{2k})]∈α₁" — doesn't show why |
| Q3 | 1 step: E_{12} has row 1, any bracket result also has row 1 → ∈α₁ |
| Q4 | "Now we consider" → announces new sub-target within Step 1 |
| Q5 | High: chain of inclusions (M_k ∩ (α₁+α₂+β_k) ⊆ α₁+α₂) forces the conclusion |
| Q6 | Neutral — efficient computation, no emotional arc |

### Paragraph 5 (φ(E_{12}) — longest computation in the paper)

| Q | Answer |
|---|--------|
| Q1 | Compute φ(E_{12}) mod α₁ — two constraints via commuting pairs |
| Q2 | "This implies that Σ s_{ik}E_{in}≡0 (mod α₁), forcing s_{ik}=0" — compresses 3-4 steps |
| Q3 | 3-4 steps: expand bracket, identify α₁ terms, eliminate |
| Q4 | "We next consider" → sequential. "Thus" → reduction. "So" → final simplified form |
| Q5 | Very high — HL equations ARE the constraints, no choice involved |
| Q6 | Explore → narrow → clean. "Suppose" (open) → "forcing" (close) → "So" (order restored) |

### Paragraph 7 (Construct ρ, transition to φ₁)

| Q | Answer |
|---|--------|
| Q1 | Three actions merged: choose r,s → construct ρ → check → denote φ₁ |
| Q2 | "it is not difficult to check" — compresses the verification that ρ is a derivation |
| Q3 | 2-3 steps: substitute ρ's definition into HL equation |
| Q4 | "Now we choose... Then... Now we denote" — Ou's three-part transition |
| Q5 | Natural — r,s are exactly the coefficients extracted from the previous computation |
| Q6 | Release — computation done, construction complete, entering new stage |

## Application to KK's Lemma 7 Proof (2023)

### Single paragraph (7 sentences)

| Q | Answer |
|---|--------|
| Q1 | Compute — numbered equation chain: (13) → multiply → whence → substitute → □ |
| Q2 | Nothing omitted — every algebraic step is visible as a numbered equation |
| Q3 | 0 steps — follow the equation numbers |
| Q4 | "Multiplying (13) by e_{ii} on the left, we get" → "whence" → "Substituting this into (13)" |
| Q5 | High — the equations are the proof, no justification needed beyond the algebra itself |
| Q6 | Mechanical precision — "(recall that char(F)=0)" is the only human interjection |

## When to Use

Run this analysis on your own paper's proof paragraphs. Compare the Q1-Q6 profile
against the reference paper you're imitating. Mismatches in compression level (Q2/Q3)
or transition style (Q4) are the most actionable differences.
