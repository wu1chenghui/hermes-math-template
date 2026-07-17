# LaTeX & Document Toolchain

## Tectonic: Lightweight LaTeX Engine

Instead of TeX Live (several GB), we use **Tectonic** — a self-contained
LaTeX engine that downloads packages on demand.

### Why Tectonic?

| | TeX Live | Tectonic |
|---|---|---|
| Size | 4-8 GB | 57 MB |
| Package management | Manual (tlmgr) | Auto-download |
| Output | .pdf | .pdf |
| Speed | Baseline | Comparable |

### Installation

```bash
# Download the latest release (inside container):
cd /tmp
curl -L -o tectonic.tar.gz \
  https://github.com/tectonic-typesetting/tectonic/releases/download/tectonic%400.15.0/tectonic-0.15.0-x86_64-unknown-linux-gnu.tar.gz
tar xzf tectonic.tar.gz
mv tectonic ~/.local/bin/tectonic
export PATH="$HOME/.local/bin:$PATH"
tectonic --version
```

### Usage

```bash
# Compile a LaTeX file
tectonic paper.tex

# Watch mode (auto-recompile on changes)
tectonic -w paper.tex
```

## Pandoc: Universal Document Converter

Pandoc converts between Markdown, LaTeX, Word, HTML, and dozens of other
formats. While not part of the core LaTeX pipeline, it's invaluable for
quick conversions.

### Installation

```bash
# Download the latest static binary (inside container):
cd /tmp
curl -L -o pandoc.tar.gz \
  https://github.com/jgm/pandoc/releases/download/3.6.4/pandoc-3.6.4-linux-amd64.tar.gz
tar xzf pandoc.tar.gz
mv pandoc-3.6.4/bin/pandoc ~/.local/bin/pandoc
export PATH="$HOME/.local/bin:$PATH"
pandoc --version
```

### Common Conversions

```bash
# Markdown → LaTeX
pandoc paper.md -o paper.tex

# Markdown → Word (.docx)
pandoc paper.md -o paper.docx

# LaTeX → Markdown
pandoc paper.tex -o paper.md

# With bibliography
pandoc paper.md --bibliography=refs.bib --citeproc -o paper.pdf
```

## GitHub CLI (`gh`)

GitHub's official CLI for repo management, PRs, issues, etc.

```bash
# Install (inside container):
curl -fsSL https://github.com/cli/cli/releases/download/v2.70.0/gh_2.70.0_linux_amd64.tar.gz \
  -o /tmp/gh.tar.gz
tar -xzf /tmp/gh.tar.gz -C /tmp
mkdir -p ~/.local/bin
cp /tmp/gh_2.70.0_linux_amd64/bin/gh ~/.local/bin/
export PATH="$HOME/.local/bin:$PATH"
gh --version
```

## Path Persistence

Add to `~/.bashrc` inside the container:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Or set it in `docker-compose.yml` environment if preferred.
