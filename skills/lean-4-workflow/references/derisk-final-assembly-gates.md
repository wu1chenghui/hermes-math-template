# De-risking a final-assembly proof: the gate ladder + characteristic audit

**When to read:** the hard mathematics of a classification theorem is "done" and only
the final-assembly / structure-decomposition step remains (e.g. `HalfDer = F·Id ⊕ kerΦ`
via an image-containment induction). Do NOT bulk-implement. The user (referee) runs a
**GATE LADDER**: a sequence of cheap probes, each answering ONE binary question with an
explicit pass/fail criterion, ordered by **information gain**, before committing to the
implementation. This caught a falsified *frozen* theorem on 2026-06-25.

## The gate-ladder discipline

- One gate = one question + one pass criterion + what it unlocks. Don't conflate gates.
- Choose the next gate by INFORMATION GAIN, not engineering volume. Spreading into
  enumeration before a risk is retired by a *compiled* probe is forbidden.
- Classify every risk by LAYER before acting: **theory / architecture / statement / engineering**.
  A "I can't close this lemma" problem is often a STATEMENT-LEVEL problem in disguise.
- verify-in-`lean_run_code`-first, then land. NEVER commit a `sorry`-skeleton whose
  statement an open design decision might rewrite.
- Don't auto-freeze and don't auto-advance to the next proof object; the referee decides
  scope each turn. Offer the freeze/decision at the gate boundary.

### Gate types that worked (reusable templates)

1. **REPRESENTATIVE PROBE** (Gate 1): prove 1–2 concrete instances of each mechanism class
   (here: a boundary + a transfer coeff-vanishing lemma) → confirms the mechanism compiles
   into stable, axiom-clean lemmas *before* generalizing. "API is organizable" check.
2. **EDGE-TYPE COMPLETENESS AUDIT** (Gate 2): prove a representative of EACH remaining
   structural edge-type; confirm no NEW landing type appears → "enumeration = pure
   engineering, no new structure." Retires the "枚举中冒出新 case" risk.
3. **ASSEMBLY-INTERFACE SKELETON** (Gate 3): write the final theorem's REAL statement +
   induction + branch split; wire ONE branch fully (test the `ih` / helper-lemma APIs),
   `sorry` the rest, compile in `lean_run_code`. Surfaces hidden interface risks BEFORE
   bulk work. Here it caught: the leaves need a hypothesis `hI` that the (architecturally
   hI-free) target theorem cannot supply — a statement-level circularity, not a gap.
4. **HYPOTHESIS-USE-POINT AUDIT** (Gate 4): to decide whether hypothesis `H` can be
   replaced by an induction hypothesis (a joint induction), tabulate EVERY use of `H` and
   the well-founded measure (e.g. target-width `v−u`) of the object it discharges.
   All-strictly-smaller ⟹ candidate is internally well-founded. **NECESSARY, not
   SUFFICIENT**: also check cross-edges — an `(a)→(b)` call that jumps UP in the measure
   (e.g. a width-2 boundary leaf depending on a fixed width-`n−2` fact) breaks a
   single-axis induction even when every `H`-use is smaller.
5. **COUPLING-NODE LANDING TAXONOMY** (Gate 5): when a residual lands back on a coeff,
   classify by parameter (zero-residual vs re-invokes-the-same-node). A same-node 2-cycle
   with coupling determinant `d` is solvable iff `char ∤ d` — which triggers the
   characteristic audit below.

## The characteristic-dependence audit (the big catch)

When a coefficient-coupling node yields a **small-integer coupling determinant** (here
`det = 3` at the `(1,2;3,n)/(1,3;2,n)` node, from `x − y = 0 ∧ x + 2y = 0`), the theorem
may be CHARACTERISTIC-DEPENDENT. Test it computationally — runnable:
`scripts/halfleibniz_nullspace_modp.py`.

- Compute the constraint-nullspace dimension over MULTIPLE primes: the suspect char
  (𝔽₃), a control char ∉ {2, suspect} (𝔽₅), and a big prime (≈ char 0).
- **SELF-CHECK**: the big prime + control MUST reproduce the known/expected dim. If they
  do, a differing value at the suspect prime is real, not a setup bug.
- 2026-06-25 result: `dim HalfDer(N_n) = n+5` over char 0 / 𝔽₅ but **`n+7` over 𝔽₃**
  (n=5,6,7, +2 uniform). The frozen "char ≠ 2 ⟹ dim n+5" was FALSE; correct range
  **char ≠ 2,3** (char 3 ⟹ n+7).
- Restrict the same system to the I-filtered subspace (`restrict_I=True`) to compute
  `kerΦ`: here `n+4` in ALL characteristics → the char-3 extra dims are OUTSIDE kerΦ, so
  the already-proven Lean lemmas (`[CharNeTwo F]`, hence valid at 𝔽₃) and `finrank kerΦ
  = n+4` were UNAFFECTED. Blast radius was confined to the final-assembly char range.

### Extracting the characteristic-specific extra solutions

To get the EXPLICIT extras (not just the dimension): solve the AUGMENTED system
`M x = 0 ∧ x[node] = 1` mod p (`solve_affine` in the script):
- consistent mod p ⟹ an explicit extra solution exists in char p; read off its action
  as `T(E_src) = Σ coeff·E_tgt`.
- INCONSISTENT mod the control prime ⟹ that coeff is forced 0 in char ≠ p, **proving** the
  solution is characteristic-p-specific.
- 2026-06-25: extracted `T₁(E_{1,2})=E_{3,n}, T₁(E_{1,3})=E_{2,n}` and its σ-mirror `T₂`,
  the two char-3-only directions; n-independent across n=5,6,7.
- Then CONFIRM the STRUCTURAL reason on paper: find the single binding bracket whose
  half-Leibniz forces the determinant. `[E_{1,2},E_{2,3}]=E_{1,3}` gives
  `2·T(E_{1,3}) = [T E_{1,2}, E_{2,3}] = [E_{3,n},E_{2,3}] = −E_{2,n}` ⟹ `2 E_{2,n} =
  −E_{2,n}` ⟹ `3 E_{2,n}=0` ⟺ char 3. (= the chronology's old "3t=0 detour", now explicit.)

## Meta-lesson

A "char ≠ 2" (or any single-prime-exclusion) frozen claim is UNTESTED in the other small
characteristics until you run the multi-prime nullspace test. A coupling determinant
divisible by `p` is the tell to test char `p`. Mark the frozen node UNDER REVIEW (don't
rewrite the statement) until the new char-split statement has both a general construction
(lower bound, explicit cocycles) AND an ∀n upper bound; record the audit as a project
fact in AGENTS so the next session doesn't re-derive it.
