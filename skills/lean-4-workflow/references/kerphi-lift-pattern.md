# kerPhi_lift — Propless Structure Constructor Pattern

When a Submodule's membership predicate exactly provides the field conditions
for a Prop-guarded structure, the lift from the Submodule to the structure
is a single `refine { ... }` block — no new mathematics.

## The pattern (general)

```lean
-- Structure: carrier type T + Prop field P
structure S (params) where
  carrier : T
  prop_field : P carrier  -- some proposition about the carrier

-- Submodule: T-level condition ⇔ P
-- kerPhi : Submodule F T
--   membership = A(carrier) ∧ B(carrier) ∧ (Φ(carrier) = 0)
--   where Φ(carrier) = 0 ↔ P(carrier)

-- The lift: trivial!
def lift (k : kerPhi) : S := {
  carrier := (k : T)
  prop_field := -- from k.property.2.2 and the Φ↔P equivalence
}
```

## Concrete instance: HalfDerivation ← kerΦ

```lean
structure HalfDerivation (F) [Field F] [CharNeTwo F] (n : ℕ) where
  coeff : ℕ → ℕ → ℕ → ℕ → F
  half_leibniz : ∀ i j p q u v, (validity guards) →
    2 * bracketSource coeff i j p q u v = bracketIdentity coeff i j p q u v

def IsKerPhi (hn : 3 ≤ n) (c : V F) : Prop :=
  I_filtered c hn ∧ IsValidSupp (n := n) c ∧
    (∀ i j p q u v : ℕ, (validity guards) → Phi c i j p q u v = 0)

def kerPhi (hn : 3 ≤ n) : Submodule F (V F) := ...
```

Key fact: `Phi c i j p q u v = 0` unfolding gives `2*bracketSource = bracketIdentity`,
which IS the `half_leibniz` condition. So `k.property.2.2` directly proves
`half_leibniz` for `k.val`.

The lift (Centering.lean, D4.1):
```lean
def kerPhi_lift (hn : 3 ≤ n) : kerPhi (F := F) hn → HalfDerivation F n := by
  intro k
  refine {
    coeff := (k : V F)
    half_leibniz := kerPhi_half_leibniz hn k
  }
```

where `kerPhi_half_leibniz` is a lemma extracting `2*bracketSource = bracketIdentity`
from `Φ = 0` (one `unfold Phi; calc` block).

## When this pattern applies

- Structure has a carrier field + one or more Prop fields
- Submodule membership includes a condition that's equivalent (↔) to those Prop fields
- The equivalence is either definitional (unfold) or via a one-direction lemma

## When it DOESN'T apply

- Submodule membership carries extra invariants not needed by the structure
  (e.g. `I_filtered` and `IsValidSupp` are NOT needed for `HalfDerivation`
  construction — they're guarantees about the result, not prerequisites)
- Structure has additional invariants beyond what Submodule provides

## Complementary pattern: coeff_injective

When the structure equality is determined by the carrier field alone
(Prop fields are `Subsingleton`), the carrier projection is injective:

```lean
lemma coeff_injective : Function.Injective (fun (D : HalfDerivation F n) => D.coeff) := by
  intro D1 D2 h
  rcases D1 with ⟨c1, hl1⟩
  rcases D2 with ⟨c2, hl2⟩
  have hc : c1 = c2 := h
  subst hc
  rfl
```

This avoids needing `LinearMap`/`Module` instances on the structure type.
