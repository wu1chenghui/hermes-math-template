# Embedding + Concept Diffusion + Temperature Sampling Retrieval

Built on 2026-06-01 as the third iteration of the Hermes Brain retrieval system.
See `references/hermes-brain-implementation.md` for the initial build (first two iterations).

## Motivation

The initial system used concept-based keyword matching (`concepts` JSON array + `LIKE` queries).
This failed on semantically-similar but lexically-different queries (e.g. "灵感" vs "创造力").

The second problem: pure Top-K retrieval is deterministic and can never produce the serendipitous
cross-domain connections that characterize human "inspiration."

## Architecture overview

```
Query text
    │
    ├─[1] Embedding: fastembed (bge-small-zh-v1.5) → 512d vector
    │
    ├─[2] Cosine similarity → top 20 candidates
    │
    ├─[3] Concept graph diffusion from Top-1:
    │       Direct concept share → +0.30 bonus
    │       1-hop concept link → +0.15 bonus
    │
    ├─[4] Segmented selection:
    │       First N-1: similarity + diffusion + importance (deterministic)
    │       Last 1:     temperature noise added, from pool positions 3-5
    │
    └─[5] Return top_k with source labels (🔍 embedding / 🌐 concept / 🎲 岔路)
```

## Setup

### Install fastembed in the persistent venv

```bash
source /opt/data/.venv/bin/activate
uv pip install fastembed
```

`fastembed` uses ONNX Runtime (no PyTorch dependency). The model
`BAAI/bge-small-zh-v1.5` produces 512-dimensional vectors and handles Chinese text well.

### Add embedding column to SQLite

```sql
ALTER TABLE memories ADD COLUMN embedding BLOB;
```

### Generate embeddings for existing memories

```python
import sqlite3, pickle
from fastembed import TextEmbedding

model = TextEmbedding('BAAI/bge-small-zh-v1.5')
conn = sqlite3.connect("/opt/data/workspace/hermes-brain/memory.db")
cur = conn.cursor()
cur.execute("SELECT id, abstract FROM memories WHERE embedding IS NULL")
for row in cur.fetchall():
    emb = list(model.embed(row[1]))[0]
    cur.execute("UPDATE memories SET embedding=? WHERE id=?", (pickle.dumps(emb), row[0]))
conn.commit()
conn.close()
```

Note: use `abstract` (the structural summary) rather than `raw_summary` — it captures
the pattern-level semantics, not surface-level details.

## Retrieval algorithm (three modes)

### Mode A: `--query` (default — semantic + diffusion + temperature)

```bash
source /opt/data/.venv/bin/activate
python3 query_memory.py --query "你的问题" --temp 0.3 --top-k 3
```

**Parameters:**

| Param | Default | Meaning |
|-------|---------|---------|
| `--temp` | 0.5 | Temperature for the wildcard slot. 0=deterministic, 0.8=very random |
| `--top-k` | 3 | Total results to return. First k-1 are deterministic, last 1 is wildcard |

**Detailed algorithm:**

