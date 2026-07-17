---
name: development-methodologies
description: "Software engineering methodologies: plan mode, spikes, systematic debugging, TDD, and pre-commit code verification. Complete development workflow from exploration through delivery."
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [planning, spike, debugging, tdd, code-review, quality, methodology]
  note: "⚠️ External reference compilation — not fully verified in practice. Use as inspiration, not prescription."
    absorbed_from:
      - plan
      - spike
      - systematic-debugging
      - test-driven-development
      - requesting-code-review
---

# Development Methodologies

A complete software development workflow: explore (spike) → plan → implement (TDD) → debug → verify (code review). Each section is a labeled methodology; use them in sequence or independently depending on where you are in the cycle.

## 1. Spike — Throwaway Experiments

Validate feasibility before committing to a build. Spikes are disposable by design.

**When to use:** User says "let me try this", "see if X works", "spike this out", "quick prototype", "compare A vs B".

**Method:**
```
decompose → research → build → verdict
```

**Step 1 — Decompose:** Break into 2-5 independent feasibility questions, presented as Given/When/Then. Order by risk (hardest first).

**Step 2 — Research:** Before building, research enough to pick the right approach. Use `web_search`, `web_extract`, or read docs.

**Step 3 — Build:** One directory per spike (`spikes/NNN-description/`). Bias toward interactive output (CLI, HTML page, web server, unit test). Test edge cases, don't just run happy path.

**Step 4 — Verdict:** Each spike's README closes with:
```markdown
## Verdict: VALIDATED | PARTIAL | INVALIDATED
### What worked / What didn't / Surprises / Recommendation
```

**Comparison spikes:** Build N approaches back-to-back, then head-to-head comparison table.

## 2. Plan — Implementation Plans

Write actionable markdown plans when the user wants a plan instead of execution. Do not implement, do not edit project files except the plan.

**Save location:** `.hermes/plans/YYYY-MM-DD_HHMMSS-slug.md`

**Core principle:** A good plan makes implementation obvious.

**Task granularity:** Each task = 2-5 minutes of focused work. Include:
- Exact file paths
- Complete code (copy-pasteable)
- Exact commands with expected output
- Verification steps
- TDD cycle per task

**Structure per task:**
```markdown
### Task N: Descriptive Name
**Objective:** One sentence
**Files:** Create/Modify/Test paths
**Step 1:** Write failing test  [code]
**Step 2:** Run to verify fail  [command + expected output]
**Step 3:** Minimal implementation [code]
**Step 4:** Run to verify pass  [command + expected output]
**Step 5:** Commit
```

**Principles:** DRY, YAGNI, TDD, frequent commits after every task.

## 3. Test-Driven Development (TDD)

Write the test first. Watch it fail. Write minimal code to pass.

**The Iron Law:** `NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST`

**RED-GREEN-REFACTOR cycle:**
1. **RED** — Write one minimal failing test
2. **Verify RED** — Run it, confirm it fails for expected reason (feature missing, not typo)
3. **GREEN** — Write simplest code to pass. Cheating is OK (hardcode, copy-paste, skip edge cases)
4. **Verify GREEN** — Run the test to see it pass, then run ALL tests for regressions
5. **REFACTOR** — Clean up duplication. Keep tests green throughout

**If you didn't watch the test fail, you don't know if it tests the right thing.**

**Common rationalizations to reject:**
- "Too simple to test" → Simple code breaks. Test takes 30 seconds.
- "I'll test after" → Tests-after answer "what does this do?" not "what should this do?"
- "Already manually tested" → No record, can't re-run, easy to forget cases
- "Deleting X hours is wasteful" → Sunk cost fallacy. Keep unverified code = technical debt

## 4. Systematic Debugging

4-phase root cause debugging. Understand bugs before fixing.

**The Iron Law:** `NO FIXES WITHOUT ROOT CAUSE INVESTIGATION FIRST`

**Phase 1 — Root Cause Investigation:** Read errors carefully, reproduce consistently, check recent changes (`git log -10`, `git diff`), gather evidence, trace data flow upstream. **Fix at the source, not at the symptom.**

**Phase 2 — Pattern Analysis:** Find working examples, compare against references, identify every difference, understand dependencies.

**Phase 3 — Hypothesis and Testing:** Form a single hypothesis, test with the smallest possible change (one variable at a time), verify before continuing. Form a NEW hypothesis if it fails.

