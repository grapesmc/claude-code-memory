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
- Control UI: `https://openclaw-001.amesbury.r23.ai` (Caddy reverse proxy, internal CA, LAN only)
- Gateway token: `9a15986270f631a907c289c732a5c020c61d32b9926969dd` (from `openclaw.json` → `gateway.auth.token`)
- Auth URL: `https://openclaw-001.amesbury.r23.ai/#token=9a15986270f631a907c289c732a5c020c61d32b9926969dd`
- New devices need one-time approval: `openclaw devices approve <requestId>`
- Caddy config: `/etc/caddy/Caddyfile`, service: `sudo systemctl reload caddy`

## Memory sync (set up 2026-05-29)
- OpenClaw `MEMORY.md` → Claude Code: auto-synced via `memory-watcher.service` (inotifywait)
- Claude Code → OpenClaw: done manually when writing significant memory updates
- Claude Code memory backed up to GitHub: `github.com/grapesmc/claude-code-memory` (auto-push on write)

## Homelab SSH Management
openclaw-001 is the control plane for all homelab servers. SSH key: `~/.ssh/id_ed25519_homelab` (ed25519, no passphrase). SSH config at `~/.ssh/config`.

| Host | IP | Port | User | Sudo | OS |
|---|---|---|---|---|---|
| hal9000 | 10.0.255.230 | 31392 | root | n/a | TrueNAS |
| lexx3000 | 10.0.255.235 | 31392 | grapesmc | passwordless | Linux |
| sydney | 10.0.255.238 | 31392 | grapesmc | passwordless | Linux |
| case | 10.0.255.253 | 31392 | root | n/a | TrueNAS |
| pi4-1 | 10.0.255.100 | 31392 | grapesmc | passwordless | Linux |
| debiandocker | 10.0.255.243 | 31392 | grapesmc | passwordless | Linux |
| homebridge | 10.0.255.229 | 31392 | grapesmc | passwordless | Linux |
| netdata-master | 10.0.255.195 | 31392 | grapesmc | passwordless | Linux |
| plex | 10.0.255.251 | 22 | root | n/a | Proxmox |

- TrueNAS/Proxmox hosts connect as root (key added via web UI) — no sudo needed
- Linux hosts use grapesmc with passwordless sudo via `/etc/sudoers.d/grapesmc-nopasswd`
- All verified working 2026-05-29

## Work bot
OpenClaw was removed from the work bot — too much maintenance overhead. Only running on OpenClaw-001 (homelab).

## Digital Twin Project (OpenClaw-02)
- VM not yet built — architecture designed, specs: 12 vCPU / 24GB RAM / 200GB / RTX 4060 passthrough
- IP: 192.168.5.176 (test network)
- See `workspace/jl-twin-project/OVERVIEW.md` for full setup checklist
