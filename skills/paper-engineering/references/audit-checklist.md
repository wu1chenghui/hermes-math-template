# Rigorous Mathematical Audit — Four-Dimension Checklist

> Run this BEFORE submitting to a journal. Performed as AUTHOR, not referee.
> Goal: catch issues a referee would find, before they find them.

---

## Audit 1: Dependency DAG

### Check
Trace every `Proposition X.Y`, `Theorem X.Y`, `Lemma X.Y`, `Corollary X.Y`
referenced in each proof. Verify the referenced section number ≤ current section number.

### Method
Create a table:

| § | Statement | References | Max Ref § | Status |
|---|---|---|---|---|
| §2 | Lem 2.1 | (2.3), (1.1) | §2 | ✅ |
| §4 | Thm 4.2 | Prop 4.1 | §4 | ✅ |

### Red flags
- Any reference to a later section (forward reference)
- Any circular dependency (A uses B, B uses A)
- Any "hidden" dependency not listed in the theorem statement

### After fix
The dependency graph must be a pure DAG. Every edge points to an earlier section.

---

## Audit 2: Definition Consistency

### Check
List every defined term. For each, find its first definition. Check all subsequent uses.

### Method
Create a table:

| Term | First Defined | §2 | §3 | §4 | §5 | Status |
|---|---|---|---|---|---|---|
| source | §2.1 | used | used | used | — | ✅ |

### Red flags
- Same concept called by different names in different sections
  (leaf = terminal node = adjacent source = width-1 node)
- Term used before it is defined
- Term defined but never used

---

## Audit 3: Notation Consistency

### Check
Every math symbol: defined once, used consistently throughout.

### Method
Collect all symbols. Check for redefinition or overload.

| Symbol | Meaning | Defined At | Consistent? |
|---|---|---|---|
| c_I | diagonal coefficient | §4.1 | ✅ |
| w(I) | source width | §3.1 | ✅ |

### Red flags
- Symbol used before definition
- Same symbol used for different things in different sections
- Typo in subscript/superscript (e.g., c_{jj}^{jj} instead of c_{pq}^{pq})
- Notation transition undocumented (c_{ij} → c_I with no bridge statement)

---

## Audit 4: Proof Compression

### Check
Can each proof be expressed in fewer sentences without loss of rigor?

### Method
For each proof: count sentences. Ask: is every "thus/hence/clearly" fully justified
by the PREVIOUS sentence (not by general mathematical knowledge)?

### Specific checks
- "Multiplying by 2 and rearranging" → show the algebra explicitly
- "Canceling, multiplying, simplifying" → expand to step-by-step
- "By induction" → verify base case + recursive call decrease
- "Without loss of generality" → justify why the reduction is valid
- "Routine case analysis" → either show it or cite appendix/Lean

### Red flags
- Any proof longer than ~10 sentences (probably contains buried case analysis)
- Any "clearly" or "obviously" (referee magnet)
- Algebra compressed into one sentence with multiple operations
