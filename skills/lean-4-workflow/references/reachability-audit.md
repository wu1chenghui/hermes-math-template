# Reachability Audit — Zero-Sorry Verification for Project Release

When a project approaches release, a **reachability audit** determines whether
any `sorry` in the repository is reachable from the main theorem's dependency
closure. The goal is NOT "zero sorry in the entire repo" but "zero reachable
sorry in the active proof graph."

## Method

### Step 1: Identify active files

The entry point (e.g., `E.lean`) defines the full import tree. For a targeted
audit, identify ONLY the files imported by the classification/theorem you care
about (e.g., `HalfClassification.lean`).

### Step 2: Exact sorry count

```bash
cd <project>
for f in $(find -name '*.lean' | sort); do
  count=$(grep -c "\bsorry\b" "$f" 2>/dev/null || echo 0)
  [ "$count" != "0" ] && echo "  $count $f"
done
```

### Step 3: Dependency closure check

For each file with a sorry, check whether it is in the dependency closure of
the classification theorem:

```bash
grep "^import" E/Classification/HalfClassification.lean
```

For each import, trace transitive imports. Any sorry file NOT in this closure
is unreachable legacy.

### Step 4: Logically unreachable blocks

Sometimes a file in the dependency closure has a `sorry`, but the code path
to it is guarded by a hypothesis that the caller always provides. Example:

```lean4
-- HalfNonadjacent.lean (deferred adjacent case):
theorem coeffOf_nonadjacent ... (h_off : u ≠ i ∨ v ≠ j) : ... := by
  by_cases h_ge2 : j - i ≥ 2
  · ... -- full proof
  · sorry  -- adjacent case

-- HalfClassification.lean (caller blocks the sorry-path):
theorem offdiag_zero_of_nonadjacent ...
    (h_nonadj : 2 ≤ j - i) (h_off : u ≠ i ∨ v ≠ j) : ... :=
  coeffOf_nonadjacent ... h_off
  -- h_nonadj ensures j-i ≥ 2, so the `sorry` branch is NEVER reached
```

Tag these as "unreachable — guarded by caller hypothesis."

### Step 5: Legacy files in separate import trees

Files imported by `E.lean` but NOT by the classification file's dependency
closure are legacy. These sorry's are genuine but isolated. Do NOT delete them
— they document the historical development path.

## Audit Report Template

```
Reachability Audit — <date>

Active files: <count>
Active sorry: <count>

| File | sorry | Reachable from <target>? | Resolution |
|------|-------|--------------------------|------------|
| HalfNonadjacent.lean | 1 | NO (h_nonadj guard) | Documented boundary |
| Coefficient.lean | 1 | NO (separate import tree) | Legacy, frozen |
| Nonadjacent.lean | 1 | NO (separate import tree) | Legacy, frozen |
| Diagonal.lean | 1 | NO (separate import tree) | Legacy, frozen |

Conclusion: Active Proof Graph — zero reachable sorry.
```

## Guard pattern for boundary cases

When a theorem has a known boundary case that is separately handled, add a
hypothesis to the CALLER that blocks the boundary, rather than trying to
prove it in the callee:

```lean4
-- Callee: full theorem with boundary case deferred as `sorry`
theorem coeffOf_nonadjacent {j-i ≥ 1} : ...

-- Caller: add hypothesis that blocks the boundary
theorem offdiag_zero_of_nonadjacent (h_nonadj : 2 ≤ j - i) ... := ...
```

This is preferred over modifying the callee's signature (which would break
the induction hypothesis generalization in recursive calls).

## When to use

- Before declaring a project "release candidate"
- When a paper maps lemmas to Lean theorems
- After major refactoring that changes import structure

## NOT a goal

- Zero sorry in the entire repository (legacy code has historical value)
- Removing all dead code (risk of breaking historical context)
