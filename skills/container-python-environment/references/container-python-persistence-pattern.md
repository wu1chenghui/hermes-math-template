# Container Python Persistence Pattern — Reference

## Environment at time of writing

- **Container host**: Docker (Desktop) running under WSL2
- **Root filesystem**: overlay (ephemeral)
- **Persistent volume**: `/opt/data/` mapped to Windows `C:\` via 9p protocol
- **Python**: 3.13.5 (system-installed in container image)
- **Package manager**: `uv` installed (no `pip` due to PEP 668)
- **User home**: likely on persistent volume or overlay — check with `df -hT ~`

## Verbatim setup commands that worked

```bash
# Create persistent venv on volume
uv venv /opt/data/.venv

# Activate
source /opt/data/.venv/bin/activate

# Verify
which python                    # → /opt/data/.venv/bin/python
python --version                # → Python 3.13.5

# Install packages (works despite PEP 668 because we're in a venv)
uv pip install cowsay

# Test import
python -c "import cowsay; print(cowsay.get_output_string('cow', 'test'))"
```

## Verification output

```
/opt/data/.venv/bin/python
Python 3.13.5

# Site-packages location (on persistent volume):
/opt/data/.venv/lib/python3.13/site-packages/

# Installed package files:
/opt/data/.venv/lib/python3.13/site-packages/cowsay/
/opt/data/.venv/lib/python3.13/site-packages/cowsay-6.1.dist-info/
```

## Detection commands used

```bash
# Container check
ls /.dockerenv                  # exists
cat /proc/1/cgroup              # shows 0::/

# Filesystem type
df -hT /opt/data                # 9p (volume mount via drvfs)
df -hT /                        # overlay (ephemeral)

# Mount info
mount | grep -E 'opt/data|overlay'
```

## Key insight for this setup

The venv `bin/python` is a symlink to `/usr/bin/python3`. Each new container from the same image has Python at the same path, so the symlink remains valid. The venv does NOT hardcode a hash of the Python binary — it just references the path — so minor patch version updates (3.13.1 → 3.13.5) are transparent.

The only breaking change is a Python minor version bump (3.13 → 3.14) in the container image, because the venv's `lib/python3.13/` path is hardcoded.
