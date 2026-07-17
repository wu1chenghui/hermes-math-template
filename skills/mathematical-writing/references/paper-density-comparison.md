# Paper Density Comparison — Methodology & Findings (2026-07-11)

## The Diagnostic Trap

When comparing your paper against a reference paper and finding your paper
feels "too long," do NOT reflexively blame display equations.  The real
diagnosis requires a four-step factual check first:

### Step 1: Measure Actual Font Sizes

Extract font metadata from both PDFs (pymupdf `page.get_text("dict")` →
`span["size"]`).  Reference papers in LAA (Ou 2007) use **9pt** Times.
elsarticle 3p uses **10pt** TeXGyreTermes.  Your font may be LARGER than
the reference, not smaller.

### Step 2: Count Display Equations Per Section

`grep -c '\\begin{equation}\\|\\begin{aligned}\\|^\\\\\\[' *.tex`

If a section has zero display equations (all inline), then "convert display
to inline" is not the fix — the verbosity is in the prose, not the formatting.

### Step 3: Compare Column Layout

Single-column (amsart, LAA published version) vs double-column (elsarticle 3p).
Double-column fits ~1.5× more text per page.  Same page count in double-column
means substantially more content.

### Step 4: Compare Section Lengths Directly

Count lines (or PDF text blocks) for analogous sections:
- Construction section (§2): Ou ~20 lines vs us ~85 lines
- Main proof section: Ou ~2.5 single-column pages vs us ~3 double-column pages
  (≈ 4.5 single-column pages)

## The Real Issue: Verification vs Assertion

The root cause is NOT formatting.  It's what the text says.

| | Ou (2007) | Our paper (v3) |
|---|---|---|
| Construction verification | "it is easy to check" (4 words) | 6-10 lines per ω_i, explicit bracket expansion |
| Proof density | One dense paragraph = 4 reasoning steps | Each step = its own paragraph + labelled equations |
| Reader trust | Assumes reader can verify | Spells out every bracket term, including those that vanish |

**Ou's "it is easy to check" is not laziness — it's a genre convention.**
In derivation-classification papers, constructions are stated, not verified.
The reader (and referee) knows how to check that a linear map is a derivation;
showing the check reads as padding.

Similarly, bracket expansions that produce a single non-zero term should
state the result ("the (1,n)-coefficient is -c_k"), not enumerate all three
terms including the two that vanish.

## Concrete Compression Targets (from v3 audit)

| Location | Lines | Issue | Compression Potential |
|----------|-------|-------|----------------------|
| §2 ω_i verifications | ~43 | Explicit bracket expansion for each | → "it is easy to check" (~5 lines) |
| Lemma 1.2 d₁=d₂ | 20 | Step-by-step algebra chain | → 3 lines inline |
| Lemma 3.1 u=1 | 26 | Three display equations with tags | → 6 lines inline paragraph |
| Lemma 3.2 Case 2(iv) | 32 | "First pair"/"Second pair" labels | → 6 lines inline |
| Lemma 3.3 F2 bracket expansion | 6 | Lists all 3 terms, 2 are zero | → 2 lines: only the non-zero term |

## Three-Tier Verdict System (2026-07-11)

Not all over-explanation should be cut equally. Classify each candidate:

- **Compress** (Ou-level trust): The claim is genuinely obvious to the
  intended reader. Cut to a bare statement. Example: "No E_{k,k+1} is a
  commutator in N(n,F)" — no proof needed.
- **Shorten** (keep structure, trim explanation): The verification carries
  non-trivial information (e.g., distinguishes two subcases, uses a lemma
  from elsewhere). Keep the logical skeleton but cut the bracket-expansion
  play-by-play. Example: "The second bracket vanishes: trivially if u>i,
  by Lemma 3.1 if u=i" — keeps the distinction, drops the bracket algebra.
- **Keep**: The verification IS the contribution, or uses a genuinely
  non-obvious technique. Do not compress.

This prevents over-compression — some candidates that look "too detailed"
actually carry structural information that the Ou-style one-liner would lose.

## The Display-Equation Trap (2026-07-11)

When a section feels "too detailed," the instinct is to blame display
equations. BEFORE doing that, check whether the section HAS any display
equations at all. In our v3 paper, §2 had ZERO display equations — every
equation was already inline. The verbosity was in the prose (explicit
verification), not in the formatting. Converting display→inline when
there are no displays is a category error.

## Ou vs KK: Two Reference Styles

Not all reference papers have the same density. When choosing a benchmark:

- **Ou (2007, LAA)**: Extreme compression. "It is easy to check" for
  all constructions. Zero display math in the main proof. Bracket
  membership claims stated without verification. This is the gold standard
  for compression.

- **KK (2023, arXiv)**: Shows detailed verification in technical lemmas
  (Lemma 6 has full equation-by-equation derivations with explicit
  multiplication steps). Compresses only after the pattern is established.

