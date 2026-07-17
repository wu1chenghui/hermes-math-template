# Web Search Debugging Checklist

When `web_search` returns empty or errors, follow this diagnostic order:

## 1. Config file permissions
```bash
ls -la /opt/data/config.yaml
```
Must be readable by the hermes user. If owned by `root`, Hermes falls back
to default empty config silently (no search backend configured).
**Fix**: `chown hermes:hermes /opt/data/config.yaml`

**Pitfall**: `hermes config set` can change file ownership to root.

## 2. Check active provider
```python
from agent.web_search_registry import list_providers, get_active_search_provider
providers = list_providers()
print(f"Providers: {len(providers)}")
for p in providers:
    print(f"  {p.name}: avail={p.is_available()}")
active = get_active_search_provider()
print(f"Active: {active.name if active else 'NONE'}")
```

If 0 providers:
- Plugin discovery didn't run → `discover_and_load(force=True)` from `hermes_cli.plugins`
- Or agent process started with broken config → restart process

If providers exist but active is wrong:
- Check `web.search_backend` in config.yaml
- Check `is_available()` returns True for the desired provider

## 3. Provider availability checks

### SearXNG (needs SEARXNG_URL env var)
```bash
curl -s "http://searxng-core:8080/search?q=test&format=json"
```
Check `unresponsive_engines` in response. Common: brave suspended,
duckduckgo CAPTCHA, bing timeout (container network issue).

### DDGS (needs ddgs Python package)
```bash
/opt/hermes/.venv/bin/python3 -c "from ddgs import DDGS; print('OK')"
```
Install: `uv pip install ddgs --target /opt/hermes/.venv/lib/python3.13/site-packages`

**Pitfall**: `uv pip install ddgs` without `--target` may install to wrong location.
**Pitfall**: Agent process started before package install → `httpcore` cached without
`http2` module → `module 'httpcore._sync' has no attribute 'http2'`. Fix: restart
agent process.

## 4. Env vars not loaded
`execute_code` runs in a sandbox; `os.getenv('SEARXNG_URL')` may differ from
the agent process. Check agent logs for the actual env:
```bash
grep "SEARXNG\|web.*backend" /opt/data/logs/agent.log | tail -10
```

## 5. Config resolution precedence
From `web_search_registry.py`:
1. `web.search_backend` (explicit per-capability) → used regardless of availability
2. `web.backend` (shared fallback)
3. Legacy preference: firecrawl → parallel → tavily → exa → searxng → brave-free → ddgs
   (filtered by `is_available()`)

If searxng is explicitly configured but unavailable, it still gets selected
(step 1) and returns empty. Clear the config or set a working backend.

## 6. Plugin discovery in agent vs sandbox
`execute_code` runs in a separate sandbox process. `PluginManager()` creates
a new instance. `discover_and_load(force=True)` in the sandbox does NOT
affect the agent's own PluginManager. Check agent logs for "Plugin discovery
complete" to verify the agent's own discovery succeeded.
