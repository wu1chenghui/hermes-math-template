# Formalization Stability Patterns (Bridge Layer Era)

Reusable proof-engineering heuristics discovered while building a
`representation-bridge` layer (a `Submodule` of a coefficient function space,
a finite evaluation model, an injectivity spine, and a `LinearEquiv`). These
are general Lean 4 / Mathlib traps and tricks, not project-specific.

---

## 1. `subst` / `rfl`-pattern eliminates the WRONG variable (section-variable capture)

**Symptom.** After `rcases h with ⟨rfl, rfl⟩` (or `subst h`) on a hypothesis
like `v = n` whose RHS is a *section variable* `n`, later references to the
literal `n` fail with `unknown identifier 'n'`. A sibling branch with `v = n-1`
compiles fine — which makes the failure look mysterious.

**Cause.** `subst` (and the `rfl` rcases pattern) on `a = b` eliminates whichever
side is a free local variable. For `v = n` **both** `v` and `n` are variables, and
subst can eliminate `n` (substituting `n := v`), removing `n` from scope. When the
RHS is a non-variable expression (`v = n - 1`), subst can only eliminate `v`, so `n`
survives — hence only the bare-variable-RHS branches break.

**Fix (any of):**
- Rewrite the goal instead of substituting: `rcases h with ⟨hu, hv⟩` then
  `rw [hu, hv]`. `rw` rewrites occurrences without eliminating any variable, so
  `n` always survives.
- Pre-compute every result that mentions the literal `n` **before** the `rcases`,
  then the post-`rcases` branches only need `exact hwa`/`exact hwb`/… (which match
  by defeq even if a variable was renamed).
- Force the intended elimination with `subst v` (naming the variable to remove).

**Example.**
```lean
-- (u,v) ∈ {(1,n-1),(1,n),(2,n)}; want T i j u v = 0
-- BAD: branch (1,n) substs n := v, then `coeffOf_f D i j 1 n` → unknown `n`
-- GOOD: establish results first, rewrite goal, exact:
have hwa := width_stability_a D hn hI i j hi hij2 hjn
have hwb := width_b_zero      D hn hI hadjD i j hi hij2 hjn
have hwc := width_stability_c D hn hI i j hi hij2 hjn
rw [coeffOf_f D i j 1 n …] at hwb   -- literal `n` used BEFORE rcases
rcases hItgt with ⟨hu, hv⟩ | ⟨hu, hv⟩ | ⟨hu, hv⟩
· rw [hu, hv]; exact hwa
· rw [hu, hv]; exact hwb
· rw [hu, hv]; exact hwc
```

---

## 2. ℕ → `Fin` decomposition without `Nat.sub` (transport-free)

**Problem.** Turning `∀ p : ℕ, 1 ≤ p → …` into something indexed by `Fin k`
(e.g. an adjacent source `(p, p+1)` ↔ `q : Fin k` with `q.val + 1 = p`) tempts you
to write the index as `⟨p - 1, _⟩`. The `Nat.sub` then leaks into every downstream
rewrite and `omega` step, and `(p-1)+1` does not reduce to `p` definitionally.

**Trick.** Pattern-match `p` as `m + 1`, confining the subtraction to the *existence
witness* only:
```lean
intro p hp1 hpn
obtain ⟨m, rfl⟩ : ∃ m, p = m + 1 := ⟨p - 1, by omega⟩
have hm : m < k := by omega
-- goal now has clean `m + 1`, `m + 2` literals; Fin index is `⟨m, hm⟩`
-- and `(⟨m, hm⟩ : Fin k).val + 1` is defeq `m + 1`, `+ 2` defeq `m + 2`
```
The resulting plumbing is `Nat.sub`-free. (`k` here is often `n - 1`; that `n-1`
lives in the `Fin (n-1)` *type* and is fine — only avoid `Nat.sub` in the
*index arithmetic* you `rw`/`omega` over.)

**Extraction companion.** To split a tuple-valued evaluation `f = 0` into its
pointwise components, `simp only [f_def, Prod.ext_iff, funext_iff, Prod.fst_zero,
Prod.snd_zero, Pi.zero_apply] at h` turns `h : (g1,g2,g3) = 0` into
`h : (∀ q, g1 q = 0) ∧ (∀ q, g2 q = 0) ∧ (∀ q, g3 q = 0)`; then
`obtain ⟨ha, hb, hc⟩ := h` and apply at `⟨m, hm⟩`.

---

## 3. ★ Applying a STRUCTURE-API lemma to a FUNCTION-level `Submodule` element
   (ephemeral witness pattern)

This is the core meta-pattern when your `finrank` carrier is a `Submodule` of a
function space but the lemmas you need are phrased over a *structure*.

