# D4 Centering Debug — Case Study (2026-06-30)

## The Problem

`lake build E.Classification.Centering` failed with 4 errors:
- line 1242: `Max Type` on `Module.finrank`
- line 1264: `simp` made no progress
- line 1271: `LinearEquiv.finiteDimensional` type mismatch
- line 1281: `assumption` failed

Previous investigation (2026-06-29) had classified these as:
- Groups A, B: RESOLVED (lines 544, 1124)
- Group C: IN PROGRESS (finrank), suspected "Max Type universe issue"

## The Investigation (Evidence-First)

### Step 1: Collect full error contexts
- Read source ±30 lines around each error
- Get LSP goals at each error site
- Discovered: LSP showed `h_eq` from `Submodule.finrank_sup_add_finrank_inf_eq` SUCCEEDING at line 1278, but lake build failing at line 1242

### Step 2: Verify mathlib API existence
- `#check Submodule.finrank_sup_add_finrank_inf_eq` — EXISTS ✓
- `#check Submodule.finrank_sup_eq_of_disjoint` — DOES NOT EXIST ✗ (phantom API in old proof)
- `#check finiteDimensional_of_finrank` — DOES NOT EXIST ✗ (phantom API)

### Step 3: Diff against pre-session backups
- 11 modules reclassified from "dead" to "imported by live code"
- Old `finrank_halfDer` proof used non-existent lemmas — NEVER compiled

### Step 4: Build diagnostic matrix
| Error | Confirmed cause | Status |
|-------|----------------|--------|
| 1264 | Dead `simp [v]` on Subtype | Remove line |
| 1271 | `h_equiv.symm.finiteDimensional` wrong direction | Use `h_equiv.finiteDimensional` |
| 1242 | Cascade: FD instance missing → `Submodule.finrank_sup_add_finrank_inf_eq` fails | Hypothesis |
| 1281 | Cascade of all above | Hypothesis |

### Step 5: Fix and rebuild
After fixing 1264 + 1271: errors reduced from 4 to 2 (1242 + 1281).
After eliminating `have h_fin` wrapper (inline proof): 0 errors.

## Root Causes

1. **`LinearEquiv.finiteDimensional` direction**: `h_equiv.symm.finiteDimensional` was wrong (needs FD(sId)→gives FD(F), but we have FD(F)→need FD(sId)). Fix: `h_equiv.finiteDimensional inferInstance`.

2. **`have`-type elaboration**: `Module.finrank F (sId ⊔ kerPhi ...)` as a `have` TYPE triggers elaboration failure (reported as "Max Type"). Same expression as goal works fine. Fix: eliminate `have h_fin` + `simpa` wrapper.

3. **Dead `simp`**: `simp [v] at this` where `v : sId` (Subtype) does nothing.

## Lessons

1. "Max Type" error was NOT a universe issue — it was a downstream artifact of a `LinearEquiv.finiteDimensional` direction error + `have`-type elaboration path.

2. Investigation-first workflow saved days: what appeared to be a fundamental typeclass/universe problem was actually two simple mistakes and a syntactic workaround.

3. Stale olean caches had hidden ALL of these errors in previous sessions. The old proof never compiled.

4. `lake clean && lake build` is the only trustable verification method.
