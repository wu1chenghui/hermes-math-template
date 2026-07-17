# Pandoc LaTeX → Word Conversion (Ctex Math Papers)

## Workflow

1. **Flatten `\input` files.** Pandoc does not resolve `\input`. Concatenate all
   sub-files into a single `.tex` file in document order.

2. **Strip ctex-specific commands.** Pandoc skips `\songti`, `\heiti`, `\kaishu`,
   `\zihao`, `\fontfamily`, `\selectfont`, `\raggedright`, `\pagestyle`,
   `\ctexset`, `\vspace`, `\quad`. Remove them before conversion.

3. **Minimal preamble.** Use only packages pandoc understands:
   ```latex
   \documentclass{ctexart}
   \usepackage{amsmath,amssymb,amsthm}
   \newtheorem{theorem}{定理}[section]
   ...
   \DeclareMathOperator{\ad}{ad}
   ...
   ```

4. **Define stub environments** for custom environments (cnabstract, cnkeywords)
   so pandoc doesn't error:
   ```latex
   \newenvironment{cnabstract}{}{}
   \newenvironment{cnkeywords}{}{}
   ```

5. **Replace `\cite` with placeholder** if you don't need real bibliography:
   `\cite{key}` → `[key]`

6. **Convert:**
   ```bash
   pandoc paper-flat.tex -o paper.docx --from=latex --to=docx
   ```

## What survives

- **Math: YES** — stored as OMML (Office Math Markup Language) in the docx XML.
  Python's `docx.Document.text` shows empty spaces for math, but Word renders
  it correctly. Count with: `doc.element.findall('.//m:oMath', ns)`.

- **Theorem environments: partially** — headings like "定理 1." survive but
  section-qualified numbering may be lost.

- **Abstract/keywords: partially** — text survives but `【摘要】` labels and
  font formatting are stripped by pandoc.

## What is lost

- All ctex font/size commands
- `\cite` → bibliography links
- Section-qualified theorem numbering (becomes "定理 1." not "定理 1.1")
- Proof environment uses English "Proof." and "◻"
- All font formatting (must be reapplied manually in Word)

## Common issues

- **"Permission denied" on output file** — the file may be open in Word.
  Use `-o paper-v2.docx` with a different name.

- **Math appears as empty spaces in python-docx** — this is expected.
  Math is in OMML XML, not plain text. Use Word to verify.

- **`\frac` in section titles** — Pandoc may produce garbled output.
  Keep section titles simple.
