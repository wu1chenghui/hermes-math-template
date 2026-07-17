# 英文数学论文写作风格审计 — 参考论文对比分析

> 基于 2026-07-11 对 Ou-Wang-Yao (2007)、KK (2023)、Ghimire-Huang (2016)、
> Yusupov 等 (2025) 的全文提取和量化对比。

---

## 参考论文位置

| 论文 | 本地路径 |
|------|----------|
| Ou-Wang-Yao (2007) | `/workspace/参考论文/derivations of the Lie algebras of strictly upper.pdf` |
| KK (2023) | `/workspace/参考论文/Transposed Poisson strucpper triangular matrices.pdf` |
| Ghimire-Huang (2016) | `/tmp/block-deriv.pdf` |
| Yusupov 等 (2025) | `/tmp/easychair-halfder.pdf` |

## 风格量化对照

### 连接词密度

| | Ou (6pp) | KK (~15pp) | GH (~13pp) | Yusupov (~10pp) | **我们 v3 (8pp)** |
|------|:--:|:--:|:--:|:--:|:--:|
| "we" | **44** | 10 | 50 | 22 | 5 |
| "thus" | 5 | 5 | 1 | 0 | **27** |
| "hence" | 0 | 4 | 1 | 0 | 0 |
| "obtain" | 1 | 3 | — | 1 | **6** |
| "so" | 7 | — | 11 | — | 11 |
| "then" | 19 | — | 23 | 8 | 6 |

**结论**: "thus" 密度是所有参考论文的 5 倍以上，是唯一显著偏离的指标。

### 口语化程度

| | Ou | KK | GH | 我们 |
|------|:--:|:--:|:--:|:--:|
| "get" | 7 | — | — | 0 |
| "see" | 9 | — | — | 0 |
| "check" | 4 | — | — | 0 |
| "easy to" | 3 | — | — | 0 |
| "note that" | 6 | — | 0 | 0 |
| "recall" | 2 | — | — | 0 |

**结论**: 我们的论文比所有参考论文都更正式。

### 证明结构

| 特征 | Ou | KK | GH | 我们 |
|------|:--:|:--:|:--:|:--:|
| "Step 1/2" 标签 | **有** (主证明中) | 无 | 无 | 已删除 |
| 命名分类 (A)(B)(C) | **有** | 无 | 无 | **有** (Trivial/Simple-root/Boundary) |
| 子节粒度 | 3 节，无子节 | 极少 | 0 | 4 (§1) → 已压缩 |
| 摘要风格 | 纯结果 | 纯结果 | 纯结果 | 纯结果 ✓ |

**结论**: Ou 也使用命名分类和 "Step" 标签。我们的分类命名是正常的。

### 术语使用

| 特征 | 所有参考论文 |
|------|:--:|
| "source"/"target" | **0** — 无任何论文使用 |
| "projection" | **0** — 无任何论文使用 |
| "chain bridge" 等自创技术名 | **0** — 无任何论文使用 |
| "a-channel" 等隐喻 | **0** — 无任何论文使用 |

---

## 去名方法（已验证有效）

以下自创术语已在 v3 中删除，替换方案经过 Ou/KK 风格验证：

| 自创术语 | 替换 |
|----------|------|
| "chain bridge" | `\eqref{eq:chainbridge}` |
| "width reduction" | "reduction to adjacent basis elements" / Lemma 编号 |
| "diagonal propagation" | "equality of diagonal entries" |
| "image restriction" | "restriction to the ideal I" |
| "endpoint reduction" | "constraints on endpoint coefficients" |
| "a-channel"/"c-channel" | 直接写系数分量 |
| "the Observation" (大写) | "Observe that no adjacent..." |
| "source (i,j)"/"target (u,v)" | 直接写 "(i,j)"/"(u,v)" |

## 讲稿特征消除

| 特征 | 状态 |
|------|:--:|
| Step 1/2/3/4 编号框架 | ✅ 已删 |
| 元评注 ("analyzed through the following reductions") | ✅ 已删 |
| Dynkin 对合装饰段落 | ✅ 已删 |
| 单等式独立子节 | ✅ 已合并 |
| "Parameter tally" 独立子节 | ✅ 已合并 |
| 摘要描述证明方法 | ✅ 改为纯结果 |
| Lemma 1.2 内 "Step 1/2" | ✅ 改为散文过渡 |
| 讲稿引导语 ("we now prove") | 仅剩 1 处，可接受 |

## 数学论文 vs 讲稿的本质区别

数学论文的隐含假设是**读者是同行**。所有讲稿特征都源于作者站在教师位置：
- 给技术起名 → "让读者记住"
- 编号步骤 → "让读者看到框架"
- 装饰段落 → "让读者欣赏对称美"
- 给指标起外号 → "帮读者区分角色"

论文应该让数学本身传达结构，不需要额外框架。
