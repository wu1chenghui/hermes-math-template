# Translation Workflow for Chinese Mathematical Papers

## Overall Process

```
glossary (freeze first)
    ↓
Translate §1 → Review → Freeze §1
    ↓
Translate §2 → Review → Freeze §2
    ↓
...
    ↓
Translate §8
    ↓
Phase 2: Unified Chinese polish
    ↓
Final PDF
```

## Phase 1: Faithful Conversion (per section)

### Must preserve
- Mathematics, proof logic, formulas
- Labels, theorem/lemma/equation numbers
- All LaTeX structure
- Cross-references

### Must NOT do
- Add/delete any explanation
- Modify proof structure
- Translate Lean theorem names
- Change equation numbers

### Allowed
- Adjust Chinese word order for natural flow
- Merge short English sentences
- Change passive to active voice

### After each section
Output this checklist:
```
✓ Math consistent with English
✓ LaTeX structure unchanged
✓ All labels preserved
✓ All formulas preserved
✓ All references preserved
✓ Glossary check passed
✓ Paragraph-by-paragraph correspondence (§§2-8)
```

## Phase 2: Unified Polish (whole document)

After ALL sections translated:
- Remove English flavor
- Unify sentence patterns and connectors
- Fix overfull hboxes
- Check terminology consistency globally

## Agent Forbidden Actions

Unless explicitly requested by user:
- Do NOT modify paper structure
- Do NOT reorder sections
- Do NOT modify proofs
- Do NOT modify theorems/lemmas
- Do NOT modify formulas or references
- Do NOT modify appendix

## Style

Aim for the style of 中国科学/数学学报/数学年刊, not general technical
translation. Introduction should use Chinese academic conventions:
"In this paper" → "本文", not literal translation.
