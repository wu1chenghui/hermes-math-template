# Characteristic-Exception Detection via Mod-p Nullspace

A classification/dimension theorem stated for "char F ≠ 2" (one excluded prime) can
**secretly fail in another small characteristic**. Treat the characteristic range as
PART of the statement to validate — not a footnote. Catching a wrong char-range early
prevents building the whole downstream proof (D3/D4, paper) on a false claim.

Worked example this skill is drawn from: `dim HalfDer(N_n)` was LOCKED as
"char ≠ 2 ⟹ n+5". A mod-p test found **char 3 ⟹ n+7** (+2 extra cocycles). The
LOCKED theorem's char-range was wrong; it should be "char ≠ 2,3 ⟹ n+5".

## Signals that a specific prime p is exceptional
- A coupling/relation node whose small constraint matrix has **determinant = p**
  (e.g. `det [[1,-1],[1,2]] = 3`). In char p the det vanishes ⟹ extra freedom.
- A recurring "`p·t = 0`" / "`p·E = 0`" identity dropping out of two independent
  relations on the SAME coefficient (the classic "3t=0 detour").
- The original dimension was only ever computed over ℚ (char 0); small-char never checked.

## The hypothesis test (exact, cheap, decisive)
Build the FULL linear system — every defining identity (e.g. `2·bracketSource =
bracketIdentity` over ALL `coeff[(source),(target)]` unknowns on the valid basis) — then
compute `nullity = #unknowns − rank` by **exact mod-p Gaussian elimination** across:
- a **big prime** (e.g. 10007) as a char-0 proxy,
- a **control prime** (char ≠ suspect, e.g. 5) — must match char 0,
- the **suspect prime** (e.g. 3).

**SELF-CHECK FIRST:** confirm big-prime + control-prime nullity reproduces the KNOWN
char-0 dimension for a couple of sizes. If not, your encoding is wrong — fix before
trusting the suspect-prime number. (This is what makes the result trustworthy.)

Condensed recipe (adapt `comm()` to your Lie bracket; here N_n strictly-upper):
```python
def basis(n): return [(i,j) for i in range(1,n+1) for j in range(i+1,n+1)]
def comm(s1,s2):                       # [E_s1,E_s2] = δ_{jk}E_il − δ_{li}E_kj
    (i,j)=s1;(k,l)=s2;r={}
    if j==k: r[(i,l)]=r.get((i,l),0)+1
    if l==i: r[(k,j)]=r.get((k,j),0)-1
    return r
def build(n, restrict=None):           # restrict=set of allowed targets ⇒ sub-predicate dim
    B=basis(n); ok=(lambda t:t in restrict) if restrict else (lambda t:True)
    var={}; c=0
    for s in B:
        for t in B:
            if ok(t): var[(s,t)]=c; c+=1
    rows=[]; seen=set()
    for s1 in B:
        (i,j)=s1
        for s2 in B:
            (k,l)=s2; cb=comm(s1,s2)
            for t in B:
                row={}
                for (a,b),cc in cb.items():            # 2·T([s1,s2])|_t
                    if ok(t): vid=var[((a,b),t)]; row[vid]=row.get(vid,0)+2*cc
                for (p,q) in B:                          # −[T s1, E_s2]|_t
                    if not ok((p,q)): continue
                    cf=(q==k and (p,l)==t) - (l==p and (k,q)==t)
                    if cf: vid=var[(s1,(p,q))]; row[vid]=row.get(vid,0)-cf
                for (p,q) in B:                          # −[E_s1, T s2]|_t
                    if not ok((p,q)): continue
                    cf=(j==p and (i,q)==t) - (q==i and (p,j)==t)
                    if cf: vid=var[(s2,(p,q))]; row[vid]=row.get(vid,0)-cf
                row={a:b for a,b in row.items() if b}
                if row:
                    key=tuple(sorted(row.items()))
                    if key not in seen: seen.add(key); rows.append(row)
    return rows,c
def rank_mod_p(rows,nv,p):              # incremental RREF, sparse rows → dense pivots
    piv={}; rank=0
    for row in rows:
        cur=[0]*nv
        for vid,val in row.items(): cur[vid]=val%p
        for col in range(nv):
            if cur[col] and col in piv:
                f=cur[col]; pr=piv[col]
                for cc in range(col,nv):
                    if pr[cc]: cur[cc]=(cur[cc]-f*pr[cc])%p
        lead=next((c for c in range(nv) if cur[c]),None)
        if lead is not None:
            inv=pow(cur[lead],p-2,p); cur=[(x*inv)%p for x in cur]; piv[lead]=cur; rank+=1
    return rank
# nullity = nv - rank_mod_p(*build(n), p).  De-dupe makes n≤7 finish in <1s.
```