If your paper aligns with KK's genre (transposed Poisson / 1/2-derivations),
using KK as the density benchmark is more appropriate than Ou. Don't
over-compress to match Ou if your constructions require non-obvious
cancellation checks that Ou's don't.

## The Nesting Pattern for Compressed Algebra (2026-07-11)

When compressing step-by-step chain-bridge expansions, use nested
fractions rather than sequential equalities:

    $2π = d₁ + ½(d₂ + ½(d₃ + d₄))$

This encodes two rounds of chain bridge in one expression. The reader
just saw the same pattern in the preceding d_k = d_{k+2} proof, so
"iterating the chain bridge on the resulting pairs" tells them what
happened without showing each step.

## The Lean-Verification Safety Step (2026-07-11)

Before applying ANY compression to a paper that has a companion Lean
formalization, verify every proposed change against Lean. The process:

### Per-change verification workflow

1. **Read the Lean theorem** corresponding to the paper's lemma/proof.
   Use `lean_declaration_file` or `read_file` on the relevant .lean file.

2. **Trace the Lean proof** to confirm its structure matches the paper's
   claim. Key checks:
   - Do the indices and ranges match? (e.g., `3 ≤ p ≤ n-2` in Lean = `3 ≤ k ≤ n-2` in paper)
   - Does the Lean use the same commuting partner? (e.g., `(E_{1,u}, E_{i,i+1})`)
   - Does the Lean's conclusion match the paper's? (e.g., `coeff ... = 0`)

3. **Verify the compressed version**: the compressed prose must be logically
   equivalent to the original. If the original says "the first bracket
   contributes π_{u,v} and the second vanishes because..." and the compressed
   says "projecting gives π_{u,v}=0", confirm that the Lean proof of the
   second bracket's vanishing is straightforward (e.g., a simple `omega`
   index check) and not a multi-step sub-lemma invocation.

4. **Classify the verdict**: after Lean verification, assign each candidate
   to the three-tier system (Compress / Shorten / Keep).  Lean verification
   may reveal that a seemingly "trivial" bracket expansion actually uses
   a non-trivial sub-lemma — promoting it from Compress to Shorten.

### Example: Verifying Case 2(ii) compression

Original (9 lines): explains why the pair commutes, which bracket term
contributes, and the bracket algebra.

Compressed (3 lines): "The commuting pair projected to (1,n) gives π=0
(only the bracket-right term contributes)."

Lean check: `adjacent_below_bi` (Centering.lean line 426) confirms
`bracketIdentity coeff 1 u i (i+1) 1 n = coeff i (i+1) u n` — only the
bracket-right term survives. The bracket-left term vanishing is confirmed
by the `split_ifs` in the bracketIdentity expansion (index conditions
are excluded by `omega`).  This is genuinely straightforward — the
compressed version is correct.  Verdict: **Shorten**.

### The "Split Personality" Trap

When compressing, watch for constructions that appear similar but have
different verification structures.  In our v3 audit:

- **τ_k**: image is in ⟨E_{1,n}⟩, verification uses centrality → "right side
  involves only brackets of central element" ✓ compressible
- **ω₁**: image is E_{2,n} (NOT central), verification uses pairwise
  cancellation → cannot use the same compressed phrase as τ_k

Each construction must be verified independently against its Lean
`half_leibniz` proof, even if the prose pattern looks similar.

### Tools used

- `mcp_lean_lsp_lean_declaration_file` / `lean_file_outline` — locate theorems
- `read_file` on .lean files — trace proof structure
- `search_files` across .lean files — find specific patterns (e.g., `coeff.*i.*i+1.*u.*i+1`)
- `execute_code` with pymupdf — verify PDF output (font sizes, page count, word count)

### When Lean disagrees

If Lean verification reveals a discrepancy between the paper and the
formalization, the compression is BLOCKED. Fix the mathematical error
first, then reconsider compression.  Never compress a proof that isn't
already verified correct.

Not all verification should be cut.  Keep explicit verification when:
- The computation uses a non-obvious trick (char≠2/3 elimination)
- The verification IS the main contribution of the lemma
- The construction has genuinely subtle cancellations

But when the verification is "bracket formula → one non-zero term → done,"
trust the reader.

## Three-Round Execution Methodology (2026-07-11)

The systematic workflow for compressing a derivation-classification paper
from "textbook-like" to journal density. Each round is a full pass through
the paper, with Lean verification at every step.

### Round 1: Obvious Display → Inline Targets

Focus on proof segments where single-step algebra occupies a full display
equation. These are the "lowest hanging fruit" — no bracket-analysis
expertise needed, just reformatting.

**Pattern**: find `\[...\]` or `\begin{equation}...\end{equation}` blocks
that contain a single equals sign with no multi-line structure. Convert
to inline `$...$` and merge surrounding prose into one paragraph.

