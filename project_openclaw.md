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

## Current state (as of 2026-05-29)
- Version: 2026.5.27 (updated from 2026.5.22 on 2026-05-29)
- Config: `/home/grapesmc/.openclaw/openclaw.json`
- Primary model: `anthropic/claude-sonnet-4-6` (changed 2026-05-26, was Haiku)
- Fallback model: `anthropic/claude-haiku-4-5-20251001`
- Web search provider: `gemini` (kept intentionally — no browser-as-search option in OpenClaw)
- Auth: `anthropic:claude-personal` (type: token, sk-ant-oat01 OAuth token — same as Claude Code)
- Gateway: LAN mode (`bind: lan`), port 18789, accessible at `http://192.168.5.64:18789`
- Service: `systemctl --user restart openclaw-gateway` (user-level, enabled on boot)
- Workspace: `/home/grapesmc/.openclaw/workspace/`
- State dir: `/home/grapesmc/.openclaw/` (chmod 700)

## Security (hardened 2026-05-29)
- `allowInsecureAuth: false`
- Auth rate limiting: 10 attempts / 60s window, 5 min lockout
- State dir permissions: 700
- Security audit: 0 critical, 1 warn (Haiku fallback — acceptable)

## Access
- Control UI: `http://192.168.5.64:18789` from LAN
- Auth token in: `/home/grapesmc/.openclaw/identity/device-auth.json`

## Memory sync (set up 2026-05-29)
- OpenClaw `MEMORY.md` → Claude Code: auto-synced via `memory-watcher.service` (inotifywait)
- Claude Code → OpenClaw: done manually when writing significant memory updates
- Claude Code memory backed up to GitHub: `github.com/grapesmc/claude-code-memory` (auto-push on write)

## Work bot
OpenClaw was removed from the work bot — too much maintenance overhead. Only running on OpenClaw-001 (homelab).

## Digital Twin Project (OpenClaw-02)
- VM not yet built — architecture designed, specs: 12 vCPU / 24GB RAM / 200GB / RTX 4060 passthrough
- IP: 192.168.5.176 (test network)
- See `workspace/jl-twin-project/OVERVIEW.md` for full setup checklist
