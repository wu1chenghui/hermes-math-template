# Python Environment Persistence

## The Problem

Hermes Agent runs in Docker. When the container restarts or the image updates,
any `pip install`-ed packages are lost. But many tools (ddgs, scrapling,
playwright, browserforge, etc.) require Python packages that are NOT in the
base image.

## The Solution: `PYTHONPATH` + `pip install --target`

The docker-compose.yml sets:

```yaml
environment:
  - PYTHONPATH=/opt/data/pip-packages
```

And `/opt/data` is a bind-mounted volume from `~/.hermes`. This means:

1. Install packages with `uv pip install --target` (NOT `pip` — the Hermes
   Docker image uses `uv` instead of `pip`):

   ```bash
   uv pip install --target /opt/data/pip-packages ddgs scrapling playwright ...
   ```

2. Packages survive container restarts and image updates.

3. Hermes' Python tool (`execute_code`) also inherits `PYTHONPATH`, so these
   packages are available in sandboxed code execution too.

## Core Packages

These are the essential packages installed in `pip-packages/`:

| Package | Purpose |
|---|---|
| `ddgs` | DuckDuckGo search (Python API) — fallback when SearXNG is down |
| `scrapling` | Cloudflare-resistant web scraping (works with Playwright) |
| `playwright` | Browser automation — Chromium for web extraction |
| `curl_cffi` | TLS fingerprint impersonation (prevents bot detection) |
| `browserforge` | Realistic browser fingerprint generation |
| `httpx` | HTTP client (async) |
| `httpx_sse` | Server-Sent Events for streaming APIs |
| `cryptography` | AES encryption (required for WeChat media, etc.) |
| `IPython` | Enhanced interactive Python shell |
| `beautifulsoup4` | HTML parsing |
| `certifi` | TLS certificate bundle |

Full list: `pip-packages/` contains ~160 packages total.

## Playwright Browser Installation

Playwright needs more than the Python package — it needs actual browser binaries:

```bash
# Inside the container:
python3 -m playwright install chromium
# Browsers are installed to /opt/data/.playwright/ (persisted via volume)
# Note: PYTHONPATH must include /opt/data/pip-packages (set in docker-compose.yml)
```

## Installation

If your container is fresh:

```bash
# Enter the container
docker compose exec hermes bash

# Install core packages
uv pip install --target /opt/data/pip-packages \
    ddgs scrapling playwright curl_cffi browserforge \
    httpx httpx_sse cryptography beautifulsoup4

# Install Playwright browser
python3 -m playwright install chromium
```

## Verifying

```bash
docker compose exec hermes python3 -c "import ddgs; print('ddgs OK')"
docker compose exec hermes python3 -c "import scrapling; print('scrapling OK')"
docker compose exec hermes python3 -c "import playwright; print('playwright OK')"
```
