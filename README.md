# Hermes Math Template

A reproducible template for setting up Hermes Agent as a mathematical
research assistant — with Lean 4 formal proof support, LaTeX document
compilation, and privacy-respecting web search.

Built following the official [Hermes Agent](https://hermes-agent.nousresearch.com/docs/) docs:
[Configuration](https://hermes-agent.nousresearch.com/docs/user-guide/configuration) ·
[Skills](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) ·
[MCP Servers](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp) ·
[Messaging Gateway](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/)

## Repo Structure

```
.
├── skills/                         # 11 agent skills (SKILL.md + references/)
│   ├── lean-4-workflow/
│   ├── lean-4-proof-writing/
│   ├── mathematical-writing/
│   ├── chinese-math-paper/
│   ├── paper-engineering/
│   ├── latex-workflow/
│   ├── web-search-strategy/
│   ├── agent-memory-architecture/
│   ├── container-python-environment/
│   ├── hermes-web-search-debugging/
│   └── development-methodologies/
├── config/
│   ├── config.yaml.template        # Hermes configuration template
│   └── env.template                # API keys and environment variables
├── docker-compose.yml              # Container orchestration
├── docker/
│   └── searxng/settings.yml        # Search engine configuration
├── infra/                          # Infrastructure setup guides
│   ├── python.md
│   ├── lean.md
│   ├── latex.md
│   └── search.md
├── README.md                       # You are here
├── README.zh.md
├── AGENTS.md                       # Machine-readable context for AI agents
└── .gitignore
```

This repository contains the **configuration, skills, and infrastructure
documentation** needed to replicate a complete Hermes Agent environment
optimized for:

- **Formal mathematics** — Lean 4 + Mathlib 4 via MCP server
- **Paper writing** — Tectonic (lightweight LaTeX) + Pandoc
- **Research search** — Self-hosted SearXNG + ddgs fallback
- **Code & proof engineering** — Proven workflows encoded as agent skills

If you clone this repo and follow the instructions, you'll get an AI agent
that can help with formal proofs, paper compilation, web research, and code
development — all running locally in Docker.

## Prerequisites

- Docker Engine 24+
- 6 GB RAM free (for the Hermes container)
- A model provider API key (DeepSeek, Anthropic, OpenAI, or OpenRouter)
- (Optional) WSL2 on Windows for Lean 4 support

## Quick Start

```bash
# 1. Clone this repo
git clone https://github.com/wu1chenghui/hermes-math-template.git
cd hermes-math-template

# 2. Create directories
mkdir -p ~/.hermes
mkdir -p workspace docker-data/searxng/config docker-data/searxng/cache

# 3. Set up Hermes config
cp config/config.yaml.template ~/.hermes/config.yaml
cp config/env.template ~/.hermes/.env
# Edit ~/.hermes/.env — add your API keys (at minimum DEEPSEEK_API_KEY)

# 4. Install skills
cp -r skills/* ~/.hermes/skills/

# 5. Start the stack
docker compose up -d

# 6. Install Python packages (first time only)
docker compose exec hermes bash -c "
    uv pip install --target /opt/data/pip-packages ddgs scrapling playwright curl_cffi browserforge httpx
    python3 -m playwright install chromium
"

# 7. Enter Hermes
# NOTE: /opt/hermes/bin is not in default PATH inside the container.
# Use the absolute path to the privilege-drop shim.
docker compose exec hermes /opt/hermes/bin/hermes
```

## What's Included

### Skills (Agent Knowledge)

| Skill | Description |
|---|---|
| `lean-4-workflow` | Lean 4 project setup, build, dependency management |
| `lean-4-proof-writing` | Proof tactics, patterns, debugging |
| `mathematical-writing` | Mathematical paper writing conventions |
| `chinese-math-paper` | Chinese mathematical paper formatting (GB/T 7713) |
| `paper-engineering` | Pipeline: Lean formalization → journal paper |
| `latex-workflow` | Markdown → LaTeX → PDF compilation |
| `web-search-strategy` | Effective web research techniques |
| `agent-memory-architecture` | Designing persistent memory for AI agents |
| `container-python-environment` | Python venv and dependency management |
| `hermes-web-search-debugging` | Search backend troubleshooting |
| `development-methodologies` | External methodology reference (unverified) |

> **Note:** These skills were developed alongside a real Lean 4 project.
> File paths in skill references point to `/opt/lean-home/lean-projects/e/`
> — replace `e` with your own Lean project directory.

### Infrastructure Docs

| Document | Covers |
|---|---|
| `infra/python.md` | Persistent Python packages (ddgs, scrapling, playwright) |
| `infra/lean.md` | Lean 4 + Mathlib installation, MCP server config |
| `infra/latex.md` | Tectonic, Pandoc, GitHub CLI installation |
| `infra/search.md` | SearXNG setup, ddgs fallback, Playwright browser |

### Configuration Templates

| File | Purpose |
|---|---|
| `config/config.yaml.template` | Hermes configuration (model, tools, MCP servers) |
| `config/env.template` | API keys and environment variables |
| `docker-compose.yml` | Container orchestration |
| `docker/searxng/settings.yml` | Search engine configuration |

### What's NOT Included

This repo provides templates and skills — it does **not** contain runtime data:
- No API keys or secrets (fill in `env.template` with your own)
- No session history (`state.db`, `sessions/`)
- No personal memory or user profiles (`memories/`)
- No Python packages or Playwright browsers (installed during Quick Start step 6)

Everything here is source material to be copied into `~/.hermes/` on your machine.

## Architecture

```
┌─────────────────────────────────────────────────┐
│ Docker Compose                                  │
│                                                 │
│  ┌──────────────┐  ┌──────────┐  ┌──────────┐  │
│  │   Hermes     │  │ SearXNG  │  │  Valkey  │  │
│  │   Agent      │  │  :8080   │  │  (cache) │  │
│  │              │  │          │  │          │  │
│  │ config.yaml  │  └──────────┘  └──────────┘  │
│  │ skills/      │                               │
│  │ pip-pkgs/    │  ┌──────────────────────────┐ │
│  │ playwright/  │  │  Lean 4 + Mathlib        │ │
│  └──────────────┘  │  (named volume)          │ │
│        │           │  /opt/lean-home          │ │
│        │           └──────────────────────────┘ │
│        │                                        │
│  ~/.hermes ← bind mount                         │
│  lean-home ← named volume (WSL ext4)            │
└─────────────────────────────────────────────────┘
```

## Model

Default configuration uses **DeepSeek** (`deepseek-chat`), which offers strong
math and reasoning performance at low cost. Change the model in
`config.yaml` → `model.default`.

## WSL Notes

If you're on Windows with WSL2:

1. **Lean 4 must live on WSL native filesystem** (`/opt/lean-home/`), not
   under `/mnt/c/`. NTFS-mounted drives break `lake build`.
2. The docker-compose.yml uses a named volume (`lean-home`) for this.
3. Ensure `systemd=true` in `/etc/wsl.conf` for Docker to work correctly.

## License

MIT — use freely, modify, share.

---

*For AI agents: see [AGENTS.md](AGENTS.md) for machine-readable project context.*
