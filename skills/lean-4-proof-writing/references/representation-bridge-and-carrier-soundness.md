# Representation Bridge & Carrier Soundness

Formalizing a "function-on-a-finite-basis" object (½-derivations, cocycle spaces)
as a `Submodule` of an ℕ-indexed coefficient function space `V := ℕ→…→ℕ→F`,
building a finite coordinate model `AdjData`, an evaluation `LinearMap`, proving
its injectivity, and `LinearEquiv.ofInjective`. Distilled from the
`RepresentationBridge.lean` session (2026-06-23).

## ★ Carrier soundness: the valid-support condition is MANDATORY and DEFINITIONAL

`V = ℕⁿ→F` OVER-represents the object: spurious coordinates at illegal index
tuples (source i≥j, out of [1,n], …). A kernel cut only by `I_filtered` (target
restriction) + GUARDED `Φ=0` (half-Leibniz only at VALID indices) does NOT
constrain illegal-index coordinates → the Submodule contains infinite-dimensional
junk and any evaluation into a finite model is NOT injective.

- Always have a counterexample ready when auditing such a carrier:
  `g i j u v := if (i,j,u,v) = (n+1,n+2,1,n) then 1 else 0` — illegal source,
  I-target column ⇒ `I_filtered g` ✓; `Φ g = 0` at all VALID indices ✓ (its support
  is unreachable by valid-index bracketSource/Identity); `g ≠ 0`; not valid-supported.
- The "no-junk" theorem `kerΦ ⊆ valid-supported` is **FALSE** (g). So valid-support
  CANNOT be a post-hoc lemma; it is a DEFINING conjunct:
  `IsValidSupp c := ∀ i j u v, ¬(legal-tuple) → c i j u v = 0`,
  `IsKerPhi := I_filtered ∧ IsValidSupp ∧ (Φ=0 guarded)`. This is a SUBMODULE
  (closed under +,•,0 — trivial, same shape as `I_filtered` closure), finite-dim,
  and contains the basis cocycles (indicator functions at valid points).
- Reject alternatives: subtype ambient `{f // ValidSupp f}` (double coercion, 0 math
  gain) and finite-index carrier `Fin(n-1)×…` (CIRCULAR — bakes the width-collapse
  THEOREM into the type; that type is the evaluation IMAGE/AdjData, not the kernel).
- Keep the ℕ-based ambient so the FROZEN toolbox (`Φ`, width lemmas, basis,
  `coeffOf`, `HalfDerivation`) still applies via `.val : ℕⁿ→F`.

## Implicit `n` synthesis trap (helpers over `V F = ℕⁿ→F`)

If a def/lemma RESULT type mentions section var `n` but its args don't pin it
(args are just `c : V F`), elaboration fails: "don't know how to synthesize implicit
argument n". Fix: pin via LHS type ascription `(evAdjFun (a+b) : AdjData F n) = …`
or `(n := n)` on the call `IsValidSupp (n := n) c`. Bites on EVERY such helper.

## Submodule closure boilerplate (predicate carrier over a Pi space)

Need pointwise `Φ` linearity first: `Phi_add/Phi_smul/Phi_zero` via
`unfold Phi bracketSource bracketIdentity bracketLeft bracketRight;
 simp only [Pi.add_apply / Pi.smul_apply,smul_eq_mul / Pi.zero_apply]; split_ifs <;> ring`.
mem proofs: `rintro a b ⟨hI,hV,hΦ⟩ ⟨…⟩; refine ⟨?_,?_,?_⟩` per conjunct.
- I_filtered(+): `simp only [Pi.add_apply] at hne; by_cases a i j u v = 0 …`.
- ValidSupp(+): `intro …hnv; simp only [Pi.add_apply]; rw [hVa …, hVb …, add_zero]`.
- Φ(+): `rw [Phi_add, hPa…, hPb…]; ring`.
- zero ValidSupp: `intro …; rfl` (Pi.zero defeq 0); zero I_filtered: `exact absurd rfl hne`.

## `evAdjFun_add/smul` are `rfl`; LinearMap fields via `show`

`(evAdjFun (a+b) : AdjData F n) = evAdjFun a + evAdjFun b := rfl` (Pi/Prod ops defeq).
`map_add' := by intro x y; show evAdjFun (↑x+↑y) = evAdjFun ↑x + evAdjFun ↑y;
 exact evAdjFun_add _ _` (the `show` bridges Submodule coe `↑(x+y)` defeq `↑x+↑y`).

## Injectivity skeleton (coordinate rigidity → global vanishing)

