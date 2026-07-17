# Search Infrastructure

Hermes uses a two-tier search setup:

1. **SearXNG** (self-hosted, primary) — privacy-respecting meta-search engine
2. **ddgs** (Python package, fallback) — DuckDuckGo search API

## SearXNG Setup

SearXNG runs as a Docker container alongside Hermes. Configuration lives at
`docker/searxng/settings.yml`.

### Engine Selection

The provided `settings.yml` is tuned for reliability:

| Engine | Status | Notes |
|---|---|---|
| Bing | ✅ Enabled | Reliable, good results |
| Mojeek | ✅ Enabled | Independent index, no API key needed |
| DuckDuckGo | ❌ Disabled | Rate-limited, unreliable through SearXNG |
| Brave | ❌ Disabled | Requires API key |
| Startpage | ❌ Disabled | Often blocked |
| Google | ❌ Disabled | Requires complex setup |

### Timeout Settings

```yaml
outgoing:
  request_timeout: 10.0   # Global timeout
engines:
  - name: bing
    timeout: 15.0         # Longer timeout for Bing
```

### Access

- Internal (from Hermes container): `http://searxng-core:8080`
- External (from host browser, testing): `http://localhost:8085`

### Environment Variable

Set in `~/.hermes/.env`:

```
SEARXNG_URL=http://searxng-core:8080
```

## ddgs Fallback

When SearXNG is unavailable, Hermes falls back to the `ddgs` Python package
(DuckDuckGo search via Python API).

```bash
pip install --target /opt/data/pip-packages ddgs
```

Configured in `config.yaml`:

```yaml
web:
  search_backend: ddgs
```

## Playwright for Web Extraction

Hermes uses Playwright + Scrapling to extract content from JavaScript-heavy
and Cloudflare-protected pages.

### Setup

Already covered in `infra/python.md`, but key points:

```bash
playwright install chromium
# Installs to /opt/data/.playwright/ (persisted via volume)
```

```bash
pip install --target /opt/data/pip-packages scrapling
```

### How It Works

When `web_extract` fails due to Cloudflare or JS requirements, Hermes
automatically falls back to the browser tool, which uses Playwright Chromium
with realistic browser fingerprints (via `browserforge`) and TLS fingerprint
impersonation (via `curl_cffi`).

## Verifying

```bash
# Check SearXNG health
curl http://localhost:8085/healthz

# Test search
curl "http://localhost:8085/search?q=test&format=json" | head -c 200

# Check ddgs from inside container
docker compose exec hermes python3 -c "from ddgs import DDGS; print('ddgs OK')"
```
