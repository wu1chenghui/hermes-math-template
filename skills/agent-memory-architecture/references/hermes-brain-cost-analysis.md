# Hermes Brain — Real-World Cost/Benefit Analysis

## Context

Hermes Brain was a custom external memory system built on top of Hermes Agent's built-in capabilities. It used:
- SQLite database with 3-level abstraction hierarchy (L1 concrete → L3 meta)
- Embedding-based semantic retrieval (fastembed / bge-small-zh-v1.5, 512-dim vectors)
- Concept graph with diffusion scoring (+0.30 direct, +0.15 one-hop)
- Temperature-sampled last-slot selection for serendipitous recall
- Importance decay + forgetting pipeline (scheduled every 30 min)
- Separate AGENTS.md file on the persistence volume for cross-instance instructions

## Original design intent

The user's core insight motivating the system: **AI's large context windows are a crutch, not a strength** — they let the model "remember and pattern-match" without needing to abstract. Human working memory (4±1 chunks) forces abstraction. A real learning agent needs:
1. Abstraction (extracting structure from detail)
2. Selective forgetting (making important patterns salient)
3. Reconstructive retrieval (regenerating from compressed structure, not replaying exact data)

Hermes Brain was an attempt to engineer these properties externally.

## What prompted the downgrade

After using the system for a period, the user revisited the question: **"Is this custom system actually necessary?"** The built-in memory tool + skills already handled most day-to-day needs:

| Need | Built-in solution | Custom solution |
|------|------------------|-----------------|
| User preferences | `memory` (auto-injected every session) | Hermes Brain query |
| Environment facts | `memory` (auto-injected) | — |
| Tool conventions | `memory` (auto-injected) | — |
| Complex workflows | `skills` (full MD, on-demand `skill_view`) | — |
| Exact commands + pitfalls | `skills` with code blocks | — |
| Cross-session concept recall | `session_search` (FTS5) | Hermes Brain semantic retrieval |
| Analogical discovery | None | Concept graph + diffusion + temperature |
| Forgetting old info | `memory` has no decay | Importance decay pipeline |

The only **truly unique** value Hermes Brain provided (not covered by built-in tools):
1. **Semantic embedding retrieval** — find conceptually related memories without shared keywords
2. **Concept graph diffusion** — one-hop association discovery
3. **Temperature-sampled serendipity** — deliberately pulling in "岔路" memories

These are valuable, but the cost of maintaining a separate query pipeline, cronjob, embedding model, and AGENTS.md workflow was high relative to how often they were actually used.

## Decision

**Downgrade from mandatory (every session) to occasional (on-demand).** The system is retained (database, code, scripts) but no longer required at session start or end. The agent uses judgment — only queries Hermes Brain when a clear signal fires (concept association need, cross-session recall, counter-intuitive insight to preserve).

See current `AGENTS.md` at `/opt/data/workspace/hermes-brain/AGENTS.md` for the full occasional-use protocol.

## Lessons for future custom memory builders

1. **Check what built-in tools already do.** The `memory` tool + `skills` + `session_search` cover a lot of ground. Don't build a third system before exhausting what's already there.
2. **Maintenance cost is real.** Embedding models, cronjobs, pipelines, AGENTS.md protocols — all require upkeep. If you don't actively use the system, it decays into dead weight.
3. **Semantic retrieval is the one genuinely hard problem.** Built-in memory is flat key-value. Built-in `session_search` is FTS5 (keyword matching, no embedding). If you need "find things that mean the same but use different words," that's where custom embedding-based retrieval earns its keep.
4. **Temperature-sampled serendipity is an under-explored pattern.** Deterministic top-k retrieval always returns the safest matches. Injecting noise into the last slot (or using a "探索 vs 利用" split) is a novel idea worth preserving for any future system.
5. **Don't put the operation protocol in the agent's memory.** Put it in a file on the persistent volume (AGENTS.md) so it survives container recreation and isn't lost when memory entries age out. This pattern worked well — the AGENTS.md survived every container rebuild.

## Files preserved (read-only, no longer maintained)

- `/opt/data/workspace/hermes-brain/memory.db` — SQLite database with existing memories
- `/opt/data/workspace/hermes-brain/scripts/query_memory.py` — three-mode retrieval script
- `/opt/data/workspace/hermes-brain/AGENTS.md` — updated occasional-use protocol