**Real example — Lemma 1.2 d₁=d₂ (22 lines → 6)**:
```
Before:
Splitting at 2:
\[ 2π_{1,5}=d₁+π_{2,5}=d₁+½(d₂+π_{3,5})=d₁+½d₂+¼(d₃+d₄). \]
Since d₃=d₁ and d₄=d₂ as shown above,
\[ 2π_{1,5}=d₁+½d₂+¼(d₁+d₂)=5d₁/4+3d₂/4. \]
Splitting at 4:
\[ 2π_{1,5}=π_{1,4}+d₄=½(d₁+π_{2,4})+d₂=½d₁+¼(d₂+d₃)+d₂=3d₁/4+5d₂/4. \]
Equating the two expressions gives d₁=d₂.

After:
Splitting at 2 and iterating the chain bridge on the resulting pairs gives
$2π_{1,5}=d₁+½(d₂+½(d₃+d₄))=5d₁/4+3d₂/4$ (using d₃=d₁, d₄=d₂).
Splitting at 4 gives $2π_{1,5}=½(d₁+½(d₂+d₃))+d₄=3d₁/4+5d₂/4$.
Equating yields d₁=d₂.
```

**Real example — Lemma 3.1 u=1 (27 lines → 10)**:
Three tagged equations (1)(2)(3), each in its own display block.
Compressed to one inline paragraph with semicolon-separated equations.

**Real example — Lemma 3.2 Case 2(iv) (32 lines → 9)**:
"First pair" / "Second pair" labels with two display equations (5.1)(5.2).
Compressed to one paragraph with two semicolon-separated clauses.

**Result**: 741 → 691 lines, PDF 75.20 → 73.68 KB. Verification: Lean
`coeffOf_cond` formula matches the chain bridge identity used throughout.

### Round 2: Prose-Verification Targets

Focus on prose that explains rather than states — "why this commutes",
"which bracket term contributes", step-by-step bracket expansions.

**Patterns to target**:
- "commutes because the index sets {a,b} and {c,d} are disjoint" → "commutes"
- "the first bracket contributes X (from E_{a,b} paired with E_{c,d}); the
  second bracket contributes nothing (every term has row i or column i+1≠v)" →
  "projecting gives π=0"
- Display equations listing 3 bracket terms, 2 of which vanish → one-line
  description of the non-zero term only

**Real example — §2 constructions (77 lines → 34 lines, −43)**:
Compressed 9 locations across two files. Key techniques:
- "not a commutator" justification → bare statement (no proof)
- τ_k verification → "right side involves only brackets of central element"
- ω₁/ω₅ verification → "the only non-trivial check is..."
- ω₄ Dynkin motivation → deleted (construction origin is irrelevant)
- Case 1/2 subcases → compressed bracket analysis to Ou-level one-liners

**Real example — F2 bracket expansion (7 lines → 3)**:
```
Before: display equation listing [a_kE_{1,n-1},E_{12}]+[b_kE_{1,n},E_{12}]+[c_kE_{2,n},E_{12}],
then "Since [E_{1,n-1},E_{12}]=[E_{1,n},E_{12}]=0 and [E_{2,n},E_{12}]=-E_{1,n}..."

After: The bracket-left term contributes -c_k at (1,n)
(only the c_kE_{2,n} component interacts non-trivially with E_{12}).
```

**Lean verification for this round**: Each change verified against specific
Lean theorems: `bracket_1n_central` (E_{1,n} centrality), `coeff_A` through
`coeff_F` (basis element definitions), `adjacent_below_bi` (Case 2(ii)),
`F2_coeff_relation` (endpoint constraints). All 9 changes passed.

**Result**: 691 → 653 lines, PDF 73.68 → 70.88 KB. Page count: 6 → 5.

### Round 3: Polish and Typo Fixes

Final pass catches (a) redundant restatements of lemma content at proof
start, (b) verbose wrap-up paragraphs, (c) genuine typos invisible to
normal reading.

**Patterns to target**:
- "By (8), write [φ₀(...)=...] for k=1,...,n-1. We determine which a_k,b_k,c_k
  are non-zero by (1) applied to two commuting pairs." → DELETE (lemma
  already stated this)
- "By Lemma 1.1, the conclusions of Case 1 extend from adjacent pairs to
  all (p,q): for every (p,q) with 1≤p<q≤n and every (u,v) with v<n,
  (u,v)∉I, we have π_{uv}(φ(E_{pq}))=0." → "By Lemma 1.1, Case 1 extends
  to all (p,q)."
- "we call such maps \emph{trivial} 1/2-derivations" → "such maps are
  called \emph{trivial}"

