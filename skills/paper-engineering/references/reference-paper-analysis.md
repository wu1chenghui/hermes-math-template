# Reference Paper Writing DNA — Sentence-Level Imitation Guide

> Extracted from forensic analysis of Ou-Wang-Yao (2007) "Derivations of N(n,R)"
> and Kaygorodov-Khrypchenko (2023) "Transposed Poisson structures on T_n(F)".
> Use this when rewriting a paper to match the voice of published derivation-classification papers.

## Part 1: Two Computational Philosophies (Never Mix)

| Scenario | Philosophy | Source | Key Tools |
|----------|-----------|--------|-----------|
| Subspace-level reasoning | **Ou style** | Ou §3 | ≡ (mod I), ∈ inclusion chains, "By this we may suppose" |
| Entry-level matrix reasoning | **KK style** | KK Lemma 7 | "Multiplying by X on the left", whence, numbered intermediate equations |

**Iron rule**: One proof block = one philosophy. Never mix mod-subspace and entry-level reasoning in the same paragraph.

## Part 2: Ou's Six-Step Bracket Calculation Template

```
1. ANNOUNCE target:
   "Now we consider the action of φ on E_{ab}."

2. PARAMETERIZE unknown:
   "Suppose that φ(E_{ab}) = Σ r_{ij} E_{ij}."
   OR "By this we may suppose that φ(E_{ab}) ≡ ... (mod I)."

3. INVOKE derivation condition:
   "By applying φ to [E_{ab}, E_{cd}] = 0, we see that
    [φ(E_{ab}), E_{cd}] + [E_{ab}, φ(E_{cd})] = 0."

4. EXPAND brackets and deduce constraints:
   "This implies that Σ ... ≡ 0 (mod I), forcing r_{ij} = 0 for i = ..."

5. STATE reduced form:
   "Thus φ(E_{ab}) ≡ ... (mod I)."

6. TRANSITION to next:
   "Now we choose X = ... to construct ..."
   OR "Now we denote φ − X by φ₁."
```

**Key rules**: Steps 1-6 are four distinct rhythms (announce/analyze/construct/name). Never merge them. Skip intermediate verification with "It is easy to check" (max 3× per paper). Skip repeated calculations with "Similarly". But NEVER skip the bridge bracket relation (E_{ab} = [E_{cd}, E_{ef}]).

## Part 3: KK's Entry-Level Computation Template

```
"By (8) and (i),(ii),(iv) of Lemma 6 we have
 2φ(e_{ij}) = (...).                                    (13)

 Multiplying (13) by e_{ii} on the left, we get
 2e_{ii}φ(e_{ij}) = (...).
 whence
 e_{ii}φ(e_{ij}) = (...).

 Substituting this into (13) and dividing by 2
 (recall that char(F) = 0), we prove the first equality. □"
```

**Key rules**: Number every intermediate equation. "Multiplying X by Y on the left/right" — always specify direction. "whence" = skip one algebraic manipulation step. Conditions go in parentheses: "(recall that char(F)=0)".

## Part 4: KK's Merge Pattern ("Combining... we see that...")

```
"Combining (19)–(24) and denoting a_{ij} := z^j_{ii}, b := y_{11},
 we see that the only (possibly) non-zero products are

   [simplified form A]                                        (25)
   [simplified form B]                                        (26)

 where [additional condition]."

Immediately followed by classification or parameter count.
```

**Key rules**: Every elimination layer gets a numbered equation. The final merge cites all of them. Rename parameters in the merge step. Put extra conditions in a "where" clause.

## Part 5: Construction Template for §2 — (A)-(E) Format

```
(A) [Name].
    [One-sentence definition + verification].   ← simple constructions
    OR
    [One-sentence definition].
    [One-sentence verification].                 ← complex constructions
```

**Constraints**: No "Definition" or "Lemma" environment — use letter labels. No "Verification:" sub-heading. Verification never expands the computation — use "by (1.1)", "by index constraints", "since ... vanishes". Each construction is self-contained.

## Part 6: Proposition vs Theorem Semantics

| Type | Meaning | Use for |
|------|---------|---------|
| Lemma | Single technical fact | bridge lemmas, reduction steps |
| Proposition | Intermediate structural result ("X equals Y") | basis linear independence |
| Theorem | Final classification result | dim Δ(N_n) = n+5 |

