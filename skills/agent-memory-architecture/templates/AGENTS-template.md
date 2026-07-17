# {{SKILL_NAME}} — Abstracted Memory System

This file guides Hermes Agent on how to use the persistent abstraction memory system.
All data lives on the Docker volume and survives container recreation.

## Location

```
{{BRAIN_ROOT}}/
├── memory.db                ← SQLite database
├── scripts/
│   ├── compress_pipeline.py ← periodic maintenance
│   └── query_memory.py      ← query helper
├── work_orders/
│   └── pending.json         ← consolidation tasks for the agent
└── logs/
    └── pipeline.log         ← pipeline audit log
```

## On every session start

### 1. Query relevant memories

Use the query helper or raw SQL to find memories matching the current topic:

```python
import sqlite3, json
conn = sqlite3.connect("{{BRAIN_ROOT}}/memory.db")
conn.row_factory = sqlite3.Row

# Extract key concepts from the user's current question
concepts = ["concept_1", "concept_2"]  # ← YOU fill this from the question

# Query by concept matching
placeholders = ",".join("?" for _ in concepts)
cur = conn.cursor()
cur.execute(f"""
    SELECT DISTINCT m.id, m.abstract, m.concepts, m.importance, m.abstraction_level
    FROM memories m
    WHERE m.is_active = 1
      AND (SELECT COUNT(*) FROM json_each(m.concepts) WHERE value IN ({placeholders})) > 0
    ORDER BY m.abstraction_level DESC, m.importance DESC
    LIMIT 10
""", concepts)

memories = [dict(r) for r in cur.fetchall()]
conn.close()
```

Use abstraction levels to guide retrieval depth:
- **L3 (meta)**: first — broad structural patterns
- **L2 (abstract)**: second — specific patterns and relationships
- **L1 (concrete)**: last — only when detail is needed
- **Full-text search**: final fallback if concept match finds nothing

### 2. Process pending work orders

Check `{{BRAIN_ROOT}}/work_orders/pending.json`. If it contains consolidation tasks, use your own summarization ability to merge the listed memories into a higher-abstraction entry, then remove the completed order.

### 3. Integrate retrieved memories into your response

Treat retrieved memories as context — they represent what was learned from past conversations. Weave them into your thinking naturally.

## On every session end

### 4. Compress and store this conversation

```python
import sqlite3, json, uuid
from datetime import datetime

memory_id = "mem_" + str(uuid.uuid4())[:8]
concepts = ["concept_a", "concept_b"]  # ← extracted from the conversation

conn = sqlite3.connect("{{BRAIN_ROOT}}/memory.db")
cur = conn.cursor()

cur.execute("""
    INSERT INTO memories (id, session_id, source, raw_summary, abstract,
                          abstraction_level, concepts, importance, created_at, last_accessed)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
""", (
    memory_id,
    "<session_id>",  # ← from session_search or current context
    "conversation",
    "<1-2 sentence concrete summary>",
    "<2-3 sentence structural abstraction>",
    2,  # abstraction level
    json.dumps(concepts, ensure_ascii=False),
    0.8,  # importance (0.9+ for core insights, 0.3-0.5 for details)
    datetime.now().isoformat(),
    datetime.now().isoformat()
))

# Update concept graph
for concept in concepts:
    cur.execute("""
        INSERT INTO concepts (name, related_concepts, abstraction, occurrences, last_updated)
        VALUES (?, NULL, NULL, 1, ?)
        ON CONFLICT(name) DO UPDATE SET
            occurrences = occurrences + 1,
            last_updated = excluded.last_updated
    """, (concept, datetime.now().isoformat()))

conn.commit()
conn.close()
```

## Importance guide

| Range | What to store at this level |
|-------|----------------------------|
| 0.9-1.0 | User's core claims, insights, anti-commonsense conclusions |
| 0.6-0.8 | Valuable discussions, important background |
| 0.3-0.5 | General information, procedural content |
| 0.1-0.2 | Details, examples, transient context |

## Database schema reminder

```sql
-- memories:
--   id TEXT PK, session_id TEXT, source TEXT,
--   raw_summary TEXT, abstract TEXT,
--   abstraction_level INT (1=concrete, 2=pattern, 3=meta),
--   concepts TEXT (JSON array), importance REAL (0-1),
--   access_count INT, created_at TEXT, last_accessed TEXT,
--   is_active INT (1=active, 0=forgotten)

-- concepts:
--   name TEXT PK, related_concepts TEXT,
--   abstraction TEXT, occurrences INT, last_updated TEXT
```
