# Local 3-Equation SCC Pattern

When a single `bracketIdentity` or `coeffOf_cond` leaves a coefficient underdetermined,
look for a **self-contained 3-equation linear system** with no hI, no induction, no width
machinery.

## Design recipe

1. **Choose 3 variables** (x, y, z) = three coefficients that form a closed system.
2. **Find two commuting-partner bracketIdentity equations** that couple the variables.
3. **Find one coeffOf_cond** equation that links two of them with factor 2.
4. **Verify the 3×3 determinant** is nonzero in the working characteristic.

## Example: BND-L local SCC (char-independent, det=1)

```
x = coeff(3,n; 2,n)      (wide ∈I residual)
y = coeff(3,4; 2,4)      (adjacent off-diag ∉I)
z = coeff(1,2; 1,3)      (boundary target, width=2)

(1) bracketIdentity(1,2, 3,n, 1,n) = 0  →  z + x = 0   [commuting: 2≠3, 1≠n]
(2) bracketIdentity(3,4, 1,2, 1,4) = 0  →  y + z = 0   [commuting: 4≠1, 3≠2]
(3) coeffOf_cond(3,n; 2,n) @ k=4        →  2x = y      [sole survivor: A-term]
```

→ `z = -x, y = -z = x, 2x = x → x = 0 → y = z = 0`.  Det = 1.

## Commuting-partner verification checklist

For sources `(i,j)` and `(c,d)` to commute:
- `j ≠ c` (first source column ≠ second source row)
- `i ≠ d` (first source row ≠ second source column)

Then `D.bracket_zero i j c d u v ...` gives `bracketIdentity = 0`.

## bracketIdentity expansion reference

```
bracketIdentity(i,j,c,d,u,v) =
  (if u < c ∧ v = d then coeff(i,j; u,c) else 0)   -- T1
- (if u = c ∧ d < v then coeff(i,j; d,v) else 0)    -- T2
+ (if u = i ∧ j < v then coeff(c,d; j,v) else 0)    -- T3
- (if u < i ∧ v = j then coeff(c,d; u,i) else 0)    -- T4
```

## When bracketIdentity DOESN'T constrain an adjacent source

For adjacent source `(i,i+1)` targeting `(u,v)` with `v ≠ i+1`:
- T1: needs `v = d` (partner source col), but then gives `coeff(i,i+1; u,c)`, NOT `(u,v)`.
- T2: needs `d = u` and `d > c = u` → impossible (d > u, d = u).
- So bracketIdentity is trivially zero for ALL partners.

The constraint comes from **bracketSource** via half-Leibniz:
`2·(coeff(i,d; u,v) - coeff(c,i+1; u,v)) = bracketIdentity = 0`
This couples the adjacent coefficient to wider nonadjacent coefficients,
requiring induction or width machinery.

## coeffOf_cond single-survivor analysis

```
coeffOf_cond(i,j; u,v) @ split k:
  2·coeff = (A - B + C - D)  where
  A = if u < k ∧ v = j then coeff(i,k; u,k)      [adjacent interior]
  B = if u = k ∧ j < v then coeff(i,k; j,v)       [boundary leaf]
  C = if u = i ∧ k < v then coeff(k,j; k,v)        [wider source]
  D = if u < i ∧ v = k then coeff(k,j; u,i)        [edge]
```

To verify a single survivor: check B/C/D conditions are impossible with omega.