## Part 7: Voice Principles — Creating "Inevitability"

### Zero hedging
NO perhaps, maybe, might, seems, appears anywhere in the paper. State facts without qualification. If something is only conditionally true, find the right condition and state it unconditionally.

### Never call things "obvious"
Ou: 0× "clearly", 0× "obviously". If something is obvious, stating it without commentary IS the most obvious presentation.

### Never explain why you chose this argument
Present the argument as if it were the only possible approach. Don't write "We choose this bracket decomposition because...". Just write the decomposition.

### Atomic paragraphs
Each paragraph does exactly one thing: announce → compute → conclude. 2-3 sentences. No "Moreover/Furthermore/Additionally" between paragraphs — the blank line is the transition.

### Minimal transition vocabulary
Use ONLY: Then, Now, Thus, So. Never: Moreover, Furthermore, Also, Next, Finally, Consequently, Therefore, Hence.

### Four-layer inevitability engine
1. §2 constructions → used immediately in §3 Step 1 (no waiting)
2. Each Step title = the goal ("Step N: we can choose X such that...")
3. Every Step shrinks the problem (φ → φ₁ → φ₂ → φ₃ → φ₄)
4. Final Step reveals a known structure ("φ₄ exactly is a central derivation")

### Let:we ratio ≈ 0.32
Use "we" 3× more often than "Let". Guide the reader through discovery. Passive voice only in definitions ("is called", "is denoted") — twice per paper max.

### Math density ≈ 0.24
One math symbol per 4 English words. English is the primary language; symbols are decoration.

## Part 8: Forbidden Words

| Forbidden | Replace With |
|-----------|-------------|
| Therefore, Hence, Consequently | **Thus** or **So** |
| Namely, i.e. (prose explanation) | State the result directly |
| vanishes, is zero | = 0 |
| lies in, belongs to | ∈ |
| contradiction | forcing ... = 0 |
| induction | Step structure or chained equalities |
| immediately | (omit) |
| respectively | Write two separate sentences |
| clearly, obviously | (omit) |

i.e. exception (2026-07-11): i.e. that performs symbol substitution
is ALLOWED. "= π, i.e., 2Y = X" is fine. "π = 0, i.e., the diagonal
vanishes" is forbidden. "whence" is always allowed.

## Part 9: Sentence-Level Quick Reference

