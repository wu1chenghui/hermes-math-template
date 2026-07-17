# Research Methodology — Chinese Math Paper Formatting

> Captured 2026-07-11 during systematic research for the `chinese-math-paper` skill.

## Recommended Research Order

When researching Chinese math journal formatting requirements:

### Step 1: Search journal submission guidelines
- Use `web_search` with patterns like `"期刊名" 投稿 格式 要求`
- Key journals: 数学学报 (Acta Math. Sinica), 数学年刊 (Chinese Ann. Math.),
  中国科学:数学 (Sci. Sin. Math.), 应用数学学报
- Extract: abstract length, keyword count, reference style, required metadata

### Step 2: Find LaTeX templates
- Search `github "期刊名" LaTeX 模板 ctex`
- Check if the journal has a custom `.cls` file (e.g., `SCIA2023cn` for 中国科学)
- Download and analyze: document class, theorem naming, frontmatter commands

### Step 3: Check the GB national standard
- GB/T 7713.2-2022 is the authoritative source for Chinese academic paper formatting
- Covers: formula punctuation (§5.7.4), font sizes (Appendix B), abstract types (§4.2.3),
  section numbering (§5.2.2), reference style (§4.3.6)
- PDF is available for download; use `pymupdf` (`pip install pymupdf`) for extraction

### Step 4: Verify punctuation conventions
- The key issue: Chinese period "。" can be confused with subscript "₀"
- Most Chinese math journals prefer English punctuation (`.`, `,`)
- Some journals (e.g., 数学年刊 征稿简则) use Chinese punctuation — always check

### Step 5: Cross-reference journal requirements
- Different journals have surprisingly different conventions
- Create a comparison table: abstract length, keywords, references, document class
- Example finding: 数学学报 requires English references in alphabetical order;
  数学年刊 uses Chinese sequential numbering

## Key Sources (verified 2026-07-11)

| Source | URL | Access |
|--------|-----|:---:|
| 数学年刊A辑 征稿简则 | camath.fudan.edu.cn | Browser (public) |
| 数学学报 投稿指南 | actamath.cjoe.ac.cn | 403 (blocked) |
| 中国科学:数学 模板 | github.com/goblinunde/LaTex-Resource | GitHub (public) |
| GB/T 7713.2-2022 PDF | aes.org.cn download | Direct download |
