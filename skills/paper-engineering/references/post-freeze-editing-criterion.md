# Post-Freeze Editing Criterion

> When the paper enters pre-submission polishing, the risk shifts from
> "mathematical errors" to "reader misunderstandings." But not all reader
> feedback warrants changes — most is preference, not correctness.

## The Criterion

> **Only modify the paper when two or more independent readers report
> the same misunderstanding at the same location. Otherwise, keep the
> paper frozen.**

This prevents the paper from being rewritten infinitely as each new
reviewer brings their own stylistic preferences.

## Classification Guide

### MUST FIX (★★★★★)
- Genuine exposition contradictions: §N says X, §M says ¬X
- Two+ independent readers confused by the same sentence
- Missing logical bridge: §N uses a result that isn't proved until §N+1
  without marking it as a forward reference

### SUGGESTED FIX (★★★★)
- Missing recall sentence: object defined in §2, used in §5, no reminder
- Missing parameter tally: reader has to count parameters themselves
- Vague bridge: "modulo channel-coupling terms" without explaining why
  those terms vanish

### DO NOT FIX (★★ and below)
- "Why is this intermediate object here?" — mathematical papers use
  intermediate objects (VEq, tensor products, resolutions, sheaves)
- "I would organize this differently" — T0-T3 classification vs.
  mechanism-by-mechanism: both are valid organizational choices
- "This forward reference is uncomfortable" — "proved later in §X"
  is standard mathematical writing
- "I want to know how you discovered this" — mathematical papers
  define, they don't narrate discovery
- "This section doesn't advance the proof" — Interpretation sections
  that are clearly labeled as such are normal

## Case Study: §5.5 Contradiction

Three reviewers read the paper. One flagged that §5.5 claims `a_k=0
for k≤n-2` while §5.6 treats `a_1` as non-zero (F_eq). This is a
genuine contradiction — not a preference.

Root cause: the cross-source chain paragraph treated a-channel and
c-channel symmetrically, but the a-channel's bracket terms vanish for
interior k (E_{1,n-1} only brackets with endpoint sources), so the
chain coupling doesn't propagate for a-channel. The text incorrectly
claimed it did.

Fix: separated a-channel and c-channel treatment. Added a_1 and c_{n-1}
to the surviving variables list with explicit F_eq coupling notation.

Lesson: the contradiction was real, the fix was small. The other 5
reviewer suggestions on the same round were preferences — VEq purpose,
Greek letter usage, T0-T3 organization, forward references, §6 role.
None were changed.
