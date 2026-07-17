# Hermes Agent Docker Upgrade — Reference

## Core principle

The Hermes Agent Docker image is **stateless**. All user data is stored in a single directory mounted from the host at `/opt/data` (typically `~/.hermes` on the host). The image can be upgraded by pulling a new version **without losing any configuration**.

From the official docs:

> *"The container stores all user data (config, API keys, sessions, skills, memories) in a single directory mounted from the host at `/opt/data`. The image itself is stateless and can be upgraded by pulling a new version without losing any configuration."*

## Verified data survival

After upgrading via `docker compose pull` + `docker compose up -d`, the following all survive because they live under `/opt/data/` (the persistent bind mount):

| Data | File/Dir in `/opt/data/` |
|------|--------------------------|
| API keys & secrets | `.env` |
| Config (model, approvals, tools, etc.) | `config.yaml` |
| All sessions (chat history) | `state.db` (SQLite, ~32MB typical) |
| All installed skills | `skills/` |
| Session exports | `sessions/` |
| Persistent memories | `memories/` |
| Cron jobs | `cron/` |
| Plugins | `plugins/` |
| Kanban board | `kanban.db` |
| Auth tokens | `auth.json` |
| System tool configs (git, ssh, npm) | `home/` (profile-scoped HOME, see #7357) |
| Audio cache | `audio_cache/` |
| Dashboard state | `config.yaml`, `.env` |
| Hermes history | `.hermes_history` |

## Upgrade methods

### Docker Compose (recommended)

```yaml
# docker-compose.yml
services:
  hermes:
    image: nousresearch/hermes-agent:latest   # use official image
    container_name: hermes
    restart: unless-stopped
    volumes:
      - ~/.hermes:/opt/data
    # ... other config
```

Upgrade command:

```bash
docker compose pull hermes
docker compose up -d hermes
```

### docker run

```bash
docker pull nousresearch/hermes-agent:latest
docker rm -f hermes
docker run -d \
  --name hermes \
  --restart unless-stopped \
  -v ~/.hermes:/opt/data \
  nousresearch/hermes-agent gateway run
```

## Switching from `build: .` to `image:`

If your compose file currently uses `build: .` (building from local source), switch to the official image by changing one line:

```yaml
services:
  hermes:
    # build: .                              ← comment out
    image: nousresearch/hermes-agent:latest  ← add this
    # volumes, environment, ports all stay the same
```

**Nothing is lost.** The build SHA is only relevant for the image layer; your `/opt/data` volume is mounted identically regardless of how the image was produced.

## Config migration during upgrade

When the new image starts, Hermes runs **non-interactive config-schema migrations** against the mounted `config.yaml`. Before making changes, it writes timestamped backups next to `config.yaml` and `.env`.

To suppress automatic migration (if you want to inspect changes first):

```bash
export HERMES_SKIP_CONFIG_MIGRATION=1
```

## Verifying no source code was modified

Before switching to the official image, confirm the local checkout hasn't been patched:

```bash
# Find the build SHA
cat /opt/hermes/.hermes_build_sha

# Compare key files against upstream at that SHA
SHA=$(cat /opt/hermes/.hermes_build_sha)
for f in run_agent.py cli.py toolsets.py agent/prompt_builder.py; do
  local=$(sha1sum /opt/hermes/$f | awk '{print $1}')
  remote=$(curl -sL "https://raw.githubusercontent.com/NousResearch/hermes-agent/$SHA/$f" | sha1sum | awk '{print $1}')
  if [ "$local" = "$remote" ]; then echo "✅ $f matches"; else echo "❌ $f DIFFERS"; fi
done
```

If all files match (as they should for an unmodified install), switching to the official image changes exactly nothing about runtime behavior.

## Known issue: per-profile HOME persistence (#4426, fixed in #7357)

Early Docker images stored system tool configs (git, ssh, gh, npm, gcloud) under `/root/` which was on the ephemeral overlay layer. A `docker pull` + recreate would lose `git config --global`, SSH keys, etc.

**Fixed in #7357** (merged). The fix injects a per-profile HOME into subprocess environments at `/opt/data/home/` (inside the persistent volume). As long as your volume has a `home/` directory with `.gitconfig` etc., the fix is active.

To verify:

```bash
ls -la /opt/data/home/   # should show .gitconfig, .cache, .elan, etc.
```

## Related

- Official docs: https://hermes-agent.nousresearch.com/docs/user-guide/docker
- Official updates guide: https://hermes-agent.nousresearch.com/docs/getting-started/updating
- GitHub releases: https://github.com/NousResearch/hermes-agent/releases
- Issue #4426 (HOME persistence): https://github.com/NousResearch/hermes-agent/issues/4426
- PR #7357 (fix): https://github.com/NousResearch/hermes-agent/pull/7357
- `docker-data-persistence-patterns.md` in this skill — general Docker persistence tiers
