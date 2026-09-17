# Hermes Agentbox (latest upstream)

Deploys the **official** `nousresearch/hermes-agent` image to GMI Agentbox with
a thin compatibility layer — no fork of Hermes itself.

## Files
- `Dockerfile.agentbox` — FROM official image, exposes 8080, sets API-server env
- `gmi-maas-init` — cont-init script mapping AgentBox's `GMI_MAAS_*` injection to
  Hermes' GMI provider env vars + seeds `config.yaml` on first boot
- `.github/workflows/agentbox-publish.yml` — builds/publishes to GHCR (weekly
  auto-rebuild; bump `HERMES_TAG` to upgrade Hermes)

## Build & publish
1. Push this folder to a GitHub repo (workflow at
   `.github/workflows/agentbox-publish.yml`).
2. Actions tab → "Publish Agentbox image" → Run workflow.
3. Make the GHCR package public (first time only): profile → Packages →
   hermes-agentbox → Package settings → Change visibility → Public.
   (Or keep private and set Enable Credentials: ON with a PAT `read:packages`.)

## Register on GMI Agentbox
| Field | Value |
|---|---|
| Image URL | `ghcr.io/<your-username>/hermes-agentbox:v2026.9.14` (or `:latest`) |
| Region | IOWA IDC-1 |
| Port mapping | 443 → 8080 (default) |
| MaaS integration | ON, select model(s) — becomes `$GMI_MODELS` |

### Env variables (Step 4)
| Variable | Type | Value |
|---|---|---|
| `API_SERVER_KEY` | SECRET | `openssl rand -hex 32` — chat-UI/API auth token |

Do NOT add `GMI_MAAS_API_KEY` / `GMI_MAAS_BASE_URL` (auto-injected, locked).
Optional adds: `TAVILY_API_KEY` (SECRET) for unthrottled web search,
`DISCORD_BOT_TOKEN` (SECRET) if you want the Discord gateway.

## Verify after launch
```bash
curl https://<instance-url>/health          # {"status":"ok"}
curl https://<instance-url>/v1/models \
  -H "Authorization: Bearer <API_SERVER_KEY>"
```
Chat UI is served at `/`; OpenAI-compatible API at `/v1`.
