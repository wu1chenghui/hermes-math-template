---
name: agent-memory-architecture
description: Design persistent, tiered memory systems for LLM agents — abstraction-compression pipelines, selective forgetting, hierarchical retrieval, and container-persistence engineering.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [memory, abstraction, compression, persistence, docker, architecture, agent]
    related_skills: [hermes-s6-container-supervision, hermes-agent]
---

> ⚠️ **IMPORTANT — Hermes Brain is OPTIONAL, not mandatory**
>
> This skill serves TWO roles: (1) a reference for designing memory architectures, AND (2) **reference documentation for the Hermes Brain external memory system** at `/opt/data/workspace/hermes-brain/`.
>
> **Hermes Brain is OPTIONAL — use on-demand, not every session.** See "When to use this skill" below for the judgment criteria.
>
> The Hermes Brain system exists at `/opt/data/workspace/hermes-brain/`. Its `AGENTS.md` has the exact commands. This skill is the WHY behind it — load this when you're designing or maintaining the system, not automatically on every session.

# Agent Memory Architecture

## When to use this skill

### On-demand triggers (Hermes Brain usage signals)
Load this skill and query Hermes Brain (`query_memory.py`) only when ONE OR MORE of these signals are true:

1. The user mentions a topic you **vaguely recall discussing before but don't remember the conclusion** (across-session recall)
2. The current topic **clearly spans multiple past sessions** (not just last turn)
3. The user explicitly asks "did we ever discuss X?" or "what did we conclude about X?"
4. You need **concept association** — not exact keyword match, but semantically related directions
5. The conversation produced a **counter-intuitive insight worth preserving long-term** (time to write, not just read)

Do NOT auto-query Hermes Brain on every session start. Do NOT auto-store on every session end.

### Design/architecture triggers (this skill as reference)
- The user asks about making the agent "remember" across sessions or container restarts
- You're discussing or implementing **abstraction compression** (selective forgetting, hierarchical summarization) on top of an existing LLM
- You need to engineer a persistent memory layer that survives Docker container deletion
- You're evaluating research approaches to LLM context compression (gist tokens, auto-compressors, KV cache pruning, MemGPT, etc.)

## Core concept: abstraction compression

The key insight is that **human memory is not exact storage** — it's a process of:
1. **Abstraction**: extracting the structural pattern from an experience, discarding surface details
2. **Selective forgetting**: actively suppressing what doesn't matter, making the important patterns more salient
3. **Reconstructive retrieval**: not replaying exact data, but regenerating it from the compressed structure

This is distinct from current LLM memory approaches (flat text injection, full-text session search). The goal is to make the agent **remember the shape of things, not the things themselves**.

## Architectural pattern: tiered memory

When building a memory layer on top of an existing LLM (without modifying the model), use three tiers:

### Tier 1: Working memory (context window)
The current conversation. Limited, ephemeral, but fast. No changes needed — this is the LLM's native capability.

### Tier 2: Abstracted long-term memory
Storage in an external database (SQLite, file-based), organized by:
- **abstraction_level** (1=concrete/specific → 3=highly abstracted)
- **concepts** (tags for cross-reference)
- **importance** (decay metric for selective forgetting)
- **access_count + last_accessed** (for LRU-like eviction)

Retrieval order: **high abstraction first → drill down → full-text only as fallback**

### Tier 3: Concept graph
A separate table/index that maps relationships between concepts. Enables analogical reasoning — when the agent encounters something new, it can find structurally similar past experiences.

## Philosophical foundation: why abstraction compression exists

This architecture is not arbitrary. It follows from a specific claim about the nature of intelligence and creativity:

> **Human creativity is not a mysterious "mathematical beauty" faculty. It's the same pattern-matching engine as AI, but running on a memory architecture that naturally performs abstraction compression.**

