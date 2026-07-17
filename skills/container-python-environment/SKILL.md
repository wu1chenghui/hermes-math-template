---
name: container-python-environment
description: Set up and manage Python virtual environments and packages inside a Docker container where the root filesystem is ephemeral and only a persistent volume mount survives container recreation.
version: 1.0.0
author: Hermes Agent
metadata:
  hermes:
    tags: [docker, python, venv, uv, persistence, setup]
    related_skills: [agent-memory-architecture, hermes-s6-container-supervision]
---

# Container Python Environment

## When to use this skill

Load this skill when:
- The user asks to install Python packages and you're running inside a Docker container
- You need packages to survive container deletion and recreation
- The user wants a persistent development environment in a container
- You're discussing or debugging why pip installs don't survive a container restart
- PEP 668 (externally-managed-environment) blocks direct pip install

## Core insight: two filesystem layers

Docker containers have two distinct storage layers:

| Layer | Typical location | Survives container delete? |
|-------|-----------------|---------------------------|
| **Overlay rootfs** | `/` (system packages, /usr/lib/) | ❌ Gone |
| **Persistent volume** | `/opt/data/` (or wherever mounted) | ✅ Retained |

Python packages installed via `pip` (without a venv) go to system `site-packages` under `/usr/local/lib/python3.x/dist-packages/` — which is ON the ephemeral overlay. Container delete = packages lost.

**The fix**: create the virtual environment **on the persistent volume**, install packages there.

## How to detect the environment

Before making any claims about persistence, **check the facts**:

```bash
# 1. Are we in Docker?
ls /.dockerenv 2>/dev/null && echo "In Docker" || echo "Not in Docker"
cat /proc/1/cgroup 2>/dev/null | head -3

# 2. Find persistent volume mounts
df -hT /opt/data 2>/dev/null   # Look for 9p, ext4, or nfs — NOT overlay

# 3. Where does pip install currently go?
python3 -c "import site; print(site.getsitepackages())"

# 4. Check for existing venvs
ls /opt/data/.venv/bin/python 2>/dev/null && echo "venv exists" || echo "no venv"

# 5. Check available tools
which uv 2>/dev/null && uv --version || echo "no uv"
which pip3 2>/dev/null && pip3 --version || echo "no pip"
```

## How to create a persistent venv

### Step 1: Choose your tool

Check what's available:
- **`uv` (recommended)** — fast, self-contained, single binary. Works even when PEP 668 blocks pip.
- **`python3 -m venv`** — standard library, always available.
- **`pip`** — may be blocked by PEP 668 (externally-managed-environment error).

### Step 2: Create the venv on the volume

```bash
# With uv (preferred when available)
uv venv /opt/data/.venv

# With stdlib venv
python3 -m venv /opt/data/.venv
```

The venv directory must be inside the persistent volume path (e.g. `/opt/data/`), NOT in the home directory (`~`) unless home is on the volume.

### Step 3: Activate and install

```bash
source /opt/data/.venv/bin/activate

# Verify the Python interpreter is the venv one
which python
# Expected: /opt/data/.venv/bin/python

# Install packages
uv pip install <package1> <package2>
# OR: pip install <package1> <package2>
```

### Step 4: Verify persistence

```bash
# Confirm packages are on the volume
ls /opt/data/.venv/lib/python3.*/site-packages/ | grep <package>

# Test an import
source /opt/data/.venv/bin/activate && python -c "import <package>; print('OK')"
```

## How it works across container recreation

The venv directory (`/opt/data/.venv/`) lives on the persistent volume, which is mapped to the host filesystem (e.g. Windows C:\ via 9p).

```
Container 1                    Container 2
┌───────────────┐             ┌───────────────┐
│ python3 (new) │             │ python3 (new) │
│ /usr/bin/     │             │ /usr/bin/     │
└───────┬───────┘             └───────┬───────┘
        │ symlink (created once)      │ symlink (same target path)
        ▼                             ▼
┌───────────────────────────────┐
│ /opt/data/.venv/              │ ← PERSISTENT VOLUME
│  ├─ bin/python  → /usr/bin/python3
│  ├─ bin/activate
│  └─ lib/python3.13/site-packages/
│       ├─ numpy/               │ ← Still here after recreation
│       ├─ requests/            │
│       └─ ...                  │
└───────────────────────────────┘
```

