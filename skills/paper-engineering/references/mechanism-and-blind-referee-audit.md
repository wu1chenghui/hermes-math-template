# Mechanism Audit + Blind Referee Audit

> Two final-stage audits for the paper-writing pipeline. Run AFTER all
> other audits (Architecture, Narrative, Dependency, Provenance) and
> AFTER the Proof Architecture document is frozen.

---

## Mechanism Audit

### When to run

After the Proof Architecture is frozen and sections have been rewritten
to follow it. This audit checks whether the rewritten sections actually
respect the architecture's boundary rules.

### The core question

> Does each subsection do EXACTLY one thing, and nothing from a
> neighboring section's responsibility?

### Method

For each subsection in the mechanism-heavy section (typically §5):

1. Identify its SINGLE responsibility from the Proof Architecture.
2. Read every sentence. Flag anything that:
   - Uses terminology defined in a LATER subsection (forward reference).
   - Starts counting parameters (that's §7's job).
   - Names basis vectors (that's §7's job).
   - States or implies the final dimension (that's §7's job).
   - Repeats historical narrative from §1.
   - Discusses the proof structure itself (that's §6's job).

### Common violations

| Violation | Example | Fix |
|-----------|---------|-----|
| Forward reference | §5.3 uses "a-channel" before §5.4 defines it | Replace with neutral language ("component proportional to E_{1,n}") |
| Parameter counting | §5.3 ends with "isolates s as the only diagonal parameter" | Delete; done by §7 |
| Basis naming | §5.5 says "c_2 and a_{n-2} from boundary cocycles C and D" | Remove basis names; just list the non-zero variables |
| Final dimension | §5.6 says "yields the full parameter count of n+5" | Replace with "completes the characterization" |
| Historical narrative | §5.6 says "absence from dim=n analysis was the root cause" | Belongs in §1; delete from §5 |

### Writing Discipline (from frozen Architecture)

```
§5 — CONSTRAINT REDUCTION
  Only proves reduction mechanisms.
  Never counts dimensions (n+5).
  Never names basis vectors (B, C, D, E, F).
  Never states the main theorem.
  Uses a/b/c channel notation only after §5.4 defines it.
  §5.6 ends with: characterization complete.

§6 — INTERPRETATION
  Only interprets the constraint system.
  No proof obligation.
  No new mechanisms.
  Key verbs: admits, interprets, summarizes (NOT proves, verifies, establishes).

§7 — ASSEMBLY
  Only assembles results from §3 and §5.
  No new mechanism.
  No new lemmas.
  Proof fits on one page.
```

### When the audit passes

Every subsection does only its assigned layer. §5 proves all reduction
mechanisms but never counts. §6 only interprets. §7 only assembles. No
section leaks into another's territory.

---

## Blind Referee Audit

### When to run

LAST. After the Mechanism Audit passes. This is the final pre-submission
readability check.

### The core question

> Can a reader who has NEVER seen the Phase documents, Lean code, or
> project history read §1–§7 and understand the proof?

### Method: 7-Round Structure

#### Round 1: Introduction Test

Read §1 only. Can you answer:
- What is the problem?
- Why is it important?
- What is the main result?
- Why should I read the rest?

#### Round 2: Object Test

Read §2. For every symbol/object introduced:
- Is it uniquely defined?
- Would a Lie algebra researcher understand it without external references?
- Is there any moment of "what IS this symbol?"

#### Round 3: Motivation Test

For every section and subsection:
- Does the first paragraph answer "why should I read this?"
- Not "what is this?" but "why is this next?"

#### Round 4: Proof Test

For every theorem/lemma:
- Is it clear why THIS theorem follows the previous one?
- Is the logical chain unbroken?

#### Round 5: Dependency Test

During reading:
- Do you ever need to flip back more than 2 pages to recall a definition?
- If yes: the definition needs a recall sentence at point of use.

#### Round 6: Story Test

After reading the full paper:
- Is there exactly ONE narrative thread?
  (HalfDer → Constraint → Reduction → Count)
- Or are there parallel/competing storylines?

#### Round 7: Referee Interruptions

Read at natural speed. Record every place you stop for >10 seconds.
Classify each interruption:
- **Math gap**: genuinely missing step
- **Confusion**: unclear notation or logic
- **Positioning**: section seems out of place or its purpose is unclear
- **Density**: too many new ideas at once

Fix interruptions in priority order: Math gap > Confusion > Positioning > Density.

### §6 Positioning (common pitfall)

§6 (evaluation matrix / interpretation) is the most common source of
reader confusion. Key rules for §6:

- NEVER say "does not depend on" — it makes the reader wonder why the
  section exists.
- NEVER say "provides an independent verification" — if §7 already
  proves the theorem, this creates a false expectation of a second proof.
- USE: "admits a compact linear-algebraic interpretation", "summarizes
  the interaction", "provides a global picture".
- Verbs for §6: admits, interprets, summarizes, clarifies.
- Verbs NOT for §6: proves, verifies, establishes, determines.

### When the audit passes

The reader can read §1–§7 in one sitting without:
- Flipping back more than 2 pages.
- Stopping for >10 seconds (except normal reflection).
- Asking "why is this section here?"
- Confusing which narrative thread they're on.
