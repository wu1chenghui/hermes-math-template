# Lean 4 Container Installation (Persistent Volume)

## When to reference this

Install Lean 4 (elan + lean compiler) inside the Hermes container on WSL where:
- Root filesystem is ephemeral (overlay)
- `/opt/data` (C:\ via 9p) is the persistent volume
- `apt-get` requires root (unavailable)
- `releases.lean-lang.org` has TLS handshake failures

## Toolchain architecture

```
~/.elan/
  bin/                     → elan, lean, lake (symlinks to current toolchain)
  toolchains/
    leanprover--lean4---v4.X.0/
      bin/lean, bin/lake          (9 MB)
      lib/*.so, lib/lean/*.so     (shared libs, ~30 MB)
      lib/lean/Init/, Lean/, Std/, Lake/  (compiled .olean files, 2.6 GB)
```

## Key workaround: TLS failure on releases.lean-lang.org

The server at `releases.lean-lang.org` (195.201.80.226) does not complete TLS 1.3 handshakes from this container's OpenSSL. Fix: download from GitHub Releases directly:

```bash
curl -sL "https://github.com/leanprover/lean4/releases/download/v4.30.0/lean-4.30.0-linux.tar.zst" \
  -o /tmp/lean-4.30.0-linux.tar.zst
```

Save to persistent storage: `cp /tmp/lean-4.30.0-linux.tar.zst /opt/data/`

## Extract without zstd binary

Container lacks `zstd` and has no root. Use Python zstandard via persistent venv:

```bash
source /opt/data/.venv/bin/activate
uv pip install zstandard

python3 -c "
import zstandard, tarfile, os
SRC = '/tmp/lean-4.30.0-linux.tar.zst'
DST = '/tmp/lean-extract'
dctx = zstandard.ZstdDecompressor()
os.makedirs(DST, exist_ok=True)
with open(SRC, 'rb') as f:
    reader = dctx.stream_reader(f)
    tf = tarfile.open(fileobj=reader, mode='r|')
    tf.extractall(DST, filter='data')
"
```

## Place toolchain (two strategies)

### A: Symlink to /tmp (fast but ephemeral)

```bash
TCDIR=~/.elan/toolchains/leanprover--lean4---v4.30.0
rm -rf "$TCDIR"
ln -sf /tmp/lean-extract/lean-4.30.0-linux "$TCDIR"
```

On container restart: re-extract from `/opt/data/lean-4.30.0-linux.tar.zst` and re-symlink.

### B: Persistent copy (resilient but slow over 9p)

Copy bin/ (9 MB) and .so files first for quick verification, then copy the 2.6 GB .olean dirs in background.

## Minimum .olean directories needed

- `Init/` (387 MB) — core definitions
- `Lean/` (1.2 GB) — tactics, `simp`, `native_decide`
- `Std/` (317 MB) — standard library
- `Lake/` (~100 MB) — build system

**If `Lean/` is missing**, all tactic proofs fail with misleading `unknown module prefix 'Init'` even though Init.olean exists.

## Environment setup

```bash
cat > ~/.elan/env << 'EOF'
export PATH="$HOME/.elan/bin:$HOME/.elan/toolchains/leanprover--lean4---v4.30.0/bin:$PATH"
export LD_LIBRARY_PATH="$HOME/.elan/toolchains/leanprover--lean4---v4.30.0/lib:$HOME/.elan/toolchains/leanprover--lean4---v4.30.0/lib/lean"
EOF
```

Then `source ~/.elan/env` each session.

## 9p data corruption (CRITICAL)

Both `/opt/data` (C:\\) and `/workspace` (D:\\) are **9p network mounts** from Windows. Copying 2.6 GB of tiny `.olean` files via `cp -r` over 9p **silently corrupts the data**. Files appear the right size and have matching md5 hashes, but Lean fails with `missing data file for module Init` even though `Init.olean` exists at the correct path. Root cause: 9p's write buffering for many small files does not guarantee data integrity.

