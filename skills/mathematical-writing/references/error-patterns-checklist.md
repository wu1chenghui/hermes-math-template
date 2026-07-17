# Error Prevention Checklist

After writing each sentence in a proof section, check:

```
[ ] 结论词：只用 Thus/So，不碰 Hence/Therefore/Consequently
[ ] 零的表达：永远 =0 或 is zero，不写 vanishes
[ ] 归纳法：不写 induction，改用 "the claim for smaller width" 或 chains/propagates
[ ] 反证法：不写 contradiction，用 "forcing ... = 0"
[ ] 情况枚举：连续 If/Suppose ≥3？→ 插入变化或中间结论句
[ ] 条件前置："For X, we Y" → "If X, then Y" 或 "Now Y"
[ ] 评价词：不出现 key/crucial/important/fundamental/surprising
[ ] 重述词：不出现 Namely/i.e./In other words
[ ] 预告词：不出现 "as follows:"/"We begin by"/"We are now ready to"
```

## Common Error Patterns (from exercise audit)

| Error | Frequency | Root Cause | Fix |
|-------|-----------|------------|-----|
| `vanishes` → `is zero` | 6× | Academic English default vocabulary | Train muscle memory: only `= 0` or `is zero` |
| `Hence` → `Thus` | 3× | "Therefore" family is the default conclusion word | Only two allowed: Thus, So |
| Consecutive `Suppose` (5×) | 5× | Parallel structure inertia in case enumeration | Vary openers; insert mid-conclusion |
| `induction` → rephrase | 2× | Standard proof terminology | Use Step structure or "the claim for smaller width" |
| `Consequently` → `Thus` | 1× | Same family as Hence | |
| `contradiction` → delete | 1× | Proof-by-contradiction phrasing | Use "forcing ... = 0" |
| Prepositional openers | 3× | "For X, we Y" is natural for precise conditions | "If X, then Y" or "Now Y" |

## Proof-Structure Pitfalls (added 2026-07-10)

| Pitfall | Detection | Fix |
|---------|-----------|-----|
| Ψ=0 written for non-commuting pair | Check [x,y] before writing Ψ(x,y,φ)=0 | Write full half-Leibniz: 2φ[x,y] = Ψ(x,y,φ) |
| Chain bridge claimed to couple c_k, c_{k+1} at target (2,n) | Both brackets give zero for interior k | Use F1/F2 commuting-pair constraints instead |
| Channel restriction bound includes boundary index | For k=n-1, [E_{n-1,n}, I] ≠ 0 | Use correct interval: 3≤k≤n-2 for c_k, 2≤k≤n-3 for a_k |
| Projection target (u,v) gives 0=0 for commuting partner E_{v,v+1} | First bracket has col v+1≠v, second has index mismatch | Project to (u, v+1) instead |
