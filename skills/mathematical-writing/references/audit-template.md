# Mathematical Paper Audit Template

Developed: 2026-07-11 during HalfDer(N_n) v3 paper audit.
Six-layer framework + per-file A/B/C group template.
One pass per file — read once, check everything.

## The 6 Layers

| Layer | Name | Method | Blocking? |
|-------|------|--------|-----------|
| L0 | Mathematical correctness | Lean `lean_verify` per theorem | Yes |
| L1 | Forbidden words + negative space | Mechanical grep (14 categories) | Yes |
| L2 | Sentence structure quantification | Per-sentence tagging | No |
| L3 | Notation audit | Grep + manual | Yes |
| L4 | Structure + organization | Section count, setup:proof ratio | Yes |
| L5 | LaTeX/typesetting | tectonic 0 errors | No |

## A-Group: Mechanical Scan (before reading)

Grep for all 14 forbidden word categories:
- Hence/Therefore/Consequently/Namely/i.e./vanishes/contradiction/induction/clearly/obviously
- key/crucial/important/fundamental/surprising
- Moreover/Furthermore/Also/Next/Finally/respectively/immediately
- perhaps/maybe/might/seems/appears
- "Note that" / "as follows" / "In other words" / "We begin by" / "We are now ready"
- "belongs to" / "lies in"
- Display equation missing terminal punctuation (manual — grep unreliable)
- Undefined labels → check compilation warnings

## B-Group: Per-Sentence (while reading)

| # | Question | Action |
|---|----------|--------|
| B1 | Meta-commentary? ("key tool", "main contribution") | Flag line |
| B2 | Grammar subject type? (Thus/Now/Let/we/If/By/This) | Count for diversity |
| B3 | Redefines something already defined? | Flag line |
| B4 | Bracket/derivation claimed but not shown? | Flag line |
| B5 | Cross-reference accurate? | Verify label → content |
| B6 | Lemma preceded by motivation sentence? | Y/N |
| B7 | Display equation ends with punctuation? | Y/N |

## C-Group: Per-Paragraph (after reading)

| # | Question |
|---|----------|
| C1 | How many distinct things does this paragraph do? ≥3 → split |
| C2 | Does first sentence preview the paragraph? → delete it |
| C3 | Delete-first-sentence test: argument still holds? → delete |
| C4 | "Moreover/Furthermore/Also" between paragraphs? → delete connector |

## File Order

Read in logical-dependency order:
abstract → intro → prelime → sec2 → strategy → image-restriction → endpoint →
appendix → acks → refs → main.tex

## Post-Audit Rescan

After all fixes: re-run A-group grep (fixes may introduce new forbidden words),
then tectonic compile. 0 errors required.

## "i.e." Rule (Corrected 2026-07-11)

- `= π, i.e., 2Y = X` — symbol substitution → **allowed**
- `π = 0, i.e., the diagonal vanishes` — prose explanation → **forbidden**

## "organized as follows" Rule (Corrected 2026-07-11)

Ou-Wang-Yao (2007) DOES contain "This paper is organized as follows" in the
introduction. A short section-outline paragraph is standard and should NOT
be flagged by the negative-space checklist.

## "whence" Rule

Always allowed (KK style: skip one algebraic manipulation step).
Not on the forbidden word list despite archaic feel.