## Reading the result
- suspect-prime nullity == known dim ⟹ characteristic NOT special (the det=p was pinned
  by other relations); original char-range holds.
- suspect-prime nullity > known dim ⟹ **characteristic p is exceptional**; dim is
  char-dependent. Split the theorem by characteristic.

## Blast-radius containment (what frozen results survive)
Recompute nullity RESTRICTED to a sub-predicate (`build(n, restrict={…})`, e.g. only
"I-filtered" targets). If the restricted dim is the SAME across all chars, that frozen
sub-result is SAFE in char p; the extra dims live entirely in the complement.
This session: `kerΦ = n+4` in char 0/5/3 alike ⟹ the Lean `finrank kerΦ = n+4` (proven
for `[CharNeTwo F]`, hence valid at 𝔽₃) is untouched; **no false Lean theorem exists**,
the +2 anomaly is entirely outside kerΦ.

## Extracting the EXPLICIT exceptional cocycles
For the actual extra elements (not just the count), solve the AUGMENTED affine system
`M x = 0 ∧ x[node] = 1` mod p (node = the suspect coupling coordinate):
- **consistent** mod p ⟹ particular solution = an explicit exceptional cocycle; read off
  its support (e.g. `T₁: E₁₂↦E₃ₙ, E₁₃↦E₂ₙ`).
- **inconsistent** mod (control prime) ⟹ that coordinate is forced 0 ⟹ confirms
  char-p-specificity.
Repeat across several n — confirm the cocycle structure is n-independent (don't
extrapolate from one n; the +k must be the same for n, n+1, n+2).
```python
def solve_affine(rows,nv,p,sel):       # particular sol of M x=0 ∧ x[sel]=1, or None
    A=[[0]*nv for _ in rows]+[[0]*nv]; b=[0]*len(rows)+[1%p]
    for r,row in enumerate(rows):
        for vid,val in row.items(): A[r][vid]=val%p
    A[-1][sel]=1
    pr=0; piv=[]
    for col in range(nv):
        s=next((r for r in range(pr,len(A)) if A[r][col]%p),None)
        if s is None: continue
        A[pr],A[s]=A[s],A[pr]; b[pr],b[s]=b[s],b[pr]
        inv=pow(A[pr][col],p-2,p); A[pr]=[x*inv%p for x in A[pr]]; b[pr]=b[pr]*inv%p
        for r in range(len(A)):
            if r!=pr and A[r][col]%p:
                f=A[r][col]; A[r]=[(A[r][c]-f*A[pr][c])%p for c in range(nv)]; b[r]=(b[r]-f*b[pr])%p
        piv.append(col); pr+=1
    if any(all(x%p==0 for x in A[r]) and b[r]%p for r in range(len(A))): return None
    x=[0]*nv
    for i,pc in enumerate(piv): x[pc]=b[i]%p
    return x
```
Also verify the extracted cocycle directly: check `2·T([x,y]) − [Tx,y] − [x,Ty] ≡ 0
mod p` for ALL generator pairs (0 violations in char p, >0 in char≠p) over n=5..14.

## The structural reason (always find it — don't stop at numbers)
A computed det=p / extra-dim is a NUMBER. Find the single BRACKET that forces it: the
explicit cocycle T satisfies the defining identity at ONE generator pair only because
`p·(coeff)=0`. This session: T₁'s half-Leibniz at `[E₁₂,E₂₃]=E₁₃` gives
`2·E₂ₙ = [E₃ₙ,E₂₃] = −E₂ₙ ⟹ 3·E₂ₙ = 0 ⟺ char 3`. The structural reason (a) makes the
∀n claim airtight (the binding identity is n-independent), and (b) IS the lower-bound
proof — the explicit cocycles + this one bracket give `dim ≥ N+k ∀n`.

## Consequence for the theorem (freeze discipline)
- Split: `char ≠ 2,p ⟹ dim = N`;  `char = p ⟹ dim = N+k` (exceptional).
- Establish the char-p LOWER bound first (explicit cocycles + structural reason — cheap,
  ∀n). The char-p UPPER bound (`dim ≤ N+k ∀n`) is a SEPARATE, harder side-line; don't
  let it block the mainline.
- Mark the old single-statement "char ≠ 2" theorem **UNDER REVISION**, not re-LOCKED.
  Record the lower bound as ESTABLISHED, upper bound PENDING. Touching a frozen/LOCKED
  theorem node needs the referee's sign-off.
- The mainline then needs `char ≠ p` ADDED to the affected lemmas (the det=p coupling
  node becomes invertible). See lean-4-proof-writing `references/coupling-node-and-circularity.md`.
