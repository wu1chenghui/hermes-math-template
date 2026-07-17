# Docker Data Persistence Patterns — Reference

## Core concept: three durability tiers

| Tier | Mechanism | Survives `docker compose down`? | Survives `docker compose down -v`? | Survives Docker Desktop reinstall? |
|------|-----------|-------------------------------|----------------------------------|-----------------------------------|
| 1 | **Host bind mount** (`D:/path:/container/path`) | ✅ Yes | ✅ Yes | ✅ Yes (on host filesystem) |
| 2 | **Named volume** (`volumename:/container/path`) | ✅ Yes | ❌ Deleted | ❌ Lost |
| 3 | **Container layer** (inside overlay fs) | ❌ Deleted | ❌ Deleted | ❌ Lost |

**Key rule**: For anything you never want to lose, use a bind mount to a host path. Named volumes are convenient but can be wiped with `-v`.

## Windows path format in docker-compose

```yaml
# ✅ CORRECT — use forward slashes
volumes:
  - D:/docker-data/searxng/config/:/etc/searxng/

# ✅ ALSO CORRECT — use relative path (relative to compose file location)
volumes:
  - ./docker-data/searxng/config/:/etc/searxng/

# ❌ WRONG — backslash is YAML escape character
volumes:
  - D:\docker-data\searxng\config\:/etc/searxng/     # ← BUG: \t is tab, \s is space, etc.
```

**Remember**: `\` in YAML is an escape character. `D:\tools` → YAML sees `D:` followed by `\t` (tab). Always use `/` in docker-compose paths, even on Windows. Docker Desktop converts them correctly.

## Mount overlap: don't double-mount the same tree

```yaml
services:
  app:
    volumes:
      - D:/:/workspace          # Mounts entire D: drive
      - D:/docker-data:/data    # ❌ REDUNDANT — D:/docker-data is already accessible at /workspace/docker-data/
```

When a parent directory is already mounted (`D:/` → `/workspace`), mounting a subdirectory separately (`D:/docker-data/` → `/data`) is technically harmless but creates confusion: the user now sees the same files at two different container paths. Only add a separate mount when:
- You need a shorter/cleaner path for that specific subdirectory
- The subdirectory is on a **different filesystem** than the parent mount

## Prerequisites: bind mount directories must exist

Many Docker images (e.g. SearXNG) **exit on startup** if the bind mount directory doesn't exist:

```bash
# Entrypoint pattern (simplified):
check_directory() {
    if [ ! -d "$1" ]; then
        exit 127   # Container exits immediately
    fi
}
```

**Action needed before `docker compose up -d`**:

```bash
# Create directories first
mkdir -p D:\docker-data\searxng\config
mkdir -p D:\docker-data\searxng\cache
```

Docker bind mounts **do** auto-create the host directory on first use, but this happens **during** container startup. Some images check prerequisites **before** Docker creates the mount, resulting in a crash loop. Always create target directories manually ahead of time.

## Named volumes vs bind mounts: choosing wisely

**Use named volumes when:**
- You only need persistence across `docker compose down` (not full system rebuild)
- You don't need to inspect/edit the files from the host
- The data is cache/temp that can be regenerated
- Example: Redis/Valkey data, search engine cache

**Use bind mounts when:**
- The data MUST survive anything (Docker reinstall, machine rebuild) 
- You need to edit config files from the host with a regular editor
- You want to back up the data with regular file-system tools
- Example: application config files, user-uploaded content, databases

## Typical directory layout

```
D:\docker-data\                 ← Container: /workspace/docker-data/
├── searxng\
│   ├── config\                 ← Bind mount: settings.yml (host-editable)
│   └── cache\                  ← Could be named volume (regeneratable)
├── app2\
│   ├── config\
│   └── data\
└── ...                         ← One subdirectory per service
```

This pattern keeps all service data in one place on the host, survives anything, and is easy to back up.
