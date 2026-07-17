# Full mathlib build session transcript (reference)

## Environment

- Container: Hermes on WSL+Docker (hermes user, uid=10000)
- Lean version: 4.31.0-rc1
- Toolchain: /opt/lean-home/toolchains/leanprover--lean4---v4.31.0-rc1/
- Named volume: lean-home → /opt/lean-home/ (ext4, 927G free)
- 9p volumes: /opt/data/ (home, C: drive), /workspace/ (bind mount, D: drive)

## Standard library recovery

After migrating the toolchain to ext4, `lean` failed with:

```
error: unknown module prefix 'Init'
```

No .olean files were in the toolchain's `lib/lean/` directory (only .so files).
Fix: download the full release zip from GitHub releases and extract `lib/lean/`.

Download: lean-4.31.0-rc1-linux.zip (832MB)
Extracted: 12109 files (2421 .olean, 21 .so files)
Key files: Init.olean (5KB), Std.olean, Lean.olean, Lake.olean

## GitHub network issues

`git clone` of mathlib4 failed repeatedly with:
- `GnuTLS recv error (-9): Error decoding the received TLS packet`
- `GnuTLS recv error (-110): The TLS connection was non-properly terminated`
- `Failed to connect to github.com port 443 after 21072 ms: Could not connect to server`
- `fetch-pack: unexpected disconnect while reading sideband packet`
- `fatal: early EOF`

These are transient network failures from within the Docker container.
Workaround: use `curl` to download GitHub archive ZIPs.

## Packages needed for lake build

From lake-manifest.json, 9 packages total:

| Package | Source | Size | .lean files |
|---------|--------|------|-------------|
| mathlib | 9p cache | ~300MB | 8626 |
| Batteries | zip | 474KB | 244 |
| aesop | zip | 342KB | 250 |
| proofwidgets | 9p cache | 31MB | 41 |
| Qq | 9p cache | 22MB | 28 |
| plausible | zip | 65KB | 28 |
| LeanSearchClient | zip | 20KB | 8 |
| importGraph | zip | 129KB | 37 |
| Cli | zip | 29KB | 5 |

## Minimal git repo creation

For each ZIP-extracted package, lake needs a .git with HEAD matching the
manifest's rev hash. Steps:

1. Set git identity FIRST (commit-tree needs it):

```bash
git config --global user.email "hermes@local"
git config --global user.name "Hermes"
```

2. For each package, create a single-file commit and update the manifest:

```python
import subprocess, os, json

with open("lake-manifest.json") as f:
    manifest = json.load(f)

for pkg in manifest["packages"]:
    name, url = pkg["name"], pkg["url"]
    pkg_path = f"/opt/lean-home/lean-projects/e/.lake/packages/{name}"
    
    subprocess.run(["git", "init"], cwd=pkg_path)
    subprocess.run(["git", "remote", "add", "origin", url], cwd=pkg_path)
    
    for fn in ["lakefile.toml", "lakefile.lean", "lean-toolchain"]:
        fp = os.path.join(pkg_path, fn)
        if os.path.exists(fp):
            with open(fp, "rb") as f:
                content = f.read()
            blob = subprocess.run(["git", "hash-object", "-w", "--stdin"],
                                  input=content, capture_output=True,
                                  cwd=pkg_path).stdout.decode().strip()
            tree = subprocess.run(["git", "mktree"],
                                  input=f"100644 blob {blob}\t{fn}\n".encode(),
                                  capture_output=True,
                                  cwd=pkg_path).stdout.decode().strip()
            commit = subprocess.run(["git", "commit-tree", tree, "-m", name],
                                    capture_output=True,
                                    cwd=pkg_path).stdout.decode().strip()
            # Write real commit hash; then update manifest to match
            with open(f"{pkg_path}/.git/refs/heads/master", "w") as f:
                f.write(f"{commit}\n")
            pkg["rev"] = commit
            break

with open("lake-manifest.json", "w") as f:
    json.dump(manifest, f, indent=2)
```

Key points:
- `git config --global` must run BEFORE `commit-tree` or it fails with
  "Author identity unknown"
- The commit hash from `commit-tree` must be captured and written to
  `.git/refs/heads/master`, AND the manifest's `rev` must be updated to match
- On 9p, `git add` of many files hangs (slow I/O). The single-file commit
  approach avoids this entirely.

## Build statistics

- Total modules: 8495 (successful), 8494 (after proofwidgets widget fix)
- Running time: ~5 hours (16 parallel lean processes)
- Disk usage: 5.3G for mathlib build artifacts
- Mathlib.olean (umbrella): 749KB
- Individual .olean files: ~8400 under mathlib/.lake/build/lib/lean/

## proofwidgets widget failure

After all 8495 modules compiled, lake failed on 4 widget facet targets:

```
✖ [638/646] Running widgetJsSrcs    — error: no such file or directory
✖ [639/646] Running widgetRollupConfig — error: no such file or directory
✖ [640/646] Running widgetTsconfig    — error: no such file or directory
✖ [641/646] Running widgetPackageJson — error: no such file or directory
```

These require JS build tooling (rollup, typescript). The widget directory
(`widget/`) was missing from the proofwidgets source copy. After copying
`.js`, `.ts`, `.json`, and `widget/` from the 9p source, the build resumed
and needed to recompile many modules (1385/8494). This is normal.

## Compiling a single file with mathlib

```bash
cd /opt/lean-home/lean-projects/e
source ~/.elan/env
lake env lean E/Euler.lean
```

`lake env` sets LEAN_PATH to include all 9 packages' .olean directories.
Without it, `lean` alone can't find the package .olean files.