**Symptoms of corruption:**
- `lean --version` works, `lean --print-libdir` returns the correct path
- Root `.olean` stubs (Init.olean, Lean.olean) are present and md5-identical
- But `lean file.lean` fails with `missing data file for module Init`
- Even `LEAN_PATH` pointing to the corrupted directory doesn't help
- Copying the same files to `/tmp/` (container-local overlay, ext4) works immediately—proving the source data is intact and the corruption happens during 9p write

## Fix: LEAN_PATH hybrid approach

**Do NOT copy .olean files to the persistent volume.** Use this hybrid layout instead:

```
Persistent volume (works):              /tmp/ (local overlay, works):
  bin/lean, bin/lake (9 MB)              lean-4.31.0-rc1-linux/
  lib/*.so, lib/lean/*.so                   lib/lean/Init/        (387 MB)
                                            lib/lean/Lean/        (1.2 GB)
                                            lib/lean/Std/         (317 MB)
                                            lib/lean/Lake/        (100 MB)
```

Set `LEAN_PATH` to point at the /tmp copy:

```bash
export PATH="$TOOLCHAIN_DIR/bin:$PATH"
export LD_LIBRARY_PATH="$TOOLCHAIN_DIR/lib:$TOOLCHAIN_DIR/lib/lean"
export LEAN_PATH="/tmp/lean-4.31.0-rc1-extract/lean-4.31.0-rc1-linux/lib/lean"
```

### Container restart recovery

The tarball is saved on the persistent volume at `/opt/data/lean-4.31.0-rc1-linux.tar.zst`. After restart:

```bash
source ~/.elan/env                                 # PATH, LD_LIBRARY_PATH, LEAN_PATH
mkdir -p /tmp/lean-4.31.0-extract
cd /tmp/lean-4.31.0-extract
python3 -c "
import zstandard, tarfile
with open('/opt/data/lean-4.31.0-rc1-linux.tar.zst', 'rb') as f:
    reader = zstandard.ZstdDecompressor().stream_reader(f)
    tarfile.open(fileobj=reader, mode='r|').extractall('.', filter='data')
"
lean --version  # verify
```

## Ultimate fix: Docker named volume (bypass 9p entirely)

The root cause is the 9p protocol, not Docker or WSL. **Docker named volumes** live on the Docker VM's ext4 partition (not Windows filesystem), so they have native ext4 performance and no 9p corruption.

Add to `docker-compose.yml`:

```yaml
services:
  gateway:
    volumes:
      - ~/.hermes:/opt/data         # existing (user config, 9p)
      - lean-home:/opt/lean-home    # NEW: Docker volume, ext4, fast

volumes:
  lean-home:                         # Docker-managed, persists across rebuilds
```

Then move the toolchain there and set `ELAN_HOME=/opt/lean-home/.elan`. This is the only permanent fix—requires modifying docker-compose.yml and restarting the container.

## Versioning

The project's `lean-toolchain` file specifies the required version (e.g. `leanprover/lean4:v4.31.0-rc1`). elan naming convention for toolchain dir: `leanprover--lean4---v4.30.0` or `leanprover--lean4---v4.31.0-rc1` (replace `/` → `--`, `.` → `-`).

### Version migration

To switch versions, repeat the same process for the new version:
1. Find the release on GitHub (check expanded_assets page for the Linux tarball URL)
2. Download to /tmp
3. Extract via Python zstandard
4. Copy bin/ and .so to persistent volume (small files, safe over 9p)
5. Create the new toolchain directory name matching elan's convention
6. Update `~/.elan/env` to point PATH, LD_LIBRARY_PATH, and LEAN_PATH at the new version
7. Save the tarball to `/opt/data/` for restart recovery

## .olean dependency graph

These root `.olean` files act as **stubs** and are always at the root of `lib/lean/`:
- `Init.olean` — references everything in `Init/` subdirectory
- `Lean.olean` — references `Lean/`
- `Std.olean` — references `Std/`
- `Lake.olean`, `LakeMain.olean`, etc.

If any of the subdirectories (`Init/`, `Lean/`, `Std/`) is missing or has corruption, the error message will say `missing data file for module X` even though the root stub `.olean` exists. Always check the **full subdirectory**, not just the root stub.
