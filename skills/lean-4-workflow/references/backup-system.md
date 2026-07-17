# Backup System

The `e/` project maintains backups under `/workspace/backups/`. Two kinds:

## Standalone AGENTS.md Backups

```
/workspace/backups/AGENTS_pre_update.md
/workspace/backups/AGENTS_post_update.md
```

Created immediately before and after AGENTS.md edits. Use these for quick
restore of the project documentation file alone.

## Full Project Snapshots (tar.gz)

Named `e_<stage>_<timestamp>/e_backup.tar.gz`. Each contains the entire
project tree at that point in time. Naming pattern:

```
e_pre_agents_20260630_140007/      # Before AGENTS.md update
e_post_agents_20260630_140043/     # After AGENTS.md update
e_pre_pubaudit_20260702_034342/    # Before publication audit
e_post_pubaudit_20260702_034436/   # After publication audit
e_final_baseline_20260701_153531/  # Final baseline — transition to writing
```

Key milestones have descriptive suffixes:
- `_D3_COMPLETE`, `_D4_clean_build_verified`, `_boundary_rigidity_proved`
- `_CHAR3_DISCOVERY`, `_v1.0-proof-complete`
- `_attack`, `_certificates`, `_constraint`, `_coverage_audit`

## Restore Procedure

To restore AGENTS.md from the most recent full snapshot:

```bash
cd /workspace/backups/e_pre_pubaudit_20260702_034342
tar xzf e_backup.tar.gz ./AGENTS.md
cp ./AGENTS.md /opt/lean-home/lean-projects/e/AGENTS.md
```

To restore the entire project from a snapshot (DANGER — overwrites current state):

```bash
cd /opt/lean-home/lean-projects
rm -rf e
mkdir e
cd e
tar xzf /workspace/backups/<snapshot_dir>/e_backup.tar.gz
```

## When to Create a Backup

Before any AGENTS.md edit, the user typically creates:
- `e_pre_<descriptor>_<timestamp>/` — full snapshot before edit
- `e_post_<descriptor>_<timestamp>/` — full snapshot after edit

The agent should NOT create backups autonomously. The agent should:
1. Propose edits and wait for confirmation
2. If a mistake is made, restore from the most recent backup
3. Know the backup directory path and naming conventions
