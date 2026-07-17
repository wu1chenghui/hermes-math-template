# Hermes Brain — Reference Implementation

Built on 2026-05-31 during a discussion about mathematical creativity and AI memory architecture.
Location: `/opt/data/workspace/hermes-brain/` (on mounted Docker volume).

## Architecture

```
/opt/data/workspace/hermes-brain/
├── memory.db                ← SQLite database
├── AGENTS.md                ← cross-instance agent instructions
├── scripts/
│   ├── compress_pipeline.py ← periodic maintenance pipeline
│   └── query_memory.py      ← query helper for agents
├── logs/
│   └── pipeline.log         ← pipeline run log
├── work_orders/
│   └── pending.json         ← LLM-intensive tasks queued for agent

/opt/data/scripts/
└── hermes-brain-cron.py     ← thin wrapper (bridges to scripts/ on volume)
```

## Database Schema

### memories
- `id` TEXT PK — e.g. `mem_creativity_theory_20260531`
- `source` TEXT — `"conversation"`, `"web"`, etc.
- `raw_summary` TEXT — concrete 1-2 sentence summary of original content
- `abstract` TEXT — 2-3 sentence structural abstraction
- `abstraction_level` INT — 1=concrete details, 2=structural pattern, 3=meta/conceptual
- `concepts` TEXT — JSON array of concept tags for cross-reference
- `importance` REAL — 0.0-1.0, decays 5% per day
- `access_count` INT, `created_at` TEXT, `last_accessed` TEXT
- `is_active` INT — 1=active, 0=forgotten/archived

### concepts
- `name` TEXT PK — concept name
- `related_concepts` TEXT — comma-separated related names
- `occurrences` INT — count of how many times this concept has appeared
- `last_updated` TEXT

## Cronjob Configuration

```
schedule: every 30 minutes
script: hermes-brain-cron.py (relative path in ~/.hermes/scripts/)
no_agent: true
```

The cronjob does purely mechanical work (importance decay, forgetting, consolidation detection).
LLM-dependent work (abstraction generation) is queued as work orders for the agent.

## Initial Memory Entries

Three entries were stored as the initial seed, designed to survive container restart:

| ID | Focus | Importance | Concepts |
|---|---|---|---|
| `mem_creativity_theory_20260531` | Human creativity = pattern matching + abstraction compression + structured randomness + embodiment feedback | 0.95 | 数学创造力, 抽象化压缩, 选择性遗忘, 记忆架构 |
| `mem_hermes_brain_build_20260531` | External memory system architecture on Docker volume | 0.90 | 记忆系统工程, 外挂大脑, 抽象化压缩管线 |
| `mem_ai_math_20260531` | OpenAI + DeepMind AI math breakthroughs on May 21, 2026 | 0.85 | AI数学, DeepMind, AlphaProof Nexus, 埃尔德什 |

## Key Design Decisions

### Why SQLite (not a vector DB)
- Zero dependencies, ships with Python stdlib
- JSON array in `concepts` column + `json_each()` provides adequate concept matching
- Full-text search available if needed (FTS5 extension)
- Single file — trivially backed up, copied, or archived

### Why importance decay + threshold forgetting (not recency-only)
- Recency-only (e.g. StreamingLLM) loses important structural insights in favor of trivial recent details
- Decay rate (5%/day) can be tuned per user or per domain
- The `FORGET_THRESHOLD` + `FORGET_DAYS_NO_ACCESS` dual condition prevents premature loss of rare-but-valuable memories

### Why two abstraction steps (agent + cronjob)
- LLM-based abstraction requires understanding of context and user intent — only the agent can do this well
- Mechanical maintenance (decay, forgetting, stats) is deterministic and doesn't need an LLM call
- Separating them keeps the cronjob fast and cheap

### Why AGENTS.md on the volume
- Memory tool entries are short and don't survive if `~/.hermes/` isn't on the volume
- AGENTS.md is a full document with code examples and protocols
- Fresh-start agents that have zero memory of the conversation can read AGENTS.md and operate the system correctly

## AGENTS.md Protocol

The AGENTS.md must answer four questions for a future agent:

1. **Where is the database?** — absolute path on the volume
2. **Query first, act second** — always retrieve relevant memories before responding
3. **Store at session end** — compress the conversation and write to the database
4. **Process work orders** — check `work_orders/pending.json` for consolidation tasks

## Status

- [x] Database schema + creation
- [x] AGENTS.md with full protocol
- [x] Compress pipeline (mechanical: decay, forgetting, consolidation detection)
- [x] Query helper script
- [x] Cronjob (30m interval, no LLM)
- [x] First 3 memory entries seeded
- [ ] Automatic abstraction at session end (requires agent action)
- [ ] Work order processing (requires agent action)
- [ ] Multi-level concept abstraction (L1→L2→L3 consolidation)

## Lessons Learned (from first implementation session)

### Cronjob repeat default

The first cronjob was created with `schedule="30m"` but **no repeat argument**. This produced a one-shot job with `schedule.kind = "once"` and `repeat.times = 1`. The job will fire once at the computed time and never again.

**Fix**: Re-create with explicit repeat:
```python
cronjob(action="create", schedule="30m", no_agent=True,
        script="hermes-brain-cron.py", repeat=0)  # 0 = unlimited
```

### Cronjob scheduler reads from disk every tick

Confirmed by reading `/opt/hermes/cron/scheduler.py` and `/opt/hermes/cron/jobs.py`:
- `tick()` calls `get_due_jobs()` → `load_jobs()` → reads `~/.hermes/cron/jobs.json`
- This happens **every 60 seconds**, so job definitions on the volume are always in sync
- **Implication**: cronjob definitions survive container recreation. As long as the volume is mounted, the scheduler will find and execute pending jobs.
- **Fast-forward behavior**: if the container was down for a period, missed job ticks are NOT retroactively executed. The scheduler fast-forwards to the next future schedule point.

### AGENTS.md is the most critical survival layer

The memory tool entry is a short pointer. AGENTS.md is the full protocol. Together they ensure that a fresh-start agent with zero memory of the conversation can:
1. Find the system (via memory entry)
2. Understand how to use it (via AGENTS.md)
3. Execute correctly

A future agent that only has the memory entry (no AGENTS.md) will know "something exists" but won't know the operating protocol.

### Docker volume verification

Always verify the actual docker-compose.yml before making claims about persistence. In this deployment, the only volume mount is `~/.hermes:/opt/data`. The Hermes Brain system lives under `/opt/data/workspace/`, which is on the volume and survives container deletion.