The human brain achieves this through:
- **Selective forgetting**: forgetting detail IS the mechanism by which structure emerges. The brain doesn't try to retain everything and then summarize — it *cannot* retain detail, and that constraint forces abstraction.
- **Reconstructive retrieval**: memory is not replayed faithfully, but regenerated from compressed schemas. Each recall modifies the memory itself.
- **Structured randomness**: neural noise is not error — it's a search strategy that kicks the system out of local optima.
- **Embodied feedback**: emotional/physiological signals (Damasio's somatic markers) provide a valuation function for "what matters" in a high-dimensional search space.

Current LLMs lack ALL four of these. The engineering challenge is to approximate them through external infrastructure without modifying model weights.

This perspective shapes every design decision in the system below.

## Engineering considerations for container persistence

### NEVER speculate about your deployment config — CHECK it
This is the most important lesson. Before making claims about what survives a container restart:
1. Read the actual `docker-compose.yml` to see volume mounts
2. Inspect the mounted paths (`/opt/data/` structure)
3. Verify where the specific data lives (cron definitions, memory files, config)

### Key persistence rules

| Component | Survives container delete? | Why |
|---|---|---|
| Memory tool data (MEMORY.md, USER.md) | ✅ If `/opt/data/memories/` is on a volume | File-based, on mounted path |
| Cronjob definitions | ✅ If `/opt/data/cron/` is on a volume | Hermes scheduler re-reads at startup |
| Cronjob scripts | ✅ If stored on the volume | Script files persist |
| Cronjob execution | ✅ Hermes scheduler restores | Loads from persisted cron/ directory on boot |
| In-process state | ❌ | Lost with container |

### Data you create yourself (SQLite DBs, scripts, AGENTS.md)
Always place in the mounted volume — e.g. `/opt/data/workspace/hermes-brain/`. This survives container deletion.

### Multiple redundancy layers
Don't rely on the agent "remembering to call the pipeline." Build infrastructure:
1. **Cronjob** (persisted in volume, auto-restored on boot) for periodic consolidation
2. **AGENTS.md** on the volume — Hermes Agent reads this on every fresh start. This is the most critical layer: it works across container restarts WITHOUT depending on memory tool injection
3. **Memory tool** entry pointing to the database — injected every session as a fallback reminder

Any one layer failing, the others catch it.

### AGENTS.md as cross-instance instruction
The AGENTS.md file placed on the volume serves as **instructions from one agent instance to the next**. It answers:
- "What memory system exists?"
- "Where is the database?"
- "What must you do at session start?"
- "What must you do at session end?"

Write it so a future agent that has **zero memory of this conversation** can pick up and use the system correctly. This is fundamentally different from memory tool injection (which tells the agent that something exists) — AGENTS.md tells the agent **how to operate** the system.
### Cronjob script path constraint
### Cronjob persistence across container recreation

Hermes' cron scheduler reads from disk on EVERY tick (every 60 seconds):

```
tick() → get_due_jobs() → load_jobs() → reads ~/.hermes/cron/jobs.json from disk
```

This means:
- Jobs that have already run (state="completed" or repeat completed) will NOT fire again
- Jobs that were scheduled but haven't run yet WILL be restored
- Recurring jobs with pending schedules WILL resume
- **Fast-forward rule**: if a job was scheduled during a period the container was down, and more than one period has elapsed, the scheduler fast-forwards to the next future run rather than firing a burst of backlogged executions

This is confirmed by reading the scheduler source code at `/opt/hermes/cron/jobs.py` line 993-999.

## Practical implementation steps

### Step 1: Initialize
```python
# SQLite schema for abstraction-compression memory
CREATE TABLE memories (
    id TEXT PRIMARY KEY,
    source TEXT,              # e.g. "session_2026-05-31"
    raw_summary TEXT,         # brief summary of original content
    abstract TEXT,            # compressed abstract structure
    abstraction_level INTEGER DEFAULT 1,  # 1=concrete, 2=pattern, 3=meta
    concepts TEXT,            # JSON array of related concept names
    access_count INTEGER DEFAULT 0,
    importance REAL DEFAULT 0.5,
    embedding BLOB,           # pickle'd float vector (512d) for semantic search
    created_at TEXT,
    last_accessed TEXT,
    is_active INTEGER DEFAULT 1  # 0 = forgotten/archived
);

CREATE TABLE concepts (
    name TEXT PRIMARY KEY,
    related_concepts TEXT,    # JSON array
    abstraction TEXT,         # abstract definition of this concept
    last_updated TEXT
);
```

### Step 2: Consolidation pipeline (run via cronjob)
Periodically (every 30m or daily):
1. **Decay importance**: reduce `importance` of all memories by a daily rate (e.g. 5%/day)
2. **Mark forgotten**: memories below a threshold (e.g. 0.15) AND not accessed for N days → `is_active = 0`
3. **Detect consolidation candidates**: find memories sharing the same concepts with 3+ entries, generate a work order for LLM to merge them into a higher-abstraction entry
4. **Update access stats**: log active/forgotten/concept counts for monitoring

The pipeline should be purely mechanical (no LLM calls needed). The LLM-intensive work (actual abstraction generation) can be triggered separately — either by the agent at session start, or by a separate cronjob with `no_agent=False`.

```python
# Core SQL for importance decay
IMPORTANCE_DECAY_PER_DAY = 0.05
cur.execute("""
    UPDATE memories
    SET importance = MAX(0.01, importance - ?)
    WHERE is_active = 1
""", (IMPORTANCE_DECAY_PER_DAY,))

# Core SQL for forgetting
cutoff = (datetime.now() - timedelta(days=30)).isoformat()
cur.execute("""
    UPDATE memories
    SET is_active = 0
    WHERE is_active = 1
      AND importance < 0.15
      AND last_accessed < ?
""", (cutoff,))
```
### Step 3: Three-mode selective retrieval

On each new interaction, use the retrieval script (`scripts/query_memory.py`) which supports three modes:

#### Mode A: Semantic embedding + concept diffusion + temperature sampling (primary)

```bash
source /opt/data/.venv/bin/activate
python3 scripts/query_memory.py --query "your question" --temp 0.3 --top-k 3
```

Algorithm:
1. **Embedding encoding**: encode the query into a 512-dim vector (via fastembed / bge-small-zh). Compute cosine similarity against all active memories.
2. **Concept graph diffusion**: from the Top-1 memory, traverse the concept graph. Memories sharing concepts with Top-1 get +0.30 bonus; 1-hop indirect links get +0.15. This surfaces structurally related memories with different wording.
3. **Segmented selection**: the first `top_k-1` results are pure similarity + concept diffusion (deterministic). The **last result** is selected from positions 3-5 in the candidate pool using a **temperature-sampled random walk** — noise is added to the fusion score to occasionally pull in a "岔路" (serendipitous) memory that wouldn't make the deterministic cut.

This is the recommended mode. It combines reliable factual recall with creative analogy discovery.

#### Mode B: Concept-based keyword matching (fallback)

```bash
python3 scripts/query_memory.py --keyword "关键词"
```

#### Mode C: List all active memories

```bash
python3 scripts/query_memory.py --query-all
```

Full implementation details in `references/embedding-retrieval-setup.md`.

### Step 4: Session-end storage (agent action — OPTIONAL)

**Only do this when signal #5 from the triggers fired** — a counter-intuitive insight worth preserving emerged. Otherwise skip it. Most sessions don't need it.

If you do store:

1. **Built-in memory** (`memory` tool) — if there are durable facts (preferences, environment details, lessons learned), save them here. This is usually sufficient for most sessions.
2. **Hermes Brain** (optional) — only if the insight is truly worth semantic retrieval later:
   - `raw_summary`: 1-2 sentence concrete summary
   - `abstract`: the structural pattern / key insight (2-3 sentences)
3. Store with appropriate importance (0.9+ for user's core claims, 0.3-0.5 for process details)
4. Update the concept graph if new concepts emerged

This CANNOT be automated — it requires the LLM's understanding of what was important. But you are NOT required to do it at every session end. Use judgment.

## Research landscape

The references/ directory contains detailed summaries of key approaches:

- **LLMLingua/LongLLMLingua** (Microsoft, 2024) — prompt-level token pruning
- **Gist Token** (Mu et al., 2023) — single-embedding context compression
- **AutoCompressor / ICAE** — encoder-decoder compression via the LLM itself
- **MemGPT / Letta** — OS-like virtual memory management
- **H₂O** (2023) — KV cache heavy-hitter retention
- **StreamingLLM** (2024) — anchor-token retention pattern
- **DRAGIN** (2024) — triggered retrieval based on model uncertainty

## Reference files

| File | Contents |
|------|----------|
| `references/context-compression-research.md` | Survey of academic context compression methods |
| `references/hermes-brain-implementation.md` | Initial Hermes Brain build (SQLite schema, cron setup, AGENTS.md protocol, container persistence) |
| `references/embedding-retrieval-setup.md` | Embedding + concept diffusion + temperature sampling iteration (fastembed, 512d vectors, three-mode retrieval, segmented selection) |
| `references/hermes-brain-cost-analysis.md` | Post-mortem: why Hermes Brain was downgraded from mandatory to occasional-use, cost/benefit analysis, lessons for future custom memory builders |
| `templates/AGENTS-template.md` | Template for cross-instance agent instructions |
| `scripts/compress_pipeline.py` | Pipeline: decay, forgetting, consolidation, auto-embedding |
| `scripts/query_memory.py` | Three-mode retrieval: semantic, keyword, list-all |

## Common pitfalls

### Claiming persistence without checking
Always verify volume mounts in the actual docker-compose.yml before telling the user what survives a container delete.

### Premature instruction encoding (prefer factual over imperative)

When storing memory system usage rules, **factual entries outlast instruction entries**. A factual description of what exists allows the LLM to flexibly infer the right action across different phrasings. An instruction entry ("when X happens, do Y") is brittle — the LLM must perfectly match the condition.

**Example of the principle:**

```
✗ INSTRUCTION (brittle):
  "用户会用'整理我们的两个记忆系统'作为精确触发词。听到这句话必须做两件事..."

✓ FACTUAL (resilient):
  "两个记忆系统：内置memory + Hermes Brain(/opt/data/workspace/hermes-brain/)"
```

The factual version occupies ~50 tokens vs ~200 tokens, triggers naturally on any variant of "整理记忆/存储/保存/两个系统", and is robust to the user phrasing it differently next session. The instruction version requires exact lexical matching and forces the LLM into a rigid if-then pattern that fails at the edges.

**When to use factual vs instruction:**
- For **knowledge that exists** (things, locations, count): always factual
- For **actions to perform** (specific sequences, security rules): instruction is appropriate, but consider documenting the full procedure in AGENTS.md or a SKILL.md and only putting the pointer in the user profile

### Over-investing in custom memory infrastructure

The Hermes Brain system itself went through a real-world cost/benefit evaluation that led to downgrading it from mandatory to occasional-use. The lesson:

**What built-in memory + skills already cover well:**
- User preferences, environment facts, tool conventions → `memory` (auto-injected, zero overhead)
- Complex workflows, exact commands, pitfalls → `skills` (full MD files, on-demand load)

**What custom memory adds (but at a cost):**
- Semantic retrieval (find conceptually related memories without keyword overlap)
- Concept graph + diffusion (analogical discovery)
- Forgetting/decay (important info doesn't stay at equal weight forever)

**The cost:** Manual query script every session, separate maintenance pipeline, cronjob upkeep, embedding model dependencies. If these aren't actively used, the system becomes dead weight.

**Rule of thumb:** Before building a custom memory system, ask: "Will the built-in memory tool and skills cover 90% of this?" If yes, invest in keeping those two pristine rather than building a third system. Custom memory is for the remaining 10% — cross-session semantic discovery and concept association that the flat key-value memory can't provide.

See `references/hermes-brain-cost-analysis.md` for the full decision record.

### Designing the pipeline to depend on agent "remembering"
The consolidation pipeline must be infrastructure (cronjob, file watcher), not agent behavior. Agents are stateless between container starts.

### Over-compression
Compressing too aggressively loses information needed for detailed reasoning. Use abstraction levels: always keep a path from abstract back to concrete when needed.

### Flat memory storage
Don't just dump all memories into a single injection string in the system prompt. This defeats the purpose — it's no different from the basic memory tool. The value is in **hierarchical retrieval**, not flat storage.

### Separating code from data
All scripts, databases, and configuration for the memory system must live on the volume, not in the container image. If you place any component in the container's writable layer (outside `/opt/data/`), it's lost on container delete. The only things that should live in `~/.hermes/scripts/` are thin wrappers that bridge to volume-stored code.

### Time-blind importance decay

The naive implementation subtracts `IMPORTANCE_DECAY_PER_DAY` EVERY run regardless of actual elapsed time. If the pipeline runs every 30 minutes, all memories collapse to floor in under 10 hours.

**Fix**: Store `last_decay_at` in config table, compute `elapsed_days = (now - last_decay).total_seconds() / 86400`, and decay by `IMPORTANCE_DECAY_PER_DAY * elapsed_days`. First run records baseline and skips decay.

### No reinforcement mechanism

Without a mechanism to INCREASE importance when a memory is used, the system only forgets — it never strengthens frequently-used memories. This creates a one-way degradation rather than a dynamic equilibrium.

**Fix**: Add `reinforce_memory(memory_id, boost=0.01)` that increments `access_count`, updates `last_accessed`, and adds `boost` to `importance` (capped at 1.0). Call this from `query_memory.py` on matched queries and from the agent whenever it references a memory in a response. `query_all_active()` (browsing mode) should NOT reinforce.

The equilibrium: +0.01 per use vs -0.10 per day means a memory needs to be used ~1 per 2.5 days to maintain importance. For users who only run containers occasionally, this prevents in-session over-boosting while still letting frequently-referenced memories survive. (See `references/embedding-retrieval-setup.md` for the complete parameter rationale.)

### Skip the AGENTS.md layer
The memory tool entry is not enough. A memory entry just says "something exists." AGENTS.md says "how to use it." Both are needed — the memory entry points to AGENTS.md, AGENTS.md provides the full protocol.

### Embedding model import in execute_code sandbox

`execute_code` uses the system Python, NOT the persistent venv Python. If you install fastembed in the venv, you cannot import it inside `execute_code` — you must run through `terminal()` with `source /opt/data/.venv/bin/activate`.

```python
# WRONG — execute_code uses system python:
# from fastembed import TextEmbedding  → ModuleNotFoundError

# RIGHT — run through terminal:
terminal("source /opt/data/.venv/bin/activate && python3 -c \"from fastembed import TextEmbedding; ...\"")
```

### Numpy array truthiness on pickle.load

`pickle.loads(embedding_blob)` returns a numpy array. You CANNOT use `if emb:` to check for existence — numpy arrays raise `ValueError` when evaluated as booleans. Always use `if emb is not None:`.

## Related skills

- `hermes-s6-container-supervision` — understanding the Docker container lifecycle
- `hermes-agent` — basic agent configuration (protected/bundled, cannot edit)
