# Per-File Audit Template

Execute one file at a time, in dependency order. Each file gets all three
groups. Do NOT scan a file twice — do everything in one pass.

## File Order
abstract → intro → prelime → sec2 → strategy → image-restriction →
endpoint → appendix → acks → refs → main

## A-Group: Mechanical (run before reading, via grep)
Scan ALL files at once for these patterns BEFORE starting B/C read:

| Pattern | Tool |
|---------|------|
| 14 forbidden words | `grep -niE '(Hence|Therefore|Consequently|Namely|i\.e\.|vanishes|contradiction|induction|clearly|obviously)'` |
| hedge words | `grep -niE '\b(perhaps|maybe|might|seems|appears)\b'` |
| connectors | `grep -niE '\b(Moreover|Furthermore|Also|Next|Finally|respectively|immediately)\b'` |
| preview phrases | `grep -n 'Note that\|as follows\|In other words\|We begin by'` |
| membership prose | `grep -ni 'belongs to\|lies in'` |
| undefined refs | `grep -c 'Warning: Reference' main.log` |

Output: filename → line:content for each hit.

**Re-run A-group after all fixes complete.** An earlier B-group fix may
introduce a new forbidden word.

## B-Group: Per-Sentence (read each sentence, ask)
| # | Question |
|---|----------|
| B1 | Is this sentence meta-commentary? (describing own structure/importance) |
| B2 | What is the grammatical subject? (track types: Thus/Now/Let/we/If/By/This/noun/imperative) |
| B3 | Is it redefining something already defined? |
| B4 | Is a bracket expansion claimed without showing the derivation? |
| B5 | Is every \ref / \eqref pointing to the correct label? |
| B6 | Does each Lemma/Theorem have a motivation sentence before it? |
| B7 | Does every display equation end with . or , ? |

## C-Group: Per-Paragraph (after finishing a paragraph, ask)
| # | Question |
|---|----------|
| C1 | How many distinct things does this paragraph do? If ≥3, split it. |
| C2 | Does the first sentence preview the paragraph's content? If yes — delete it. |
| C3 | **Delete the first sentence.** Does the argument survive? If yes — keep deleted. |
| C4 | Are paragraphs connected by Moreover/Furthermore/Also? If yes — delete connector. |

## Reporting
After each file, output only PROBLEMS found (not "nothing found").
After all files, compile a master table: file → issues → fixed?

## Pitfalls Caught by This Method (2026-07-11 session)

### "Summary ≠ Body" bounds mismatch
strategy.tex said "c_k=0 for k≥3" but endpoint.tex says "3≤k≤n-2".
The summary oversimplified the bounds. **Rule**: strategy overviews must
use the same index ranges as the proof they summarize.

### Factual error masked by plausibility
"I is maximal abelian" sounded plausible but was false (I+⟨E_{1,n-2}⟩ is
also abelian). **Rule**: any claim containing "maximal" or "minimal" needs
explicit verification — these are rarely true in classification papers.

### "i.e." only caught on re-scan
The initial A-group scan used overly broad regex and missed "i.e."
A more targeted follow-up found it. **Rule**: re-run A-group with a
focused scan after all B/C fixes.
