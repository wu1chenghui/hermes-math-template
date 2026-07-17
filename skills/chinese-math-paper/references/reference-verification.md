# Reference Verification Checklist

> Added 2026-07-12 after discovering that Kaygorodov-Khrypchenko (2023)
> had been published in Portugaliae Mathematica (2024) while our bib
> still cited the arXiv preprint.

## When to verify

Before finalizing a paper for submission, verify EVERY cited reference
against authoritative sources. Do not trust the bibliography as-is.

## Verification workflow

### 1. arXiv preprints → check for published version

For any reference listed as `arXiv:XXXX.XXXXX`:

1. Navigate to `https://arxiv.org/abs/XXXX.XXXXX`
2. Look for **"Related DOI:"** field on the page
3. If present, the paper has been published — follow the DOI link
4. Extract the published journal, volume, year, and pages
5. Example: arXiv:2305.00727 → Related DOI: 10.4171/PM/2120 → Port. Math. 81 (2024), no. 1/2, 135–149

### 2. Journal articles → verify page numbers and issue

For journal references, verify via web search or DOI link:

- **Page numbers**: Cross-check against at least one independent source (Google Scholar, Semantic Scholar, journal website)
- **Issue number**: Some journals (especially Russian ones like Sibirsk. Mat. Zh.) require issue numbers
- **Volume**: Confirm against journal's official listing
- **Year**: Verify against publication date

### 3. Author initials

- Confirm that abbreviated author names (S. Ou, V. T. Filippov, etc.) match the paper's actual author list
- Check for missing co-authors

### 4. Title

- Verify the title matches exactly (check for "the" vs no "the", "certain" vs no "certain", etc.)

## Common pitfalls

| Pitfall | Example | Fix |
|---------|---------|-----|
| Citing arXiv when published | `arXiv:2305.00727v1 (2023)` | Check Related DOI → `Port. Math. 81 (2024)` |
| Missing issue number | `Sibirsk. Mat. Zh. 39 (1998)` | Add `no.~6` |
| Wrong page range | Various | Cross-check with journal website |
| Uncited orphan bibitem | `\bibitem{lean}` never cited | Delete or add \cite |
