# Unicode Math → LaTeX Mapping

Full mapping table with correct `{}` separators. All commands below are safe when followed by letters.

## Arrows
| Unicode | LaTeX | Notes |
|---------|-------|-------|
| → | `\to{}` | |
| ← | `\leftarrow` | rarely followed by letter |
| ⇒ | `\Rightarrow` | |
| ⇐ | `\Leftarrow` | |
| ↝ | `\rightsquigarrow` | requires amssymb |

## Set / Logic
| Unicode | LaTeX |
|---------|-------|
| ∈ | `\in{}` |
| ∉ | `\notin` |
| ⊆ | `\subseteq` |
| ⊇ | `\supseteq` |
| ⊂ | `\subset` |
| ⊃ | `\supset` |
| ∩ | `\cap` |
| ∪ | `\cup` |
| ∧ | `\land{}` |
| ∨ | `\lor` |
| ¬ | `\neg` |

## Relations
| Unicode | LaTeX |
|---------|-------|
| × | `\times{}` |
| · | `\cdot{}` |
| ≤ | `\le{}` |
| ≥ | `\ge{}` |
| ≠ | `\ne{}` |
| ≡ | `\equiv` |
| ≈ | `\approx` |
| ∞ | `\infty` |
| ∂ | `\partial` |
| − | `-` (ASCII hyphen; works in math mode) |

## Greek Lowercase (ALL need `{}`)
| Unicode | LaTeX |
|---------|-------|
| α | `\alpha{}` |
| β | `\beta{}` |
| γ | `\gamma{}` |
| δ | `\delta{}` |
| ε | `\varepsilon{}` |
| ζ | `\zeta{}` |
| η | `\eta{}` |
| θ | `\theta{}` |
| ι | `\iota{}` |
| κ | `\kappa{}` |
| λ | `\lambda{}` |
| μ | `\mu{}` |
| ν | `\nu{}` |
| ξ | `\xi{}` |
| π | `\pi{}` |
| ρ | `\rho{}` |
| σ | `\sigma{}` |
| τ | `\tau{}` |
| υ | `\upsilon{}` |
| φ | `\varphi{}` |
| χ | `\chi{}` |
| ψ | `\psi{}` |
| ω | `\omega{}` |

## Greek Uppercase (ALL need `{}`)
| Unicode | LaTeX |
|---------|-------|
| Γ | `\Gamma{}` |
| Δ | `\Delta{}` |
| Θ | `\Theta{}` |
| Λ | `\Lambda{}` |
| Ξ | `\Xi{}` |
| Π | `\Pi{}` |
| Σ | `\Sigma{}` |
| Υ | `\Upsilon{}` |
| Φ | `\Phi{}` |
| Ψ | `\Psi{}` |
| Ω | `\Omega{}` |

## Subscripts / Superscripts
| Unicode | LaTeX |
|---------|-------|
| ₀–₉ | `_{0}`–`_{9}` |
| ⁰–⁹ | `^{0}`–`^{9}` |

## Punctuation / Misc
| Unicode | LaTeX |
|---------|-------|
| … | `\dots` |
| — | `---` (em dash) |
| – | `--` (en dash) |
| ∎ | `$\square$` |
| § | `\S` |

## Why `{}` Is Required

TeX parses control sequence names by consuming all consecutive letters (A-Z, a-z).
Without the `{}` terminator:

```
λId    →  \lambdaId    ✗ (undefined control sequence, not \lambda Id)
2·D    →  2\cdotD      ✗ (undefined control sequence, not \cdot D)
≤u     →  \leu         ✗ (undefined control sequence, not \le u)
```

With `{}`:

```
λId    →  \lambda{}Id  ✓
2·D    →  2\cdot{}D    ✓
≤u     →  \le{}u       ✓
```

Exception: subscripts `₁` → `_{1}` already have braces from the subscript syntax, so no extra `{}` needed.
