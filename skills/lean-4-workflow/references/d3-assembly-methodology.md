# D3 Assembly Methodology — 先装配骨架，后填缺口

**Session**: 2026-06-28
**Trigger**: User redirected from "钻最后一个 v=n sorrie" to "先把 D3 整体装配写出来"

## Principle

当项目的"机制发现阶段"结束（local SCC inventory 完成、occurrence audit 穷尽、generator audit 通过），不再继续对单个 stubborn sorrie 做深度分析。而是：

> **把完整证明骨架先写出来，编译通过（含 sorries），然后分析骨架暴露了什么。**

## Why

1. 孤立地研究一个 sorrie 时，你不知道它在完整证明中是否真的需要。可能骨架装配后发现可以用更弱的接口。
2. 骨架装配会**暴露不可见的结构性缺口**（如 IsValidSupp），这些缺口在逐个攻 sorrie 时永远不会被发现。
3. 装配后可能出现三种情况：
   - **A**（最好）：最终 obligation 比预期弱，不需要最强的 lemma
   - **B**（预期）：确实需要完整 lemma，但骨架让 obligation 的形状清晰了
   - **C**（重构）：接口设计需要调整（如 kerΦ 的定义），改变后整个循环消失

## How

1. **写完整的 `centered_part_mem_kerPhi` skeleton**
   - 写成三个子条件：I_filtered、IsValidSupp、Phi=0
   - 每个子条件分别写成独立 lemma（带 sorries），而不是全塞在一个定理里
   - 标注每个 gap 的分析和可能解路径

2. **写 `halfDer_decomposition` skeleton**
   - 引用已完成的 `decompose`、`id_not_kerPhi`、`id_kerPhi_disjoint`
   - 引用 `centered_part_mem_kerPhi`（sorried）
   - 即使填 `trivial` placeholder 也要写出完整的 statement shape

3. **写 D4 `finrank_halfDer` skeleton**
   - 标注装配计划（7步：coeff injectivity → image decomposition → Spanning finrank → direct sum → result）
   - 即使全 placeholder 也要暴露接口需求

4. **编译验证**（`lake build` 必须通过）
   - 确保骨架不引入新编译错误
   - 列出所有 sorries 并分类（live vs orphaned）

## Example: this session

装配后发现的新缺口：**GAP-A (IsValidSupp)**。之前的分析从未讨论过 `IsValidSupp`，因为所有关注点都在 `I_filtered` 和 `Phi=0` 上。但骨架显示 `kerΦ` 要求 `IsValidSupp`，而 `HalfDerivation.coeff` 的类型 `ℕ⁴→F` 不强制非法下标处为 0。

## Pitfall: HalfDerivation equality

`HalfDerivation` 是一个 `structure`，有 `coeff`（ℕ⁴→F）和 `half_leibniz`（Prop proof）两个字段。**没有 `@[ext]` lemma**。证明两个 HalfDerivation 相等时：

```lean
-- 有用 helper lemma（需在 Centering.lean 或其他下游文件中定义）：
private lemma ext_coeff {A B : HalfDerivation F n} (h : A.coeff = B.coeff) : A = B := by
  rcases A with ⟨cA, hlA⟩
  rcases B with ⟨cB, hlB⟩
  have h' : cA = cB := by simpa using h
  cases h'
  rfl
```

- `rcases` 解构后 `h : HalfDerivation.mk cA hlA |>.coeff = ...` 归约为 `cA = cB`
- `cases h'` 替换 `cA` 为 `cB`（`rw` 会因为 dependent type 报 "motive is not type correct"）
- 最终 `rfl` 闭合同一结构（`mk cB hlA = mk cB hlB` 由 proof irrelevance 自动处理）

## Pitfall: HalfDerivation 缺 bundled typeclass

`HalfDerivation` 有独立的 `Add`、`Zero`、`Neg`、`Sub`、`SMul` 实例，但**没有** `AddSemigroup`、`AddMonoid`、`AddGroup`、`AddCommGroup` 等 bundled typeclass。后果：

- `simp` 无法使用 `add_zero`、`zero_add`、`add_comm`、`add_assoc` 等（这些 lemma 要求对应的 typeclass）
- `abel`、`ring` 在 HalfDerivation 级别完全不可用
- 简单代数恒等式（如 `D = s·Id + (D - s·Id)`）需要退到 coeff 级别证明，再用 `ext_coeff` 回升

**修复方案**: 在 `HalfDerivation.lean` 中添加 bundled typeclass 实例（如果允许改动 Core），或永远在 coeff/V F 级别做代数运算。
