<p align="center">
  <img src="icon.png" alt="GreatArrow.ai" width="128" height="128">
</p>

# GreatArrow.ai — Kiro Power

Connect Kiro to your [GreatArrow.ai](https://www.greatarrow.ai) workspace:
persistent semantic memory shared across every AI you use, 22 agents, and deep
integrations (Google, Microsoft, Slack, Notion, GitHub, and more) — all through
a single MCP server.

## Install

**One-click (recommended):** sign in at
[greatarrow.ai/install](https://www.greatarrow.ai/install), choose **Kiro
(Power)**, and download a personalized power with your token already filled in.

**From this repo (Import from GitHub):**

1. In Kiro: **Powers → Add Custom Power → Import from GitHub**
2. Paste this repository's URL and **Install**
3. Mint a token at [greatarrow.ai/install](https://www.greatarrow.ai/install)
   and replace `YOUR_GAD_TOKEN` in `mcp.json`

See [`POWER.md`](./POWER.md) for full setup, workflows, and troubleshooting.

## What's here

- `plugin.json` — Agent Plugins manifest (name, keywords, author, license)
- `POWER.md` — power docs (setup, workflows, troubleshooting), read by Kiro on activation
- `mcp.json` — MCP server config (HTTP transport, `kiro-power` client)
- `icon.png` — 512×512 Great Arrow logo for the power listing

The power connects over HTTP to `https://www.greatarrow.ai/api/mcp`. All auth,
scopes, rate limiting, and audit logging are enforced server-side. No secret is
stored in this repo — you supply your own `gad_` token.

## Support

`support@greatarrow.ai` · [Support](https://www.greatarrow.ai/support) ·
[Privacy](https://www.greatarrow.ai/legal/privacy)