**Real typos found in this round**:
1. "Fix the adjacent **the** pair (i,i+1)." → "Fix an adjacent pair (i,i+1)."
2. "gives **the the** a_k-coefficient analogue" → "gives the a_k-coefficient analogue"
3. "apply (5)" \[blank line\] "Split at k+1:" — missing object → "apply (5) to (k,k+3). Splitting at k+1 gives"

**Result**: 653 → 635 lines, PDF 70.88 → 69.73 KB. Still 5 pages.

### Round 4: Micro-Polish — Redundancy Cleanup (2026-07-11)

Final pass targets the smallest remaining redundancies — unused adjectives,
restated conclusions, verbose bracket-right explanations that mirror an
already-established pattern.

**Patterns to target in this round**:
- Unused properties: \"This ideal is abelian and satisfies\" → \"This ideal
  satisfies\" (\"abelian\" is never used in any proof; Lean's IdealI.lean only
  proves bracket closure, not internal commutativity)
- Restated parity conclusions: \"Thus d₁=d₃=d₅=⋯ and d₂=d₄=d₆=⋯\" → DELETE
  (this is an immediate consequence of d_k=d_{k+2}, already proven two
  lines earlier)
- Unspecific gap-fill: \"apply (5)\" \\[blank line\\] \"Split at k+1:\" →
  \"apply (5) to (k,k+3). Splitting at k+1 gives\" (missing grammatical object)
- Verbose construction verification: ω₂'s 6-line verification compressed
  to 3 lines via \"The only non-trivial check is [E₁₂,E₂₃]=E₁₃:\n
  2ω₂(E₁₃)=[E₁₂,ω₂(E₂₃)]=E₁ₙ. Commuting pairs contribute zero.\"
- \"Since X, so Y\" → \"Since X, Y\" (grammar fix)
- \"independence is determined by\" → \"it suffices to know\" (wrong word:
  \"independence\" suggests linear independence, but the sentence is about
  unique determination of coefficients)
- Bracket-right explanations for F2 and F1: the same pattern appears twice
  — \"generators of I have no index overlap with E_{k,k+1}\" — compressed to
  \"The bracket-right term vanishes: φ₀(E₁₂)∈I by Lemma 3.1, and for k≤n-2
  its generators are disjoint from {k,k+1}.\" The F1 version mirrors this.

**Real typos found**: none new. The two typos from R3 were already fixed.\n\n**Result**: 635 → 623 lines, PDF 69.73 → 68.83 KB. Still 5 pages.\n\n### Cumulative Results\n\n| Round | Lines | PDF Size | Pages | Focus |\n|-------|-------|----------|-------|-------|\n| Start | 741 | 75.20 KB | 6 | — |\n| R1 | 691 | 73.68 KB | 6 | Display→inline |\n| R2 | 653 | 70.88 KB | 5 | Prose compression |\n| R3 | 635 | 69.73 KB | 5 | Polish + typos |\n| R4 | 623 | 68.83 KB | 5 | Micro-polish |\n\nTotal: −118 lines (−16%), −6.37 KB (−8.5%), −1 page. Zero mathematical\ncontent changed. All changes verified against Lean formalization with 0\nerrors across 4 independent rounds.

## Word-Count Benchmarking (2026-07-11)

Use pymupdf to extract text and count words per page for comparison:

```
import fitz
doc = fitz.open("paper.pdf")
for i in range(doc.page_count):
    words = len(doc[i].get_text().split())
```

| Paper | Pages | Words | Format |
|-------|-------|-------|--------|
| Ou (2007) | 6 | 2,881 | single-col, 9pt Times |
| Our paper (final) | 5 | ~2,649 | double-col, 10pt TeXGyreTermes |

Double-column at 10pt fits ~1.5× more text per page than single-column at 9pt.
Our 5 double-column pages ≈ 7.5 single-column pages at Ou's density. This
means we have MORE content than Ou — any further compression beyond 5 pages
in double-column would require cutting mathematical content, not just prose.

## Typo Detection During Compression (2026-07-11)

Systematic compression rounds naturally expose typos that survived normal
reading. When reading each line carefully to assess compressibility:

1. **Adjacent word repetition**: "the adjacent the pair", "gives the the" —
   these hide at line breaks in the LaTeX source and are invisible in the
   compiled PDF (the line break disappears).

2. **Missing grammatical objects**: "apply (5)" followed by a blank line
   then "Split at k+1:" — the verb has no direct object. In the PDF, the
   blank line looks like a paragraph break, hiding the grammatical gap.

3. **Redundant restatements**: when a proof's first paragraph restates the
   lemma statement verbatim (with display math), the compression review
   catches it because it's an obvious candidate for deletion. These
   restatements are otherwise invisible — they look like "setting up the
   proof" rather than "unnecessary repetition."

**Methodology**: After every compression round, do one pass specifically
looking for: doubled words across line breaks, sentences with missing
grammatical objects, and paragraph openings that restate the lemma.