### Verbs of deduction (prefer top half)
- **we see that** — read directly from expansion (Ou's favorite)
- **we get** — after simple algebra
- **forcing ...** — strong conclusion
- **This shows that** — directional conclusion
- we obtain, it follows that — KK style, use sparingly

### Reminder phrases (semantically distinct)
- **Note that** — side observation during calculation
- **Recall that** — remind of already-proved fact
- **Observe that** — non-obvious consequence (use rarely)

### "Now" has three meanings
- "Now we consider" → announce new target
- "Now we choose" → parameter extraction + construction
- "Now we denote" → name the result (= step complete)

### How to write zero and membership
- Always `r_{ij} = 0`, never "r_{ij} vanishes" or "is zero"
- Always `φ(E_{ij}) ∈ I`, never "lies in" or "belongs to"

### How to handle the center
- Number the key identity: "E_{1,n} is central in N_n. (1.1)"
- Thereafter always cite "by (1.1)", never "since E_{1,n} is central"

### Equation references
- Just the number: "by (5)", never "by equation (5)" or "by Eq. (5)"

### Closing proofs
- End with □ alone — no "This completes the proof" (used 1× across both papers)

### Quantifier style
- Setup: "Let R be an arbitrary commutative ring" (arbitrary)
- Ongoing: "any derivation φ", "any x ∈ N₁" (any)
- Ranges: "k = 1,...,n−1" (not "for all k")

### If-then
- Always write "If X, then Y" — never "If X, Y" (Conrad's rule)

### Display vs inline
- Routine calculations: inline, in the paragraph flow
- Key equalities and definitions: display with equation number
- Subspace inclusion chains: inline with ∈ and ⊆

## Part 10: Structural Imitation Checklist

- [ ] 3 numbered sections + unnumbered Introduction (matching Ou)
- [ ] No "Conclusion" section — main theorem IS the conclusion
- [ ] Setup:execution ratio ≤ 1:2 (Ou is 1:2.25, KK is 1:1.2)
- [ ] All notation defined in §1, never re-introduced
- [ ] Each Step in §3 has goal in the first sentence
- [ ] Final assembly cites all intermediate results by number
- [ ] Small-n cases dismissed in one sentence: "n≤3 are trivial; we assume n≥5"
- [ ] "Conversely" verification: constructions from §2 are cited in §3's closing
- [ ] AMS classification, keywords, author affiliation all present
- [ ] References: numeric [1],[2],... following Ou/LAA convention

## Part 12: Qualitative Comparison — Ou vs KK (2026-07-11)

Discovered during paragraph-by-paragraph Q1-Q6 analysis of both papers'
proof sections. This complements the sentence-level templates in Parts 2-4
with paragraph-level rhythm analysis. Full framework in
`references/qualitative-reading-framework.md`.

### Ou's Progressive Reduction Architecture

Six numbered Steps. Each shrinks the problem: φ → φ₁ → φ₂ → φ₃ → φ₄.
Each Step heading states the goal. Each Step outputs a named object.
Paragraphs follow a 3-phase rhythm: compute → construct → name.

Key technique: modular reasoning ("≡ (mod α₁)"). All clutter is pushed
into a designated subspace, then eliminated in the next Step.

### KK's Lemma-Chain Architecture

Lemma 6 → 7 → 8 → Proposition 10. Each lemma accumulates one layer of
constraints. No named reduction objects; the reader follows equation numbers
instead. Compression uses "whence" (skip one step) and "Multiplying X by Y
on the left/right" (always explicit direction).

### When to Use Which

- Use **Ou style** when your proof reduces unknowns through a chain of
  increasingly constrained forms (our φ → φ₀ → a_k,b_k,c_k → survivors)
- Use **KK style** when your proof accumulates constraints through
  numbered matrix-entry equations
- Our paper uses a HYBRID: Ou's case-analysis structure with KK's
  explicit bracket expansions. This is correct for Lie bracket computations
  where neither pure mod-I nor pure matrix multiplication applies.

Common errors found when writing half-derivation classification proofs,
discovered 2026-07-10 during a paper audit against the Lean formalization.

### Pitfall 1: Ψ=0 requires commuting pairs
The notation Ψ(x,y,φ) = [φx,y] + [x,φy] is the bracket-identity side of the
half-Leibniz equation.  The full equation is 2φ[x,y] = Ψ(x,y,φ).
Writing Ψ=0 is only correct when [x,y]=0 (commuting pair).  For non-commuting
pairs, the bracket-source term 2φ[x,y] must appear on the left-hand side.

**Check**: Always verify [x,y]=0 before writing Ψ(x,y,φ)=0.  If [x,y]≠0,
write the full equation: 2φ[x,y] = Ψ(x,y,φ).

**Example** (from the paper): For u>i+1, the pair (E_{i+1,u}, E_{i,i+1})
has [E_{i+1,u}, E_{i,i+1}] = -E_{i,u} ≠ 0.  Writing Ψ=0 here is wrong;
the bracket-source contributes -2X on the left, giving the correct equation
-2X = bracket-identity terms.

### Pitfall 2: Chain bridge gives zero coupling for c-channel
When applying the chain bridge to an adjacent pair (E_{k,k+1}, E_{k+1,k+2})
projected to the c-channel target (2,n), both bracket terms evaluate to
zero for interior k (2 ≤ k ≤ n-3):
  [E_{2,n}, E_{k+1,k+2}]_{(2,n)} = 0  (no index overlap for k≤n-2)
  [E_{k,k+1}, E_{2,n}]_{(2,n)} = 0    (no index overlap for k≥2)
The chain bridge provides NO coupling between c_k and c_{k+1}.
Any claim of a recurrence "2c_{k,k+2} = c_k + c_{k+1}" from the chain
bridge at this target is false.

**Correct approach**: Channel constraints come from half-Leibniz equations
with commuting pairs, not from the chain bridge.  See F1/F2 below.

### Pitfall 3: Channel restrictions have boundary exceptions
When proving [E_{k,k+1}, I] = 0 for k≥3, the argument fails at k=n-1
because the generator E_{1,n-1} of I shares the index n-1 with E_{n-1,n}:
  [E_{n-1,n}, E_{1,n-1}] = -E_{1,n} ≠ 0.
The k=n-1 case gives the Feq constraint (a_1 + c_{n-1} = 0), not c_{n-1}=0.

**Correct bounds**: c_k = 0 for 3 ≤ k ≤ n-2 (not k≥3).
a_k = 0 for 2 ≤ k ≤ n-3 (not k≤n-3).

### Pitfall 4: Strategy summaries must match proof bounds exactly
When a strategy overview says "c_k=0 for k≥3" but the actual proof says
"c_k=0 for 3≤k≤n-2", the reader computes the wrong surviving variables.
Always copy the precise interval from the proof into every summary. Never
abbreviate bounds in an overview paragraph — the overview is what readers
skim first.

### Pitfall 5: Unverified adjectives on ideal properties
Claiming "I is maximal abelian" when only "abelian" is proved (and
"maximal" is actually false) is a self-inflicted wound. Only state
properties you have verified. If a property is never used in the proof,
delete it entirely — it can only attract referee objections.

### Allowed "i.e." vs forbidden "i.e."
- `= π_{...}, i.e., 2Y = X` — **allowed**: substitutes defined symbols.
- `π = 0, i.e., the diagonal entries vanish` — **forbidden**: rephrases
  math in prose. The distinction: "i.e." that performs a symbol
  substitution is fine; "i.e." that explains the math is not.
  "whence" is always allowed (KK-style skip-one-step).

### F1/F2 Constraint Pattern (preferred over cross-source chain)
The cleanest way to constrain channel variables a_k, c_k uses two
half-Leibniz equations per adjacent source k:

  **F2** (pair with E_{12}, target (1,n)): constrains c-channel
    - For 3 ≤ k ≤ n-2: c_k = 0
    - For k = n-1: c_{n-1} + a_1 = 0 (Feq)

  **F1** (pair with E_{n-1,n}, target (1,n)): constrains a-channel
    - For 2 ≤ k ≤ n-3: a_k = 0
    - For k = 1: a_1 + c_{n-1} = 0 (Feq, same as F2 at k=n-1)

This pattern is verified by the Lean formalization and avoids the erroneous
cross-source chain bridge argument entirely.

### Technique: Project to "next column" for coefficient isolation
When isolating a coefficient π_{uv}(φ(E_{i,i+1})) via a commuting partner
E_{v,v+1}, project the half-Leibniz equation to target (u, v+1), NOT (u,v).
At (u,v+1), the first bracket [φ(E_{i,i+1}), E_{v,v+1}] contributes the
desired coefficient through [E_{uv}, E_{v,v+1}] = E_{u,v+1}.

Projecting to (u,v) gives no constraint because both brackets vanish at
that target (the first bracket has column v+1≠v; the second has row
or column mismatches).

**Used in**: Image restriction Case 1a, 1b, 1c.

### Image Restriction Case 1d: Three-equation chain bridge proof
For u < i with v = i+1, the coefficient π_{u,i+1}(φ(E_{i,i+1})) cannot be
isolated by a single commuting partner. The Lean proof uses a 3-equation
system (chain bridge + two half-Leibniz equations):

  Regime u = 1:
    (1) half-Leibniz at (i,i+1)(i+1,i+2) target (1,i+2): GOAL = 2X
    (2) half-Leibniz at (i,i+1)(i+1,n) target (1,n): GOAL = 2Y
    (3) chain bridge at (i,n) split i+2, target (1,n): 2Y = X
    Solve: GOAL = 2X = 4Y, GOAL = 2Y → 2Y=0 → GOAL=0

  Regime u ≥ 3:
    x = π_{u-1,i}(φ(E_{u-1,u})), y = π_{u,i+1}(φ(E_{i,i+1})), z = π_{u,n}(φ(E_{i,n}))
    (1) bracketIdentity at (i,i+1)(u-1,u) target (u-1,i+1): y + x = 0
    (2) bracketIdentity at (u-1,u)(i,n) target (u-1,n): x + z = 0
    (3) chain bridge at (i,n) split i+1, target (u,n): 2z = y
    Solve: y = -x, z = -x → y = z, 2y = y → y = 0