`rw [injective_iff_map_eq_zero]` (NOT `LinearMap.injective_iff_map_eq_zero` — wrong name).
`intro T hT; obtain ⟨hI,hV,hΦ⟩ := T.property; apply Subtype.ext; funext i j u v;
 show (T:V F) i j u v = 0; by_cases hvalid : 1≤i ∧ i<j ∧ j≤n ∧ 1≤u ∧ u<v ∧ v≤n`.
- invalid → `exact hV i j u v hvalid` (by_cases predicate MUST be the same conjunction
  as IsValidSupp's, so `hvalid : ¬(…)` applies directly).
- valid, `by_cases (u,v) ∈ I_target_set`: ∉ → `by_contra hne; exact hItgt (hI i j u v hne)`;
  ∈ → `by_cases j = i+1` (adjacent vs width≥2).

## Fin↔ℕ channel extraction (`hch` — the one fiddly bit)

```
intro p hp1 hpn
obtain ⟨m, rfl⟩ : ∃ m, p = m + 1 := ⟨p - 1, by omega⟩   -- avoids Nat.sub in the index plumbing
have hm : m < n - 1 := by omega
have hT' : evAdjFun (T : V F) = (0 : AdjData F n) := hT   -- defeq from FunLike coe
simp only [evAdjFun, Prod.ext_iff, funext_iff, Prod.fst_zero, Prod.snd_zero, Pi.zero_apply] at hT'
obtain ⟨ha, hb, hc⟩ := hT'                               -- three channel ∀-statements
exact ⟨ha ⟨m, hm⟩, hb ⟨m, hm⟩, hc ⟨m, hm⟩⟩               -- (m+1)+1 defeq m+2
```

## 🔴 `rcases ⟨rfl, rfl⟩` can subst-eliminate the WRONG variable

`rcases hItgt with ⟨rfl, rfl⟩` where membership decomposed to `u = 1 ∧ v = n`:
the second `rfl` substs `v = n` and may eliminate the SECTION variable `n`
(replacing n with v everywhere) instead of v. Any later LITERAL `n` then errors
"unknown identifier n". A `v = n-1` branch is SAFE (n-1 isn't a variable → v is
eliminated, n survives), so only the `v = n` branches break — easy to misdiagnose.
- Fix A (used): establish all `n`-mentioning results BEFORE the `rcases`, then per
  branch `rw [hu', hv']` on the goal (rewrite, no subst, no elimination).
- Fix B: `rcases … with ⟨hu',hv'⟩` then force `subst v` (named variable).

## Ephemeral structure-witness (apply HalfDerivation-API lemmas to a raw coeff)

Width/`coeffOf` lemmas are stated over `D : HalfDerivation`. To use them on a raw
`T.val : V F` in the kernel, build a LOCAL throwaway witness — never enters any signature:
```
have hleib : ∀ a b c d w x, (9 validity hyps) →
    2*bracketSource (T:V F) … = bracketIdentity (T:V F) … := by
  intro …; have hz := hΦ … ; unfold Phi at hz; exact sub_eq_zero.mp hz
set D : HalfDerivation F n := ⟨(T : V F), hleib⟩ with hDdef
```
`D.coeff` defeq `T.val`, so `width_stability_a D hn hI …` (hI : I_filtered (T:V F)
accepted by defeq) gives `coeffOf D … = 0`; `rw [coeffOf_f D … (by omega)×6] at hw;
exact hw` (goal `(T:V F) … = 0` closes by defeq `D.coeff = ↑T`). DAG stays acyclic.

## bridge_equiv is free + namespace opens

`LinearEquiv.ofInjective (evAdj hn) (evAdj_injective hn)` (target `kerPhi ≃ₗ
ConstraintKernel` matches since `ConstraintKernel := (evAdj).range`).
`coeffOf`/`coeffOf_f` ∈ `HalfDerivation` namespace; `width_stability_*`/`width_b_zero`
∈ `WidthFiltration` — `open HalfDerivation` + `open WidthFiltration` to use unqualified.

## Verification = triple cross-check (this user's audit standard)

`lake env lean <file>` (0 error/0 sorry) → `lake build <Module>` (.olean produced,
integrates into build graph) → `lean_verify <Namespace.decl>` axioms = {propext,
Classical.choice, Quot.sound} ONLY (no sorryAx). Do all three before claiming done;
this user distrusts self-reports. Incremental discipline: ONE lemma → compile →
verify → next; scaffold complex single lemmas with internal `sorry`, compile the
skeleton, then fill case-by-case (use MCP `lean_goal`/`lean_multi_attempt` for fiddly
leaves like the Fin extraction instead of blind compile loops).
