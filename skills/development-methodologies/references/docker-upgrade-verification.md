# Docker Upgrade Verification Checklist

Before switching a Docker image tag (e.g., `build: .` → `image: ...:latest` or bumping a version tag), verify these 8 points to ensure data and functionality survive.

## 1. Dockerfile — local modifications

Check if the Dockerfile has extra `RUN` commands beyond upstream:

```bash
diff <(cat Dockerfile) <(curl -sL "https://raw.githubusercontent.com/org/repo/$BUILD_SHA/Dockerfile")
```

If it differs, note what custom packages/steps must be carried forward.

## 2. Extra packages / tools installed at runtime

```bash
# apt: check history.log
cat /var/log/apt/history.log | grep Commandline

# pip: check installed packages
pip list

# npm: check global packages
npm list -g --depth=0
```

## 3. Environment variables (.env)

Check for env vars that point to internal service names, custom URLs, or credentials. These must be compatible with the new image.

```bash
cat /opt/data/.env | grep -v "^#" | grep -v "^\s*$"
```

Key things to look for: API keys, service URLs (SearXNG, inference servers), feature flags.

## 4. Config.yaml — path dependencies

```bash
cat /opt/data/config.yaml
```

Look for:
- Hardcoded paths that might not exist in the new image
- References to the old build directory
- Custom backend configurations

## 5. Network connectivity between services

If the compose file has multiple services (SearXNG, inference servers, databases), verify:

- Are services on the same Docker Compose network? (Same docker-compose.yml → yes)
- Does the .env reference service names (`searxng-core:8080`) that resolve on the default bridge network?
- Is `network_mode: host` required? (Not compatible with Docker Desktop for Windows)

## 6. Env var compatibility with new image

Check the official image's documentation for supported env vars. The Docker docs page usually lists them in a table. Common ones to verify:

- `HERMES_DASHBOARD` — `1` to enable the supervised dashboard
- `HERMES_DASHBOARD_HOST` / `HERMES_DASHBOARD_PORT`
- `HERMES_DASHBOARD_INSECURE` — `1` to disable OAuth gate
- `HERMES_UID` / `HERMES_GID` — UID remap
- `HERMES_SKIP_CONFIG_MIGRATION` — `1` to skip auto-migration

## 7. Volume mounts

Verify that the volume mount in docker-compose.yml maps the correct host directory:

```bash
# Inside the container, check what's mounted:
mount | grep "/opt/data"

# The compose volume `~/.hermes:/opt/data` uses the HOST's ~, not the container's.
# On Windows Docker Desktop, ~ = C:\Users\<username>
```

The official image stores ALL user data (config, API keys, sessions, skills, memories) at `/opt/data`. The container image itself is stateless — upgrading it by pulling a new version preserves all data in the volume.

## 8. Source code modifications

If using `build: .`, check if the local source differs from the build SHA:

```bash
BUILD_SHA=$(cat /opt/data/.hermes_build_sha 2>/dev/null || git rev-parse HEAD)
# Compare key files against upstream at that SHA
for f in run_agent.py cli.py toolsets.py; do
  diff <(sha1sum /opt/hermes/$f) <(curl -sL "https://raw.githubusercontent.com/org/repo/$BUILD_SHA/$f" | sha1sum)
done
```

If no modifications, switching to the official image is safe.

## Upgrade Procedure (Docker Compose)

```bash
docker compose pull             # Pull the new image
docker compose up -d            # Recreate the container (volume preserved)
docker compose logs -f          # Watch for startup + migration messages
```

The image runs non-interactive config-schema migrations automatically and writes timestamped backups of `config.yaml` and `.env` first.
