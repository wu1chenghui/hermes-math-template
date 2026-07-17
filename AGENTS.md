# AGENTS.md — Hermes Math Template

This repo provides a reproducible Hermes Agent environment for mathematical
research: Lean 4 formal proofs, LaTeX paper writing, privacy-respecting web
search. It contains configuration templates, skills (agent knowledge), and
infrastructure documentation.

## Quick Start (execute in order)

```bash
git clone https://github.com/wu1chenghui/hermes-math-template.git
cd hermes-math-template

mkdir -p ~/.hermes workspace docker-data/searxng/config docker-data/searxng/cache
cp config/config.yaml.template ~/.hermes/config.yaml
cp config/env.template ~/.hermes/.env
# EDIT ~/.hermes/.env — add DEEPSEEK_API_KEY
cp -r skills/* ~/.hermes/skills/
docker compose up -d

# First-time package install (inside container):
docker compose exec hermes bash -c "
    uv pip install --target /opt/data/pip-packages ddgs scrapling playwright curl_cffi browserforge httpx
    python3 -m playwright install chromium
"

# Enter Hermes:
docker compose exec hermes /opt/hermes/bin/hermes
```

## Key Paths

| Path | Purpose |
|---|---|
| `~/.hermes/` | Host-side Hermes home (mounted to `/opt/data`) |
| `/opt/lean-home/` | Lean 4 toolchain (named Docker volume, WSL ext4) |
| `config/config.yaml.template` | Hermes config — copy to `~/.hermes/config.yaml` |
| `config/env.template` | API keys — copy to `~/.hermes/.env` |
| `skills/` | Agent skills — copy to `~/.hermes/skills/` |
| `infra/` | Infrastructure setup docs (python, lean, latex, search) |
| `docker-compose.yml` | Container orchestration |

## Environment Variables Required

```
DEEPSEEK_API_KEY=sk-...    # Primary model provider
SEARXNG_URL=http://searxng-core:8080   # Set in docker-compose or .env
```

## Infrastructure Components

| Component | How | Doc |
|---|---|---|
| Python packages | `uv pip install --target /opt/data/pip-packages` | `infra/python.md` |
| Playwright | `python3 -m playwright install chromium` | `infra/python.md` |
| Lean 4 + Mathlib | `elan` + `lake exe cache get` | `infra/lean.md` |
| LaTeX | Tectonic binary in `~/.local/bin/` | `infra/latex.md` |
| Pandoc | Pandoc binary in `~/.local/bin/` | `infra/latex.md` |
| SearXNG | Docker container, settings at `docker/searxng/settings.yml` | `infra/search.md` |
| ddgs fallback | Python package in pip-packages | `infra/search.md` |

## Critical Warnings

- The Docker image has `uv` but NOT `pip`. Always use `uv pip install --target`.
- The `hermes` CLI is at `/opt/hermes/bin/hermes`, NOT in default PATH.
- `HERMES_HOME=/opt/data` is set in docker-compose.yml — do NOT remove.
- Lean 4 CANNOT compile on NTFS-mounted paths (WSL). Must be on WSL ext4.
- Skills reference `/opt/lean-home/lean-projects/e/` throughout — this is the
  original project path. Replace `e` with your own project directory.
- config.yaml.template uses `your-project` as LEAN_PROJECT_PATH placeholder.

## Skills Included

The `skills/` directory contains 11 skills for mathematical research:

| Skill | Purpose |
|---|---|
| `lean-4-workflow` | Lean 4 project setup, build, dependency management |
| `lean-4-proof-writing` | Proof tactics, patterns, debugging |
| `mathematical-writing` | Mathematical paper writing conventions |
| `chinese-math-paper` | Chinese math paper formatting (GB/T 7713, ctexart) |
| `paper-engineering` | Pipeline: Lean formalization → journal paper |
| `latex-workflow` | Markdown → LaTeX → PDF compilation |
| `web-search-strategy` | Effective web research techniques |
| `agent-memory-architecture` | Designing persistent memory for AI agents |
| `container-python-environment` | Python venv and dependency management |
| `hermes-web-search-debugging` | Search backend troubleshooting |
| `development-methodologies` | External methodology reference (unverified) |

Each skill has a `SKILL.md` with YAML frontmatter and linked `references/`,
`templates/`, `scripts/` directories.

## License

MIT
