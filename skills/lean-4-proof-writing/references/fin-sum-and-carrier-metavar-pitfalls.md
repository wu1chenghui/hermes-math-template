# Fin/Finset-sum & carrier-metavar tactic pitfalls

Concrete Lean 4 (mathlib v4.31.0-rc1) tactic gotchas that each cost an iteration.
All are non-environment-dependent (pure tactic/elaboration facts).

## 1. Dirac-delta finite sum over `Fin`

Goal shape (after simplifying a single-generator coordinate sum):
```
⊢ c p = ∑ x, if ↑p = ↑x then c x else 0        -- p x : Fin k, ↑ = Fin.val
```
Close with:
```lean
simp only [Fin.val_inj, Finset.sum_ite_eq, Finset.mem_univ, if_true]
```
`Fin.val_inj` rewrites the condition `↑p = ↑x` to `p = x` (Fin.val is injective),
then `Finset.sum_ite_eq` evaluates the delta.

Do NOT use `Finset.sum_ite_eq'` here — wrong condition orientation; the rewrite never
fires and the diagnostic reports every simp arg as "unused" while the goal survives
(a silent "unsolved goals" pinned to the enclosing `:= by`). `simp [Finset.sum_ite_eq']`
is a classic false fix for this goal.

## 2. Metavar from a NON-DEPENDENT carrier type (the real bug behind a failing `omega`)

If a function's result type does not mention the size parameter, applying it leaves
that parameter as a metavariable, and a downstream `Fin` index proof fails:
```lean
-- V F := ℕ → ℕ → ℕ → ℕ → F   (does NOT mention n)
set a0 := (evAdjFun (T : V F)).1 ⟨0, by omega⟩    -- ⟨0,_⟩ : Fin (?n - 1)
-- omega error: "could not prove the goal; counterexample n ≥ 4"  ← ?n is a metavar
```
The bug is the metavar, NOT omega. FIX: pin the size with a type ascription on the
non-dependent application:
```lean
set a0 := (evAdjFun (T : V F) : AdjData F n).1 ⟨0, by omega⟩   -- Fin (n-1), omega OK
```
Tell: `omega` "counterexample" mentions only the size var with no contradicting
hypothesis, and the error column sits on a `Fin` literal's proof obligation.

## 3. `set x := e with h` is DEFEQ-transparent (not opaque)

`set` introduces `x` as a let-binding; `x` is **definitionally equal** to `e`, so
`show`, `change`, `rfl`, and `exact` see through it (and `h : x = e` is available for
`rw` when you need a syntactic fold). Do not assume `set` blocks defeq — if a proof
that "should" be `rfl` fails after `set`, the obstruction is almost always a metavar
(see §2) or a Nat-subtraction non-defeq (see §5), not `set` opacity.

## 4. `show` that changes the goal ⟹ linter wants `change`

`show <G>` is for re-stating the CURRENT goal verbatim. If you use it to convert the
goal to a defeq-but-different form, `linter.style.show` fires:
"this tactic invocation changed the goal — use `change` instead". Use `change <G>`
for any defeq goal-conversion; reserve `show` for restating.

## 5. Nat subtraction is not definitionally `±1`

`(n-2)+1` is NOT defeq to `n-1` (truncated Nat subtraction). `rfl`/`congr` fail on
`f ((n-2)+1) = f (n-1)`. Bridge with omega-rewrites:
```lean
change f ((n-2)+1) ((n-2)+2) = f (n-1) n
rw [show (n-2)+1 = n-1 from by omega, show (n-2)+2 = n from by omega]
```

## 6. Sandbox-verify whole proofs BEFORE landing (workflow)

For any non-trivial new lemma, run the COMPLETE proof in `lean_run_code`
(self-contained, `import` the project module) or test individual tactics with
`lean_multi_attempt` — both are side-effect-free. Only `patch` it into the file once
`lean_run_code` returns `success: true`. This session landed five lemmas with zero
post-write rework by always sandbox-verifying first; `lean_multi_attempt` is ideal
for choosing among candidate closers without editing the file.
