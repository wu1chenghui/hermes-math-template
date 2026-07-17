# split_ifs goal direction vs IsValidSupp lemmas

## The pitfall

After `unfold coeffOf; split_ifs`, the false-branch goal is `0 = ...`
but `IsValidSupp` lemmas (like `k.property.2.1`) return `... = 0` —
the REVERSE direction.

```lean
-- ❌ FAILS: goal is 0 = k.val i j u v, lemma gives k.val i j u v = 0
lemma coeffOf_kerPhi_lift (hn : 3 ≤ n) (k : kerPhi (F := F) hn) (i j u v : ℕ) :
    coeffOf (kerPhi_lift hn k) i j u v = (k : V F) i j u v := by
  unfold coeffOf
  split_ifs with h
  · rfl
  · rcases h with (hi | hij | hjn | hu | huv | hvn)
    · exact k.property.2.1 i j u v (by intro hvalid; exact hi hvalid.1)
    -- Goal: 0 = ↑k i j u v
    -- Term: ↑k i j u v = 0    ← WRONG DIRECTION
```

## The fix

Add `.symm` to all 6 branches:

```lean
  · rcases h with (hi | hij | hjn | hu | huv | hvn)
    · exact (k.property.2.1 i j u v (by intro hvalid; exact hi hvalid.1)).symm
    · exact (k.property.2.1 i j u v (by intro hvalid; exact hij hvalid.2.1)).symm
    · exact (k.property.2.1 i j u v (by intro hvalid; exact hjn hvalid.2.2)).symm
    · exact (k.property.2.1 i j u v (by intro hvalid; exact hu hvalid.2.2.1)).symm
    · exact (k.property.2.1 i j u v (by intro hvalid; exact huv hvalid.2.2.2.1)).symm
    · exact (k.property.2.1 i j u v (by intro hvalid; exact hvn hvalid.2.2.2.2)).symm
```

## Why this is deceptive

1. It compiles in some Lean versions depending on `split_ifs` behavior
2. It may appear in AGENTS.md checkpoints that claim "compiles" but actually relied on stale olean caches
3. The error message (unsolved goals) is clear, but only surfaces after a clean rebuild

## General pattern

After ANY `unfold` + `split_ifs`, check the goal direction. If the goal is `0 = expr` but your lemma produces `expr = 0`, add `.symm`.
