# Context Compression & Abstraction Memory — Research Survey

Summarized from May 2026 discussions. Approaches ordered from most "lightweight/plug-in" to most "architectural."

---

## 1. LLMLingua / LongLLMLingua (Microsoft, 2024)

**Idea**: Use a *small* model (e.g. a BERT variant) to score each token in the prompt for redundancy, then prune low-value tokens before sending to the large model.

**Key results**:
- Compresses prompts to ~20% of original size with negligible task-performance loss
- LongLLMLingua adds query-aware retention — preserves tokens most relevant to the current question

**Why it matters for abstraction compression**:
- No model modification needed
- Lightweight auxiliary model does the "forgetting" (dropping detail)
- The small model acts as a learned attention filter — what survives is the structure, not the noise

**Limitations**:
- Operates at token level, not semantic level — can't reorganize or restructure
- One-shot per prompt, no persistent memory across turns

**Link**: arXiv:2310.01287 / arXiv:2402.10657

---

## 2. Gist Token (Mu et al., 2023)

**Idea**: Append a special `<GIST>` token to a long input, then fine-tune (LoRA or prefix-tuning) so that token's hidden state encodes the entire content. Downstream use only needs that single embedding.

**Key insight**: Compresses arbitrarily long context into the representation space of a single token.

**Why it matters**: This is the closest existing approach to "abstract structure extraction" — the model is forced to learn what's essential and discard the rest.

**Limitations**:
- Requires fine-tuning (even if light)
- The gist token's information is opaque — you can't inspect what was kept vs. discarded

---

## 3. AutoCompressor / ICAE (Input Compression via Auto-Encoding, 2023-2024)

**Idea**: Use the LLM itself as an autoencoder. Feed a long document, have it produce a short sequence of "summary vectors" (soft tokens), then reconstruct from those vectors.

**Why it matters**: The compressed representation is learned end-to-end — the model decides what to keep and how to structure it. Multiple approaches:
- **AutoCompressors**: prepend soft tokens that accumulate information as the model reads the context
- **ICAE**: encoder produces compressed key-value pairs; decoder (same LLM) reconstructs; the compressed form can be reused

**Limitations**:
- Requires training (even if parameter-efficient)
- Compression is domain-agnostic — no explicit "forgetting" mechanism

---

## 4. MemGPT / Letta (2023-2024)

**Idea**: Give the LLM an OS-like virtual memory architecture:
- **Main context** = working memory (fixed size, visible at all times)
- **External memory** = disk (vector DB, recursively summarized)
- **Memory management** = the LLM itself decides: when context fills up, the model chooses what to "archive" (compress into external storage) and what to "recall" (pull back in)

**Key innovation**: The model is *active* in managing its own memory — it decides what's important enough to keep and what to forget.

**Why it matters**: This is the most direct implementation of the abstraction-compression + selective forgetting concept.

**Limitations**:
- Complex architecture — external DB, recursive summarization, trigger conditions
- The model's memory-management decisions (what to archive) can be suboptimal

**Link**: arXiv:2310.08560 (MemGPT); Letta.dev (open-source implementation)

---

## 5. H₂O — Heavy Hitter Oracle (2023)

**Idea**: During inference, examine the attention scores in the KV Cache. Most past tokens receive near-zero attention from future tokens. H₂O identifies the "heavy hitter" tokens (those with consistently high attention scores) and drops the rest.

**Key result**: Discarding 50% of KV cache entries has no measurable impact on generation quality.

**Why it matters**: This is "selective forgetting" at the *mechanistic* level — not a policy, but a discovered property of the transformer's attention patterns.

**Limitations**:
- Only works within a single generation session
- Doesn't produce persistent, re-usable abstractions

---

## 6. StreamingLLM (2024)

**Idea**: The model only keeps:
1. A small set of initial tokens (the "attention sink" anchor)
2. The most recent tokens
Everything in between gets dropped.

**Why it matters**: Extremely simple, no training needed. Demonstrates that models can function with only "recent + anchor" context — the rest is noise.

**Limitations**:
- No semantic prioritization — just positional
- Can't selectively retain important information from the middle of the context

---

## 7. DRAGIN (2024)

**Idea**: Instead of retrieving external knowledge at every step (standard RAG), only retrieve when the model displays *uncertainty* — measured by entropy of the output token distribution.

**Why it matters**: This is the computational analogue of "only consulting notes when stuck." It reduces unnecessary context while preserving critical retrieval triggers.

**Limitations**: Only addresses *when* to retrieve, not *how* to compress or forget.

---

## 8. Human Creativity & Memory — Foundational References

### Poincaré (1908), "Mathematical Creation"
Four-stage model: Preparation → Incubation (unconscious processing) → Illumination (insight) → Verification. The unconscious phase performs combinatorial exploration — the "noise" of random neural activity produces novel combinations, and aesthetic judgment selects the promising ones.

### Hadamard (1945), "The Psychology of Invention in the Mathematical Field"
Surveyed leading mathematicians (including Einstein). Finding: most mathematicians think in images and kinesthetic feelings, not words. Creative insight emerges from pre-verbal, non-linguistic pattern manipulation.

### Hawkins (2021), "A Thousand Brains"
The neocortex runs a universal **memory-prediction algorithm**: constantly predicting sensory input from stored patterns, updating when prediction fails. Creativity = exploring the gap between prediction and reality.

### Friston — Free Energy Principle
The brain minimizes prediction error. Exploration and creativity emerge from "active inference" — the brain changes the world (via actions) to match its predictions, not just the reverse.

### Damasio — Somatic Marker Hypothesis
Emotion is not separate from rational decision-making. The body generates feedback signals (heart rate, etc.) that mark certain options as "good" or "bad" — this tagging is essential for efficient decision-making in high-dimensional spaces.

---

## Synthesis: What's missing in current approaches

| Human capability | Closest current analogy | Gap |
|---|---|---|
| Selective forgetting | H₂O (drop low-attention tokens) | No persistence across sessions |
| Abstraction compression | Gist Token / AutoCompressor | Requires training, not plug-and-play |
| Self-directed memory management | MemGPT | Complex, model-dependent |
| Emotional/body-based valuation | None | No analogue exists in pure-LM systems |
| Incubation / random exploration | Temperature sampling (weak analogue) | Randomness is blind, not structurally guided |
| Reconstructive recall | None | Current systems retrieve exact matches, not regenerated structures |