The `bin/python` symlink points to the **same path** (`/usr/bin/python3`) which exists in every fresh container built from the same image. So the venv just works — `source activate` and all packages are available.

## When the venv breaks

The one scenario where the venv needs recreation: **the container image is updated to a different Python MAJOR or MINOR version.**

- Python 3.13 → Python 3.14: venv is tied to 3.13's ABI. All installed C extensions are incompatible.
- Python 3.13.1 → Python 3.13.5: **same minor version** — symlink still resolves, packages work fine.

To check the relationship:

```bash
# What Python does the venv expect?
head -1 /opt/data/.venv/bin/python   # shows the shebang path

# What Python does the new container have?
python3 --version
```

If the minor versions match, no action needed. If not, delete and recreate the venv.

## Common pitfalls

### Activating without telling future sessions

The venv activation is per-shell, not automatic. Each new Hermes session that needs the packages must:
1. Know the venv exists
2. Run `source /opt/data/.venv/bin/activate`

**Save this to memory** so future you knows about it.

### PEP 668 blocks pip directly

If you see `error: externally-managed-environment`, it means the OS (e.g. Debian/Ubuntu) explicitly blocks system-wide pip installs. Don't fight it — use `uv` (which ignores PEP 668) or create a venv and use pip inside it.

### Wrong Python in venv bin

After container recreation, `ls -la /opt/data/.venv/bin/python` should show a symlink to `/usr/bin/python3` (or wherever the system Python is in the new container). If it points to a path that no longer exists, the venv is broken — recreate it.

### Home directory on overlay

If the user's `~` is on the overlay layer (common in Docker), creating the venv at `~/.venv/` loses it on container delete. Always use the persistent volume path.

### Forgetting to activate

`source /opt/data/.venv/bin/activate` must be sourced (not executed as a script). `bash /opt/data/.venv/bin/activate` spawns a subshell and does NOTHING to the parent shell. Always use `source` or `.`.

## Related: Docker data persistence (beyond Python)

The same "what survives container rebuilds" principle applies to **all Docker services**, not just Python venvs. See `references/docker-data-persistence-patterns.md` for:

- **Three durability tiers**: host bind mount vs named volume vs container layer
- **Windows path format**: why `D:\\path` breaks in docker-compose (YAML escape), use `D:/path`
- **Mount overlap**: why mounting a subdirectory under an already-mounted parent is confusing
- **Bind mount prerequisites**: some images crash if the target directory doesn't exist at startup
- **Choice guide**: when to use named volumes vs bind mounts
- **Directory layout convention**: `D:\\docker-data\\<service>\\` for all persistent service data

### Hermes Agent Docker upgrade

See `references/hermes-docker-upgrade.md` for upgrading Hermes Agent running in Docker:

- **Two upgrade paths**: Docker Compose vs docker run — exact commands for each
- **Switching from `build: .` to `image:`**: what changes, what doesn't
- **Verified data survival table**: every file and directory under `/opt/data/` that survives a container recreate
- **Config migration automatic behavior**: what runs on first boot after upgrade
- **Source verification**: how to check the local checkout hasn't been patched (build SHA comparison)
- **Known issue #4426 / fix #7357**: per-profile HOME persistence for system tool configs (git, ssh, npm, etc.)
- **All authoritative sources linked**: official docs, GitHub issues, PRs

### Lean 4 installation in the container

See `references/lean-4-container-installation.md` for installing Lean 4 (elan + lean compiler) in the same container environment. Covers:
- TLS workaround for `releases.lean-lang.org` (download from GitHub Releases)
- zstd extraction via Python (no `zstd` binary available, no root)
- Symlink-to-/tmp vs persistent copy strategies for the 2.6 GB `.olean` libraries
- 9p filesystem performance considerations

## Verification checklist

After setting up a persistent venv:

- [ ] `.dockerenv` exists (we're in a container)
- [ ] `/opt/data/` is on a non-overlay filesystem (volume mount)
- [ ] `source /opt/data/.venv/bin/activate` works
- [ ] `which python` returns the venv path
- [ ] Installed packages are importable
- [ ] Package files physically reside under `/opt/data/.venv/lib/`
- [ ] Memory has been updated so future sessions know about the venv