**Situation.** Carrier `kerPhi : Submodule F (ℕ→…→F)`; element `T : kerPhi` has
`(T : ℕ→…→F)` a raw coefficient table. But the verified lemmas you need
(`width_stability_*`, etc.) take `(D : HalfDerivation F n)` and conclude about
`coeffOf D …` — they are NOT stated over the raw function. You CANNOT pass `↑T`.

**Pattern.** Build a *local, ephemeral* witness:
```lean
-- half_leibniz from the membership predicate (e.g. Φ = 0 ⟹ 2·src = idn):
have hleib : ∀ …, 2 * bracketSource (T:V F) … = bracketIdentity (T:V F) … := by
  intro …; have hz := hΦ …; unfold Phi at hz; exact sub_eq_zero.mp hz
set D : HalfDerivation F n := ⟨(T : V F), hleib⟩ with hDdef
-- use D only to invoke the structure-API lemma, then bridge back:
have hw := width_stability_a D hn hI i j hi hij2 hjn   -- coeffOf D … = 0
rw [coeffOf_f D i j 1 (n-1) …] at hw                   -- D.coeff … = 0
exact hw   -- closes goal `(T:V F) … = 0` because D.coeff is DEFEQ ↑T (via `set`)
```
**Why it is sound / acyclic.** The witness is forged from data the element
*already definitionally holds* (the membership predicate), in a module that does
not import the bridge — so it is not circular. Use `set` (keeps the body defeq, so
`D.coeff` ≡ `↑T` and `exact` closes by defeq) rather than `have` (which forgets the
body). The witness must never appear in any *signature*; it is a proof artifact
only, keeping the import DAG one-directional.

**`I_filtered`-style hypotheses transfer by defeq:** `hI : I_filtered (↑T) hn`
is accepted where `I_filtered D.coeff hn` is expected, since `D.coeff ≡ ↑T`.

---

## 4. Explicit type anchoring when an implicit is uninferable

**Symptom.** `don't know how to synthesize implicit argument 'n'` on a call like
`f a`, where `f` has an implicit `{n}` (often an auto-bound section variable) that
appears only in `f`'s RESULT type, and `a`'s type does not mention `n`.

**Fix.** Anchor `n` explicitly, either by ascribing the result type or by a named
argument:
```lean
lemma evAdjFun_add (a b : V F) :
    (evAdjFun (a + b) : AdjData F n) = evAdjFun a + evAdjFun b := rfl  -- ascription pins n
-- or, at a use site:
IsValidSupp (n := n) c       -- named-arg pins n
```
Type ascription on one side of an equation forces the other side to the same type,
so a single anchor usually suffices.

---

**Meta-lesson**

When a verified toolbox is frozen over one representation (here `ℕ⁴→F`), the
*honest* finrank carrier is usually a **predicate-cut `Submodule` of that same
ambient** (selecting the real object out of an over-representation), NOT a fresh
finite-index type. Changing the ambient type either bakes a downstream theorem
into the type (circular) or severs the frozen API. Add a `valid-support`-style
*defining* condition to the carrier predicate; recover finiteness later as a
*theorem* (an injective evaluation into a finite model), not as a type refinement.

---

## 5. Prefer direct equality lemmas over existential `∃` lemmas

**Symptom.** You need a normal-form lemma: "any X satisfying constraint C can
be expressed as a linear combination of generators." The natural first impulse
is `∃ params, X = linear_combination(params)`. This creates `obtain ⟨...⟩`
witness-management noise at every use site.

**Fix.** Write the lemma as a **direct equality** where the parameters are read
from `X` itself, not existentially quantified:

```lean4
-- ❌ Existential: forces unpacking at every call site
lemma normal_form (hC : constraint X) : ∃ p q r, X = p•g1 + q•g2 + r•g3 := ...

-- ✅ Direct: parameters read from X, zero witness overhead
lemma normal_form (hC : constraint X) :
    X = (field_of_X)•g1 + (field_of_X')•g2 + (field_of_X'')•g3 := ...
```

The right-hand side computes parameters as projections from `X`'s own data
(e.g. `x.1 ⟨0, ...⟩`, `x.2.2 ⟨n-2, ...⟩`). At the call site, the lemma is a
single `rw` or `have h := normal_form hC; rw [h]` — no `obtain`, no witness
variables to thread through downstream steps.

**Why this matters for Lean.** `∃` parameters multiply the proof state: each
`obtain` adds binder + hypothesis. In a 5-step proof chain (extract coords →
normal form → construct S → evAdj equality → injective), existential unpacking
adds 5–10 extra lines of variable plumbing. The direct form eliminates all of it.

This principle applies broadly to normalization, parametrization, and
classification lemmas where the parameters are *determined* (not free).
Only use `∃` when the parameters represent genuine nondeterministic choice.
