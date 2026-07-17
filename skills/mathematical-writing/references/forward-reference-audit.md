# Forward-Reference Audit

## Purpose

Catch symbols that are used before they are defined in the paper's reading order.
This is distinct from structural or completeness audits — it checks that every
notation the reader encounters has been introduced earlier in the text.

## When to run

Last audit, once all sections are in final order and no more restructuring is planned.
Section reordering can invalidate previous trace results.

## Methodology

### Step 1: Map the reading order

List every input file in the order it appears in the main document:

```
abstract → intro → §1 → §2 → §3 (strategy → image-restriction → endpoint) → references
```

### Step 2: Trace key symbols

For each symbol, record the FIRST occurrence in reading order and whether
it's a definition or a usage. A "definition" is: an explicit equation, a
named environment (lemma, proposition), or a clear declarative sentence
("Let X be...", "Denote by X the...", "Set X = ...").

### Step 3: Check forward references

If a symbol's first occurrence is a USAGE, check whether the definition
appears EARLIER. If the definition appears LATER, that's a forward reference.

## Common forward-reference patterns

### `\DeclareMathOperator` commands used before defined in prose

Example: `\id` declared in preamble but first explained in §2. The preamble
makes the command available to LaTeX but the READER doesn't know what `\id`
means until §2 says `\id(E_{ij}) = E_{ij}`.

**Fix**: Add a one-sentence definition at the first point of use. In §1's
centered decomposition subsection: "Let \id denote the identity map."
Then §2 can reference this as already-established notation.

### Notation used in abstract but defined in body

The abstract may use notation like `⟨E_{1,n-1},E_{1,n},E_{2,n}⟩` or `Δ₀`.
This is acceptable IF the abstract itself contains a self-contained
mini-definition (e.g., "to the ideal I = ⟨...⟩"). The abstract is a summary
and readers expect notation to be re-defined in the body. But a clean abstract
should minimize reliance on body-defined notation.

### Temporary symbols used only in one proof

Symbols like `d_k`, `X`, `Y`, `a`, `b`, `g` that exist only within a single
proof block don't need forward-reference checks — they're defined inline.

## Example audit (from our paper, 2026-07-11)

| Symbol | First use (file:line) | First definition (file:line) | Status |
|--------|----------------------|------------------------------|--------|
| N(n,F) | intro:3 | intro:3 | ✓ |
| Δ(N(n,F)) | intro:10 | intro:10 | ✓ |
| E_{ij} | prelime:7 | prelime:7 | ✓ |
| π_{uv} | prelime:13 | prelime:13 | ✓ |
| I | prelime:18 | prelime:18 | ✓ |
| chain bridge | prelime:27 | prelime:27 | ✓ |
| **\id** | **prelime:129** | **sec2:16** | **✗ FORWARD REF** |
| Δ₀ | prelime:125 | prelime:125 | ✓ |
| τ_k | sec2:23 | sec2:23 | ✓ |
| ω_i | sec2:38 | sec2:38 | ✓ |
| a_k,b_k,c_k | strategy:20 | strategy:20 | ✓ |
| Ψ | image-restriction:206 | image-restriction:209 | ✓ |

Only one forward reference found: `\id` used in Lemma 1.4 before its §2 definition.

## Also check

- Dead declarations: `\DeclareMathOperator` commands in preamble that are never
  used in the body (e.g., `\ad` in ours — declared but unused).
- Notation that relies on implicit understanding: `\im` for image, `⟨·⟩` for
  linear span. These are universally understood and don't need explicit
  definitions, but verify they're used consistently.
