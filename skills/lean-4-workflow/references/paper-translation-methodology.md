# Paper Translation Methodology (English → Chinese)

> How to translate a mathematical paper with Lean formalization from English to
> Chinese, following domestic Chinese mathematical journal conventions.
> Established during the `e/` project paper translation, 2026-07-03.

---

## Core Principle

The English paper is the **master version**. The Chinese version is a **faithful
mapping**, not a rewrite. After translation: any Chinese sentence maps uniquely
to one English sentence. This ensures future edits to the English version can be
synchronized to the Chinese version mechanically.

---

## 10 Translation Principles

### 1. Chinese version always corresponds to English version
- Do NOT adjust proof order.
- Do NOT add explanations.
- Do NOT rewrite proof logic for Chinese fluency.
- Do NOT add new mathematical exposition.

### 2. Mathematical content is immutable
- Definition, Lemma, Theorem, Proof, Remark, Corollary structures preserved.
- Formulas preserved.
- Numbering preserved.
- Paragraph positions preserved.

### 3. Unified glossary (one term, one translation)
- Establish a glossary BEFORE translating.
- Every English term maps to exactly one Chinese term.
- No synonym variation (e.g. "像限制" not sometimes "像约束" or "值域限制").

### 4. Keep English term in parentheses on first occurrence
- Example: "中心化（centered）" on first use, then "中心化" thereafter.
- Example: "链式桥接（chain bridge）" on first use, then "链桥" thereafter.

### 5. Lean names are NEVER translated
- `coeffOf_cond` stays `coeffOf_cond`.
- In text: "对应于 Lean 中的 `coeffOf_cond`".
- Never: "系数条件引理".

### 6. Section titles: translate, but keep numbering
- `5.4 Image Restriction` → `5.4 像限制`.

### 7. Formulas are NEVER modified
- `Im(T₀)⊆I` stays as-is.
- Explanation can be in Chinese, but the formula itself is immutable.

### 8. Narrative is not "polished"
- "We now prove..." → "现在证明……"
- NOT "接下来我们将详细证明……" (that's rewriting, not translating).

### 9. Per-section verification
- Translate §1 → verify against English §1 → confirm → then translate §2.
- Do NOT translate all sections before checking.

### 10. Chinese version does not participate in mathematical revisions
- English version remains the canonical paper.
- Chinese version is read-only.
- If English version changes: fix English first, confirm, then sync Chinese.

---

## Two-Phase Translation Workflow

### Phase 1: Faithful Conversion (per section)
- No modification of math, logic, formulas, labels, numbering.
- No addition or deletion of content.
- Allowed: reorder Chinese phrasing, merge English short sentences, passive → active voice.

### Phase 2: Chinese Polish (after all sections translated)
- Eliminate English-ism.
- Unify sentence patterns.
- Adjust long sentences.
- Check full-document glossary consistency.
- Do NOT polish while translating — separate the phases.

---

## Chinese Mathematical Writing Conventions

### Introduction (§1)
- Do NOT translate literally.
- "In this paper, we study..." → "本文研究……"
- "The main result is..." → "本文的主要结果如下。"
- "This paper is organized as follows." → "本文的结构如下。"

### Body text (§2–§8)
- Paragraph-to-paragraph correspondence.
- Do NOT merge paragraphs across section boundaries.
- No "本文" in body — use "定义……", "注意……", "下面证明……", "我们首先证明……".
- Style reference: 《中国科学》《数学学报》《数学年刊》.

### Key word choices (do NOT use these)
| Avoid | Use instead |
|-------|-------------|
| encode | 刻画 / 描述 |
| dominant | 起主导作用 |
| assembly | 综合前述结果 |
| invisible (to analysis) | 未能被……检测到 |
| free (parameter) | 自由参数 |
| paper proof | 本文中的证明 |
| morphism (not category theory) | 映射 |
| reduction (constraints) | 化简 |

---

## Glossary for e/ Project (key terms)

| English | 中文 |
|---------|------|
| half-derivation | 1/2-导子 |
| centered | 中心化（首次：中心化（centered）） |
| image restriction | 像限制 |
| chain bridge | 链式桥接（首次）/ 链桥（后续） |
| width reduction | 宽度归约 |
| diagonal propagation | 对角传播 |
| endpoint coupling | 端点耦合 |
| constraint reduction | 约束化简 |
| evaluation matrix | 求值矩阵 |
| evaluation map | 求值映射 |
| channel | 信道 |
| boundary cocycle | 边界 cocycle（不译） |
| nilpotent Lie algebra | 幂零李代数 |
| strictly upper triangular | 严格上三角 |
| rank-nullity | 秩–零度定理 |
| commutative diagram | 交换图 |
| adjoint representation | 伴随表示 |
| scalar equation space | 标量方程空间 |

Full glossary: `paper-cn/glossary.tex` (v3, frozen).

---

## Per-Section Checklist

After translating each section, verify:
- □ Math content matches English version
- □ LaTeX structure unchanged
- □ All labels preserved
- □ All formulas preserved
- □ All references preserved
- □ Theorem/Lemma/Equation numbering unchanged
- □ Lean names not translated
- □ Glossary terms pass check
- □ Paragraph correspondence maintained (§2–§8)

---

## Agent Forbidden Actions

Unless explicitly requested by the user:
- Do NOT modify paper structure.
- Do NOT reorder sections.
- Do NOT modify proofs.
- Do NOT modify theorems, lemmas, formulas, references, cross-references.
- Do NOT modify the Appendix.

---

## Frozen Section Discipline

Once a section translation is approved by the user, it is **frozen**.
Do not re-edit it unless a mathematical error is discovered.
Infinite polishing of Introduction is a known anti-pattern.