**Phase 4 — Implementation:** Create a failing regression test first (TDD), implement the single fix, verify, run full suite.

**The Rule of Three:** If 3+ fixes failed, STOP and question the architecture. Don't attempt Fix #4 without architectural discussion.

**Red flags to STOP:**
- "Quick fix for now, investigate later"
- "Just try changing X and see if it works"
- "Add multiple changes, run tests"
- Proposing solutions before tracing data flow

## 5. Pre-Commit Code Verification

Automated verification pipeline before code lands. No agent should verify its own work — fresh context finds what you miss.

**Step 1 — Get the diff:** `git diff --cached` (or `git diff HEAD~1 HEAD` if empty)

**Step 2 — Static security scan:** Check for hardcoded secrets, shell injection (`os.system`, `shell=True`), `eval()`/`exec()`, unsafe pickle, SQL injection in added lines.

**Step 3 — Baseline tests and linting:** Detect project framework (pytest, npm test, cargo test), run with baseline comparison (stash changes → run → pop → compare).

**Step 4 — Self-review checklist:** No hardcoded secrets, input validation, parameterized SQL, no debug prints, no commented-out code, new code has tests.

**Step 5 — Independent reviewer subagent:** Call `delegate_task` with the diff and static scan results. Reviewer gets ONLY the diff — no shared context. Fail-closed on parse errors.

**Step 6 — Evaluate:** Combine static scan + reviewer verdict. If all pass → commit. If failures → auto-fix loop.

**Step 7 — Auto-fix loop:** Max 2 fix-and-reverify cycles. Spawn a separate fix agent that touches ONLY the reported issues. If still failing after 2 attempts, escalate.

**Step 8 — Commit:** `git add -A && git commit -m "[verified] <description>"`

### Reference: `references/docker-upgrade-verification.md`

Pre-upgrade safety checklist for Docker image swaps: verify Dockerfile, packages, .env, config.yaml, network, env var compatibility, volume mounts, and source code.

## 6. Writing Implementation Plans (Plan Crafting)

Write plans assuming the implementer has zero codebase context and questionable taste. Document everything: exact file paths, complete code, testing commands, verification steps.

**Core principle:** A good plan makes implementation obvious. If someone has to guess, the plan is incomplete.

**Plan document header:**
```markdown
# Feature Implementation Plan
**Goal:** [one sentence]
**Architecture:** [2-3 sentences]
**Tech Stack:** [...]
```

**Task format (each = 2-5 min):**
```markdown
### Task N: [Name]
**Objective:** One sentence
**Files:** Create/Modify/Test paths
**Step 1:** Write failing test [code]
**Step 2:** Run to verify failure [command + expected output]
**Step 3:** Minimal implementation [code]
**Step 4:** Run to verify pass [command + expected output]
**Step 5:** Commit
```

Save plans to `.hermes/plans/YYYY-MM-DD_HHMMSS-slug.md`. After saving, offer execution via subagent-driven-development.

**Principles:** DRY, YAGNI, TDD, frequent commits. Never write vague tasks like "Add authentication" — write "Create User model with email and password_hash fields".

## 7. Subagent-Driven Development (Plan Execution)

Execute plans by dispatching fresh subagents per task with two-stage review.

**Core principle:** Fresh subagent per task + independent spec review then quality review = high quality, fast iteration.

### Per-Task Workflow

**Step 1 — Read the plan once:** Extract ALL task context upfront. Provide full text in delegate_task context — never make subagents read the plan file.

**Step 2 — Dispatch implementer:** Complete context including plan snippet, project context, TDD instructions, and commit instructions.

**Step 3 — Spec compliance review:** After implementer finishes, a fresh reviewer subagent checks: "All requirements met? File paths correct? Nothing extra added?" If issues → fix, re-review.

**Step 4 — Code quality review:** After spec passes, quality reviewer checks: conventions, error handling, naming, test coverage, security. If issues → fix, re-review.

**Step 5 — Mark complete, proceed to next task.**

### Integration with other methodologies
- Uses TDD cycle for each implementation subtask
- Spec review catches under/over-building early
- Quality review catches code issues before they compound
- Final integration review after all tasks complete

### References
- `references/context-budget-discipline.md` — Four-tier context degradation model (PEAK / GOOD / DEGRADING / POOR)
- `references/gates-taxonomy.md` — Four canonical gate types (Pre-flight, Revision, Escalation, Abort)