1. Encode query text via fastembed → 512d vector
2. Compute cosine similarity for all active memories
3. Sort by similarity → keep top 20
4. Collect concepts from Top-1 → index all memories by concept membership
5. For each candidate:
   - `diffusion_bonus = 0.30` if shares ≥1 concept with Top-1
   - `diffusion_bonus = 0.15` if linked via 1-hop (a memory that shares Top-1's concept)
   - `importance_bonus = importance * 0.1` (reward long-valued memories)
6. **Segmented selection** (key innovation):
   - First `top_k-1` slots: deterministic, score = similarity + diffusion_bonus + importance_bonus
   - Last slot: from pool positions 3–5, add `random.gauss(0, temp * 0.12)` noise,
     re-sort within pool, take highest
7. Display each result with source labels

### Mode B: `--keyword` (legacy)

```bash
python3 query_memory.py --keyword "搜索词"
```

Concept-based `LIKE` search on `abstract`, `raw_summary`, and `concepts`.
Preserved for backward compatibility when exact term matching is needed.

### Mode C: `--query-all` (browse)

```bash
python3 query_memory.py --query-all
```

Lists all active memories sorted by importance.

## How temperature sampling creates "岔路" (serendipity)

The wildcard slot only operates on candidates in positions 3–5 (not the full pool).
This means:

- Position 1 and 2 are always the most semantically relevant (deterministic)
- The wildcard picks from the "close-but-not-best" group
- At `temp=0.3`, the noise is small — the wildcard is usually position 3 (best of the rest)
- At `temp=0.8`, the noise can push a position-5 memory past position 3, creating genuine
  cross-domain connections
- Re-running the same query with the same temperature produces different wildcard results

This simulates the brain's stochastic resonance — neural noise that occasionally
activates distant schema associations.

## Pipeline auto-embedding

`compress_pipeline.py` includes a `generate_embeddings()` step that runs every cycle.
It finds memories where `embedding IS NULL` and generates vectors automatically.
This ensures that any memory written without an embedding (e.g. by manual SQL insert
or a previous-version agent) gets indexed on the next pipeline run.

## Parameter rationale (the "occasional user" adjustment)

The user runs the container infrequently — sometimes weeks between sessions.
Original parameters (decay=0.05/day, reinforce=+0.02, forget=30d) caused:
- In-session over-boosting: one intense conversation could boost a memory 0.10+
  (5+ references × +0.02), exceeding a full day's decay
- Too-slow forgetting: at -0.05/day, a memory untouched for a 2-week gap only
  decays -0.70 — still well above the 0.15 threshold

Tuned parameters:
| Parameter | Old | New | Why |
|-----------|-----|-----|-----|
| Decay/day | 0.05 | 0.10 | Faster fading during multi-week gaps |
| Forgive days | 30 | 14 | Faster identification of cold memories |
| Reinforce boost | 0.02 | 0.01 | Prevents one-session boost from dominating |

Result: a memory used once per 3 sessions (~42 days) maintains equilibrium.
A memory never referenced falls to forget threshold in ~14 days.

## Embedding model choice: BAAI/bge-small-zh-v1.5

| Property | Value |
|----------|-------|
| Dimensions | 512 |
| Language | Chinese (also handles English) |
| Runtime | ONNX (no PyTorch) |
| Size | ~30MB |
| Context length | 512 tokens |
| Speed | ~10ms per encode (CPU) |

Alternatives considered:
- `all-MiniLM-L6-v2` (384d, English-only) — poorer for Chinese queries
- OpenAI `text-embedding-3-small` — API call overhead, no offline capability

## Lessons learned

### SQLite column indexes are easy to get wrong

When reading from a `cur.fetchall()` result, index 0 = first SELECT column.
If SELECT has 7 columns and you read index 3 thinking it's `importance` when
it's actually `concepts`, you get `TypeError: can't multiply sequence by non-int`.
Always verify column order against the SELECT statement, and prefer named
access via `conn.row_factory = sqlite3.Row` if available.

### fastembed returns a generator

`model.embed(text)` does NOT return a list — it returns a generator.
You MUST consume it with `list()`: `emb = list(model.embed(text))[0]`.

### numpy array truthiness

`pickle.loads(pickled_embedding)` returns a numpy array.
`if my_array:` raises `ValueError: The truth value of an array with more than
one element is ambiguous`. Use `if my_array is not None:` instead.

### Pipeline runs from system cron, not venv

The cronjob pipeline uses the system Python. To access fastembed, the
`generate_embeddings()` function injects the venv path into `sys.path`:
```python
import sys
sys.path.insert(0, "/opt/data/.venv/lib/python3.13/site-packages")
from fastembed import TextEmbedding
```

### Small memory pools limit 岔路 effect

When there are only 5-6 memories, the "positions 3-5" pool is very small.
The wildcard slot will often pick the same memory repeatedly (just with
different noise values). The 岔路 mechanism becomes more interesting as
the memory pool grows to 20+ entries.

## File changed

| File | Change |
|------|--------|
| `scripts/query_memory.py` | Rewritten: three-mode retrieval |
| `scripts/compress_pipeline.py` | Added `generate_embeddings()` step |
| `AGENTS.md` | Updated retrieval instructions |
| `memory.db` | Added `embedding BLOB` column, indexed 5 memories |
