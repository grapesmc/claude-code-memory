---
name: project-openclaw
description: "OpenClaw AI agent platform setup, config, and upgrade history for Rob's homelab"
metadata: 
  node_type: memory
  type: project
  originSessionId: ca389924-dc34-49d0-aadf-4196a83ac71f
---

OpenClaw is Rob's self-hosted AI agent platform running on pve1 (OpenClaw-001, IP: 192.168.5.64).

**Why:** Long-term personal AI assistant with memory, smart home control, and multi-channel access (Discord, WhatsApp/BlueBubbles, etc.)

**How to apply:** When touching OpenClaw config or upgrading, use this as reference.

## Current state (as of 2026-05-24)
- Version: 2026.5.22 (updated from 2026.5.12)
- Config: `/home/grapesmc/.openclaw/openclaw.json`
- Primary model: `anthropic/claude-sonnet-4-6` (changed 2026-05-26, was Haiku)
- Fallback model: `anthropic/claude-haiku-4-5-20251001`
- Web search provider: `gemini` (kept intentionally — no browser-as-search option in OpenClaw)
- Auth: `anthropic:claude-personal` (type: token, sk-ant-oat01 OAuth token — same as Claude Code)
- Gateway: local mode, port 18789, auth token in identity/device-auth.json
- Workspace: `/home/grapesmc/.openclaw/workspace/`

## c4monitor (Amesbury Dashboard)
- Location: `/home/grapesmc/.openclaw/workspace/c4monitor/`
- Service: `systemctl --user restart c4monitor`
- Port: 8766
- Chat endpoint was rewritten (2026-05-12): now POSTs directly to OpenClaw gateway at `http://127.0.0.1:18789/v1/chat/completions` with model `openclaw` instead of spawning a subprocess. Also fixed history bug (history was built but not passed to model before).
- Gateway token for c4monitor stored in `OPENCLAW_GATEWAY_TOKEN` env var in c4monitor `.env`

## Planned migration (2026-05-25)
- OpenClaw-001 VM moving to production server — networking unchanged (192.168.5.64, all ports the same)
- Nothing in c4monitor or Cloudflare Tunnel config needs to change
- After migration: confirm `loginctl enable-linger grapesmc` is set so user services start on boot

## Work bot
OpenClaw was removed from the work bot — too much maintenance overhead. Only running on OpenClaw-001 (homelab).

## Digital Twin Project (OpenClaw-02)
- VM not yet built — architecture designed, specs: 12 vCPU / 24GB RAM / 200GB / RTX 4060 passthrough
- IP: 192.168.5.176 (test network)
- See `workspace/jl-twin-project/OVERVIEW.md` for full setup checklist
