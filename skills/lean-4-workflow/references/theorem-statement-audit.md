# Theorem Statement Audit — Lean↔Paper Cross-Check Methodology

## Two Levels of Cross-Check

### Level 1: Signature Matching (weak)
Check that paper theorem names map to Lean theorem names. Verify that the
type signatures look similar. This is fast but can miss false theorems
(propositions with `sorry` in their proof).

### Level 2: Axiom Closure Audit (strong)
Use `lean_verify` to check the transitive axiom closure of each Lean theorem.
If `sorryAx` appears, the theorem transitively depends on an unproven `sorry`.
This reveals gaps that signature matching would miss.

## Example: P_1 Counterexample Discovery (June 2026)

The paper's Theorem 6.3 claimed "c_I^T = 0 for all T ≠ I." The Lean theorem
`coeffOf_nonadjacent` had the same signature. Level 1 audit would say: ✅ match.

But `lean_verify` revealed `sorryAx` in the closure. Investigation found:
- P_1(E_{12}) = E_{1n} is a valid 1/2-derivation with c_{12}^{1n} = 1 ≠ 0
- The theorem statement is **mathematically false** for adjacent sources at (1,n)
- The nonadjacent induction is NOT contaminated (see induction-contamination-check)

## Corrective Workflow

1. Run `lean_verify` on every paper-claimed theorem
2. For any `sorryAx`: check if it's a genuine unproven case or a false statement
3. Trace recursive calls in induction proofs to check for contamination
4. Restructure: split theorems into provable + exceptional cases

## Backup Pattern

After significant analysis (even without code changes), back up:
```bash
BACKUP=/workspace/backups/E_backup_$(date +%Y%m%d_%H%M%S)_<descriptor>
mkdir -p "$BACKUP"
cd /opt/lean-home/lean-projects/e
for f in E.lean $(find E -name '*.lean'); do
  mkdir -p "$BACKUP/$(dirname $f)"
  cp "$f" "$BACKUP/$f"
done
cp lakefile.toml lean-toolchain lake-manifest.json "$BACKUP/"
```
